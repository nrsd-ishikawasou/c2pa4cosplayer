# Basic Design Document Chapter 8: Repository and Data Management

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [08_Basic Design_Repository and Data Management.docx](08_Basic%20Design_Repository%20and%20Data%20Management.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

## Position of This Document

- Details Chapter 8 of the Outline Design Document. This chapter sets the arrangement of data on the User's device, what leaves the device, backup files (format, creation, restoration, merging), the Public Repository and public page, the Hong Kong mirror, integrity, capacity, and dependence on GitHub. It is designed together with Chapter 2 “Signing Information and Matching”, and goes back and forth with Chapter 6 “Registration and Evidence Preservation” on capacity.
- This chapter is based on the study memo “Study of Chapter 2 Signing Information and Matching and Chapter 8 Repository and Data Management” (NRSD's internal record of study, not distributed with this document; the facts underlying the design are written with their sources in each section of this document). The third draft added on-device encryption of records, the form of the reference information package, protection of the GitHub account and repository, operation of the mirror, and where to keep backups (the facts researched are in the second-round section of the same study memo).
- The decisions received are as in the following table.

|Number|Type|Content|
|---|---|---|
|D-3-3|Decision|Users shall be able to use the System without looking at GitHub|
|D-10-7|Decision|Evidence is stored on the User’s device and not published. The System does not collect evidence|
|D-12-1|Decision|The Public Repository is provided. No Private Repository is provided|
|D-12-2|Decision|The software is published, and its being taken away is not prevented|
|D-12-3|Decision|Rights Holder Information, the signing key, work data, the ledger, and evidence are kept on the User’s device; NRSD does not collect them|
|D-12-4|Decision|The public page is viewable and not editable, and information that could be used for impersonation is not posted on it|
|D-12-5|Decision|Fork operation by Users themselves is not presupposed|
|I-04|To Be Investigated|Access to GitHub from mainland China|
|O-05|Open (decided in Basic Design Chapter 8, 3.3 “Public page (decision on O-05“Content of the public page”)”)|Content of the public page|
|O-10|Open (decided in Basic Design Chapter 3, 7.2 “Work data fields (decision on O-10“Recording format of work data”)”)|Recording format of work data|
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
|DD-8-8|Reference information is distributed as a “package” (version number, issue date/time, expiry, and a list of files with SHA-256), and the package's index is signed with NRSD's reference-information signing key (ECDSA P-256 in a hardware token). The app does not accept packages of an older version than it has, and keeps using expired packages while notifying|Protection against, besides fake reference information, attacks that keep serving old valid packages (rollback and freeze). Addresses the attacks listed in The Update Framework (TUF) specification (rollback, indefinite freeze, endless data, mix-and-match)|Attaching only record signatures (old valid packages cannot be distinguished)|
|DD-8-9|NRSD's GitHub is an organization account, with two owners (a trusted separate person as the second; fewer than three). Until a second can be appointed it is operated with one owner, and the risk of not being able to recover the organization is declared as a gap (Chapter 13, H-45“While the GitHub organization has one owner, if that person loses all authentication and recovery methods, the organization cannot be recovered”). An owner is not the same person's second account. All members are required to use two-factor authentication, and owners register several authentication methods (two or more passkeys or security keys, one kept in another place; time-based one-time codes). The Public Repository prohibits force-pushes and deletion of the main branch by rules, and releases are immutable releases|GitHub recommends having two or more “people” as organization owners (“Maintaining ownership continuity for your organization”). Free accounts are limited to one per person (GitHub Terms of Service B.3). GitHub does not recover an account that has lost all two-factor methods and recovery methods, nor does it recover by identity verification (“GitHub account recovery policy”). OpenSSF's SCM Best Practices say fewer than three owners. GitHub has required two-factor authentication for users who create releases and others (since March 2023). Immutable releases cannot have assets and tags changed after publication, and release attestations are created automatically (GitHub's guide). Prevents replacement of distributables through account takeover|Placing it under a personal account (stops with one person's takeover or accident). Making a second account of the Lead Developer an owner (free accounts are limited to one per person by the terms, so it cannot be had for free; even if paid, it does not meet GitHub's recommended “two or more people” and does not prepare for accidents in which that person can no longer act)|
|DD-8-10|Access logging (log storage) of the Hong Kong mirror (object storage) is not enabled. Listing is prohibited, and only public reading is allowed|Access logs would keep the IP addresses of Users who downloaded, meaning NRSD collects Users' information (Design Plan D-5-1“The System does not confirm Users and collects no User information. No registration is required in order to use the System”). Access logging of Alibaba Cloud OSS is disabled by default, and no operation is done to enable it|Keeping logs for investigating failures (collects Users' information)|

## 2. Device Data

### 2.1 Arrangement

- Location: the app's folder under the app's data location (app_local_data_dir of Chapter 1, 9.1 “Locations per OS”).

|Path|Content|Writer|Included in backup|
|---|---|---|---|
|`identity/`|Personal root and signing certificate, entered Rights Holder Information, history of changes (`history.jsonl`; Chapter 2, 6 “Rights Holder Information”)|App|Included|
|`identity/past_roots/`|Certificates of past personal roots (public keys only), their notice codes, periods of use, reasons for remaking (Chapter 2, 3.6 “Past personal roots”)|App|Included|
|`identity/key.age`|Only on Linux without Secret Service: the key encrypted with a passphrase (Chapter 2, 7.5 “Storage method per OS”)|App|Not included (keys go into the backup separately; 5.2)|
|`works/<year>/<batch number>.jsonl` and `.sig`|Work data (Chapter 3, 7.2 “Work data fields (decision on O-10“Recording format of work data”)”). One file per batch|App|Included|
|`cases/<case number>/case.json` and `.sig`|Case record (Chapter 6)|App|Included|
|`cases/<case number>/status/<serial>.json` and `.sig`|Status additions (Chapter 6)|App|Included|
|`cases/<case number>/evidence/`|Evidence files (Chapter 6)|App|Included|
|`templates/<template number>/`|Templates (Chapter 4, 8 “Templates”) and their drafts. Location of sample photos (`local.json`; not included in template files for passing on; Chapter 4, 9.2 “View”)|App|Included|
|`assets/`|Imported fonts and images (named by SHA-256; Chapter 4, 5 “Image Layers”, 6.4 “Imported fonts”)|App|Included|
|`sessions/<work session number>/`|Work sessions (Chapter 4, 11 “Work Sessions (the State in the Middle of Editing)”)|App|Included|
|`approvals/`|Authorizations, revocations, joint-rights documents (Chapter 2, 5 “Authorizations and Joint Rights”)|App|Included|
|`reference/<version>/`|Obtained reference information and NRSD's record signatures|App|Not included (can be re-obtained from the Public Repository)|
|`settings.json`|Settings (Chapter 10, 10 “Settings”)|App|Included|
|`state/`|Running mark, in-progress state of export and registration (Chapter 1, 7.8 “Starting after an abnormal exit”)|App|Not included|
|`migration-backup/<original version>/`|Copies before record format migration (Chapter 9, 4.4 “Record format migration”; deleted after 30 days)|App|Not included|
|`thumbs/`|Reduced images of photos (can be regenerated). Placed under the cache location (app_cache_dir). On Windows the cache location is the same as the app's data location, so they are separated by this name|App|Not included|
|`logs/`|Operation logs. The log location (app_log_dir) is `logs/` under the app's data location on Windows and Linux, and `~/Library/Logs/<identifier>` on macOS|App|Not included|
|`EBWebView/` (Windows only)|Screen component data created by WebView2 (Tauri's default location). Not used by this app, but included in what is erased (Chapter 9, 2.3 “Procedure for “Erase this device's records””)|WebView2|Not included|

- Source for locations: the description of Tauri's PathResolver (the per-OS locations of app_cache_dir and app_log_dir; checked 2026-09-30). Data created by macOS WKWebView (`~/Library/WebKit/<identifier>`, etc.) is also included in what is erased ([To be measured] the actual location on macOS).
- Reduced images and operation logs are not included in backups.
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
|`reference/<version>/` (under the public page)|Reference information (contact points, texts, references to each country's law, wording of the Enclosed Document) and NRSD's record signatures. The app obtains it from the public page (GitHub Pages) URL (Chapter 1, 7.5 “Fetching reference information”)|
|Public page|3.3|

- No personal information or per-User information is placed there (R-8-1-1“It holds the software, the public page, and reference information, and no personal information”).

### 3.2 Distinguishing modified versions

- Reference information and updates are signed with NRSD's keys, and the app holds the public keys internally to verify them. Reference information and updates distributed by modified versions do not pass verification in the official app.
- How to tell the official version: Chapter 9, 3 “Proof of the Official Version”.

### 3.3 Public page (decision on O-05“Content of the public page”)

|What is placed|What is not placed|
|---|---|
|Explanation of the System, how to obtain it (stating the official sources)|Names and URLs of reposting sites|
|The matching procedure (how it looks on general verification sites, how to read the clues; Chapter 2, 4.1 “Matching with the Notice account”) and the “Check notice code” page (a static page that reads an image only within the browser and computes and shows the notice code from the certificate chain of the C2PA signature; the image is sent nowhere; Chapter 2, DD-2-9“The personal root certificate is included in x5chain”)|Users' handle names and Notice accounts|
|Version and update date of the reference information|Individual content of cases|
|That the System is not involved in the rights of the original works, and that it is not legal advice|—|
|Outages and announcements (Chapter 1, 18.1 “NRSD's points of involvement and structure”)|—|

- The public page is made with GitHub's public page feature, and edits go through the Public Repository (“not editable” in D-12-4“The public page is viewable and not editable, and information that could be used for impersonation is not posted on it” means viewers cannot edit it).

### 3.4 Reference information package

|File|Content|
|---|---|
|`reference/<version>/index.json`|`schema: nrsd.reference/1`, key identifier (SHA-256 of the public key of the reference-information signing key used), version number (an integer starting from 1; counted per key, only raised, never lowered), issue date/time (RFC 3339), expiry (120 days from issue; initial value; the quarterly review (Chapter 7, 2.2 “Keeping up to date”) plus a 30-day margin), the app's minimum version (Chapter 9, 4.2 “Users of old versions (forced updates)”), list of files (path, size, SHA-256)|
|`reference/<version>/index.json.sig`|ECDSA P-256 signature over `index.json` normalized with JCS (RFC 8785). The key is in NRSD's hardware token (the PIV digital signature slot; PIN entered for each signature)|
|`reference/<version>/contacts.json`, etc.|Contact points (Chapter 7, 2 “Contact Points”), sample texts (Chapter 7, 3 “Model Texts”), references to each country's law (Chapter 7, 4 “References to Each Country's Law”), wording of the Enclosed Document (Chapter 5, 3.7 “Draft sentences of the common part”), announcements (Chapter 10, G-21“Notifications”)|
|`reference/latest.json`|The latest version number and the SHA-256 of `index.json` (a pointer to which version to fetch; not itself a basis of trust)|

- The app's verification (the content of step 2 of Chapter 1, 7.5 “Fetching reference information”):

|Order|What is checked|When it fails|
|---|---|---|
|1|`index.json` is 64 KB or less, each file is no larger than the size in the list, and the total is 1 MB or less (the limits of Chapter 1, 4.2 “Communication with the outside”)|Stop fetching (protection against attacks that obstruct the device with large data)|
|2|Verify `index.json.sig` with the reference information public key embedded in the app|Not used. Recorded with a `REF` code|
|3|The key identifier is that of the public key embedded in the app, and the version number is larger than the version held locally, counted for that key|Packages smaller than or equal to the local one, and packages of old keys, are not used (protection against rollback)|
|4|The SHA-256 of each file matches the list|The whole package is not used (protection against mix-and-match)|
|5|Expiry|Even if expired, it is used as the latest held locally, and G-21“Notifications” shows “Reference information is out of date (last checked: date)” (protection against freezing). More than 30 days after expiry, it is also shown at the top of the contact point guidance (G-17“Action Guide”)|

- Bundled package: each release of the app bundles the latest reference information package at that time (with index and signature). Enclosed Documents, sample Notice texts, and model texts can be made even if the first start is offline. If a package obtained after start has a larger version than the bundled one, the obtained one is used (check 3 of 3.4).
- Replacing the public key: the reference-information signing key is replaced only by an app version containing the new public key (signed with the update signing key and distributed). The package index (`index.json`) holds the key identifier (SHA-256 of the public key), and the app counts version numbers per key. An app version containing the new public key does not accept packages of the old key, discards the version number counted with the old key, and counts package versions of the new key from 1.
  - Reason: the “minimum version” is inside the package and cannot be raised without signing with the old key (it cannot be raised when the key is lost). When the key leaks, an attacker can issue a package with an extremely large version number, and if the local version number becomes that number, correct packages after the key change would keep being rejected as “rollback”. Counting versions per key and entrusting replacement to the path of the update signing key avoids both (the same idea as TUF's specification of replacing the root of trust through a separate key path).
  - In case of a leak, until the new app version is installed, Users' apps may accept the attacker's packages (including fake contact points, fake announcements (G-21“Notifications”), and an inflated “minimum version” (Chapter 9, 4.2 “Users of old versions (forced updates)”; C2PA signing and registration stop)) (Chapter 13, H-22“If NRSD's keys (update, reference information) are stolen, fake updates or reference information may be distributed”). The procedure follows Chapter 1, 18.3 “Incidents involving NRSD's keys”.

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
|Issues and Discussions|Disabled. The feedback channel is limited to e-mail (Chapter 10, 9 “Feedback Channel”). This avoids Users writing repost URLs or their own information in public places (DD-8-4“The public page carries no repost site names or User information”, Legal Research L-18“GitHub terms”). The README says “Send feedback by e-mail”|

### 3.6 Operation of the Hong Kong mirror

|Item|Setting / procedure|
|---|---|
|Location|One bucket in Alibaba Cloud OSS's Hong Kong region. Standard storage class|
|Access|Public read only. Listing is prohibited. Writing is limited to the NRSD operator's key (a RAM user with write permission only to this bucket)|
|Access logging|Not enabled (DD-8-10“The mirror keeps no access logs”)|
|How copied|In step 6 of Chapter 9, 6.1 “Procedure”, copy the GitHub release distributables, update signatures, the mirror's update manifest, and the reference information package. After copying, re-download from the mirror and compare the SHA-256 with GitHub's|
|Old versions|Distributables are kept only for the latest and previous versions, and older ones are deleted (to keep capacity and cost down; GitHub releases keep all versions). The mirror is used for obtaining updates and for direct download from the Chinese section of the README (Chapter 9, 5 “Download Routes”). README links point to per-version distributables, and the README is revised at each release. The mirror URL is not put in Notices or Enclosed Documents. No HTML entry page is placed. All versions of the reference information package are kept (they are small; not needed for version verification, but Users can check them)|
|Cost|6.3|

## 4. Integrity

- Tamper detection: work data, case records, status additions, and authorizations are given record signatures with the User's key (Chapter 1, DD-1-5“Device records carry record signatures and are verified”) and verified when read. For evidence files, the SHA-256 is recorded in the case record and a timestamp is attached (Chapter 6).
- History: records are append-only. Corrections and withdrawals are also appended as new records (Chapter 6, DD-6-5“Case records are append-only”).
- When a record that fails verification is read, that record is not used, the User is notified (a `STO` code of Chapter 1, 10.3 “Errors”), and restoration from a backup file is suggested.
- At start, record signatures of all records are verified in the background (at most once a day if there are many; initial value).

## 5. Backup Files

### 5.1 Format

- The outer layer is age format (C2SP's age specification). The recipient stanza is a single passphrase (scrypt) stanza only (in the age specification, a passphrase stanza must be the only stanza). The scrypt work factor is 2^18 (the default of age's official Go implementation, noted as “one second on a modern machine”).
- The content is ZIP.

|Inside the ZIP|Content|
|---|---|
|`manifest.json`|Backup format version (`schema: nrsd.backup/1`), creation date/time, app version, format version of each record, notice code of the personal root, list of files (path, size, SHA-256)|
|`keys/`|The personal root key and the signing certificate key (PKCS#8). Taken out of the keystore and put in|
|`data/`|The items marked “Included” in the table of 2.1|

- The extension is specific to this app (e.g., `.nrsdbak`).
- Components: the Rust version of age (age 0.12.1, MIT OR Apache-2.0), zip (Chapter 11, 4.1 “Rust components”).

### 5.2 How it is made

|Order|Processing|
|---|---|
|1|The User chooses the destination and a passphrase. The passphrase is 12 characters or more (initial value). Passphrases in a list of commonly used ones (10,000 entries, initial value) are not accepted|
|2|Check that no export or registration is in progress (if there is, wait until it finishes)|
|3|Read the keys from the keystore, read the records, and encrypt with age while putting them into the ZIP, writing under a temporary name (processed sequentially without putting the whole in memory)|
|4|After writing, reread the file, decrypt the header, and check against the SHA-256 list in `manifest.json`|
|5|Rename to the official name and record the creation date/time in the settings (for the “last backup” display on Home and the 30-day recommendation; Chapter 1, 13 “Devices and People”)|

- Guidance on destinations: an external storage medium (USB storage, etc.) or the User's cloud is recommended rather than the same PC. As it is encrypted, it may be placed in the cloud, but it is shown that it may be read if the passphrase is weak.
- It is shown at creation that it cannot be restored if the passphrase is forgotten, and a confirmation press is required.
- [To be measured] Time to create a 5 GB backup including evidence. Criterion: within 15 minutes (initial value) on a typical PC (Chapter 1, 14.1 “Scale and performance targets”).

### 5.3 How it is restored (import)

|Order|Processing|
|---|---|
|1|The User enters the file and the passphrase|
|2|Read the age header, and do not accept scrypt work factors over 2^22 (the upper limit accepted by age's official Go implementation; prevents crafted files that take long to decrypt)|
|3|Read the ZIP while decrypting and check `manifest.json` (format, version, list of files). Abort if a path inside the ZIP contains `..` or is absolute. Abort if the total extracted size does not match the sizes in the list (Chapter 1, 10.6 “Validation of inputs from outside”)|
|4|Check the SHA-256 of each file against the list|
|5|Check whether the personal root is the same as the current device's, or whether there are no records on the device (5.4)|
|6|Put the keys in the keystore, extract the records to a temporary location, verify their record signatures, and then move them to the official location|
|7|Show the number imported and the list of records that failed verification|

- Backups made with a newer app (with newer record format versions) are not imported, and the User is prompted to update the app.

### 5.4 Merging

|Current device|Handling|
|---|---|
|No records (new device)|Imported as is|
|Has records of the same personal root (using two devices)|Merged (rules below)|
|Has records of a different personal root|Not merged (DD-8-6“Backups with a different personal root are not merged”). The User chooses “Replace the current records with the backup file (a backup of the current records is made first)” or “Cancel”|

- Merging rules (same personal root):

|Record|Rule|
|---|---|
|Work data (file per batch)|If the number and content are the same, one is kept. If the number is the same but the content differs, both are kept and the difference is shown (normally does not happen)|
|Case records and status additions|Per case number, status additions are combined and ordered by time (append-only records, so keeping both suffices)|
|Evidence|Combined by SHA-256 names (identical ones become one)|
|Templates and work sessions|If the number is the same, the one with the newer version number (last modified date/time) is taken, and the older one is kept as a backup|
|Authorizations, etc.|Combined by number|
|Settings|The current device's are kept|
|History of changes|Combined in time order|

### 5.5 Recommending backups

- After the first input, and when 30 days (initial value) have passed since the last backup, it is recommended on Home (Chapter 10, G-07“Home”).
- If, after registering evidence, there is evidence not yet backed up, Home shows the count.

### 5.6 Guidance on where to keep backups

- On the backup creation screen (G-05“Create Backup”) and in the README, the following is shown along the generally recommended “3-2-1” rule (keep three copies of important files, on two kinds of media, with one in a separate place; US CISA “Data Backup Options”).

|Copy|Example location|
|---|---|
|First|The app's records on this PC (the original)|
|Second|A backup file on an external storage medium (USB storage, external disk)|
|Third|A separate place: the User's cloud (the backup is encrypted), or a storage medium kept elsewhere|

- A backup file is made with a new name each time (including the creation date/time in the name), and old backups are not overwritten. Whether to delete old backups is up to the User. If there are three or more backups of the same personal root at the location, the app only shows that the oldest may be deleted (it does not delete the User's files).

## 6. Capacity

### 6.1 Guide to device capacity

|Data|Estimate (from initial values)|
|---|---|
|Work data|2 KB per work × 5,000 works per year = 10 MB per year|
|Evidence|20 MB per case × 200 cases per year = 4 GB per year|
|Work sessions|A few MB per work session (only photo locations and adjustments)|
|Templates and assets|Tens of MB|
|Reduced images (cache)|About 50 KB per image × 500 images = 25 MB per work session (can be regenerated if deleted)|

- When free space falls below 1 GB (initial value), the User is notified and guided to put old cases' evidence into a backup file before deleting. Before deleting, the period for claims is shown (Chapter 1, 8.4 “Retention and deletion”).
- When the cache exceeds 1 GB (initial value), reduced images of old work sessions are deleted first.

### 6.2 Mainland China

- Obtaining (updates, reference information): obtained from GitHub, and if that fails, from the Hong Kong mirror (DD-8-5“A Hong Kong mirror is set up for mainland China”). Distributables and reference information packages are verified by signatures, so replacement at the mirror can be detected. The update manifest (JSON) has no signature, so forcing an old version by rewriting the manifest is prevented by checking the manifest's version against the version in the update signature (Chapter 9, DD-9-2“Updates are fetched via Tauri's mechanism, checking the version in the manifest against the signature”).
- First download: the Chinese section of the README carries direct links to the latest distributables on the mirror and their SHA-256 (3.6, Chapter 9, 5 “Download Routes”). If the README itself cannot be reached, it cannot be obtained (Chapter 13, H-17“In mainland China the README may be unreachable”).
- Writing records is completed within the device, so no connection to GitHub is needed.
- Candidate mirror: Alibaba Cloud OSS in the Hong Kong region (static delivery). The Hong Kong region does not require an ICP filing (Alibaba Cloud's guide). There is guidance that latency from mainland China is 30 to 50 ms (same).
- Cost: determined by storage and outbound transfer. [Estimate required] Size of distributables (Chapter 11, 4.3 “Assets”) × number of downloads.
- [To be measured] Reachability and speed from mainland China to the mirror. Criterion: updates and reference information can be obtained from the mirror. If not met, it is declared as a gap (Chapter 13).

### 6.3 Estimate of mirror cost

- The cost is determined by storage and outbound transfer. Outbound transfer from the Hong Kong region is free up to 5 GB per month (Alibaba Cloud OSS guide). The unit price table is on Alibaba Cloud's pricing page, but in this document's research (2026-09-29) the screen could not be obtained. [Estimate required] NRSD checks the unit prices on the pricing page before contracting.
- Estimation formula: monthly outbound transfer = size of distributables (the expectation in Chapter 9, 2.1 “Distributables per OS”: around 200 MB per OS, with Linux larger by the bundled components; the design target is 300 MB or less) × number of Users updating or newly installing from the mirror per month + reference information (1 MB or less) × number of downloads. Storage = distributables of one version (Windows installer, macOS DMG and `.app.tar.gz` for updates, Linux AppImage: four items of about 200 MB each) × two versions (about 1.6 to 2 GB) + source packages of the LGPL components bundled in the Linux AppImage (for two versions; [Estimate required] size) + reference information.
- Example: if 200 mirror Users update per month, transfer is about 50 GB per month.

## 7. Dependence on GitHub

|Event|Preparation|
|---|---|
|GitHub outage|Updates and reference information are obtained from the mirror. Records are on the device, so work does not stop|
|Suspension of the account or repository|Move the Public Repository to the mirror and another git host, and switch the download source with an app update. The app holds several sources for updates (Chapter 9, DD-9-2“Updates are fetched via Tauri's mechanism, checking the version in the manifest against the signature”)|
|Terms|No personal information or repost information is placed in the Public Repository (Legal Research L-18“GitHub terms”)|
|Release capacity|A single release file must be under 2 GiB, and one release can have up to 1,000 files. There is no limit on the total size of releases or on bandwidth (GitHub's guide “About releases”). The design target for distributable size is 300 MB or less (Chapter 9, 2.1 “Distributables per OS”). The 500 MB in Chapter 1, 4.2 “Communication with the outside” is the download limit; distributables exceeding it are not downloaded|
|Account takeover|The protections of 3.5. Even if taken over, updates and reference information are not applied without NRSD's key signatures (3.2)|

## 8. Handling Failures

|Event|How it is reported|What the User does|
|---|---|---|
|Wrong passphrase for a backup|“The passphrase is wrong or the file is broken” (age does not distinguish the two)|Check the passphrase|
|The backup file is broken|Same. If found in the middle of extraction, what was imported is rolled back|Use another backup|
|Not enough free space at the backup destination|The estimate is shown before creation, and it does not start|Choose another destination|
|Record signature does not match|That record is not used, and a list is shown|Restore from a backup|
|Not enough free space on the device|The notice of 6.1|Put old evidence into a backup and then delete it|

## 9. Mapping to Requirements

|Requirement number|Requirement|Sections in this chapter|
|---|---|---|
|R-8-1-1|Holds the software, public page, and reference information, and no personal information|3 “Public Repository”|
|R-8-1-2|Reference information and updates of modified versions can be distinguished from official ones|3.2 “Distinguishing modified versions”|
|R-8-2-1|The arrangement of Rights Holder Information, ledger, evidence, and work data is determined|2.1 “Arrangement”|
|R-8-2-2|It is guaranteed that they do not leave the device (except when the User exports them)|2.2 “Not leaving the device”|
|R-8-4-1|Tampering can be detected, history remains, and recovery from exports is possible|4 “Integrity”, 5 “Backup Files”|
|R-8-5-1|A guide to device capacity is shown|6.1 “Guide to device capacity”|
|R-8-5-2|Distributables and reference information of the Public Repository can also be obtained from mainland China|6.2 “Mainland China”|
|R-8-7-1|There is preparation for when GitHub becomes unavailable|7 “Dependence on GitHub”|

- Four requirements on writing to the private repository and accepting Users (the former 8-3 and 8-6 of the Outline Design Document) were deleted in Design Plan Edition 2.

## 10. Gaps Declared in This Chapter

- A User who has not made a backup file loses records, evidence, and signing keys if the device is lost.
- If the passphrase of a backup file is weak, someone who steals the file may read the signing keys and records (Chapter 1, T-5“Stealing and reading a backup file”).
- If a device without full-disk encryption enabled is stolen, the records in the app's data location (evidence, case records, Rights Holder Information) may be read (DD-8-7“Encryption of records is left to full-disk encryption of the OS”; signing keys are in the OS keystore and are an exception).
- If distribution of reference information is stopped (frozen), the app keeps using the last package it has. This can be noticed through the expiry notice (3.4), but new contact points do not arrive.
- If the passphrase is forgotten, the backup file cannot be restored.
- When using two devices in parallel, merging is manual via backup files, and there is no automatic synchronization.
- If the CI or GitHub organization is taken over, releases, the README, and the public page of the Public Repository can be rewritten, and fake distributables with valid code signatures may be placed (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”).
- While the GitHub organization has only one owner, if that person loses all authentication and recovery methods, the organization cannot be recovered (Chapter 13, H-45“While the GitHub organization has one owner, if that person loses all authentication and recovery methods, the organization cannot be recovered”).

## 11. Corrections to Other Chapters and the Outline Design Document

- Chapter 2, 12 “Corrections to Other Chapters and the Outline Design Document” and Chapter 11, 4.1 “Rust components”: change the backup encryption to age (DD-8-3“Moving devices and preparing for failure use backup files (age format)”).
- Outline Design Document Chapter 8: the text of R-8-1-1“It holds the software, the public page, and reference information, and no personal information” to R-8-7-1“There is a provision for when GitHub becomes unusable” is not changed.
