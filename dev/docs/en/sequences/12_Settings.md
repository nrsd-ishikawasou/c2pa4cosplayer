# S-12 Settings, erasing records, portable kit, importing reference information and updates from files, reports, about this app

Origin: Chapter 10, 10, 9, and G-20 to G-23; Chapter 9, 2.3, 4.5, and 5; Chapter 1, 10.3; Chapter 12, 5 and 6.

## S-12a Settings (G-20)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant ST as core-store
  participant ID as core-identity
  participant RF as core-ref
  UI->>AC: get_settings (B-290), then ST Settings::get (B-025)
  UI->>AC: set_setting(key, value) (B-290)
  AC->>ST: Settings::set (device-independent items are subject to sync)
  UI->>AC: set_tsa_account(id, url, user, passphrase) (B-290)
  AC->>ID: Keystore::set_secret(tsa-<id>) (B-090; settings.json holds only the number and name)
  UI->>AC: consent(accept | decline) (B-290), then RF Consent::consent (B-223)
  UI->>AC: open_log_dir (B-290), then ST Paths::log_dir (B-020), then opener
```

## S-12b Erase this device's records (Chapter 9, 2.3)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant BK as core-backup
  participant SY as core-sync
  participant ID as core-identity
  participant ST as core-store
  UI->>AC: wipe_device (B-290)
  AC->>BK: Settings (the date/time of the last backup; if none or old, "Create a backup" first)
  AC-->>UI: RetentionNotice (the claim period) and a confirmation dialog (what is erased and what is not, the confirmation word)
  UI->>AC: wipe_device(confirm_word)
  AC->>ID: OsAuth::os_user_verify (B-092)
  AC->>SY: Link::announce_removed (the "this device was removed" row; if it does not arrive, removed by hand on another device)
  AC->>ID: Keystore::wipe_keys (B-090; the signing keys and the device key)
  AC->>ST: deletes Paths::data_dir / cache_dir / log_dir (B-020; EBWebView on Windows, WebKit data on macOS)
  AC-->>UI: the list of failed items, then exit (the next start is a first run)
```

## S-12c Portable kit and importing from files (Chapter 9, 5 and 4.5)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant RF as core-ref
  participant ST as core-store
  participant HS as core-hash
  UI->>AC: portable_kit(dest) (B-290)
  AC->>AC: the distributables in kits/ (if none, fetched by B-331 and verified by TUF and the update signature)
  AC->>RF: Fetcher::export_bundle (B-226; the package and TUF metadata)
  AC->>HS: Sha256 (SHA256SUMS)
  AC->>ST: AtomicFile (README.txt in three languages, dist/, reference/)
  UI->>AC: import_reference_file(path) (B-290), then RF Fetcher::import_bundle (B-226; TUF; an expired timestamp is only a mark)
  UI->>AC: import_update_file(path) (B-290)
  AC->>RF: Fetcher::verify_update_manifest (the kit's TUF)
  AC->>AC: OS code signature verification (B-341; WinVerifyTrust / codesign), then the update signature, then Updater::apply_on(Now)
```

## S-12d Notices, help, reports, about this app (G-21 to G-23)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant RF as core-ref
  participant CM as core-common
  participant OS
  UI->>AC: notices (B-291; gathers reference information, corrections, updates, backups, deadlines, free space, VEX, key expiry)
  AC->>RF: Store::current().notices / vex (B-221)
  UI->>AC: mark_read(id) (notices.read in settings.json; a device-independent setting)
  UI->>AC: help(screen) (B-291; help.<screen>.* of the message files)
  UI->>AC: compose_feedback(kind, body, attach) (B-291)
  AC->>CM: Logging::recent_errors (B-002; the numbers of recent errors)
  AC-->>UI: text and recipient (mailto up to 2,000 characters; beyond that only "Copy")
  UI->>AC: open_mailto (B-336)
  UI->>AC: crash_reports (B-291), then CM CrashReport::pending (B-010)
  UI->>AC: send_crash_report(id) / discard, then CM CrashReport::keep / discard
  UI->>AC: about (B-291; version, the publisher of its own code signature B-341 self_signature_info), licenses, terms_full (RF Consent::terms_current)
```

## Gaps (found later and fixed)
- Read notices are a “device-independent setting” (the saving item of Chapter 10, 10), yet app-client's data design had them in `state/read-notices.json` (per device), so it was changed to `notices.read` in `settings.json` (device-independent) and the row in app-client.md's data design was corrected.
- The destination of the “this device was removed” row (the operation name in core-sync) was not on core-sync's page, so `announce_removed()` (sent as a row of the devices chain) was added to `Link` in core-sync.md.
