# S-07 Background queue (what runs without screen operation)

Origin: Chapter 1, 7 (the background order), 7.5, 7.9; Chapter 3, 4 (pending and attaching later, bundled timestamps, ERS); Chapter 8, 3.4, 4, and 5.5; Chapter 9, 4.1–4.4; Chapter 2, 4.2; Chapter 1, 8.4; Chapter 7, 6.3; Chapter 11, 5.3. Triggers are S-01b (start-up), the `Scheduler` (core-net B-044; every 24 hours, hourly, line recovery), and closing.

## S-07a Attaching timestamps later (on connection)
```mermaid
sequenceDiagram
  participant NT as core-net
  participant AC as app-client
  participant SG as core-sign
  participant CS as core-case
  participant RF as core-ref
  participant ST as core-store
  NT-->>AC: Reachability::on_online (B-044)
  AC->>SG: Works::pending_timestamps_count (B-167)
  AC->>CS: Cases (pending status appends)
  AC->>RF: Store::current().tsa (B-221)
  AC->>SG: Timestamps::attach_pending_timestamps(targets, tsa_set) (B-163)
  loop per target
    SG->>NT: Http::post_timestamp x3 (B-042)
    SG->>SG: Rfc3161::verify
    SG->>ST: Chain::append(works | status, timestamped) (B-022; the manifest is not rewritten)
  end
  AC-->>AC: notice (updates the pending count on Home)
```

## S-07b Bundled timestamp and ERS (once a day and on every connection)
```mermaid
sequenceDiagram
  participant AC as app-client
  participant SG as core-sign
  participant ST as core-store
  participant NT as core-net
  AC->>SG: Timestamps::timestamp_chain_head(tsa_set) (B-164)
  SG->>ST: Chain::head(stream) x all streams (B-022), then chain-heads.json
  SG->>NT: Http::post_timestamp (B-042)
  SG->>SG: Ers::archive_timestamp / renew (state/ers.json; one year before the TSA certificate expires, or on a weak algorithm)
  SG->>ST: AtomicFile::atomic_write(state/ers.json) (B-021)
```

## S-07c Update manifest and reference information (at start and every 24 hours)
```mermaid
sequenceDiagram
  participant AC as app-client
  participant RF as core-ref
  participant NT as core-net
  participant SY as core-sync
  participant ST as core-store
  participant UI as ui
  AC->>RF: Fetcher::verify_update_manifest(platform) (B-222; regardless of consent)
  RF->>NT: Sources::ordered, then Http::fetch(PublicPage | Mirror, update/latest.json, tuf/*) (B-041, B-040)
  RF->>RF: BundleVerifier::verify(root, TufSet) (tough; timestamp 7 days, then snapshot, then targets, then release)
  RF->>RF: Store::minimum_version (the custom field of the release targets)
  alt new version
    AC->>AC: Updater::download (twice the free space, a temporary place; passes endpoints to Tauri's update component)
    AC->>AC: Updater::verify (update signature, version match)
    AC->>UI: update_ready (B-294; on close or "Restart and update now")
  else failure
    RF->>NT: Sources::report_failure (the mirror first from next time)
  end
  AC->>RF: Consent::consent_state (B-223)
  alt with consent
    AC->>RF: Fetcher::refresh() (B-220; the five sources: public page, mirror, BundleSource (core-sync B-228), Cloud sync folder, file)
    RF->>NT: Http::fetch(reference/latest.json, only the changed files)
    RF->>RF: BundleVerifier::verify (size limit, then TUF, then version greater than local, then each file's SHA-256, then the 120-day expiry mark)
    RF->>ST: AtomicFile (replaces reference/<version>/)
    AC->>RF: Store::corrections_since(version) (B-227)
    alt corrections present
      AC->>UI: notice (reference information corrections; listing the affected works is S-11c)
    end
    AC->>UI: notice (reference information updated / old / VEX)
  end
```

## S-07d Checking Originals, retention sweep, free space
```mermaid
sequenceDiagram
  participant AC as app-client
  participant SG as core-sign
  participant ST as core-store
  participant HS as core-hash
  participant UI as ui
  AC->>SG: Works::verify_originals() (B-169; SHA-256 of the files at the recorded locations; missing or different)
  SG->>HS: Sha256::sha256_file (B-050)
  SG-->>AC: [OriginalIssue], then UI notice ("Link Original" in G-12, link_original B-285)
  AC->>ST: the retention sweep at start (B-028; logs, crash reports, thumbnails, migration copies, leftover temporary files. Unused assets are at the consolidation of S-08c)
  AC->>ST: Space::free_space (B-031; notice under 1 GB)
```

## S-07e Automatic backup (hourly and on close), then S-08a
## S-07f Deadline notices (at start and daily)
```mermaid
sequenceDiagram
  participant AC as app-client
  participant CS as core-case
  participant GD as core-guidance
  participant RF as core-ref
  participant OS
  participant UI as ui
  AC->>CS: Cases::list (B-184; cases with a complaint filed)
  AC->>RF: Store::current().deadlines / holidays
  AC->>GD: Deadlines::alarms_due(now, cases) (B-202; 3 days before and on the day; including those missed while not running)
  AC->>AC: core-identity Grants::auto_renew (B-088; authorizations 30 days before expiry; the proposal to send to confirmed contacts goes to G-21)
  AC->>OS: notify (B-339)
  AC->>UI: notice (G-21, G-16)
```

## Gaps (found later and fixed)
- Attaching timestamps later needs `tsa_set`, but it was not among the arguments of `attach_pending_timestamps`, so `tsa_set` was added to the arguments of B-163 in core-sign.md (alongside the `targets` added in S-05).
- The “daily trigger of deadline notices” was not in the specification (Chapter 7, 6.3 has only the VALARM time and “those missed while not running, at the next start”), so while running, `alarms_due` is run on the 24-hour tick of core-net's `Scheduler` (“daily” was added at the end of the background order in Chapter 1, 7).
- Passing the order of `Sources::ordered` to the update component (the `endpoints` of Chapter 9, 4.3) is app-client, so “passes `Sources::ordered` to endpoints” was added to the note of `Updater::download` in app-client.md.
