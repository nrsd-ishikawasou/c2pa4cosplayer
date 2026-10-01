# Basic Design Document Chapter 8: Repository and Data Management

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [08_Basic Design_Repository and Data Management.docx](08_Basic%20Design_Repository%20and%20Data%20Management.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

## Position of This Document

- Details Chapter 8 of the Outline Design Document. This chapter sets the arrangement of data on the User's device, what leaves the device, backup files (format, creation, restoration, merging), the Public Repository and public page, the Hong Kong mirror, integrity, capacity, and dependence on GitHub. It is designed together with Chapter 2 “Signing Information and Matching”, and goes back and forth with Chapter 6 “Registration and Evidence Preservation” on capacity.
- This chapter is based on the study memo “Study of Chapter 2 Signing Information and Matching and Chapter 8 Repository and Data Management” (NRSD's internal record of study, not distributed with this document; the facts underlying the design are written with their sources in each section of this document). The third draft added on-device encryption of records, the form of the reference information package, protection of the GitHub account and repository, operation of the mirror, and where to keep backups (the facts researched are in the second-round section of the same study memo).
- The decisions received are as in the following table. The texts of the Design Plan's decisions, items to be investigated, and open items are per Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” and the destination table of the Outline Design Document (not reproduced in this chapter; only the numbers and the omissions found in the item breakdown are listed).

|Number|Type|Content|
|---|---|---|
|A-1|Omission found in the item breakdown|Write permissions to the private repository (the premise disappeared with Edition 2, which abolished the private repository)|
|A-17|Omission found in the item breakdown|Cross-border data transfer (the premise disappeared with Edition 2, in which NRSD receives no User information)|
|A-9|Omission found in the item breakdown|Costs, and GitHub capacity limits|

## 1. List of Design Decisions

|Number|Decision|Reason|Rejected alternatives|
|---|---|---|---|
|DD-8-1|One Public Repository is set up (under NRSD's GitHub organization account; DD-8-9“Protected by a GitHub organization and immutable releases”). It holds the source, releases, README, public page, and reference information|Design Plan D-12-1“The Public Repository is provided. No Private Repository is provided”, D-12-2“The software is published, and its being taken away is not prevented”, D-12-6“Distribution is from a GitHub link, with an easy-to-understand README”, D-12-7“The Client App receives binaries from the Public Repository and updates itself”|—|
|DD-8-2|Users' records are placed in the OS's per-user, non-synchronized app data location (2, Chapter 1, DD-1-10“Data is kept in a per-user location that is not synchronized”). The app does not send records outside the device|Design Plan D-12-3“Rights Holder Information, the signing key, work data, the ledger, and evidence are kept on the User’s device; NRSD does not collect them”, D-5-1“The System does not confirm Users and collects no User information. No registration is required in order to use the System”|Automatically synchronizing to an external location (would mean collecting Users' information)|
|DD-8-3|Moving devices and preparing for failure are done with backup files the User creates. The format is all records (including signing keys) put into a ZIP and encrypted in age format (passphrase, scrypt work factor 2^18)|Since records are not collected, backups can only be made at the User's hand. age is a public format that encrypts in 64 KiB chunks and also detects tampering with the header, and can process backups of several GB sequentially. The official implementation's default work factor (2^18, “one second on a modern machine”) is stronger than OWASP's minimum (2^17)|A self-made combination of Argon2id and AES-256-GCM (first draft; encrypting several GB in one go requires putting the whole in memory or making one's own chunking). Exporting only evidence (signing keys cannot be moved)|
|DD-8-4|The public page carries no individual reposting site names, URLs, or User information. It carries the explanation of the System, how to obtain it, the matching procedure, and the version of the reference information|Design Plan D-12-4“The public page is viewable and not editable, and information that could be used for impersonation is not posted on it”. The System does not collect repost information|—|
|DD-8-5|For mainland China, copies of the Public Repository's releases and reference information are placed in object storage in the Hong Kong region (static delivery only). The candidate is Alibaba Cloud OSS in the Hong Kong region|Connections from mainland China to GitHub are unstable (Research Materials “GitHub and Distribution”). The Hong Kong region does not require an ICP filing (Alibaba Cloud's ICP filing guide)|Placing it in a mainland China region (ICP filing procedure)|
|DD-8-6|When importing a backup file whose personal root differs from the current device's, it is not merged; the User chooses to replace or cancel|Mixing records of different personal roots would make the notice code and record signatures inconsistent|Mixing automatically|
|DD-8-7|Records in the app's data location are not encrypted by the app itself. Protection against device theft is left to the OS's full-disk encryption (Windows device encryption / BitLocker, macOS FileVault, Linux LUKS), and enabling it is recommended in the first-run guidance and README. Signing keys are kept in the OS keystore (Chapter 2, 7 “Signing Keys”)|Even if the app encrypted them, its key could only be kept in the OS keystore of the same device, and protection against someone who can get into the device would be no different from full-disk encryption. On the other hand, losing the keystore item would make evidence unreadable. Windows device encryption is enabled automatically when set up with a Microsoft account. Macs with Apple CPUs encrypt stored data automatically, but to protect a stolen machine with the login password FileVault must be enabled (Microsoft and Apple guides)|Encrypting records with the app's key (losing the key loses the evidence; double encryption with backup files)|
|DD-8-8|Reference information is distributed as a “package” (version number, issue date/time, list of files), and the hash and expiry of each file are distributed in the TUF metadata of the delegation `reference`, signed with NRSD's key (ECDSA P-256 in a hardware token) (3.4, Chapter 9, 4.3; changed on 2026-10-01 from a JWS signature on the index to TUF). The app does not accept packages of an older version than it has, and keeps using expired packages while notifying|Protection against, besides fake reference information, attacks that keep serving old valid packages (rollback and freeze). Addresses the attacks listed in The Update Framework (TUF) specification (rollback, indefinite freeze, endless data, mix-and-match)|Attaching only record signatures (old valid packages cannot be distinguished)|
|DD-8-9|NRSD's GitHub is an organization account, with two owners (a trusted separate person as the second; fewer than three). Until a second can be appointed it is operated with one owner, and the risk of not being able to recover the organization is declared as a gap (Chapter 13, H-45“While the GitHub organization has one owner, if that person loses all authentication and recovery methods, the organization cannot be recovered”). An owner is not the same person's second account. All members are required to use two-factor authentication, and owners register several authentication methods (two or more passkeys or security keys, one kept in another place; time-based one-time codes). The Public Repository prohibits force-pushes and deletion of the main branch by rules, and releases are immutable releases|GitHub recommends having two or more “people” as organization owners (“Maintaining ownership continuity for your organization”). Free accounts are limited to one per person (GitHub Terms of Service B.3). GitHub does not recover an account that has lost all two-factor methods and recovery methods, nor does it recover by identity verification (“GitHub account recovery policy”). OpenSSF's SCM Best Practices say fewer than three owners. GitHub has required two-factor authentication for users who create releases and others (since March 2023). Immutable releases cannot have assets and tags changed after publication, and release attestations are created automatically (GitHub's guide). Prevents replacement of distributables through account takeover|Placing it under a personal account (stops with one person's takeover or accident). Making a second account of the Lead Developer an owner (free accounts are limited to one per person by the terms, so it cannot be had for free; even if paid, it does not meet GitHub's recommended “two or more people” and does not prepare for accidents in which that person can no longer act)|
|DD-8-10|Access logging (log storage) of the Hong Kong mirror (object storage) is not enabled. Listing is prohibited, and only public reading is allowed|Access logs would keep the IP addresses of Users who downloaded, meaning NRSD collects Users' information (Design Plan D-5-1“The System does not confirm Users and collects no User information. No registration is required in order to use the System”). Access logging of Alibaba Cloud OSS is disabled by default, and no operation is done to enable it|Keeping logs for investigating failures (collects Users' information)|

## 2. Device Data

### 2.1 Arrangement

- Location: the app's folder under the app's data location (app_local_data_dir of Chapter 1, 9.1 “Locations per OS”). The source, storage location, scope of publication, writer, integrity protection, and backup handling of all data are set in this one table (Chapter 1, 8.1 points here).

|Data|Location|Writer|Public|Integrity protection|Included in backup|
|---|---|---|---|---|---|
|Originals (RAW, pre-development, input photos)|The User's PC (the System only reads them)|User|Non-public|— (SHA-256 and location in the work data)|Chosen at first run (included by default; Chapter 2, 4.2, 5.5)|
|Rights Holder Information (handle name, role, Notice accounts), its input draft, the personal root and the signing certificate, the day the first timestamp was obtained, the history of changes|`identity/` (the draft in `identity/draft.json` (Chapter 2, 2.1), the timestamp day in `identity/epoch.json` (Chapter 2, 3.4), the history in `identity/history/<device number>.jsonl` + `.sig`; Chapter 2, 6)|App|The range in the certificate is public together with images|The history is a chain (Chapter 1, 8.3)|Included|
|Past personal roots|`identity/past_roots/` (public key, notice code, period, reason; Chapter 2, 3.6)|App|Non-public|—|Included|
|The User's signing keys (personal root, signing)|The OS keystore. On Linux without Secret Service, `identity/key.age` (Chapter 2, 7.5)|App|Secret|—|Included (keys go separately into the backup's `keys/`; 5.2)|
|Device key (direct connections)|The OS keystore (Chapter 1, 8.5)|App (per device)|Secret|—|Not included|
|Paid TSA accounts|The OS keystore (Chapter 6, 3.2)|User|Secret|—|Included (not synchronized)|
|Work data|`works/<year>/<batch number>.jsonl` + `.sig` (a chain made by one device; Chapter 3, 7.2). The index `works/index.json` (identification number → file and row; SHA-256 of Original and output → identification number; SHA-256 of the wording files used → identification number (listing corrected versions; Chapter 5, 2.4). It can be rebuilt from the chain)|App|Non-public|Chain|Included (the index is not)|
|Output images (delivery, social media) and Enclosed Documents|The output folder the User chooses. From there to Cloud and posting sites (outside the System)|App|Distributed to purchasers / public|C2PA signature and watermark. Enclosed Documents have their SHA-256 in the manifest (Chapter 5, 3.4)|Not included|
|Case records and status additions|`cases/<case number>/case.json` + `.sig`, `status/<device number>.jsonl` + `.sig` (Chapter 6, 2.4, 6.1)|App|Non-public|Chain|Included|
|Evidence (fetched items, screenshots, the contents of the extension's `.wacz`)|`cases/<case number>/evidence/` (SHA-256 names; Chapter 6, 2.4)|App, User|Non-public|Hashes in the case record, timestamp|Included|
|Copies of complaints (with names and the like redacted)|`cases/<case number>/filings/<UUID v7>.txt` (Chapter 6, 6.1)|App|Non-public|Hash in the chain|Included|
|Authorizations, revocations, joint-rights documents|`approvals/<device number>.jsonl` + `.sig` (the import rows), `approvals/docs/` (the documents; Chapter 2, 5)|The principal Rights Holder's app; both parties for joint-rights documents|Non-public (a summary in the output's manifest)|The record signatures inside the documents, chain|Included|
|Templates, their drafts, the location of sample photos|`templates/<template number>/` (`local.json` is not put in the file for passing on; Chapter 4, 8, 9.2)|App|Non-public|—|Included|
|Imported assets (fonts, images)|`assets/` (SHA-256 names; Chapter 4, 5, 6.4)|User|Non-public|Referenced by SHA-256|Included|
|Work sessions (editing in progress)|`sessions/<work session number>/` (Chapter 4, 11)|App (saved automatically)|Non-public|— (photos are not copied; location and SHA-256)|Included|
|Reference information|`reference/<version>/` (3.4)|NRSD (fetched by the app)|Public|NRSD's TUF signatures|Only the versions referenced by work data (Chapter 5, 4)|
|Settings|`settings.json` (Chapter 10, 10)|User|Non-public|—|Included|
|Record of consent to the Terms of Use|`consent.json` (Chapter 12, 3.1; at the top level because Users who “only verify” have no `identity/`)|App|Non-public|—|Included|
|List of linked devices and confirmed contacts|`peers.json` (`schema: nrsd.peers/1`. Devices: device number, device public key, name, date/time linked, date/time last synchronized. Contacts: notice code, SHA-256 of the personal root certificate, name, date/time the safety number was confirmed, list of known device numbers (because a contact's identity is confirmed by the root, confirmation is not redone when the contact adds devices; 5.4, Chapter 2, 5.5)|App|Non-public|—|Included (not synchronized)|
|The row of chain heads|`chain-heads.json` (the target of the bundled timestamp; Chapter 3, 4)|App|Non-public|Bundled timestamp|Included|
|In-progress state, the export queue `export-queue.json`, the ERS schedule `ers.json`, the mark reserving an update swap `update-pending` (Chapter 9, 4.1), the monotonic number `seq`, the in-use mark `running` (written at start and removed on a clean exit; if it remains, the previous run did not close properly), the order of sources with failure counts and last check times `fetch-state.json` (Chapter 1, 4.2), and samples of clock skew (the differences between the TSAs' `genTime` and the request time for the last 10; Chapter 1, 10.7) `clock.json`|`state/` (Chapter 1, 7.8)|App|Non-public|—|Not included|
|Copies before record format migration|`migration-backup/<original version>/` (deleted after 30 days; Chapter 9, 4.4)|App|Non-public|—|Not included|
|Reduced images of photos|`thumbs/` in the cache location (app_cache_dir) (on Windows it is the same as the app's data location, so it is separated by this name)|App|Non-public|— (can be regenerated)|Not included|
|Copy of the distributable for the portable kit (one version)|`kits/` in the cache location (Chapter 9, 5)|App|Non-public|Verified when fetched|Not included|
|Operation log|The log location (app_log_dir; `logs/` under the app's data location on Windows and Linux, `~/Library/Logs/<identifier>` on macOS)|App|Non-public (appears only when the User attaches it to feedback)|—|Not included|
|Crash reports|`crashes/` (Chapter 10, 9)|App|Same as above|—|Not included|
|Screen component data|`EBWebView/` (Windows WebView2), `~/Library/WebKit/<identifier>` (macOS; [To be measured]). Subject to erasure (Chapter 9, 2.3)|Screen component|Non-public|—|Not included|
|Backup files|The locations the User chooses (external storage, the Cloud sync folder, another device; 5.5)|App|Non-public|age's authenticated encryption (5.1)|—|
|Recovery key|The private key only on the printed emergency kit. The public key is a device-independent setting (5.1)|App|The private key is secret|—|The public key is included|

- File names are ASCII only (numbers), and names entered by the User are kept inside JSON (avoiding problems with multilingual file names; Chapter 1, 8.6 “Multilingual file names and characters”).
- All records are written by the procedure of Chapter 1, 10.2 “Storage” (temporary file and replacement).

### 2.2 Not leaving the device

- What the app sends outside the device is limited to the parties and content in the table of Chapter 1, 4.2 “Communication with the outside”. None includes the User's records themselves.
- Except for backup files the User makes themselves (5) and feedback e-mails the User sends themselves (Chapter 10, 9 “Feedback Channel”), records do not leave the device.

## 3. Public Repository

### 3.1 Structure

- The layout inside the repository follows Chapter 11, 7 “Repository Structure”.

|Place|Content|
|---|---|
|README|Three languages. Guidance on obtaining, how to tell the official version (Chapter 9, 3 “Proof of the Official Version”), the matching procedure (Chapter 2, 4.1 “Matching with the Notice account”), how to check provenance attestations (Chapter 11, 6 “SBOM and Provenance Attestations”)|
|Releases|Distributables for the three OSes, update signatures, software bill of materials (SBOM), list of licenses, provenance attestations, a copy of the reference information package bundled in that version. They are immutable releases (3.5)|
|`update/` (under the public page)|Update manifest (JSON, for GitHub; Chapter 9, 4.3 “Form of the update manifest (static JSON)”). Placed outside releases so that it can be replaced and rolled back to the previous version (Chapter 9, 6.5 “When a version with errors has been released”)|
|`reference/<version>/` (under the public page)|Reference information (contact points, texts, references to each country's law, wording of the Enclosed Document). The app obtains it from the public page (GitHub Pages) URL (Chapter 1, 7.5 “Fetching reference information”)|
|`tuf/` (under the public page)|TUF metadata (the hashes and signatures of distributables and reference information; Chapter 9, 4.3, 3.3). `snapshot.json` and `timestamp.json` are re-signed and placed by the CI `tuf-online`|
|Public page|3.3|

- No personal information or per-User information is placed there (R-8-1-1).
- Permanent URLs: the URLs embedded in images, manifests, and Notices (`xmpRights:WebStatement`, the ODRL `nrsd:` namespace, the verification method) are under `https://c2pa4cosplayer.nrsd.jp/`, pointed to GitHub Pages by a DNS CNAME (GitHub Pages custom domain; TLS is issued by GitHub). No server returning redirects is needed; all NRSD holds is the DNS record (compatible with Chapter 1, DD-1-2). If the GitHub organization is lost, the CNAME is pointed to another static location (Chapter 13, H-45). A redirect under the company site (`nrsd.jp/c2pa4cosplayer/`) is not adopted.

### 3.2 Distinguishing modified versions

- Reference information and updates are protected by TUF metadata signed with NRSD's keys, and the app holds `root.json` internally to verify them (Chapter 9, 4.3). Reference information and updates distributed by modified versions do not pass verification in the official app.
- How to tell the official version: Chapter 9, 3 “Proof of the Official Version”.

### 3.3 Public page (decision on O-05“Content of the public page”)

|What is placed|What is not placed|
|---|---|
|Explanation of the System, how to obtain it (stating the official sources)|Names and URLs of reposting sites|
|The matching procedure (how it looks on general verification sites, how to read the clues, how to tell outputs made under a revoked authorization; Chapter 2, 4.1 “Matching with the Notice account”) and the “Verify” page (besides images, the Enclosed Document folder can be dropped too; it shows agreement with the hashes in the manifest and shows the ODRL in the form of a Creative Commons Deed; Chapter 5, 3.4; NRSD's request, 2026-10-01) (the flow and words follow Adobe's Content Authenticity Verify (drop an image, the signer and linked accounts appear, open the account and compare). A static page that reads an image only within the browser and computes and shows the notice code from the certificate chain of the C2PA signature; the image is sent nowhere; Chapter 2, DD-2-9“The personal root certificate is included in x5chain”. Written in in-house JavaScript without external components, with the HTML and JavaScript in one file so that it works even when the User saves it and opens it locally. The page's hash is listed in the README and the reference information. Preparation against threat T-23 at the end of this section; NRSD's request, 2026-09-30). The JSON Schemas of records and the test vectors (Chapter 1, 8.3)|Users' handle names and Notice accounts|
|Version and update date of the reference information|Individual content of cases|
|The “Accessibility” page (what meets and what does not meet the definitions of Chapter 10, 2.7, and how they are measured; updated per version)|—|
|`/.well-known/security.txt` (RFC 9116; Contact, Encryption, Policy, Expires) and the vulnerability reporting policy page (acknowledgement within about 7 days; not disclosed until a fixed version is released)|—|
|Vulnerability advisories for components (VEX in OASIS CSAF 2.0, or CycloneDX VEX: not affected, affected, fixed version, under investigation, with reasons. Linked to the SBOM; Chapter 11, 5.3)|—|
|The “Accessibility” page is published in the OpenACR form (US GSA; YAML and HTML). The success criteria added in WCAG 2.2 are an additional table of the same form (Chapter 10, 2.7)|—|
|The privacy policy page of the browser extension “Save evidence” (required by the extension stores; that the extension sends nothing, and the contents of the file passed to the app)|—|
|That the System is not involved in the rights of the original works, and that it is not legal advice|—|
|Outages and announcements (Chapter 1, 18.1 “NRSD's points of involvement and structure”)|—|

- The public page is generated by `tools/refbuild` (Rust; Markdown with pulldown-cmark), the same tool that makes the reference information package; no separate static site generator is brought in.
- The procedure of the “Check notice code” page (moved from Chapter 2, 4.1):
1. Open the image on the public page's “Check notice code” page. This page reads the image only within the browser (sending it nowhere; the public page is a static page, consistent with the policy of having no servers) and computes and shows the notice code from the personal root certificate at the end of the C2PA signature's certificate chain. The app's “Verify” shows the same.
2. There is no need to compute it by hand (the procedure in 2.4 describes what the page and app do). CI confirms with the same test vectors that the page and the app compute the same (Chapter 11, 8.1). The page is a single file of in-house JavaScript and works even when the User saves it and opens it locally (Chapter 8, 3.3).
3. Open the Notice account (URL) in the signer's certificate and see whether the same notice code is on the profile or pinned post.
4. If it is, it is a clue that “the owner of this account is the owner of this signing key”. If not, it is not a clue.
- The `/rights/` and terms pages are generated by CI from the reference information sources, and published after the difference has been checked in a pull request (not generated by hand; NRSD's request, 2026-09-30).
- Structure of the pages: under `/<language>/` (`ja`, `zh`, `en`; pointing to each other with `hreflang`; `/` goes to `en` regardless of `Accept-Language`), `index` (explanation and obtaining), `verify`, `rights/<country>` (each country's law), `terms/<version>` (terms and policies; past versions are kept), `accessibility`, `security` (policy and VEX), and `notices` (outages and announcements). The language-independent `update/`, `reference/`, `tuf/` (`root.json`, `targets.json`, the delegations `reference.json` and `release.json`, `snapshot.json`, `timestamp.json`; Chapter 9, 4.3), and `/.well-known/security.txt` are at the top level.
- The public page is made with GitHub's public page feature, and edits go through the Public Repository (“not editable” in D-12-4“The public page is viewable and not editable, and information that could be used for impersonation is not posted on it” means viewers cannot edit it).
- Threat T-23 (tampering): rewriting the public page's “Check notice code” page to show a fake notice code. Attacker: a party who took over the GitHub organization. Target asset: viewers performing matching. Remaining gap: when the organization is taken (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”)

### 3.4 Reference information package

|File|Content|
|---|---|
|`reference/<version>/index.json`|`schema: nrsd.reference/1`, version number (an integer starting from 1; only raised, never lowered), issue date/time (RFC 3339), list of files (path, sizes before and after compression, budget), the “correction” marks (`corrections`: a row has `file` (the path of the corrected file), `since` (the first version containing the error), and `reason` (three languages); Chapter 5, 2.4 uses these rows to find the affected works). Hashes and expiry are held by the TUF metadata (the expiry is 120 days from issue; initial value; the quarterly review (Chapter 7, 2.2 “Keeping up to date”) plus a 30-day margin)|
|`reference/<version>/index.json.sig`|Not held. The package's signature is unified into the TUF metadata: `targets.json` (the delegated targets of the reference information) lists `index.json` and the hash and size of each file, and is signed with NRSD's hardware token (the PIV digital signature slot; PIN entered for each signature). No JWS signature on the index is held, and no `x5c` certificate for NRSD's JWS is needed (NRSD's keys are distributed as the public keys in TUF's `root.json`)|
|Each file of `reference/<version>/` (`platforms.json` (one row per posting site or provider: host names for detection, profile limit, C2PA survival, contact points (two kinds: copyright and portrait). The posting sites are the same set as the list of contact points in Chapter 7, 2.1 (including TikTok, Fantia, and Cloudflare); the enumeration in Chapter 2, 2.3 is the main examples. This table is passed to the check of Notice accounts (Chapter 2, 2.3); Chapter 7, 2.1), `templates/<kind>.<language>.md` model texts, `laws.json` references to each country's law, `texts/` the wording of Enclosed Documents, `notices.json` announcements (number, date, kind, title and body in three languages, expiry), `terms/<language>.md` and `terms.json` (version, effective date), `clearurls.json`, `rdap-bootstrap/`, `tsa.json`, `relays.json` (the list of relays for direct connections; a row has `url` (the URL of an iroh relay), `region`, and `operator`; the first edition is iroh's public relays (run by n0; Chapter 1, 12); replacements are distributed by package versions), `holidays.json`, `deadlines.json`, `minimum-version.json`, `vex.json`)|Contact points (Chapter 7, 2 “Contact Points”), sample texts (Chapter 7, 3 “Model Texts”), references to each country's law (Chapter 7, 4 “References to Each Country's Law”), wording of the Enclosed Document (Chapter 5, 3.7 “Draft sentences of the common part”), announcements (Chapter 10, G-21“Notifications”)|
|`reference/latest.json`|The latest version number and the SHA-256 of `index.json` (a pointer to which version to fetch; not itself a basis of trust)|

- The app's verification (the content of step 2 of Chapter 1, 7.5 “Fetching reference information”):

|Order|What is checked|When it fails|
|---|---|---|
|1|`index.json` is 64 KB or less, each file is no larger than the size in the list and 1 MB or less, and the total is 16 MB or less (initial value; this revises the limits of Chapter 1, 4.2 “Communication with the outside” (NRSD's request, 2026-09-30))|Stop fetching (protection against attacks that obstruct the device with large data)|
|2|Verify in the order TUF `root.json` (updated from the first version embedded in the app through the chain of versions) → `timestamp.json` → `snapshot.json` → `targets.json`, and confirm that the hash of `index.json` matches `targets.json` (tough)|Not used. Recorded with a `REF` code|
|3|The package's version number is larger than the version held locally (combined with the monotonic increase of the TUF snapshot version)|Packages smaller than or equal to the local one are not used (protection against rollback)|
|4|The SHA-256 of each file matches the list|The whole package is not used (protection against mix-and-match)|
|5|The expiry of the delegation `reference` metadata (120 days)|Even if expired, it is used as the latest held locally, and G-21“Notifications” shows “Reference information is out of date (last checked: date)” (protection against freezing). More than 30 days after expiry, it is also shown at the top of the contact point guidance (G-17“Action Guide”). Expiry of the TUF timestamp (7 days; Chapter 9, 4.3) is treated as a fetch failure and tried at the next check. Only the package's expiry is reported to the User|

- The keys for signing the reference information are two hardware tokens, primary and spare, and the two public keys are placed in the role of the TUF delegation `reference` with threshold 1 (Chapter 9, 4.3). In an incident with the primary key, packages can be issued with the spare key as they are. The attestation certificate of the token used for signing is attached to the index (NRSD's request, 2026-09-30).
- The RDAP bootstrap list (the table IANA distributes of which registry to query) is included in the package. The operator tool M-01“Reference Information” fetches IANA's list, shows the difference from the previous version, and raises the version. The app does not query IANA (Chapter 6, 5). The platform detection table (Chapter 2, 2.3, Chapter 6, 2.1) is also included in the package, and every row of the table carries test URLs run in CI.
- The ClearURLs rules for tracking parameters (Chapter 1, 10.10 “Forms of URLs and QR codes”) are also included in the package, and compared with upstream quarterly to raise the version.
- A package that arrives from the User's own other device, a confirmed contact, the Cloud sync folder, or a file is accepted even if the TUF `timestamp.json` has expired (7 days), provided the `snapshot` and `targets` signatures are correct and the version is newer than the local one, and is marked “the date of checking is old (date)” (a package that arrives cannot pass without NRSD's signature, so the risk of freezing is resolved by fetching from the network). The mark disappears when a fetch from the network succeeds.
- The package can be fetched file by file, and the app fetches only the files whose SHA-256 differs from the local ones. Each file is placed compressed (zstd), and the index holds the sizes before and after compression and the budget. M-01“Reference Information” warns at 80% of the budget. The index can carry “correction” marks (Chapter 5, 2.4). There are five sources (the public page, the Hong Kong mirror, the User's own other devices and “confirmed” contacts (5.4), a package placed in the User's Cloud sync folder (the backup location), and a file the User chooses), and the newest version is taken from whichever source is reachable. Verification (the table above) is the same regardless of the source. As with app updates, nothing is shown to the User (the same idea as Windows Delivery Optimization receiving from both the CDN and other PCs). The package is public information protected by NRSD's signature, so trust does not change with the route. What flows to other devices and contacts is only the package and its version; no User information or records flow (can be turned off in G-20“Settings”) (NRSD's request, 2026-09-30).
- The package index and the update manifest are unified into TUF-form metadata (Chapter 9, 4.3) and verified with tough. Verification of the version and expiry of `index.json` is replaced by the verification of the TUF snapshot and timestamp (NRSD's request, 2026-10-01).
- Bundled package: each release of the app bundles the latest reference information package at that time (with the TUF metadata). Enclosed Documents, sample Notice texts, and model texts can be made even if the first start is offline. If a package obtained after start has a larger version than the bundled one, the obtained one is used (check 3 of 3.4).
- The procedure for replacing keys and for incidents follows Chapter 1, 18.3 “Incidents involving NRSD's keys” (replacement of the TUF role keys; Chapter 9, 4.3). The period during which fake packages may be accepted if a key leaks is in Chapter 13, H-22“If NRSD's keys (update, reference information) are stolen, fake updates or reference information may be distributed”.
- Threat T-7 (spoofing, tampering): distributing fake reference information or updates. Attacker: a person who took over the Public Repository. Target asset: all Users. Remaining gap: leak of the signing key itself
- Threat T-20 (tampering): keep returning old valid reference packages or rolling back versions (freeze, rollback). Attacker: a party who took over the mirror or the network path. Target asset: the app (reference information). Remaining gap: during a freeze, new contact points do not arrive
- Threat T-25 (tampering): distributing a fake reference information package from another device or a contact. Attacker: a party who learned the device number. Target asset: the reference information. Remaining gap: —
- Secret: the reference-information signing key. Location: hardware tokens (NRSD's management PC). If leaked: fake reference information may be distributed. Replace the TUF role keys (Chapter 1, 18.3)

### 3.5 Protection of the GitHub account and repository

|Target|Settings|
|---|---|
|Organization|NRSD's organization account. Two owners (a trusted separate person; one until one can be appointed). All members are required to use two-factor authentication (an organization setting; those who disable it are removed from the organization). Owners' activity is checked every six months (OpenSSF's SCM Best Practices)|
|Two-factor authentication|Register two or more passkeys or security keys (one kept in another place) and an app for time-based one-time codes. SMS is not used (GitHub's guide: SMS carries risks). The recovery codes (16) are copied onto paper and kept in a physical place (a safe) separate from the password manager. As recovery factors, an SSH key for authentication and registered devices are also held (GitHub's “Configuring two-factor authentication recovery methods” strongly recommends registering two or more methods and storing recovery codes)|
|Preparation for losing the organization|Keep a copy (clone) of the Public Repository on the operator's PC so it can be rebuilt elsewhere even if the organization is lost (GitHub does not recover the content of frozen or lost organizations, but public content can be cloned). The organization name, releases, and existing URLs are lost (Chapter 13, H-45“While the GitHub organization has one owner, if that person loses all authentication and recovery methods, the organization cannot be recovered”)|
|Main branch|Rulesets prohibit force-pushes and deletion, and changes go through pull requests. Changes cannot be merged unless the CI checks (Chapter 11, 8.1 “On every change”) pass|
|Tags|A rule limiting creation and deletion of `v*` tags to owners|
|Releases|Enable immutable releases. Attach all distributables, update signatures, and the SBOM in draft state before publishing (nothing can be added after publication). A version with errors is not deleted; its title is changed (Chapter 9, 6.5 “When a version with errors has been released”). The update manifest is not put in the release but placed in `update/` on the public page (3.1; for replacing and rolling back the manifest)|
|Actions|Third-party Actions are pinned by commit hash (Chapter 11, 8.1 “On every change”). Release secrets are placed in an environment usable only by the release CI that runs when a `v*` tag is created (Chapter 11, 8.3 “Handling of secrets”)|
|Checking by Users|The README shows how advanced Users can check the release attestations automatically attached to immutable releases|
|Other providers' accounts (Apple Developer, Azure, Alibaba Cloud, Chrome Web Store, AMO, the DNS registrar)|The same protection as the two-factor authentication row above (two or more passkeys or security keys, no SMS, recovery codes on paper in another place). The owners are the same two people as for GitHub|
|Issues and Discussions|Disabled. Private vulnerability reporting is enabled, and the procedure is written in SECURITY.md (so vulnerabilities are not written in public; paired with the security.txt of RFC 9116 (NRSD's request, 2026-10-01)). The feedback channel is limited to e-mail (Chapter 10, 9 “Feedback Channel”). This avoids Users writing repost URLs or their own information in public places (DD-8-4“The public page carries no repost site names or User information”, Legal Research L-18“GitHub terms”). The README says “Send feedback by e-mail”|

- Threat T-22 (spoofing, tampering): taking over CI or the GitHub organization to place correctly code-signed fake builds in official releases and rewrite the SHA-256 in the README. Attacker: a party who took over CI or an owner account. Target asset: Users obtaining the app for the first time. Remaining gap: new downloaders can only tell by the OS code signing and the SHA-256 in the README, and cannot tell if both are taken (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”)
- Secret: the GitHub organization owner accounts. Location: passkeys or security keys (2 or more), time-based one-time codes, paper recovery codes (Chapter 8, 3.5 “Protection of the GitHub account and repository”). If leaked: the Public Repository, releases, and public page (including the SHA-256 in the README) could be rewritten (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”). Report to GitHub and recover the account
- Secret: the accounts of Apple Developer, Azure, Alibaba Cloud, the extension stores (Chrome Web Store, AMO), and DNS (nrsd.jp). Location: the same protection as for GitHub in Chapter 8, 3.5 (two or more passkeys or security keys for two-factor authentication, no SMS, recovery codes on paper in another place). If leaked: report to each company and recover. Code-signing credentials and the mirror key follow the procedures of the rows above

### 3.6 Operation of the Hong Kong mirror

|Item|Setting / procedure|
|---|---|
|Location|One bucket in Alibaba Cloud OSS's Hong Kong region. Standard storage class|
|Access|Public read only. Listing is prohibited. Writing is limited to the NRSD operator's key (a RAM user with write permission only to this bucket)|
|Access logging|Not enabled (DD-8-10“The mirror keeps no access logs”)|
|How copied|In step 6 of Chapter 9, 6.1 “Procedure”, the operator copies the GitHub release distributables, update signatures, the mirror's update manifest, and the reference information package. After copying, re-download from the mirror and compare the SHA-256 with GitHub's. Only the TUF metadata (`tuf/`; Chapter 9, 4.3), because `timestamp.json` expires in 7 days, is copied automatically to both the public page and the mirror each time the CI `tuf-online` environment re-signs it (a separate key that can write only to `tuf/` of the mirror bucket; Chapter 11, 8.3)|
|Old versions|Distributables are kept only for the latest and previous versions, and older ones are deleted (to keep capacity and cost down; GitHub releases keep all versions). The mirror is used for obtaining updates and for direct download from the Chinese section of the README (Chapter 9, 5 “Download Routes”). README links point to per-version distributables, and the README is revised at each release. The mirror URL is not put in Notices or Enclosed Documents. No HTML entry page is placed. All versions of the reference information package are kept (they are small; not needed for version verification, but Users can check them)|
|Cost|6.3|

- Secret: the Hong Kong mirror write keys (two). Location: Alibaba Cloud RAM user AccessKeys. The one for distributables and packages (with write permission to this bucket only) is on the operator's management PC (step 6 of Chapter 9, 6.1 “Procedure”; the operator does the copying). The one for the TUF metadata (permission to write only under the `tuf/` prefix) is in the CI `tuf-online` environment (Chapter 11, 8.3). If leaked: the mirror's content could be rewritten. Distributables and reference information are verified by signatures and would not be applied, while rewriting of the update manifest is prevented by version checking (Chapter 9, DD-9-2“Updates are fetched via Tauri's mechanism, checking the version in the manifest against the signature”). Revoke and recreate the key

## 4. Integrity

- Tamper detection: work data, case records, status additions, and authorizations are given record signatures with the User's key (Chapter 1, DD-1-5“Device records carry record signatures and are verified”) and verified when read. For evidence files, the SHA-256 is recorded in the case record and a timestamp is attached (Chapter 6).
- History: records are append-only. Corrections and withdrawals are also appended as new records (Chapter 6, DD-6-5“Case records are append-only”).
- When a record that fails verification is read, that record is not used, the User is notified (a `STO` code of Chapter 1, 10.3 “Errors”), and restoration from a backup file is suggested.
- At start, record signatures of all records are verified in the background (at most once a day if there are many; initial value).
- Record signatures are held per row as a chain, and only corrupted rows are quarantined. Verification at start proceeds from the chain heads and does not reread everything (Chapter 1, 8.3) (NRSD's request, 2026-09-30).
- Threat T-6 (tampering): rewriting records on the device. Attacker: a person with access to the device. Target asset: case records, evidence. Remaining gap: when the signing key is stolen as well

## 5. Backup Files

### 5.1 Format

- The outer layer is age format (C2SP's age specification). The recipient stanza is a single passphrase (scrypt) stanza only (in the age specification, a passphrase stanza must be the only stanza). The scrypt work factor is 2^18 (the default of age's official Go implementation, noted as “one second on a modern machine”).
- Recipient: the backup is encrypted to the public key of the “recovery key” (an X25519 key; its text form is age's standard key string (`AGE-SECRET-KEY-1…`; Bech32). At first run it is made as an “emergency kit” (the same form as 1Password's Emergency Kit; a single PDF (PDF/A-2u; ISO 19005-2; long-term preservation and printing; it also has the PDF/UA-1 (ISO 14289-1) structure so that it can be read aloud): the QR code of the recovery key (`recovery` of Chapter 1, 10.10 “Forms of URLs and QR codes”) and its text, the backup locations, how to restore, and instructions for the person taking over; three languages), and the User is guided to print and keep it. It is not kept on the device. NRSD holds no copy), and a file with the recovery key's private key encrypted with the passphrase (scrypt) is attached to the backup. When opening with the passphrase, the attached key is decrypted first and then the backup is opened. When opening with the printed recovery key, it is opened directly. Even if the passphrase is forgotten, the recovery key can restore it. age's rule that a passphrase stanza must be the only stanza is satisfied by this two-stage form (NRSD's request, 2026-09-30).
- There is one recovery key per personal root. Its public key is reconciled to other devices as a device-independent setting (Chapter 10, 10), and every device's backups are encrypted with the same key. After “Remake the recovery key” when the printout is lost, later backups are made with the new key, and it is shown that old backups can be opened only with the old printout.
- The content is ZIP.

|Inside the ZIP|Content|
|---|---|
|`manifest.json`|Backup format version (`schema: nrsd.backup/1`), creation date/time, app version, format version of each record, notice code of the personal root, device number, `kind` (`full` or `diff`), `chain_id` (new at each consolidation), `seq` (from 1), `parent_sha256` (the SHA-256 of the previous backup's `manifest.json`), list of files (path, size, SHA-256). When restoring, the `full` and then the diffs are applied in `seq` order, the chain of `parent_sha256` is checked, and if there is a gap or a swap, it stops there and offers “diff N is missing” and restoring up to just before the gap (the same as the chain of snapshots of Duplicati and Borg)|
|`keys/`|The personal root key and the signing certificate key (PKCS#8). Taken out of the keystore and put in|
|`data/`|The items marked “Included” in the table of 2.1|

- The extension is `.nrsdbak`.
- Components: the Rust version of age (age 0.12.1, MIT OR Apache-2.0), zip (Chapter 11, 4.1 “Rust components”).

### 5.2 How it is made

|Order|Processing|
|---|---|
|1|The User chooses the destination and a passphrase. Not accepted are: passphrases in a list of commonly used ones (100,000 entries; bundled; the source is recorded in the list of assets (Chapter 11, 6)), those estimated weak by the strength estimate (the zxcvbn method; Chapter 11, 4.1), and those shorter than 15 characters (the lower limit for single-factor passphrases in NIST SP 800-63B-4; for Users who printed the recovery key of the emergency kit, the multi-factor lower limit of 8 characters or more). No composition rules such as mandatory symbols or capitals are imposed (same source; NRSD's request, 2026-10-01). An estimate of the time to crack is shown on screen (NRSD's request, 2026-09-30)|
|2|Check that no export or registration is in progress (if there is, wait until it finishes)|
|3|Read the keys from the keystore, read the records, and encrypt with age while putting them into the ZIP, writing under a temporary name (processed sequentially without putting the whole in memory)|
|4|After writing, reread the file, decrypt the header, and check against the SHA-256 list in `manifest.json`|
|5|Rename to the official name and record the creation date/time in the settings (for the “last backup” display on Home; 5.5)|

- Guidance on destinations: an external storage medium (USB storage, etc.) or the User's cloud is recommended rather than the same PC. As it is encrypted, it may be placed in the cloud, but it is shown that it may be read if the passphrase is weak.
- It is shown at creation that it cannot be restored if the passphrase is forgotten, and a confirmation press is required.
- [To be measured] Time to create a 5 GB backup including evidence. Criterion: within 15 minutes (initial value) on a typical PC (Chapter 1, 14.1 “Scale and performance targets”).
- Threat T-5 (information disclosure): stealing and reading a backup file. Attacker: a person who obtained the file. Target asset: signing keys, records. Remaining gap: weak passphrases
- Secret: the backup file passphrase. Location: the User's memory (the app does not store it). If leaked: anyone holding the backup file may read it. Create a new backup and delete the old one

### 5.3 How it is restored (import)

|Order|Processing|
|---|---|
|1|The User enters the file and the passphrase|
|2|Read the age header, and do not accept scrypt work factors over 2^22 (the upper limit accepted by age's official Go implementation; prevents crafted files that take long to decrypt)|
|3|Read the ZIP while decrypting (the common checks are in Chapter 1, 10.2 “Storage”) and check `manifest.json` (format, version, list of files). Abort if the total extracted size does not match the sizes in the list|
|4|Check the SHA-256 of each file against the list|
|5|Check whether the personal root is the same as the current device's, or whether there are no records on the device (5.4)|
|6|Put the keys in the keystore, extract the records to a temporary location, verify their record signatures, and then move them to the official location|
|7|Show the number imported and the list of records that failed verification|

- Handling of the current device at import:

|Current device|Handling|
|---|---|
|No records (new device)|Imported as is|
|Has records of the same personal root (using two devices)|Merged (rules below)|
|Has records of a different personal root|Not merged (DD-8-6“Backups with a different personal root are not merged”). The User chooses “Replace the current records with the backup file (a backup of the current records is made first)” or “Cancel”|

- The reconciliation per record follows the bullets of 5.4 “Device-to-device synchronization”.
- Backups made with a newer app (with newer record format versions) are not imported, and the User is prompted to update the app.
- Open as successor: on a device where the backup was restored with the recovery key of the emergency kit, when the User chooses “Open as successor”, it opens in a read-only state that does not use the signing key (no new C2PA signing or Registration), and only viewing works, cases, and evidence, exporting evidence packages, and regenerating Enclosed Documents are possible (Chapter 1, 13) (NRSD's request, 2026-09-30).
- Threat T-11 (tampering): tampering with imported templates or backup files. Attacker: the party handing over the file. Target asset: app core, records. Remaining gap: —

### 5.4 Device-to-device synchronization

- Procedure between devices: after connecting, the two first show each other their signing certificate chains and confirm the same personal root by signing the other's random value (mutual challenge-response). Next they exchange the heads of each chain (the `seq` per stream and device number) and send the missing rows. Large files such as evidence and assets are received with iroh-blobs (content-addressed transfer referenced by BLAKE3 hashes; can resume midway). The protocol name is ALPN `c2pa4cosplayer/sync/1`, and the messages are CBOR. When linking a new device, after the link check (QR code or passphrase), the existing device hands over the keys of the personal root and the signing certificate over the same direct connection (end-to-end encrypted), and the new device puts them in its keystore (the same result as restoring from a backup; the device key is made per device).
- Device-to-device synchronization: devices holding the same personal root connect in the same flow as Signal's “linked devices” (a QR code (`link` of Chapter 1, 10.10 “Forms of URLs and QR codes”) appears on the new device, any already linked device scans it, and from then on it is automatic. There is no “primary device”; any linked device holding the signing key can link a new device (the same as Delta Chat's “Add second device”; Signal's “only the primary device can link” is not adopted)). The mechanism underneath is the same as Syncthing and Delta Chat (a direct QUIC connection called by the device public key; iroh; a public relay punches the NAT hole, and once connected the communication is direct and end-to-end encrypted; once directly connected, it leaves the relay. The relay sees only IP addresses and random device numbers and cannot read the content). They connect automatically wherever they are (at home and away, even across countries), and the unit of reconciliation is the rows of chains (work data, case records, status additions, authorizations), plus templates, work sessions, and device-independent settings (the scope is the table below). Records are append-only, so they do not conflict. The reconciliation per record follows the bullets below. Devices also serve as each other's backup location (5.5). While not connected, diff backups catch up later through the Cloud sync folder. “Devices” in G-20“Settings” has the list (name, time last synchronized, “Remove this device”; up to 5). A setting not to use relays (only the same network and the Cloud sync folder) is also provided. NRSD has no server (D-6-2“The System shall not be built as a website”). The “automatic synchronization” rejected by DD-1-4“All records are kept on the User's device” is one that requires a location on NRSD's side, and direct synchronization between devices is not that (NRSD's request, 2026-09-30).
- Connection from a passphrase (when QR codes cannot be shown to each other; common to adding contacts in Chapter 2, 5.5 and to linking devices): the passphrase has the same form as Magic Wormhole (one number and two words from the PGP word list; one-time, expires in 10 minutes). From the passphrase, the seed of an Ed25519 key pair is derived with Argon2id, and the initiating side publishes a record signed with that key (its own device number and a 10-minute expiry) to pkarr (signed records in the BEP 44 DHT; the same mechanism as iroh's `discovery-pkarr-dht`, carried on the BitTorrent mainline DHT; no NRSD server is needed). The other side looks it up with the same key, obtains the device number, connects directly, performs a SPAKE2 (RFC 9382) key agreement with the passphrase, and lets through only a peer who knows the passphrase (the same as Magic Wormhole). The record is deleted after one use or after 10 minutes (NRSD's request, 2026-10-01).
- Means of linking a device: besides a QR code, the passphrase above can be used (the QR code is optional).
- Withdrawn evidence: a withdrawal (Chapter 6, 7) is synced as an appended row. When another device receives the row, it moves the evidence to the OS trash. It remains in backups and is removed at the next consolidation (monthly; 5.5).
- Reconciliation per record: chain rows, evidence, assets, and authorizations follow Chapter 1, 8.3 “Formats and versions”; templates and work sessions follow Chapter 4, 8.2 “Versions”; settings follow Chapter 10, 10 “Settings”. Importing backups (5.3) uses the same rules.
- Threat T-24 (spoofing): posing as the User's device or a confirmed contact to push in records or documents, or to read records. Attacker: a party who learned the device number. Target asset: the device's records, documents. Remaining gap: when the device key is stolen (the same as T-4“Stealing a User's signing key to C2PA-sign”)
- Threat T-26 (information disclosure): the relay for direct connections sees the User's IP address and device number. Attacker: the operator of the relay. Target asset: the User's location. Remaining gap: the IP address is visible to the relay (the same as connections to the TSA and GitHub)

### 5.5 Backup locations and automatic backups

- Right after the first input (G-05“Create Backup”), reachable locations (external storage, the User's Cloud sync folder, another device of the same person (5.4)) are detected and proposed, and one or more are decided. From then on, every hour (initial value; the same as the defaults of Time Machine and Windows File History) and when closing, if records have changed, the app makes a backup automatically. The word on screen is “Backup”, and Home shows “Last backup: N minutes ago”. Whether to include the Originals themselves (RAW, pre-development) in backups is offered at first run, and the default is to include them (Chapter 2, 4.2). The User does nothing. When a location is unreachable, it is made at the next opportunity (NRSD's request, 2026-09-30).
- Backing up Originals: Originals are backed up as diffs (only new Originals). Originals are not included in Cloud sync folders, only in external storage and other devices. An estimate (count and GB) is shown at the first time.
- At the first time, only external storage and Cloud sync folders are detected. Other devices are added as locations when linked in G-20.
- Detecting locations: external storage is the OS's removable media (Windows `GetDriveType` `DRIVE_REMOVABLE`; macOS DiskArbitration's `Removable` and `Ejectable`; Linux UDisks2's `Removable`). Cloud sync folders are each company's default locations (Dropbox: `path` in `~/.dropbox/info.json`; OneDrive: the environment variable `OneDrive`; Google Drive: a drive named `Google Drive` / `My Drive`; iCloud: `~/Library/Mobile Documents/com~apple~CloudDocs`). If none is found, the User chooses.
- Backups are stacked as diffs (new rows and new evidence files since the previous one) and consolidated once a month (initial value). Importing applies the full backup and then the diffs in order.
- For each location, the list of backups and the date/time of the last successful verification are shown on Home (G-07“Home”). The monthly restore check is done only when the temporary location has free space of twice the backup's size, and otherwise the User is notified. In addition to the reread right after creation (5.2), once a month the backup at the location is actually restored to a temporary location and the record signatures of all records are confirmed to pass (checking that it is a restorable backup before it is lost).
- Only when no location has been reachable for 7 days (initial value) is the User notified on Home. The manual backup operation (5.2) also remains.

### 5.6 Guidance on where to keep backups

- On the backup creation screen (G-05“Create Backup”) and in the README, the following is shown along the generally recommended “3-2-1” rule (keep three copies of important files, on two kinds of media, with one in a separate place; US CISA “Data Backup Options”).

|Copy|Example location|
|---|---|
|First|The app's records on this PC (the original)|
|Second|A backup file on an external storage medium (USB storage, external disk)|
|Third|A separate place: the User's cloud (the backup is encrypted), or a storage medium kept elsewhere|

- A backup file is made with a new name each time (`backup-<YYYYMMDD-HHMMSS>-<seq>.nrsdbak`), and old backups are not overwritten. At the monthly consolidation (5.5) the app folds the diff backups it made itself into a new full backup and deletes the folded diffs (the same as Time Machine thinning old snapshots; backup files the User copied by hand are not touched). Whether to delete full backups is up to the User, and if there are three or more full backups of the same personal root at the location, the app only shows that the oldest may be deleted.

## 6. Capacity

### 6.1 Guide to device capacity

|Data|Estimate (from initial values)|
|---|---|
|Work data|50 KB per work (including the copy of the manifest and the reduced image) × 5,000 works per year = 250 MB per year|
|Evidence|20 MB per case (typical) × 200 cases per year = 4 GB per year. The limit per case follows Chapter 6, 2.4 (1 GB)|
|Work sessions|A few MB per work session (only photo locations and adjustments)|
|Templates and assets|Tens of MB|
|Reduced images (cache)|About 50 KB per image × 500 images = 25 MB per work session (can be regenerated if deleted)|

- When free space falls below 1 GB (initial value), the User is notified and guided to put old cases' evidence into a backup file before deleting. Before deleting, the period for claims is shown (Chapter 1, 8.4 “Retention and deletion”).
- When the cache exceeds 1 GB (initial value), reduced images of old work sessions are deleted first.

### 6.2 Mainland China

- Obtaining (updates, reference information): obtained from GitHub, and if that fails, from the Hong Kong mirror (DD-8-5“A Hong Kong mirror is set up for mainland China”). Distributables and reference information packages are verified by signatures, so replacement at the mirror can be detected. The update manifest (JSON) has no signature, so forcing an old version by rewriting the manifest is prevented by checking the manifest's version against the version in the update signature (Chapter 9, DD-9-2“Updates are fetched via Tauri's mechanism, checking the version in the manifest against the signature”).
- First download: the Chinese section of the README carries direct links to the latest distributables on the mirror and their SHA-256 (3.6, Chapter 9, 5 “Download Routes”). If the README itself cannot be reached, it cannot be obtained (Chapter 13, H-17“In mainland China the README may be unreachable”).
- Writing records is completed within the device, so no connection to GitHub is needed.
- Reference information also arrives from the User's own other devices, confirmed contacts, the Cloud sync folder, and files (3.4, Chapter 1, 7.5). A “portable kit” (the app's distributable and the reference information package) can be written to USB media from G-20“Settings” (Chapter 9, 5). If one person at a venue has the latest version, it reaches those around (NRSD's request, 2026-09-30).
- Candidate mirror: Alibaba Cloud OSS in the Hong Kong region (static delivery). The Hong Kong region does not require an ICP filing (Alibaba Cloud's guide). There is guidance that latency from mainland China is 30 to 50 ms (same).
- Cost: determined by storage and outbound transfer. [Estimate required] Size of distributables (Chapter 11, 4.3 “Assets”) × number of downloads.
- [To be measured] Reachability and speed from mainland China to the mirror. Criterion: updates and reference information can be obtained from the mirror. If not met, it is declared as a gap (Chapter 13).
- Risk: unreachable from mainland China. Impact: Users in China cannot obtain or update the app. Preparation: the Hong Kong mirror (direct download from the Chinese section of the README, and fetching updates and reference information; Chapter 8, 3.6 “Operation of the Hong Kong mirror”, Chapter 9, 5 “Download Routes”)

### 6.3 Estimate of mirror cost

- The cost is determined by storage and outbound transfer. Outbound transfer from the Hong Kong region is free up to 5 GB per month (Alibaba Cloud OSS guide). The unit price table is on Alibaba Cloud's pricing page, but in this document's research (2026-09-29) the screen could not be obtained. [Estimate required] NRSD checks the unit prices on the pricing page before contracting.
- Estimation formula: monthly outbound transfer = size of distributables (the expectation in Chapter 9, 2.1 “Distributables per OS”: around 200 MB per OS, with Linux larger by the bundled components; the design target is 300 MB or less) × number of Users updating or newly installing from the mirror per month + reference information (1 MB or less) × number of downloads. Storage = distributables of one version (Windows installer, macOS DMG and `.app.tar.gz` for updates, Linux AppImage: four items of about 200 MB each) × two versions (about 1.6 to 2 GB) + source packages of the LGPL components bundled in the Linux AppImage (for two versions; [Estimate required] size) + reference information.
- Example: if 200 mirror Users update per month, transfer is about 50 GB per month.

## 7. Dependence on GitHub

|Event|Preparation|
|---|---|
|GitHub outage|Updates and reference information are obtained from the mirror. Records are on the device, so work does not stop|
|Suspension of the account or repository|Move the Public Repository to the mirror and another git host. For the public page, pointing the CNAME of the custom domain (`c2pa4cosplayer.nrsd.jp`; 3.1) to another static host leaves the app's fetch URLs unchanged. The app holds several sources for updates (Chapter 9, DD-9-2“Updates are fetched via Tauri's mechanism, checking the version in the manifest against the signature”)|
|Terms|No personal information or repost information is placed in the Public Repository (Legal Research L-18“GitHub terms”)|
|Release capacity|A single release file must be under 2 GiB, and one release can have up to 1,000 files. There is no limit on the total size of releases or on bandwidth (GitHub's guide “About releases”). The design target for distributable size is 300 MB or less (Chapter 9, 2.1 “Distributables per OS”). The 500 MB in Chapter 1, 4.2 “Communication with the outside” is the download limit; distributables exceeding it are not downloaded|
|Account takeover|The protections of 3.5. Even if taken over, updates and reference information are not applied without the TUF metadata signed with NRSD's offline keys and the update signature (3.2, Chapter 9, 4.3)|

- Risk: suspension of the GitHub account. Impact: distribution stops. Preparation: the mirror and another hosting service (Chapter 8, 7 “Dependence on GitHub”)

## 8. Handling Failures

|Event|How it is reported|What the User does|
|---|---|---|
|Wrong passphrase for a backup|“The passphrase is wrong or the file is broken” (age does not distinguish the two)|Check the passphrase|
|The backup file is broken|Same. If found in the middle of extraction, what was imported is rolled back|Use another backup|
|Not enough free space at the backup destination|The estimate is shown before creation, and it does not start|Choose another destination|
|Record signature does not match|That record is not used, and a list is shown|Restore from a backup|
|Not enough free space on the device|The notice of 6.1|Put old evidence into a backup and then delete it|

## 9. Mapping to Requirements

|Requirement number (text in the Outline Design Document)|Sections in this chapter|
|---|---|
|R-8-1-1|3 “Public Repository”|
|R-8-1-2|3.2 “Distinguishing modified versions”|
|R-8-2-1|2.1 “Arrangement”|
|R-8-2-2|2.2 “Not leaving the device”|
|R-8-4-1|4 “Integrity”, 5 “Backup Files”|
|R-8-5-1|6.1 “Guide to device capacity”|
|R-8-5-2|6.2 “Mainland China”|
|R-8-7-1|7 “Dependence on GitHub”|

- Four requirements on writing to the private repository and accepting Users (the former 8-3 and 8-6 of the Outline Design Document) were deleted in Design Plan Edition 2.

## 10. Gaps Declared in This Chapter

- The gaps of this chapter follow the table in Chapter 13, 4.1 “Gaps in the mechanism” (the rows whose chapter column is this chapter; with why they cannot be closed, the extent addressed, the remaining risks, and who bears them) (not reproduced in this chapter).

## 11. Corrections to Other Chapters and the Outline Design Document

- Chapter 2, 12 “Corrections to Other Chapters and the Outline Design Document” and Chapter 11, 4.1 “Rust components”: change the backup encryption to age (DD-8-3“Moving devices and preparing for failure use backup files (age format)”).
- Outline Design Document Chapter 8: the text of R-8-1-1 to R-8-7-1 is not changed.
