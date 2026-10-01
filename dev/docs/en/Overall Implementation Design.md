# Overall Implementation Design (16th draft, 2026-10-01)

## Position of this document
- The specification is the set of design documents (Draft Project Proposal, Design Plan 2nd edition, Outline Design, Basic Design Chapters 1–13, Research Materials, Legal Research; the Japanese md is authoritative). This document does not copy the specification; it writes only the index of how the source is organized (implementation artifacts, blocks and dependencies, block pages, boundaries with the outside, where decisions are written).
- There is one loop. Requests are taken as revisions of the design documents, and the stage state of the affected blocks is rolled back (CLAUDE.local.md 0.12).
- The design of each block (sections taken, dependencies, class diagram, bridges, algorithms, data design) is in blocks/<block>.md (under the same dev/docs/en/ as this document; the Japanese original is in dev/docs/jp, Chinese in dev/docs/cn, with the same layout. The Japanese is authoritative). This document holds only the index to them and the tables that span blocks (implementation artifacts, dependency edges, boundaries with the outside). Decisions are written in the specification sections; section 5 is the index of where they are.

## 1. Implementation artifacts
|No.|Artifact|Location (Chapter 11, 7)|
|---|---|---|
|P-1|16 core crates (the core-* of section 2)|`crates/`|
|P-2|The User's app shell (Tauri; app-client)|`apps/client/`|
|P-3|Screens (Svelte; ui)|`ui/`|
|P-4|Message files (Japanese, Chinese, English; ICU MessageFormat)|`i18n/`|
|P-5|Operator tool (Tauri; app-operator; M-01 and M-02)|`apps/operator/`|
|P-6|Public page (static; the “Check a notice code” page, `/rights/`, `update/`, `reference/`, the Terms)|Source of GitHub Pages|
|P-7|Reference information (sources and package: contact points, model texts, each country's laws, texts, example texts, deadlines, notices, minimum version, Terms)|`reference/`|
|P-8|Bundled assets (fonts, models, default templates, images for the guide and procedures, app icons (each OS's format), the passphrase word list, the initial package, license documents)|`assets/`|
|P-9|Distributables (NSIS, DMG and `.app.tar.gz`, AppImage, Flatpak; signatures, SBOM, attestations, LGPL sources)|Releases, mirror|
|P-10|Repository skeleton and CI|Top level, `.github/`|
|P-11|README (three languages), LICENSE, NOTICE, SECURITY.md, `LICENSES/`, `REUSE.toml`|Top level|
|P-12|Operations tools and procedures (copying to the mirror, releases, key incidents, publishing reference information)|`tools/` and the operations documents|
|P-13|Developer-side documents (types and permissions generated from the list of boundaries = the bridge tables, JSON Schemas and test vectors, the error list)|`dev/`|
|P-14|Browser extension “Keep evidence”|`extension/`|
- The preview edition is a difference in the build settings of P-2 and P-9 (version number, URL of the update manifest). It is not a separate artifact.

## 2. Blocks and dependencies
- The list of crates and their roles is Chapter 11, 7.1. This table is authoritative for the dependency edges (check_bridges.py reads this table to check for cycles and for the presence of bridges).
|Block|Layer|Lower blocks it depends on|
|---|---|---|
|core-common|base|—|
|core-store|base|core-common|
|core-net|base|core-common|
|core-hash|business|core-common|
|core-image|business|core-common, core-hash|
|core-identity|business|core-common, core-store|
|core-mark|business|core-common, core-image|
|core-render|business|core-common, core-image|
|core-ref|business|core-common, core-store, core-net|
|core-rights|business|core-common, core-store, core-hash, core-ref|
|core-work|business|core-common, core-store, core-hash, core-image, core-render, core-mark, core-rights, core-ref|
|core-sign|business|core-common, core-store, core-net, core-hash, core-image, core-identity, core-mark, core-render, core-rights, core-work|
|core-case|business|core-common, core-store, core-net, core-hash, core-image, core-mark, core-render, core-sign, core-rights, core-identity, core-ref|
|core-guidance|business|core-common, core-ref, core-case|
|core-backup|business|core-common, core-store, core-hash, core-identity, core-render, core-work, core-sign|
|core-sync|business|core-common, core-store, core-net, core-identity, core-ref, core-work|
|app-client|shell|core-common, core-store, core-net, core-hash, core-image, core-identity, core-mark, core-render, core-sign, core-rights, core-work, core-case, core-guidance, core-ref, core-backup, core-sync|
|app-operator|shell|core-common, core-store, core-ref|
|ui|screen|app-client|
- Dependency inversions (the lower block defines, the upper block implements; not counted as edges): `RecordSigner` (defined by core-store, implemented by core-identity; the device number is also passed here), `BundleSource` (defined by core-ref, implemented by core-sync), `BundleSigner` (defined by core-ref, implemented by app-operator), screen events (emitted by app-client, subscribed to by ui).
- Only app-client and app-operator depend on tauri.
- Correspondence with the 12 blocks of Design Plan 4.2 “System blocks” (the reasons for the division in the design documents are in each section):
|Design Plan 4.2|Implementing block|
|---|---|
|Signing information|core-identity|
|Settings|core-store (settings), core-work (templates)|
|Update|app-client (swap-in), core-ref (verification of the manifest)|
|Image editing|core-work, core-render, ui (G-25)|
|Batch application|core-work, core-image (thumbnails), core-mark (running the auto-placement model)|
|Rights document creation|core-rights, ui (G-10)|
|Signing|core-sign, core-mark, core-hash, core-image|
|Delivery output|core-sign (the flow of Chapter 3, 8.1), core-rights (Enclosed Document)|
|Social media output|core-sign, core-image (resizing, color), core-render (Visible Signature)|
|Registration and evidence preservation|core-case, core-net|
|Status check|core-case|
|Legal Action Guidance|core-guidance, core-ref (reference information)|
|(Not in the Design Plan; added in the design documents)|core-common (Chapter 1, 10, cross-cutting), the saving and integrity of records in core-store (Chapter 1, 8.3; Chapter 8, 4), core-net (Chapter 1, 4.2), core-backup (Chapter 8, 5), core-sync (Chapter 8, 5.4; Chapter 2, 5.5), app-operator (Chapter 1, 19)|

### 2.1 Algorithm blocks (points of replacement)
- Algorithms (things whose method can be chosen, that are switched by measurement, or that hold thresholds) are placed behind a trait inside the owning block, and the choice is made in one place in each block. The list is the “Algorithms” table of each block's page (NRSD's instruction, 2026-10-01).
## 3. Block designs (pages) and sequences
- The sequences (for each entry point, every route of what is called where; S-01 to S-12) start from the table in [sequences/00_How to Read.md](sequences/00_How to Read.md). Gaps found later are at the end of each file together with where they were fixed.
- Each section is owned by exactly one block. The list of owned sections and their design are at the top of each page under “Sections taken”. When another block needs something from that section, it goes through the owner's bridge (the “Bridges” table of the page).
|Block|Page|Audit (the full text of the sections taken checked against the page; one pass from the leaves up)|
|---|---|---|
|core-common|[blocks/core-common.md](blocks/core-common.md)|Done 2026-10-01|
|core-store|[blocks/core-store.md](blocks/core-store.md)|Done 2026-10-01|
|core-net|[blocks/core-net.md](blocks/core-net.md)|Done 2026-10-01|
|core-hash|[blocks/core-hash.md](blocks/core-hash.md)|Done 2026-10-01|
|core-image|[blocks/core-image.md](blocks/core-image.md)|Done 2026-10-01|
|core-identity|[blocks/core-identity.md](blocks/core-identity.md)|Done 2026-10-01|
|core-mark|[blocks/core-mark.md](blocks/core-mark.md)|Done 2026-10-01|
|core-render|[blocks/core-render.md](blocks/core-render.md)|Done 2026-10-01|
|core-ref|[blocks/core-ref.md](blocks/core-ref.md)|Done 2026-10-01|
|core-rights|[blocks/core-rights.md](blocks/core-rights.md)|Done 2026-10-01|
|core-work|[blocks/core-work.md](blocks/core-work.md)|Done 2026-10-01|
|core-sign|[blocks/core-sign.md](blocks/core-sign.md)|Done 2026-10-01|
|core-case|[blocks/core-case.md](blocks/core-case.md)|Done 2026-10-01|
|core-guidance|[blocks/core-guidance.md](blocks/core-guidance.md)|Done 2026-10-01|
|core-backup|[blocks/core-backup.md](blocks/core-backup.md)|Done 2026-10-01|
|core-sync|[blocks/core-sync.md](blocks/core-sync.md)|Done 2026-10-01|
|app-client|[blocks/app-client.md](blocks/app-client.md)|Done 2026-10-01|
|app-operator|[blocks/app-operator.md](blocks/app-operator.md)|Done 2026-10-01|
|ui|[blocks/ui.md](blocks/ui.md)|Done 2026-10-01|
- Sections not subject to implementation (not owned): each chapter's “List of design decisions”, “Correspondence with requirements”, “Known gaps declared in this chapter”, “Corrections to other chapters”; Chapter 1, 2–5, 11, 14–18.2, 20–22 (context, quality goals, trust and threats, quality requirements, risks, terms, operations); Chapter 8, 6.3; Chapter 10, 2.4; Chapter 11, 2 and 12; Chapter 12, 2 and 7; Chapter 13 (design verification; measurements and confirmations are taken by the test documents).
- Sections owned by implementation artifacts that are not blocks (no page; the content is the specification section itself):
|Artifact|Sections owned|
|---|---|
|Public page (P-6)|Chapter 8, 3.3 “Public page (decision of O-05)”|
|Reference information (P-7)|Chapter 5, 3.3 “Outline of the per-country part”, 3.7 “Draft text of the common part”, 4 “Managing the texts”, 6.3 “Draft of the short text (three languages)”; Chapter 7, 2.1 “Initial list of contact points (initial content of the reference information)”, 3.2.1 “Liability for wrong or false complaints”, 3.2.2 “Procedure for the other party's counter-notice”, 3.3.1 “United States (DMCA notice; 17 U.S.C. §512(c)(3)(A))”, 3.3.2 “China (Civil Code Article 1195; Regulation on the Protection of the Right to Network Dissemination of Information, Article 14)”, 3.3.3 “Japan (request for measures to prevent transmission of infringing information; Form A of the Copyright-related Guidelines, 3rd edition)”, 4 “References to each country's laws”; Chapter 12, 4.1 “List of clauses”, 4.2 “Draft wording of the key clauses”, 5.1 “List of clauses”, 5.2 “List of transmissions outside the device”|
|Bundled assets (P-8)|Chapter 4, 6.1 “Bundled fonts”, 6.3 “Emoji”, 8.4 “Default templates”; Chapter 11, 4.3 “Assets”, 4.4 “Incorporating TrustMark”|
|Distributables (P-9)|Chapter 1, 9.2 “Installation”; Chapter 9, 2.1 “Distributables per OS”, 5.1 “Preview edition (the edition for User trials)”|
|Repository skeleton and CI (P-10)|Chapter 9, 3.1 “Code signing”, 6.2 “Bundled license notices”, 6.4 “Versioning”; Chapter 11, 3 “Development environment and versions”, 3.1 “Minimum supported OS”, 4.1 “Rust components”, 4.2 “Screen components”, 5.1 “Permitted licenses”, 5.2 “Sources and prohibitions”, 5.3 “Vulnerabilities”, 6 “Bill of materials and provenance attestation”, 8.1 “On every change”, 8.2 “At release”, 8.3 “Handling of secrets”, 10 “Versioning and branch operation”|
|README and others (P-11)|Chapter 8, 3.2 “Distinguishing modified versions”; Chapter 9, 2.2 “Uninstalling and the device's records”, 3.2 “How to tell”, 5 “Acquisition routes”; Chapter 11, 9 “NRSD's license (decision of O-09)”|
|Operations tools and procedures (P-12)|Chapter 1, 12 “External dependencies”, 18.1 “NRSD's points of involvement and organization”, 18.3 “NRSD's key incidents”; Chapter 7, 2.2 “Keeping up”; Chapter 8, 3.5 “Protecting the GitHub account and repository”, 3.6 “Operating the Hong Kong mirror”, 6.2 “Mainland China”, 7 “Dependence on GitHub”; Chapter 9, 6.1 “Procedure”, 6.3 “Export control of software containing cryptography”, 6.5 “When a faulty version was released”; Chapter 11, 5.4 “Updates”; Chapter 12, 4.3 “Procedure for changes”, 8 “Review of the legal research”|
|Browser extension (P-14)|Chapter 6, 2.2.2 “Guidance on taking screen images”|
## 4. Bridges and boundaries with the outside
- Bridges (types and operations crossing between blocks) are in the “Bridges” table of the page of the destination (lower) block (the B- numbers were carried over). The content of a type passed over a bridge is defined as is by the table in the section named under “Origin”.
- When a request comes, the places to fix are three: the specification md, this document and the block pages, and the source.
|No.|Caller|Counterpart|What crosses|
|---|---|---|---|
|B-320|P-14 browser extension|core-case (the form of B-182)|File `.wacz` (URL, date/time, screen image, HTML, images, each hash, versions of the extension and browser)|
|B-321|P-7 reference information|core-ref (the form of B-229)|Package `nrsd.reference/1` (`index.json`, each file (zstd), `latest.json`, TUF `root.json`, `targets.json` (delegation `reference`), `snapshot.json`, `timestamp.json`. There is no `index.json.sig`)|
|B-322|P-6 public page|core-identity (B-089), core-rights (B-125), P-13|The same computation and the same test vectors (notice code, Deed), the JSON Schemas of records|
|B-323|ui, core-common|P-4|The form of the message files (ICU MessageFormat JSON, three languages) and how keys are assigned. The round trip with XLIFF 2.1 is P-10|
|B-324|app-client, ui, core-store, P-6|P-13 developer-side documents|Command types generated from the list of boundaries (this table), the permission list, JSON Schemas, test vectors, the error list|
|B-330|core-ref (via core-net)|(outside) Public Repository, mirror|HTTPS GET: reference information package, update manifest|
|B-331|app-client (Tauri's update component)|(outside) Public Repository, mirror|HTTPS GET: distributables (after core-ref's verification)|
|B-332|core-sign (via core-net)|(outside) TSA|RFC 3161 request and response|
|B-333|core-case (via core-net's `Http` and `Dns`)|(outside) RDAP, DNS|Queries|
|B-334|core-case (via core-net)|(outside) repost page|HTTP GET (only URLs the User registered)|
|B-335|core-sync (iroh)|(outside) another device, confirmed contact, relay|Direct connection addressed by public key (end-to-end encrypted)|
|B-336|app-client (opener)|(outside) e-mail software, default browser, calendar|`mailto:`, URL, `.ics`|
|B-337|core-identity (keyring)|(outside) OS keystore, OS user verification|Save and retrieve, verification request|
|B-338|core-case (trash)|(outside) OS trash|Move|
|B-339|app-client (notification)|(outside) OS notifications|Notification|
|B-340|app-client (deep-link, folder watching)|(outside) OS URL scheme, save folder|Receiving the `.wacz` made by the browser extension (passed to B-182)|
|B-341|app-client|(outside) OS code signature verification (WinVerifyTrust on Windows, codesign on macOS; none on Linux)|Checking the code signature of an update distributable applied by hand from a file (portable kit; the update signature is checked by Tauri's component), `self_signature_info()` (publisher and version of its own executable; G-23)|
## 5. Where decisions are written (decisions are written in the specification sections; this table is the index)
|Matter|Decision|
|---|---|
|Texts|Written in Chapter 1, 10.3 and Chapter 10, 5|
|Writers of records|Written in Chapter 1, 10.2|
|Values passed from the screen to the core at start|Written in Chapter 1, 6.2|
|Start-up order and states|Written in Chapter 1, 7|
|ONNX runtime|blocks/core-mark.md (owner)|
|XMP and EXIF|blocks/core-image.md (owner)|
|Posting site detection table|Written in Chapter 8, 3.4|
|Form of each reference information file|Chapter 8, 3.4; blocks/core-ref.md (owner)|
|Short form of `{license}`|Written in Chapter 5, 2.1|
|Identification number collisions|Written in Chapter 3, 7.1|
|Locations|Operations tools in `tools/`, developer-side documents in `dev/` (separated from docs/ which is shown to others)|
|Boundaries that make ownership single|“Dependencies” of each page in blocks/|
|Insertion values|blocks/core-work.md (owner)|
|Consolidating backups and devices|blocks/core-work.md and core-backup.md (owners)|
|Corrected Enclosed Documents|blocks/core-rights.md (owner)|
|Passing of setting values|blocks/core-image.md and core-rights.md (owners)|
|Version of the watermark model|blocks/core-mark.md (owner)|
|Propagation of a remade personal root|Written in Chapter 2, 7.2|
|Checking past roots|blocks/core-store.md (owner)|
|Component versions|Names, versions, and licenses were checked on crates.io and npm on 2026-09-30. Functions are checked early in the implementation|
|Identification number in the name (A-2)|Written in Chapter 3, 7.1|
|Means of linking devices (A-6)|Written in Chapter 8, 5.4|
|Deep-link and OS integration (A-7)|Written in Chapter 10, 8|
|Distributables for the portable kit (A-9)|Written in Chapter 9, 5|
|Backing up Originals (A-10)|Written in Chapter 8, 5.5|
|Automatic processing and keys (A-11, A-23)|Written in Chapter 2, 7.4|
|Set of posting sites (A-15)|Written in Chapter 8, 3.4|
|Location of the first backup (A-16)|Written in Chapter 8, 5.5|
|Sample number in previews (A-17)|Written in Chapter 4, 3.1|
|Where corrected versions are generated (A-18)|Written in Chapter 5, 2.4|
|Users without consent and the minimum version (A-19)|Written in Chapter 9, 4.2|
|Checking past roots (A-24)|Written in Chapter 2, 3.6|
|Notice account in templates (A-26)|Written in Chapter 4, 3.1|
|Permanent URLs (A-27, A-76)|Written in Chapter 8, 3.1|
|Withdrawn evidence (A-31)|Written in Chapter 8, 5.4|
|Safety number (A-36)|Written in Chapter 2, 5.5 (from the personal root's public key and the notice code; the device key is not included)|
|Removing a device (A-38)|Written in Chapter 9, 2.3|
|Start-up order (A-39)|Written in Chapter 1, 7|
|Using only for checking (A-40)|Written in Chapter 10, G-13|
|Updates at OS shutdown (A-41)|Written in Chapter 9, 4.1|
|Certificate changes during an export (A-42)|Written in Chapter 2, 7.6|
|Earlier Notice accounts (A-43)|Written in Chapter 2, 4.1|
|Documents after a remaking (A-44)|Written in Chapter 2, 3.6|
|Joint rights when the counterpart does not use the System (A-46)|Written in Chapter 2, 5.2|
|Export queue (A-47)|Written in Chapter 3, 9|
|Temporary names and sync folders (A-48)|Written in Chapter 1, 10.2|
|Serial numbers on re-export and folder collisions (A-49)|Written in Chapter 3, 10.1|
|Linking RAW Originals (A-50)|Written in Chapter 2, 4.2|
|Importing a template with the same `id` (A-51)|Written in Chapter 4, 8.3|
|`recolor: auto` of image layers (A-52)|Written in Chapter 4, 5|
|Distinguishing corrected versions (A-53)|Written in Chapter 5, 3.4|
|Matches with multiple works (A-56)|Written in Chapter 6, 4.1|
|Chain verification after a withdrawal (A-57)|Written in Chapter 6, 7|
|Signing the extension's WACZ (A-58)|Written in Chapter 6, 2.2.1|
|Extension versions and when the app is not running (A-59)|Written in Chapter 6, 2.2.2|
|Copy of the complaint sent (A-61)|Written in Chapter 6, 6.1|
|TUF for the Flatpak version (A-64)|Written in Chapter 9, 4.3|
|Form of differential backups (A-65)|Written in the `manifest.json` of Chapter 8, 5.1|
|Recovery key (A-66)|Written in Chapter 8, 5.1|
|Primary device (A-69)|Written in Chapter 8, 5.4 (there is no “primary device”)|
|Form of the reference information sources (A-70)|Written in Chapter 1, 19|
|Key collisions (A-72)|Written in Chapter 4, 9.3|
|List of record formats (A-73)|Written in Chapter 1, 8.3|
|Numbering of errors (A-75)|Written in Chapter 1, 10.3|
|Carrying out from the offline PC (A-79)|Written in Chapter 1, 19|
|Cleaning up temporary files (A-87)|Written in Chapter 3, 9|
|Reaching by passphrase (A-96)|Written in Chapter 8, 5.4 under “Connecting from a passphrase” (published on the pkarr DHT with a key derived by Argon2id, checked with SPAKE2)|
|Receiving what arrives from a contact (A-99)|Written in Chapter 2, 5.5|
|Watermark in 16-bit images (A-103)|Written in Chapter 3, 5|
|Converting u2netp (A-107)|Written in Chapter 11, 6|
|Limit on history appends (A-108)|Written in Chapter 4, 11.3|
|Rights holder fields of the Enclosed Document (A-109)|Written in Chapter 5, 3.6 (`assigner` is the single signer; the principal rights holder and joint rights holder are `nrsd:grantor` and `nrsd:coRightsHolder`)|
|App identifier (A-120)|Written in Chapter 1, 9.1|
|Extension releases (A-121)|Written in Chapter 9, 6.1|
|Temporary location of update distributables (A-122)|Written in Chapter 9, 4.1|

## 6. What NRSD provides (preconditions for implementation)
- The GitHub organization and its protection (Chapter 8, 3.5; Chapter 11, 8.3). Artifact Signing and the Apple Developer Program (Chapter 9, 3.1). The update signing key (Chapter 9, DD-9-4). The reference information signing keys (two hardware tokens; Chapter 8, 3.4). The Hong Kong mirror (Chapter 8, 3.6). The e-mail inbox for reports (Chapter 10, 9; Chapter 12, 5). Expert review of the texts (Chapter 12, DD-12-10). Trial participants (Chapter 10, 2.4).

## 7. Checking
- Judgments are made by reading the text (CLAUDE.local.md 2 “Do not use tool output as evidence”). The tools in claude_plan/impl are used only as an index of line numbers. 境界.tsv was dissolved into the block pages and deleted on 2026-10-01 (check_bridges.py lost its target and is not used).
