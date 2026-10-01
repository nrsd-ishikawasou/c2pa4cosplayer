# S-05 Registering a repost (G-14), cases (G-15 and G-16)

Origin: Chapter 1, 7.3; Chapter 6, 2.1 (steps 1 to 6), 2.2, 2.2.1, 2.2.2, 2.4, 2.5, 3.2, 3.3, 3.4, 4, 5, 6, and 7; Chapter 10, G-14 to G-16.

## S-05a The registration screen (steps 1 to 5)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant CS as core-case
  participant RF as core-ref
  participant CM as core-common
  participant IM as core-image
  UI->>AC: normalize_url(url) (B-286; when pasted)
  AC->>RF: Store::current().platforms / clearurls (B-221)
  AC->>CS: Registrar::normalize_and_hint(url, platforms) (B-181)
  CS->>CM: B-007 (UrlKind is Repost. Tracking parameters are removed by the clearurls rules, and the input as entered is also kept. Detection is Platforms::detect)
  AC-->>UI: Hint (canonical URL, platform, count on the same domain, that the IP is visible and "not fetched in this registration")
  UI->>AC: match_image(path) (step 2; B-286)
  AC->>CS: Matcher::match_image(path, works) (B-183; the 6 levels of Chapter 6, 4.1 with the same materials as S-04a)
  UI->>AC: import_capture(path) (step 3; the extension's .wacz)
  AC->>CS: Capture::import_capture(path) (B-182; compat, ZIP checks, each hash, signature and timeSignature)
  UI->>AC: adds a screen image, then CS Capture::screenshot_date (date/time taken / file date/time)
  UI->>AC: tsa_accounts (B-286; choice of a paid TSA)
  Note over UI: step 5 input fields (as far as known)
```

## S-05b Registering (step 6; a mark in state/ per step)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant CS as core-case
  participant NT as core-net
  participant IM as core-image
  participant HS as core-hash
  participant MK as core-mark
  participant RT as core-rights
  participant SG as core-sign
  participant ID as core-identity
  participant ST as core-store
  UI->>AC: register_case(input, tsa_choice) (B-286)
  AC->>CS: Registrar::register(input, refs) (B-180)
  CS->>ST: state/case-<id>.json (step marks)
  alt capture the page record (default)
    CS->>NT: Headers::for_page (B-043), then Http::fetch(RepostPage, url) (B-040; each hop of the redirects)
    CS->>CS: WarcWriter (warcinfo, request, response, metadata)
    CS->>CS: ImagePicker (og:image and the top 5 img), then Http::fetch x5
    CS->>IM: Decoder::decode (B-062; hash only if unreadable)
    alt redirect to login / 404 or 410 / blocked / over 50 MB
      CS->>CS: recorded as in the table of Chapter 6, 3.3
    end
  end
  CS->>CS: Whois::lookup(final_url, bootstrap) (B-193; DNS, RDAP IP and domain, reverse lookup, ICP)
  CS->>NT: Dns::resolve / reverse (B-045; raw responses into the WARC), Http::fetch(Rdap, …) (B-040; raw request and response into the WARC)
  CS->>HS: Sha256::sha256_file (per evidence file)
  CS->>CS: Wacz::pack, then evidence/<SHA-256>.<extension>
  CS->>RT: ViolationRules::violations(scope, facts) (B-128; the Permitted Scope of the matched work)
  CS->>CS: Collection (the form of Chapter 6, 2.5)
  CS->>ID: Keystore::with_signing_key (B-086; OS user verification)
  CS->>ST: Chain::append(cases/<id>/case.json) (B-022)
  CS->>ST: Chain::append(status/<device>.jsonl, registered)
  CS->>SG: Timestamps::timestamp(SHA-256 of the list, tsa_set) (B-162; three free ones, or the chosen paid one)
  alt no connection
    CS->>ST: Chain::append(status, timestamped is pending) (attached later in S-07a)
  else
    CS->>SG: Timestamps::verify_tsr
    CS->>ST: Chain::append(status, timestamped)
  end
  CS->>ST: deletes state/case-<id>.json
  AC-->>UI: Case, then route(G-16)
```

## S-05c Case list, details, current state, withdrawal, evidence set
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant CS as core-case
  participant GD as core-guidance
  participant SG as core-sign
  participant RD as core-render
  participant ST as core-store
  UI->>AC: list_cases(filter) (B-286), then CS Cases::list (B-184; per domain, priority marks)
  UI->>AC: case_detail(id) (B-286), then CS Cases::case (B-184)
  AC->>GD: Deadlines::deadlines(case, filing, rules, holidays) (B-202; "N days to the deadline")
  UI->>AC: check_now(id) (B-286)
  AC->>CS: Status::check_now(id) (B-185; one fetch, then capture-<serial>.warc.gz, then checked in status, then timestamp)
  UI->>AC: withdraw_case(id, reason) (B-286)
  AC->>CS: Status::withdraw(id, reason) (B-187)
  CS-->>AC: RetentionNotice (the claim period), then UI confirmation dialog (Chapter 10, 3.6)
  CS->>ST: Chain::append(status, withdrawn)
  CS->>CS: the evidence folder to the OS trash (B-338)
  UI->>AC: export_bundle(id, user_fields, include_clues) (B-286)
  AC->>CS: Bundle::export_bundle (B-188)
  CS->>SG: Timestamps::ers_for(rows) (B-165; extracting the chain leading to the case)
  CS->>SG: Works::work(id) (B-167; the matched work data)
  CS->>RD: PdfWriter::pdf_a(the report; embeds manifest-sha256.txt and the ERS) (B-115)
  CS->>ST: SafeArchive::zip_write(BagIt) (B-027)
  UI->>AC: export_ics(id) (B-286), then GD Deadlines::ics (B-202)
```

## Gaps (found later and fixed)
- The form in which app-client passes the `refs` of `register` (the reference information's contact point list, detection table, RDAP bootstrap, `tsa.json`) is in the dependencies of core-case.md, but the argument name of `Registrar::register` was `refs: RefBundle`, pointing to core-ref's type, so, because core-case does not depend on core-ref, it was changed to a type holding only the needed tables, `CaseRefs { platforms, rdap_bootstrap, tsa }` (the diagram of core-case.md).
- Where the check of the 1 GB evidence total (Chapter 6, 2.4) stops within the flow of step 6 was unclear, so `Cases::evidence_size` is called twice, before fetching (the estimate from the page record limit and the total of screen images) and after (measured), and if exceeded the import is refused and “as a separate case” is shown (added to the note of `Registrar::register` in core-case.md).
- The route for attaching a case's “pending timestamp” (the state of G-14) at the next connection existed only in core-sign's `attach_pending_timestamps` (pending ones of work data), so `attach_pending_timestamps` also covers pending case status appends (noted in B-163 of core-sign.md; the enumeration of targets is received from core-case's `Cases`).
