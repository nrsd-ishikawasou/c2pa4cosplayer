# S-04 Verify (G-13) and clue comparison (G-24)

Origin: Chapter 3, 10.2; Chapter 2, 4.1, 4.4, and 5.1; Chapter 5, 3.4; Chapter 6, 4.1; Chapter 10, G-13 and G-24. Leaves no records (Chapter 10, G-13).

## S-04a Checking an image
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant SG as core-sign
  participant ID as core-identity
  participant MK as core-mark
  participant HS as core-hash
  participant ST as core-store
  participant RT as core-rights
  UI->>AC: dropped_files / pick_files, then inspect_image(path) (B-285)
  AC->>SG: Inspector::inspect(path) (B-160)
  SG->>SG: c2pa verification (validation codes, then status_word_key B-166)
  SG->>ID: NoticeCodeCalc::expected_notice_code(x5chain) (B-089; "no notice code" unless a self-signed CA)
  SG->>ID: PastRoots::is_own_root(root) (B-085; whether it is own current or past root)
  SG->>ID: IdentityStore::history (B-083; if own earlier signing certificate, the period and the accounts at the time)
  SG->>ID: Validator::skeleton(signer_handle) vs own (B-093; caution on similar names)
  SG->>MK: Embedder::extract_image(image, all_orientations=true) (B-101)
  SG->>ST: WorkIndex::lookup(work_id) (B-029; whether it is in own work data)
  SG->>HS: Pdq::hash_all_orientations / Iscc::compute (B-051, B-052)
  SG->>ST: WorkIndex::lookup_by_hash (the distance to the Original's and output's PDQ is via Works)
  SG->>ID: Grants::by_hash(the authorization hash in the manifest), then is_revoked(grant_id) (B-088; the mark on outputs that carry the hash of a revoked authorization)
  AC-->>UI: Inspection (verification words, signer, expected notice code, timestamp, watermark, degree of match, marks)
  UI->>AC: open_url(Notice account) (B-293)
```

## S-04b Checking an Enclosed Document folder (Chapter 5, 3.4)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant SG as core-sign
  participant RT as core-rights
  participant ST as core-store
  UI->>AC: verify_enclosure_folder(folder) (B-285)
  AC->>SG: Inspector::inspect(the images in the folder) ("No images" if none)
  AC->>RT: EnclosureVerifier::verify(folder, manifest_hashes) (B-122)
  RT->>ST: Chain::verify_embedded(00_RIGHTS_ERRATA.txt.sig) (B-022; whether the root matches the image's x5chain)
  AC-->>UI: VerifyReport (match, mismatch, missing, extra; legitimacy of ERRATA)
```

## S-04c Clue comparison (G-24)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant CS as core-case
  participant SG as core-sign
  participant RD as core-render
  UI->>AC: clues(path) (B-285; from G-13 when "there is someone else's C2PA signature and it relates to own work")
  AC->>CS: Matcher::clues(path) (B-189)
  CS->>SG: Inspector::inspect (B-160), Works::work(id) (B-167; the timestamp and the Original's PDQ of own work data)
  AC-->>UI: Clues (watermark, timestamp, ingredient history, Original, Notice; no judgment)
  UI->>AC: export_clues(path, dest) (B-285; exports the list of clues)
  AC->>RD: PdfWriter::pdf_a (B-115) and TXT
  UI->>AC: route(G-17) ("See possible means", then guidance_options(case=None) S-06)
  UI->>AC: route(G-14) ("Register this repost"; the entrance to G-03 if there is no signing information)
```

## Gaps (found later and fixed)
- The form (TXT or PDF) of Chapter 2, 4.4 “the list of clues can be exported in addition to the evidence set” was not in the specification, so it was made the same as the report of Chapter 6, 3.4 (TXT and PDF/A-3u), taken by `Bundle::export_bundle(include_clues)` in core-case.md. The standalone “extracting clues” of G-24 is written in the same form by app-client's `export_clues` (this file). “The form is the same as the report of Chapter 6, 3.4” was added to Chapter 2, 4.4.
- The lookup “authorization hash in the manifest → authorization number” needed for G-13's “mark on outputs that carry the hash of a revoked authorization” (Chapter 2, 5.1) was not in core-identity's `Grants`, so `by_hash(sha256) Option<Document>` was added to `Grants` in core-identity.md.
