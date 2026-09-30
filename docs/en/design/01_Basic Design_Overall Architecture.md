# Basic Design Document Chapter 1: Overall Architecture

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [01_Basic Design_Overall Architecture.docx](01_Basic%20Design_Overall%20Architecture.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

## Position of This Document

- Details Chapter 1 of the Outline Design Document. This chapter is the premise for all chapters; the other chapters follow this chapter's structure, data locations, identifiers, trust boundaries, and cross-cutting policies.
- The sections of this chapter follow the 12 sections of arc42, a public template for describing software architecture (goals, constraints, context and scope, solution strategy, building blocks, runtime scenarios, deployment, cross-cutting concepts, decisions, quality requirements, risks, glossary). This chapter is based on the study memo “Chapter 1 Overall Architecture” (NRSD's internal record of study, not distributed with this document; the facts underlying the design are written with their sources in each section of this document).
- The decisions received are as in the following table.

|Number|Type|Content|
|---|---|---|
|D-2-1|Decision|What the System protects shall be the rights (copyright and portrait rights) that the Rights Holders have in their own photographs|
|D-2-2|Decision|The scope of the System shall be the application of licenses|
|D-2-3|Decision|The greatest value of the System shall be deterrence|
|D-2-4|Decision|Automated detection shall be a separate design plan|
|D-2-5|Decision|Sales management is not handled. Anyone who needs it shall fork and modify the System, and NRSD takes no part in the results|
|D-2-6|Decision|Representation in legal proceedings and legal judgment are not performed|
|D-3-2|Decision|Viewers and the Operator are not defined within the System|
|D-3-3|Decision|Users shall be able to use the System without looking at GitHub|
|D-5-1|Decision|The System does not confirm Users and collects no User information. No registration is required in order to use the System|
|D-5-5|Decision|The System provides the means of matching but does not perform matching on anyone’s behalf|
|D-6-1|Decision|The Client App shall be a desktop application running on Windows, macOS, and Linux|
|D-6-2|Decision|The System shall not be built as a website|
|D-12-1|Decision|The Public Repository is provided. No Private Repository is provided|
|D-12-3|Decision|Rights Holder Information, the signing key, work data, the ledger, and evidence are kept on the User’s device; NRSD does not collect them|
|D-13-1|Decision|Phase 2 shall be a separate design plan|
|D-13-3|Decision|Registration of contact details for notifying authors is dealt with in the Phase 2 design plan|
|A-6|Omission found in the item breakdown|Preparation against fake apps|
|A-9|Omission found in the item breakdown|Costs, and GitHub capacity limits|
|A-10|Omission found in the item breakdown|Mapping from Design Plan decisions to the design|
|A-14|Omission found in the item breakdown|NRSD's position and the Terms of Use|
|A-17|Omission found in the item breakdown|Cross-border data transfer (the premise disappeared with Edition 2, in which NRSD receives no User information)|

- Notation: “[To be measured]” marks matters confirmed by sending/receiving or by implementation (the assumption and the criterion are written with it). “(initial value)” marks a value set at the start of operation where there is no external figure to base it on.
- Terms are collected in 16 “Glossary”.

## 1. List of Design Decisions

|Number|Decision|Reason|Rejected alternatives|
|---|---|---|---|
|DD-1-1|The System consists of the Client App (including data on the User's device) and the Public Repository (distributions, reference information, public page). NRSD holds keys that sign distributions and reference information|Design Plan D-12-1“The Public Repository is provided. No Private Repository is provided”, D-12-3“Rights Holder Information, the signing key, work data, the ledger, and evidence are kept on the User’s device; NRSD does not collect them”|Keeping records in a private repository (Design Plan Edition 1; this would mean collecting User information)|
|DD-1-2|NRSD runs no always-on servers and has no route for receiving User information (except feedback e-mails Users send themselves; Chapter 10, DD-10-6“Feedback is sent by the User from their own e-mail”, Chapter 12, 5 “Privacy Policy”)|Design Plan D-6-2“The System shall not be built as a website”, D-5-1“The System does not confirm Users and collects no User information. No registration is required in order to use the System”|Placing a server for signing and Registration|
|DD-1-3|Certificates for C2PA signatures are created within the Client App from the information the User enters. No certificate authority is used|Design Plan D-5-2“The certificate for C2PA signatures is created inside the Client App from information entered by the User. No certificate authority is required”. The C2PA technical specification requires no qualification of the certificate issuer|NRSD acting as a certificate authority (a design from the Design Plan Edition 1 stage, which mistook a candidate in the plan for a requirement)|
|DD-1-4|All records (work data, case records, evidence, templates, work sessions, authorizations) are kept on the User's device. Moving devices and preparing for failure are done with backup files the User creates|Design Plan D-10-7“Evidence is stored on the User’s device and not published. The System does not collect evidence”, D-12-3“Rights Holder Information, the signing key, work data, the ledger, and evidence are kept on the User’s device; NRSD does not collect them”|Automatic synchronization (requires an external storage location)|
|DD-1-5|Records requiring integrity (work data, case records, authorizations) carry record signatures made with the User's key, and are verified when read|Detects rewriting of records from outside the device (another person touching the device, corrupted files, crafted backup files). Because the record signature is made with the User's own key, the User can rewrite a record and sign it again. What shows third parties (experts, providers) that a record existed at a given time is the trusted timestamp from an external TSA (Chapter 3, 4 “Trusted Timestamps”, Chapter 6, 3.2 “Trusted timestamps”). Work data created offline has no such timestamp until one is added later|Storing without signatures|
|DD-1-6|Reference information (contact points, example texts, references to each country's laws, wording of Enclosed Documents) carries NRSD's record signature and is placed in the Public Repository and the mirror; the app fetches and verifies it|Delivers changes in contact points and laws without waiting for an app update. A route to mainland China (Design Plan I-04“Access to GitHub from mainland China”)|Only embedding it in the app (updates would wait for app updates)|
|DD-1-7|The app can perform C2PA signing, editing, export, and recording of Registrations offline. Timestamps, fetching reference information and updates, and fetching repost pages happen when connected|Work after shooting does not depend on the network|Assuming a permanent connection|
|DD-1-8|The screen (WebView) handles only display and input; text rendering, image processing, file I/O, networking, and key operations all happen in the Rust core. Only permitted commands can be called from the screen|Tauri's security model places a trust boundary between the screen and the core (Tauri's official security guidance). Risks from displaying external text on screen do not reach the core. Rendering in the core gives the same result on all three OSes (Chapter 4, DD-4-5“Text shaping and rendering use the same Rust mechanism for preview and export”)|Processing images on the screen (results differ by OS; weaknesses of the screen reach the records)|
|DD-1-9|All records are saved by writing to a temporary file, flushing it to storage, and then swapping the name|No half-written, broken records remain (quality goal 1)|Overwriting in place|
|DD-1-10|Data is kept in the OS's per-user, non-synchronized location (on Windows, Local AppData). Things that can be regenerated (thumbnails, etc.) go in the OS cache location|Windows Roaming AppData is synchronized on organizational PCs, and several GB of evidence would make it heavy|Placing data in Roaming AppData|
|DD-1-11|Only one instance of the app runs per machine. A second launch brings the first window to the front and exits|Prevents two apps from writing the same record at the same time|Allowing multiple instances with a lock per record (the work-session lock in Chapter 4, 11.7 “Opening at the same time” remains as a supplement to single-instance)|

## 2. Purpose and Quality Goals

### 2.1 Purpose

- The System puts the Rights Holders of cosplay photographs in a position to show their rights in the photographs (C2PA signature, Visible Signature, Enclosed Document), set the Permitted Scope, register reposts and keep evidence, and act when appropriate (Design Plan Chapter 2 “Purpose and Scope”).

### 2.2 Stakeholders

|Party|Relation to the System|Concerns|
|---|---|---|
|Cosplayer (User)|C2PA-signs photographs, sells, posts, and deals with reposts|Little effort, good looks, photographs are not damaged|
|Photographer (User)|Same as above. May hold joint rights with the cosplayer|Credits appear correctly|
|Authorized person (User)|C2PA-signs and posts with the Rights Holder's authorization|The scope of authorization is clear|
|Purchaser (external)|Receives Delivery Images and the Enclosed Document|Knows what is permitted|
|Viewer (external)|Sees posts and tells whether a photograph is genuine|Knows how to check|
|Reposter (counterpart)|Reposts without permission|— (the target of deterrence)|
|Person posing as the Rights Holder (counterpart)|C2PA-signs someone else's photographs first, etc.|— (handled with matching clues; Chapter 2)|
|NRSD (operator)|Builds and distributes the app. Maintains reference information|Protecting keys, keeping costs down, scope of responsibility|
|Legal experts (external)|Receive evidence packages. Check wording|Procedures for verifying evidence|
|Posting sites, Cloud, complaint contact points (external organizations)|Where images are placed, receivers of complaints|—|

### 2.3 Quality goals (in order of priority)

|Order|Goal|ISO/IEC 25010:2023 characteristic|Reason|
|---|---|---|---|
|1|Do not lose or damage Originals and records|Reliability, safety|Photographs are Users' source of income, and evidence is needed for later claims. Once lost, they cannot be recovered|
|2|Non-technical people can use it without getting lost|Usability|If it is not used, deterrence does not work (Design Plan 8.1 “Appearance”)|
|3|Do not collect or leak User information|Security|Design Plan D-5-1“The System does not confirm Users and collects no User information. No registration is required in order to use the System” is the premise of the System|
|4|Output passes third-party verification, and tampering can be detected|Functional suitability|Otherwise the C2PA signature loses its meaning|
|5|Same results on the three OSes|Compatibility, flexibility|Output does not change when Users change devices|

- Performance is met within the range that does not harm the five goals above (14.1).
- When goals conflict, the higher one takes precedence. Example: even if automatic saving (goal 1) increases writes, it takes precedence over speed.

## 3. Constraints

|Type|Constraint|Origin|
|---|---|---|
|Technical|Desktop app for Windows, macOS, and Linux|Design Plan D-6-1“The Client App shall be a desktop application running on Windows, macOS, and Linux”|
|Technical|Not built as a website. No always-on servers|D-6-2“The System shall not be built as a website”, DD-1-2“NRSD has no servers and receives no User information (except feedback e-mails Users send themselves)”|
|Technical|Does not collect User information. Requires no registration|D-5-1“The System does not confirm Users and collects no User information. No registration is required in order to use the System”|
|Technical|Main work can be done offline|DD-1-7“C2PA signing, editing, export, and Registration work offline”|
|Technical|Bundled assets (fonts, models) are under licenses that permit redistribution|Chapter 4, 6 “Fonts and Emoji”, Chapter 11|
|Technical|Supported machines: Windows 10/11 and Linux (glibc 2.35 or later) on CPUs supporting x86-64-v3; macOS 14 or later on Apple CPUs. Intel Macs are not supported|Chapter 11, 3.1 “Minimum supported OS versions” (due to ONNX Runtime support)|
|Organizational|NRSD holds no certificate authority license|NRSD's decision (2026-09-29)|
|Organizational|NRSD is not a lawyer. It makes no legal judgments|Design Plan D-2-6“Representation in legal proceedings and legal judgment are not performed”|
|Organizational|NRSD's development and operations structure is small. As of 2026-09-30, development, design, and operations are all done by one Lead Developer, and there is no commissioned designer (Chapter 10, DD-10-2“Design is done and judged by NRSD's Lead Developer”). Whether to add people is NRSD's decision|—|

## 4. Context and Scope

### 4.1 Components

|Component|Holds|Does not hold|Where it runs|
|---|---|---|---|
|Client App|The User's signing key (OS keystore), certificates, Rights Holder Information (entered values), settings, templates, work sessions, work data, case records, evidence, authorizations, fetched reference information|Other Users' data|The User's PC (Windows, macOS, Linux)|
|Public Repository|Source, releases (distributions), README, public page, reference information (with record signatures)|Personal information, per-User information|GitHub (public)|
|Hong Kong mirror|Copies of the Public Repository's releases and reference information|Personal information|Object storage (static delivery only)|
|Operator tool|Editing reference information, signing distributions|User information|M-01“Reference Information” runs on NRSD's management PC (network-connected; the reference-information signing key is on a hardware token). M-02“Release Signing” runs on the operator's offline PC (update signing key)|
|NRSD's keys|Update signing key, reference-information signing key|User information|The operator's offline PC (update signing key), a hardware token (reference-information signing key)|

![Figure 1-1 Components and connections](fig/d01_構成.png)

Figure 1-1 Components and connections

- Outside the boundary: Phase 2 (automated detection, including registration of contact details for notifying authors; Design Plan D-13-3“Registration of contact details for notifying authors is dealt with in the Phase 2 design plan”), sales management, rights in source works (Outline Design Document “What the System protects and does not protect”), legal judgment and representation (Chapter 7 “Legal Action Guidance”), collection of User information, and matching or judging on others' behalf (Design Plan D-5-5“The System provides the means of matching but does not perform matching on anyone’s behalf”).

### 4.2 Communication with the outside

- The app itself communicates outside the device only with the parties in the following table. The app sends nothing not in the table (quality goal 3). Transmissions by the OS and screen components—such as mandatory diagnostic data that WebView2 (Windows) sends to Microsoft, and SmartScreen and Gatekeeper checks by the OS—follow each company's policy (Microsoft “WebView2 data and privacy”). The Privacy Policy states this (Chapter 12, 5.2 “List of what is sent outside the device”).

|Party|Method|What is sent|What is received|Timeouts and limits (initial values)|On failure|
|---|---|---|---|---|---|
|Public Repository (GitHub)|HTTPS GET|Fetch requests only|Update manifest (JSON), distributions, reference information and record signatures|Connect 10 s. Manifest and reference information: 120 s total, up to 1 MB. Distributions: 30 min total (initial value, so that they can be obtained even over slow lines from mainland China via the mirror), up to 500 MB|Try the mirror. If both fail, try at the next start|
|Hong Kong mirror|HTTPS GET|Same as above|Same as above|Same as above|—|
|Timestamp authority (TSA)|HTTP or HTTPS POST (RFC 3161)|Hash only (timestamp request)|Timestamp response|Connect 10 s, total 30 s, response up to 1 MB|Try the next TSA (Chapter 3, 4 “Trusted Timestamps”). If all fail, hold and attach when connected|
|RDAP|HTTPS GET|Domain name or IP address being looked up|Registration information (JSON)|Connect 10 s, total 30 s, response up to 1 MB|Record as no result, and show how to look it up by hand|
|DNS|OS name resolution|Domain name being looked up|IP address|Per OS settings|Same as above|
|Repost page|HTTP or HTTPS GET (only URLs the User registered; only at Registration (default; the User can turn it off for that Registration) and when “Check the current status” is pressed)|Fetch request (the User's IP address is visible to the other party. The System does not identify itself, and headers match the WebView's values. Chapter 6, DD-6-8“Fetch requests do not identify the System”)|The page's HTML and images|Connect 10 s, total 60 s, up to 50 MB per page, up to 5 redirects|Record that it could not be fetched, and supplement with the User's screen images (Chapter 6)|
|E-mail software|Pass a mailto URL to the OS|Feedback text (after the User checks it)|—|—|Allow copying the text and address (Chapter 10, 9 “Feedback Channel”)|
|OS keystore|OS API|Storing and retrieving signing keys|—|—|Chapter 2, 7 “Signing Keys”|

- The System does not communicate directly with posting sites, Cloud, or complaint contact points. Contact points are opened in the OS default browser when the User presses a button.
- Fetching a repost page shows the User's IP address to the other site. This is shown once before fetching (threat in 11.2).

## 5. Solution Strategy

|Quality goal|Strategy|This chapter / other chapters|
|---|---|---|
|1 Do not lose or damage|Originals are read only. Records are swapped in from temporary files. Work sessions are saved automatically. Backup files (encrypted) are recommended|DD-1-9“Records are written fully to a temporary file and then swapped in”, Chapter 4, 11 “Work Sessions (the State in the Middle of Editing)”, Chapter 8, 5 “Backup Files”|
|2 Easy to use without getting lost|One window, step indicators, a guide character, an error pattern. GitHub is not shown|Chapter 10|
|3 Do not collect or leak|No servers. Outgoing transmissions limited to the table in 4.2. Records kept only on the device. No secrets written to logs|DD-1-2“NRSD has no servers and receives no User information (except feedback e-mails Users send themselves)”, 10.4|
|4 Pass verification|Components following the C2PA specification (c2pa-rs), timestamps, record signatures|Chapter 3, DD-1-5“Device records carry record signatures and are verified”|
|5 Same results|Rendering, color conversion, and export use the same Rust components. OS fonts are not used|DD-1-8“The screen handles only display and input; processing happens in the Rust core”, Chapter 4|

## 6. Contents of the Components

### 6.1 App layers

|Layer|Parts|Responsibility|Not responsible for|
|---|---|---|---|
|Screen|WebView (HTML, CSS, TypeScript)|Display and input. Progress display|Text rendering, image processing, file, network, and key operations|
|Command|Tauri commands|Receives requests from the screen, validates input (10.6), passes them to business components. Long tasks send progress notifications to the screen|Business decisions|
|Business|Signing information, export (signing), editing (rendering), work sessions, Rights Documents, Registration and evidence, matching, reference information, updates, backups, settings|The functions of each chapter|How records are written, network details|
|Foundation|Storage, keystore, networking, image I/O, model execution, timestamps, logs|Cross-cutting functions (10)|Business decisions|

- The mapping to parts (crates) is in Chapter 11, 7.1 “How the Rust components are divided”.

### 6.2 Between the screen and the core

- The screen can call only permitted commands (allowed per window with Tauri capabilities and permissions). File access is limited to the app's data location and files and folders the User chooses each time (Tauri scopes).
- The screen is barred from loading external resources (CSP loads nothing but the app's own resources). External URLs open in the OS default browser when the User presses them.
- Text from outside (repost page titles, file names, template names, reference information text) is displayed only as text and never interpreted as HTML.
- Long tasks (export, Registration, auto-placement, backups) run in background tasks in the core and send progress notifications (which item, estimated remaining time) to the screen. Cancellation from the screen is accepted.

## 7. Runtime Scenarios

- The normal procedure of each scenario and the handling on failure are shown. Details are in each chapter; this chapter shows the order across chapters.

![Figure 1-2 Data flows](fig/d02_データの流れ.png)

Figure 1-2 Data flows

### 7.1 First run

|Order|Processing|On failure|
|---|---|---|
|1|Choose the language from the OS language (Japanese, Chinese, English; others fall back to English)|—|
|2|Display of and consent to the Terms of Use and Privacy Policy (Chapter 10, G-02“Consent”). The Terms apply to NRSD's service (provision of reference information)|If the User does not consent, reference information is not fetched (only the bundled package is used). C2PA signing, editing, export, Registration, verification, viewing and extracting records, and updates remain available (Chapter 12, DD-12-3“Consent gates only the fetching of reference information”)|
|3|Entering signing information, creating the personal root and the signing certificate, and storing keys (Chapter 2, 2 “Input of Information for C2PA Signatures”)|If the keystore cannot be written, show the reason and do not proceed (signing is impossible without a key)|
|4|Showing the notice code and example texts (G-04“Posting the Notice”)|—|
|5|Recommending creating a backup file (G-05“Create Backup”)|Proceed without creating one|

### 7.2 Batch export

|Order|Processing|On failure|
|---|---|---|
|1|Choose photos and purpose. Create a work session (Chapter 4, 11 “Work Sessions (the State in the Middle of Editing)”)|Mark unreadable photos and set them aside|
|2|Thumbnails and auto-placement (Chapter 4, 7.2 “Auto-placement”)|—|
|3|Adjustment (Chapter 4, 9 “Editing Operations”). The work session is saved automatically|If saving fails, notify|
|4|Permitted Scope (delivery. Chapter 5)|—|
|5|Export: for each photo, write the output under a temporary name in the processing order (Chapter 3, 8 “Processing Order and Streams”), record the work data, and fix the output name (by this order, no output without a record remains under its final name)|A failure of one photo is listed and the others continue. If stopped midway, what was exported is recorded and export can continue from there|
|6|Timestamps (when there is no connection, hold and attach when connected)|If no TSA responds, hold|

### 7.3 Registering a repost

|Order|Processing|On failure|
|---|---|---|
|1|Enter the URL, the reposted image, and screen images (Chapter 6)|—|
|2|Matching (watermark, hash)|If unreadable, record “cannot be matched”|
|3|Fetch the repost page (done by default at every Registration. The Registration screen shows that the IP address is visible to the other site, and the User can turn it off for that Registration. Chapter 6, 2.1 “Procedure”)|If it cannot be fetched, record that and supplement with screen images|
|4|RDAP and DNS lookups|Record as no result|
|5|Create the case record, record signature, and timestamp of the evidence package|If stopped midway, continue from the state of each step (8.2)|

### 7.4 Matching (when a person posing as the Rights Holder appears)

|Order|Processing|On failure|
|---|---|---|
|1|Put in the image (Chapter 10, G-13“Verify”)|—|
|2|Verify the C2PA signature, read the watermark, compute the matching hash|Show unreadable items as “none”|
|3|Lay out the clues (Chapter 2, 4.4 “Clues when a person posing as the Rights Holder appears”). No judgment|—|

### 7.5 Fetching reference information

|Order|Processing|On failure|
|---|---|---|
|1|At start and every 24 hours (initial value), check the version of the reference information on the public page (GitHub Pages: `https://<organization>.github.io/<repository>/reference/latest.json`). raw.githubusercontent.com is not used (there are reports of blocking and DNS poisoning in mainland China; Research Materials “GitHub and Distribution”). The order of sources (public page, then the Hong Kong mirror's `reference/`) moves a failed source to the back (as with updates; Chapter 9, 4.3 “Form of the update manifest (static JSON)”)|Try the mirror|
|2|If there is a new version, fetch it and verify the package (size, signature, version number greater than the local one, SHA-256 of each file, expiry) (Chapter 8, 3.4 “Reference information package”)|If verification fails, do not use it and keep using the local version. A local version past its expiry is kept in use with a notice|
|3|Replace the local version (by the procedure of DD-1-9“Records are written fully to a temporary file and then swapped in”)|—|

### 7.6 Updates

- Follow Chapter 9, 4 “Updates”. Verify the update signature; if it fails, do not apply.

### 7.7 Creating and restoring backups

- Follow Chapter 8, 5 “Backup Files”. The backup file is created under a temporary name and renamed once complete.

### 7.8 Starting after an abnormal exit

|Order|Processing|
|---|---|
|1|At start, check the “in use” record. If the previous session was not closed properly, do the following|
|2|Work sessions: confirm they open at the last saved version, and show “Continue from where you left off” (Chapter 4, 11.5 “After abnormal termination”)|
|3|Export: delete outputs left under temporary names and reconcile with the records of exported items|
|4|Registration: continue case records from the step last recorded|
|5|Verify record signatures and report those that do not match|

### 7.9 Offline export and later timestamps

- Export without a timestamp and record “awaiting timestamp” in the work data. When connected, obtain a timestamp for the output's SHA-256 and attach it to the work data (Chapter 3, DD-3-5“Timestamps are taken at C2PA signing, or later when offline”). Home shows the number waiting.

## 8. Data

### 8.1 List of data

|Data|Source|Storage location|Public|Writer|Integrity protection|
|---|---|---|---|---|---|
|Original|User (shooting, development)|User's PC (the System only reads it)|Non-public|—|—|
|Rights Holder Information (handle name, role, Notice accounts)|User input|Certificate, device|The range in the certificate is public together with C2PA-signed images|User|—|
|User's signing key|App|OS keystore|Secret|—|—|
|User's certificates|App|Device, manifest of output images|Included in the manifest|User|User's key|
|Output images (delivery)|App|From the User's PC to Cloud|Distributed to purchasers|User|C2PA signature|
|Output images (social media)|App|From the User's PC to posting sites|Public|User|C2PA signature (assumed not to survive on posting sites) and invisible watermark|
|Work data|App|Device|Non-public|User|Record signature|
|Case records (Registrations)|App|Device|Non-public|User|Record signature|
|Evidence (fetched items)|App, User|Device|Non-public|User|Hashes recorded in the case record, with a timestamp attached|
|Authorizations|The principal Rights Holder's app|Both parties' devices|Non-public (a summary appears in the manifest of the authorized person's output)|Principal Rights Holder|Record signature|
|Templates|User|Device|Non-public|User|—|
|Work sessions (editing in progress)|App (saved automatically)|Device|Non-public|User|— (photos are not copied; their location and SHA-256 are recorded)|
|Imported assets (fonts, images)|User|Device|Non-public|User|Referenced by SHA-256|
|Enclosed Documents|App (generated from the wording in the reference information)|Output folder|Purchasers|User|Hash recorded in the manifest (Chapter 5)|
|Reference information|NRSD|Public Repository, mirror, copy on the device|Public|NRSD|NRSD's record signature|
|Settings|User|Device|Non-public|User|—|
|Backup files|App|A location the User chooses (external storage, etc.)|Non-public|User|Encryption with a passphrase and tamper detection (Chapter 8)|
|Thumbnails|App|OS cache location|Non-public|App|— (can be regenerated)|
|Operation log|App|OS log location|Non-public|App|—|

### 8.2 Identifier scheme

|Identifier|Format|Issued by|Uniqueness|
|---|---|---|---|
|Identification number|A 61-bit random number. Placed as-is in the invisible watermark (the data part of TrustMark BCH_5). For display, the 61-bit value is placed in the upper bits and its 4-bit check (CRC-4 with generator polynomial x^4＋x＋1) in the lower bits, and the resulting 65 bits are converted 5 bits at a time from the top into 13 Crockford Base32 characters, split 5-5-3 (example: value 0x1604F408204C92C5, check 4 gives `P0KT0-G82CJ-B2M`). Each value has exactly one notation, and a notation whose 4-bit check does not match is invalid and shown as “the identification number may have been mistyped”|App (at export)|See the note below|
|Notice code|The first 100 bits of the SHA-256 of the public key of the User's personal root key (Chapter 2, 3.1 “Structure”) as 20 Crockford Base32 characters (Chapter 2, DD-2-7“Notice codes and identification numbers use Crockford Base32”), prefixed with `NRSD-` and split every 5 characters (28 characters)|App (at first input)|With 100 bits, accidental collisions can be ignored|
|Case number|`C` followed by a UUID v7|App|Collisions can be ignored|
|Work session, template, and batch numbers|UUID v7|App|Collisions can be ignored|
|Error number|Category (3 letters) and 3-digit sequence (example: `SIG-012`)|Design (10.3)|—|

- One identification number is issued per work (one exported file). If the same photograph is exported for X, for Instagram, and for delivery, each gets a different identification number, and that they are the same photograph is known from the Original's SHA-256 in the work data. The same identification number is recorded in the invisible watermark, the C2PA manifest, the file name (if the User chooses to append it), the Visible Signature (if the placeholder is used), the work data, and case records. It is the “identification number” of Design Plan 1.5 “Definitions”, and the Draft Project Proposal's “matching number” (a number attached to posts so viewers can identify unauthorized reposts; Sections 2 and 3) and “work number” (a number that links reported reposts to works; Section 6) are the same value.
- Uniqueness: the probability that two 61-bit random numbers collide is about 1 in 50,000 even with 10 million works (n²/2⁶²: 10¹⁴ ÷ 4.6×10¹⁸). At issuance, it is checked against the local work data. Collisions with other Users are not checked because there is no central ledger. In matching, works are distinguished by the pair of identification number and signer.
- Shortening the display to some upper bits would break uniqueness (with 40 bits, 500,000 works collide with about a 10% probability). Because viewers use it as the matching number, it is not shortened.
- A work registration number (the number of a public registration) is not issued by the System; it is treated as a string the User writes in the Notice or the Visible Signature.

### 8.3 Formats and versions

- Records are JSON (UTF-8, keys in English letters). Each record has a `schema` (for example, work data `nrsd.work/1`, work sessions `nrsd.session/1`).
- When a version is raised, the app keeps code that reads the old version and migrates records to the new version at the first start after the app update. A copy of records before migration is kept by the procedure of Chapter 9, 4.4 “Record format migration”.
- When the app reads a record of a newer version than itself, it only reads it without rewriting, and prompts an update.
- Record signatures are made over the JSON normalized with JCS (RFC 8785) with ECDSA P-256 using the signing certificate's key, and placed in a separate file (`.sig`). The `.sig` carries the signing certificate and the personal root certificate (Chapter 2, DD-2-10“Record signatures use the signing certificate's key with the certificate chain attached”). However, authorizations, revocations, and joint-rights documents are exchanged as a single file, so their signatures are placed in `signatures` within the JSON (Chapter 2, 5.3 “Document format”).

### 8.4 Retention and deletion

- All records are on the User's device and are kept unless the User deletes them. The app deletes only the following automatically: operation logs (90 days; initial value), thumbnails (when a work session is deleted, and the oldest when the cache exceeds 1 GB; Chapter 8, 6.1 “Guide to device capacity”), versions of work session backups beyond 5 (Chapter 4, 11.4 “Version backups”), temporary files left after an abnormal exit (7.8), and copies from record format migration (30 days; Chapter 9, 4.4 “Record format migration”). The User's records (work data, case records, evidence, templates, authorizations) are never deleted automatically.
- Before deleting case records or evidence, the limitation period for damages claims (in Japan and China, 3 years from knowledge and 20 years from the act; in the US, 3 years from accrual; Legal Research L-20“Limitation periods for damages claims (Japan, China, US)”) is shown and exporting is recommended.
- NRSD holds no User data, so no retention or deletion policy arises on NRSD's side.

### 8.5 List of secrets

|Secret|Location|If leaked|
|---|---|---|
|User's signing key|OS keystore (Windows Credential Manager/DPAPI, macOS Keychain, Linux Secret Service. On Linux without Secret Service, a key file encrypted with a passphrase; Chapter 2, 7.5 “Storage method per OS”). Inside backup files, encrypted with the User's passphrase|The User remakes the key and re-posts the notice code (Chapter 2, 7 “Signing Keys”)|
|Backup file passphrase|The User's memory (the app does not store it)|Anyone holding the backup file may read it. Create a new backup and delete the old one|
|Reference-information signing key|Hardware token (NRSD's management PC)|Fake reference information may be distributed. A new public key is distributed by an app update (Chapter 9)|
|App update signing key|Tauri's signing key file (encrypted with a passphrase) on the operator's offline PC. A paper backup is kept in a safe (Chapter 9, DD-9-4“The update signing key stays outside CI; code-signing credentials are CI secrets”)|Fake updates may be distributed. If lost, updates cannot be distributed (Chapter 9)|
|Code-signing credentials (Windows)|Artifact Signing signing permission (credentials holding the Azure Artifact Signing signer role; CI secrets limited to the release environment; Chapter 11, 8.3 “Handling of secrets”). The certificate itself stays within the service and never reaches NRSD|Correctly code-signed fake builds could be produced. Revoke the credentials and report to Artifact Signing (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”)|
|Code-signing credentials (macOS)|The Developer ID Application certificate and private key (.p12 and its passphrase), and the App Store Connect API key used for notarization. CI secrets (limited to the release environment). The originals are on the operator's management PC|Correctly signed and notarized fake builds could be produced. Ask Apple to revoke the certificate (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”)|
|Hong Kong mirror write key|An Alibaba Cloud RAM user AccessKey (with write permission to this bucket only). On the operator's management PC (Chapter 9, 6.1 “Procedure” step 6; the operator does the copying)|The mirror's content could be rewritten. Distributions and reference information are verified by signatures and would not be applied, while rewriting of the update manifest is prevented by version checking (Chapter 9, DD-9-2“Updates are fetched via Tauri's mechanism, checking the version in the manifest against the signature”). Revoke and recreate the key|
|GitHub organization owner account|Passkeys or security keys (2 or more), time-based one-time codes, paper recovery codes (Chapter 8, 3.5 “Protection of the GitHub account and repository”)|The Public Repository, releases, and public page (including the SHA-256 in the README) could be rewritten (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”). Report to GitHub and recover the account|

### 8.6 Multilingual file names and characters

- File names are handled in UTF-8 (normalized to NFC). Characters the OS does not allow (on Windows `\ / : * ? " < > |`, etc.) are replaced with `_`.
- Files and folders the app creates within its data location are named with alphanumerics (numbers), and names the User entered are kept inside the JSON. Export output names (using the shoot name and original file name) follow Chapter 3, 10.1 “Names and structure”.

## 9. Deployment

### 9.1 Locations per OS

|What is placed|Location (Tauri name)|Windows|macOS|Linux|
|---|---|---|---|---|
|Records, work sessions, evidence, templates, imported assets, settings, fetched reference information|app_local_data_dir|The app's folder under Local AppData|Under ~/Library/Application Support|Under $XDG_DATA_HOME (or ~/.local/share)|
|Thumbnails and other regenerable items|app_cache_dir|Under Local AppData|Under ~/Library/Caches|Under $XDG_CACHE_HOME (or ~/.cache)|
|Operation log|app_log_dir|logs under Local AppData|Under ~/Library/Logs|logs under local_data_dir|
|Signing key|OS keystore|Credential Manager/DPAPI|Keychain|Secret Service (if absent, a key file encrypted with a passphrase under app_local_data_dir; Chapter 2, 7.5 “Storage method per OS”)|

- Source: Tauri's PathResolver documentation.
- Roaming AppData (the Windows location of app_data_dir) is not used (DD-1-10“Data is kept in a per-user location that is not synchronized”).
- The layout within these locations is set in Chapter 8, 2.1 “Arrangement”.

### 9.2 Installation

- One distribution per OS (Chapter 9, DD-9-1“One distribution per OS, code-signed”). It is a per-user installation that does not require administrator rights.

### 9.3 Single instance

- Only one instance runs per machine (DD-1-11“Only one instance of the app runs per machine”). Tauri's single-instance plugin (tauri-plugin-single-instance; supports Windows, macOS, and Linux, using DBus on Linux; a second launch notifies the first and exits; the plugin must be registered first) is used. Source: Tauri's official guide (v2.tauri.app/plugin/single-instance).
- If photos or folders are passed to the second launch (such as by dropping files on the window), the first instance receives them.

## 10. Cross-cutting Concepts

### 10.1 Data model

|Entity|Holds|Relations|
|---|---|---|
|Signing information|Handle name, role, Notice accounts, certificates, notice code|Signer of works, authorizations, and cases|
|Template|Groups, layers, styles, assets|Work sessions hold a copy|
|Work session|List of photos, adjustments, export presets, history|Works are produced from a work session|
|Work (work data)|Identification number, SHA-256 of the Original, SHA-256 of the output, PDQ, timestamp, Permitted Scope, hash of the Enclosed Document|Referenced from cases|
|Case (case record)|URL of the repost, status history, evidence package|References works (matching results)|
|Evidence|Fetched items, screen images, hash list, timestamp|Belongs to a case|
|Authorization|Principal Rights Holder, authorized person, scope, period|Basis of the rights of the work's signer|
|Reference information|Contact points, texts, references to each country's laws, wording of Enclosed Documents, version|Used by Rights Documents and guidance|

### 10.2 Storage

- All records are written by the procedure of DD-1-9“Records are written fully to a temporary file and then swapped in”: write to a temporary file, flush to storage (`File::sync_all`), replace the name (on Windows `MoveFileExW` with replace-existing and write-through flags; on macOS and Linux `rename`), and on macOS and Linux also flush the folder. Sources and reasons are in Chapter 4, 11.3 “How saving works”.
- Records requiring integrity (work data, case records, authorizations) carry record signatures (DD-1-5“Device records carry record signatures and are verified”). They are verified when read, and if they do not match, they are not used and the User is notified.
- Two processes never write the same record at the same time. Each record has one writer process, and other processes ask the writer.

### 10.3 Errors

- An error has a category and number (8.2), a message for the User, and a message for the log.

|Category|Content|
|---|---|
|`IMG`|Image reading and writing|
|`SIG`|C2PA signatures, certificates, keys|
|`TSA`|Timestamps|
|`NET`|Networking|
|`STO`|Storage, capacity|
|`REF`|Reference information|
|`UPD`|Updates|
|`CAS`|Registration, evidence|
|`BAK`|Backup files|
|`INP`|Input validation|

- Display to the User follows the pattern of Chapter 10, DD-10-8“Error displays follow a uniform pattern” (what happened, whether Originals and records are safe, what to do next, number).
- The list of numbers (number, category, cause, message for the User, what to do next) is made during implementation and kept in the same three languages as the screen message files (Chapter 11, DD-11-8“Internationalization uses message files”).

### 10.4 Operation log

- Purpose: clues to causes that Users can attach themselves when giving feedback (Chapter 10, 9 “Feedback Channel”). The app never sends it automatically.
- Written: time (UTC), app version, type of operation, error number, counts and durations.
- Not written: signing keys, passphrases, contents of evidence, contents of repost pages, full paths of photos (file names only), Rights Holder Information, query parts of URLs.
- Size: rotate at 10 MB per file, up to 50 MB in total (initial values). Kept for 90 days (initial value).
- Users can open the log location from settings (to attach it to feedback).

### 10.5 Concurrency

- Long tasks (export, auto-placement, Registration, backups) run in background tasks in the core without freezing the screen. They report progress and can be canceled.
- At most one long task of each kind runs at a time (two exports do not run together). Different kinds can run in parallel (Registration during export).
- The parallelism of image processing follows the formula in Chapter 4, 14.4 “Large images and parallel processing”.

### 10.6 Validation of inputs from outside

|Input|Limits and checks (initial values)|Chapter|
|---|---|---|
|Photos|Up to 100 megapixels and 200 MB per file. Format determined by content (not extension)|Chapter 3, Chapter 4|
|Images for image layers|20 MB, up to 8,000 pixels on each side. PNG and JPEG only|Chapter 4, 5 “Image Layers”|
|Imported fonts|Up to 50 MB. TTF and OTF only|Chapter 4, 6.4 “Imported fonts”|
|Template files|Up to 50 files in the ZIP, 100 MB total after extraction, names without `..`|Chapter 4, 8.3 “Passing on (export and import)”|
|Backup files|Tampering detected by the passphrase-based authenticated encryption. The extracted size must match the manifest inside|Chapter 8, 5 “Backup Files”|
|Reference information|Up to 1 MB. Record signature verified|7.5|
|Updates|Update signature verified|Chapter 9|
|Repost pages|Up to 50 MB, up to 5 redirects, timeout 60 s. Not rendered; scripts not run|Chapter 6, DD-6-1“Repost pages are fetched without running scripts”|
|TSA responses|Up to 1 MB. Certificate chain verified|Chapter 3|
|RDAP responses|Up to 1 MB|Chapter 6|

- The pixel count of an image is checked from the width and height in the file header before memory is allocated (a precaution against decompression bombs; the width and height limits of Rust's image are strictly enforced).

### 10.7 Time

- The device clock is not trusted. Time as evidence is shown with RFC 3161 timestamps.
- Records store UTC in ISO 8601; display converts to the User's local time with the offset (for example, UTC+9).

### 10.8 Languages and countries

- The “screen language” and the “target country of Rights Documents” are separate settings (R-1-7-2“The screen language and the country of Rights Documents are handled separately”).
- Screen text is placed in message files per language (Chapter 11, DD-11-8“Internationalization uses message files”), never written directly in the screen.

### 10.9 Version compatibility

- The app version, the record `schema` version, the C2PA specification version (Chapter 3, DD-3-7“The C2PA specification version follows the c2pa-rs version”), and the reference information version are kept separately.

## 11. Trust and Threats

### 11.1 Trust boundaries

|Boundary|What crosses it|Verification|
|---|---|---|
|Between the screen and the core|Commands and results|Only permitted commands. Inputs validated (DD-1-8“The screen handles only display and input; processing happens in the Rust core”, 10.6)|
|Between the app and the Public Repository or mirror|Updates, reference information|Updates by update signature, reference information by NRSD's record signature. Not used on failure|
|Between the app and the TSA|Hashes, timestamp responses|The TSA's certificate chain is verified. Only hashes are sent|
|Between the app and repost pages|Fetched items|Not trusted. Scripts are not run and nothing is rendered (Chapter 6)|
|Between the app and files on the device|Records, evidence|Record signatures and hashes verified when read (DD-1-5“Device records carry record signatures and are verified”)|
|Between the app and other Users|Authorizations, template files|Authorizations: the principal Rights Holder's record signature is verified. Templates: format and size are checked|

![Figure 1-3 Trust boundaries](fig/d03_信頼の境界.png)

Figure 1-3 Trust boundaries

### 11.2 List of threats

- For each element (User, screen, app core, device records, OS keystore, backup files, Public Repository and mirror, TSA, repost pages, NRSD's operator tool and keys), the six STRIDE categories (spoofing, tampering, repudiation, information disclosure, denial of service, elevation of privilege; Microsoft's threat classification) were applied to identify threats.

|Number|Category|Threat|Attacker|Target asset|Countermeasure|Remaining gap|
|---|---|---|---|---|---|---|
|T-1|Spoofing|Posing as the Rights Holder to C2PA-sign first or replace the C2PA signature|A person who obtained someone else's photographs|Entitlement|Matching clues (Chapter 2)|Cannot be prevented (Chapter 13)|
|T-2|Spoofing|Treating the true Rights Holder's post as a repost and complaining to the provider|Same as above|The true Rights Holder's post|Clues and possible actions (Chapter 2, Chapter 7)|Complaints outside the System cannot be prevented|
|T-3|Spoofing|Distributing a fake app|Third parties|Signing keys, records|Code signing, official download route (Chapter 9)|Users who obtain it from somewhere other than the README|
|T-4|Spoofing|Stealing a User's signing key to C2PA-sign|A person who compromised the device|Identity|OS keystore, remaking the key (Chapter 2)|When the device is fully compromised|
|T-5|Information disclosure|Stealing and reading a backup file|A person who obtained the file|Signing keys, records|Encryption with a passphrase (Chapter 8)|Weak passphrases|
|T-6|Tampering|Rewriting records on the device|A person with access to the device|Case records, evidence|Record signatures (DD-1-5“Device records carry record signatures and are verified”)|When the signing key is stolen as well|
|T-7|Spoofing, tampering|Distributing fake reference information or updates|A person who took over the Public Repository|All Users|Signature verification (Chapter 8, 3.4 “Reference information package”, Chapter 9, 4 “Updates”)|Leak of the signing key itself|
|T-8|Elevation of privilege|Stealing NRSD's keys|Insiders, outsiders|All Users|The update signing key on an offline PC and the reference-information signing key on a hardware token, both outside build automation (Chapter 9, DD-9-4“The update signing key stays outside CI; code-signing credentials are CI secrets”)|When the keys themselves are stolen (Chapter 13, H-22“If NRSD's keys (update, reference information) are stolen, fake updates or reference information may be distributed”)|
|T-9|Elevation of privilege|Touching malicious content when fetching repost pages|Malicious sites|The User's device|Fetching without running scripts (Chapter 6)|Viewing in the User's own browser|
|T-10|Spoofing|Fake timestamp responses|A party in the middle of communication|Proof of time|Verification of the TSA's certificate chain (Chapter 3)|—|
|T-11|Tampering|Tampering with imported templates or backup files|The party handing over the file|App core, records|Format and size checks (10.6). Backups detect tampering by authenticated encryption|—|
|T-12|Repudiation|NRSD later denying the content of reference information it distributed|NRSD|The basis of the User's complaint|The version and record signature are kept on the User's device (7.5)|—|
|T-13|Information disclosure|Fetching repost pages shows the User's IP address to the other site|The operator of the repost site|The User's location|Fetched by default at Registration and shown before registering. The User can turn it off for that Registration (4.2, Chapter 6, 2.1 “Procedure”)|Visible when fetched (Chapter 13, H-26“Taking a record of the repost page (the Registration default) shows the User's IP address to the other site”)|
|T-14|Information disclosure|Keys, evidence, or personal information appearing in the operation log|Anyone who sees the log|Secrets, personal information|Deciding the items not written to the log (10.4)|—|
|T-15|Denial of service|Huge images and decompression bombs (images, templates, backup ZIPs)|The party handing over the file|App core|Limits on pixel counts, file counts, and extracted sizes (10.6)|—|
|T-16|Denial of service|Huge responses, repeated redirects, and delayed responses from repost pages|Malicious sites|App core|Timeouts, size limits, redirect limits (4.2)|—|
|T-17|Elevation of privilege|External text displayed on screen triggering screen commands|A party planting external text|Screen, core|External text is not interpreted as HTML; CSP blocks external loading (6.2)|—|
|T-18|Elevation of privilege|Calling commands from the screen that read or write arbitrary files|A party who hijacked the screen|Files on the device|Permitted commands only, limited file scope (6.2)|—|
|T-19|Elevation of privilege|Exploiting vulnerabilities in the app core with malicious fonts or images|The party handing over the file|App core|Read with Rust components; formats and sizes limited (10.6)|Unknown vulnerabilities in components|
|T-20|Tampering|Keep returning old valid reference packages or rolling back versions (freeze, rollback)|A party who took over the mirror or the network path|App (reference information)|Package version numbers (older than local are rejected) and expiry (notified when past) (Chapter 8, 3.4 “Reference information package”)|During a freeze, new contact points do not arrive|
|T-21|Information disclosure|Fetch requests for repost pages reveal to the Reposter that evidence is being collected|The operator of the repost page|Case (evidence preservation)|Fetch requests do not identify the System. Sent headers are kept in the WARC (Chapter 6, DD-6-8“Fetch requests do not identify the System”). Guidance on VPNs|The IP address is visible to the other party (H-26“Taking a record of the repost page (the Registration default) shows the User's IP address to the other site”)|
|T-22|Spoofing, tampering|Taking over CI or the GitHub organization to place correctly code-signed fake builds in official releases and rewrite the SHA-256 in the README|A party who took over CI or an owner account|Users obtaining the app for the first time|The update signature is outside CI (so it does not affect updates of existing Users). Two-factor authentication and multiple authentication methods for owners, rules on the main branch and tags, immutable releases (Chapter 8, 3.5 “Protection of the GitHub account and repository”). Release secrets are limited to the release environment (Chapter 11, 8.3 “Handling of secrets”)|New downloaders can only tell by the OS code signing and the SHA-256 in the README, and cannot tell if both are taken (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”)|
|T-23|Tampering|Rewriting the public page's “Check notice code” page to show a fake notice code|A party who took over the GitHub organization|Viewers performing matching|Two-factor authentication for owners, rules on the main branch (Chapter 8, 3.5 “Protection of the GitHub account and repository”). The same computation can be done in the app's G-13“Verify”, and the computation method is public (Chapter 2, 2.4 “How the notice code is made”)|When the organization is taken (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”)|

## 12. External Dependencies

|External party|When it stops|
|---|---|
|TSA|C2PA signing continues and timestamps are held. Multiple TSAs are tried in order (Chapter 3)|
|GitHub|Updates and reference information are fetched from the mirror. Records are on the device, so work does not stop|
|Hong Kong mirror|Fetch from GitHub|
|Posting sites|Outside the System|
|OS keystore|Signing is impossible. The reason is shown (Chapter 2)|

## 13. Devices and People

- Loss or failure of the device (R-1-6-1“In case of device loss or failure, Users can export records and import them on another device”): records and signing keys exist only on the device. The User regularly creates a backup file (Chapter 8, 5 “Backup Files”) and keeps it on external storage or elsewhere. The app shows a recommendation on Home 30 days (initial value) after the last backup. Importing it on a new device restores the signing key, notice code, and records. Without a backup, the key is remade and the notice code re-posted (Chapter 2, 7 “Signing Keys”).
- Multiple devices (R-1-6-2“One person can use multiple devices”): the backup file is imported on another device and the same signing key is used (only one notice code is needed). Merging records created on two machines in parallel follows Chapter 8, 5 “Backup Files”.
- Authorization relationships and transfer of rights (R-1-6-3“Transfer and termination of rights (including revocation of authorization) can be handled”): issuing and revoking authorizations (Chapter 2, 5 “Authorizations and Joint Rights”). A change of Rights Holder is handled by the new Rights Holder C2PA-signing with their own information. Past work data is not transferred.

## 14. Quality Requirements

### 14.1 Scale and performance targets

|Item|Target|Basis|
|---|---|---|
|One batch|Up to 500 photos per folder (initial value)|Assumes delivery for one shoot|
|Image size|Up to 100 megapixels and 200 MB per image (initial value)|Assumes developed output from medium-format and high-resolution cameras|
|Processing time per image (total of C2PA signing, invisible watermark, and hash computation)|Within 10 seconds on the CPU of a typical PC|[To be measured] TrustMark CPU inference time (Chapter 3)|
|Social media export of 500 images (24 megapixels)|Within 30 minutes (initial value)|[To be measured]|
|Evidence per User|200 cases per year, up to 20 MB per case (initial value)|Guide to device capacity (Chapter 8, 6 “Capacity”)|
|App startup|Operable within 10 seconds (initial value)|[To be measured]|

- A “typical PC” has a CPU with 4 or more cores (supporting x86-64-v3, or an Apple CPU), 16 GB of memory, and an SSD (initial value).
- Design handling when targets are missed: if startup exceeds 10 seconds, loading of ONNX Runtime and models and of bundled fonts is deferred until first use (export, opening the editing screen). If exporting 500 images for social media exceeds 30 minutes, the number of images processed simultaneously (Chapter 4, 14.4 “Large images and parallel processing”) and the watermark variant (Chapter 3, 5 “Invisible Watermark”) are reviewed. The same applies if one image exceeds 10 seconds.

### 14.2 Quality scenarios

- How to check that each scenario is met (test procedures and pass criteria) is set in the test documents. This section shows only the scenarios and responses the design must answer for each quality goal.

|Goal|Scenario|Required response|
|---|---|---|
|1|The PC loses power during export|The Original is unchanged. Exported items are recorded and export can continue. Changes to the work session lose at most the last 2 seconds|
|1|The app crashes while saving a record|The record remains as either the previous or the new version; no broken record|
|1|Restoring to another device from a backup file|The signing key and all records return, and all record signatures verify|
|2|A first-time User completes first input and posting the Notice without explanation|4 out of 5 complete them|
|3|Counting what the app sends outside|Only the parties and contents in the table in 4.2|
|4|Checking an exported image on a public verification site|The C2PA signature is shown valid (not shown as trusted; Chapter 2, 3.3 “How validators show it”)|
|4|Changing one pixel of an exported image|Verification shows tampering|
|5|Exporting the same work session on the three OSes|Pixel value differences within 1|
|Performance|Exporting 500 images for X|Within 30 minutes (14.1)|

## 15. Risks

|Risk|Impact|Preparation|
|---|---|---|
|Implementing vertical typesetting in-house|More work and more room for errors|Decide test sentences and check them (Chapter 4, 3.3 “Vertical writing”)|
|Components such as HarfRust and vello_cpu are young|Version upgrades may break things|Pin versions and test when upgrading (Chapter 11)|
|Color accuracy of moxcms|Colors may shift|Compare with lcms2 (Chapter 4, 14.2 “Processing”)|
|TrustMark watermarks unreadable after social media recompression|Fewer matching clues|Measurement (Chapter 3)|
|Suspension of the GitHub account|Distribution stops|The mirror and another hosting service (Chapter 8, 7 “Dependence on GitHub”)|
|Unreachable from mainland China|Users in China cannot obtain or update the app|The Hong Kong mirror (direct download from the Chinese section of the README, and fetching updates and reference information; Chapter 8, 3.6 “Operation of the Hong Kong mirror”, Chapter 9, 5 “Download Routes”)|
|Code-signing reputation (Windows SmartScreen)|Warnings appear early on|Guidance (Chapter 9)|
|Large distributions (fonts about 71 MB, models about 70 MB (TrustMark about 65 MB, u2netp about 4.6 MB), ONNX Runtime)|Downloads take time|The Lite version of LXGW WenKai was adopted (Chapter 4, 6.1 “Bundled fonts”). Criterion 300 MB or less (Chapter 9, 2.1 “Distributables per OS”)|
|NRSD's small structure|Maintenance of keys and reference information stops|Key backup procedures (Chapter 9). Structure is NRSD's decision|

## 16. Glossary

|Term|Meaning|
|---|---|
|C2PA signature|A cryptographic signature attached to an image (the manifest signature under the C2PA technical specification)|
|Visible Signature|A name or credit drawn into the photograph's pixels (Chapter 4)|
|Record signature|A cryptographic signature attached to work data, case records, authorizations, and reference information|
|Code signing|Attaching to distributions a signature that the OS checks (Chapter 9)|
|Update signature|A signature attached to update distributions with NRSD's update signing key (Chapter 9)|
|Personal root|A certificate created on the User's device that issues signing certificates (Chapter 2)|
|Notice code|A short string derived from the fingerprint of the personal root's public key, posted on Notice accounts (Chapter 2)|
|Identification number|The number of each work, placed in the watermark, manifest, etc. (8.2)|
|Work session|The state of editing in progress, gathered together (Chapter 4, 11 “Work Sessions (the State in the Middle of Editing)”)|
|Template|The pattern of groups and layers of a Visible Signature (Chapter 4, 8 “Templates”)|
|Group, layer|The unit of placement of a Visible Signature and the text or image elements within it (Chapter 4, 4 “Groups and Layers”)|
|Work data|The record of an exported work (Chapter 3)|
|Case, case record|A registered repost and its record (Chapter 6)|
|Evidence package|A case's fetched items, screen images, hash list, and timestamp (Chapter 6)|
|Reference information|Contact points, example texts, references to each country's laws, wording of Enclosed Documents (DD-1-6“NRSD signs reference information and places it in the Public Repository and the mirror”)|
|Backup file|A single file containing the signing key and all records, encrypted with a passphrase (Chapter 8)|
|Authorization|A record by which the principal Rights Holder grants authorization (Chapter 2, 5 “Authorizations and Joint Rights”)|

## 17. Mapping to Requirements

|Requirement number|Requirement|Sections in this chapter|
|---|---|---|
|R-1-1-1|What each component holds and does not hold is uniquely determined|4.1 “Components”, 6 “Contents of the Components”|
|R-1-1-2|It is stated explicitly that Phase 2, sales management, rights in source works, and collection of User information are outside the boundary|4.1 “Components”|
|R-1-2-1|For all data, the source, storage location, public/non-public status, and writer are determined|8.1 “List of data”|
|R-1-2-2|The identifier scheme and the versions of data formats are determined|8.2 “Identifier scheme”, 8.3 “Formats and versions”|
|R-1-2-3|The locations of secrets (Users' signing keys, NRSD's keys) are listed|8.5 “List of secrets”|
|R-1-2-4|The retention and deletion policy is determined|8.4 “Retention and deletion”|
|R-1-2-5|Multilingual file names and characters can be handled|8.6 “Multilingual file names and characters”|
|R-1-3-1|Each flow—signing, sales, social media posting, Registration, updates, and distribution of reference information—runs uninterrupted from start to end|7 “Runtime Scenarios”|
|R-1-3-2|Even after partial failure, no data inconsistency remains|7 “Runtime Scenarios”, 10.2 “Storage”|
|R-1-3-3|Work that can be done offline is distinguished from work that needs a connection, and work can be held while offline|4.2 “Communication with the outside”, 7.9 “Offline export and later timestamps”|
|R-1-4-1|The trust boundaries are shown in a figure|11.1 “Trust boundaries”|
|R-1-4-2|There is a list of threats (impersonation, fake apps, key theft, tampering)|11.2 “List of threats”|
|R-1-5-1|The behavior when the TSA, GitHub, or posting sites stop is determined|12 “External Dependencies”|
|R-1-6-1|In case of device loss or failure, Users can export records and import them on another device|13 “Devices and People”|
|R-1-6-2|One person can use multiple devices|13 “Devices and People”|
|R-1-6-3|Transfer and termination of rights (including revocation of authorization) can be handled|13 “Devices and People”|
|R-1-7-1|Scale and performance targets, handling of failures, records (operation logs), version compatibility, and handling of time are common to all chapters|10 “Cross-cutting Concepts”, 14 “Quality Requirements”|
|R-1-7-2|The screen language and the country of Rights Documents are handled separately|10.8 “Languages and countries”|
|R-1-8-1|NRSD's points of involvement, operating structure, and contact during failures are determined|18 “Operations”|
|R-1-8-2|Operating costs and who bears them are determined|18 “Operations”|
|R-1-9-1|There are a structure diagram, a data flow diagram, and a trust boundary diagram|4.1 “Components” (Figure 1-1), 7 “Runtime Scenarios” (Figure 1-2), 11.1 “Trust boundaries” (Figure 1-3)|
|R-1-9-2|There is a mapping table from Design Plan decision numbers to this document's requirement numbers|Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design”|
|R-1-9-3|There is a chapter dependency table and an order for entering the basic design (below)|Outline Design Document 1-9 “Figures and mapping”, 20 “Chapter Dependencies”|
|R-1-10-1|NRSD can make releases, maintain reference information, and receive feedback|19 “Operator Tool”|
|R-1-10-2|The operator functions themselves do not become a path for fake updates or fake reference information|19 “Operator Tool”|

## 18. Operations

### 18.1 NRSD's points of involvement and structure

|Work|Frequency|Owner|
|---|---|---|
|Maintaining reference information (contact points, texts, laws)|Quarterly, and whenever there is a change|NRSD operations (wording of Rights Documents is reviewed by experts; Chapter 5)|
|Receiving feedback (Chapter 10, 9 “Feedback Channel”)|As it comes. Acknowledge within 7 days (initial value)|NRSD operations|
|Releases|As needed|NRSD development|
|Key management (update signing key file and paper backup, reference-information signing key hardware token)|Checked once a year (initial value)|NRSD's representative|

- Operations, development, and representative are role names; as of 2026-09-30 one Lead Developer holds all of them (3 “Constraints”). A trusted other person is to be the second owner of the GitHub organization (Chapter 8, DD-8-9“Protected by a GitHub organization and immutable releases”). Until a second owner can be appointed, the organization runs with one owner, registers multiple authentication methods, keeps paper recovery codes in a separate place, and keeps a local copy of the Public Repository (Chapter 8, 3.5 “Protection of the GitHub account and repository”). The owner's own second account is not made an owner (free accounts are limited to one per person (GitHub Terms of Service B.3), and GitHub recommends “two or more people”). The risk of being unable to recover the organization is declared as a gap (Chapter 13, H-45“While the GitHub organization has one owner, if that person loses all authentication and recovery methods, the organization cannot be recovered”).
- NRSD does not accept Users, issue certificates, or store records.

### 18.2 Operating costs (initial estimate at publication, researched in September 2026)

|Item|Cost|Source|
|---|---|---|
|Apple Developer Program (macOS code signing and notarization)|99 USD per year|Research Materials “GitHub and Distribution”|
|Azure Artifact Signing (Windows code signing)|9.99 USD per month (5,000 signatures). A paid Azure subscription (such as pay-as-you-go) is required; free and trial subscriptions cannot be used|Same as above, Microsoft “Artifact Signing FAQ”|
|GitHub (Public Repository)|Standard Actions runners for public repositories are free|Same as above|
|Hong Kong mirror (object storage)|Outbound transfer free up to 5 GB per month; beyond that, Alibaba Cloud's unit price ([Estimate required]). Formula in Chapter 8, 6.3 “Estimate of mirror cost”|Chapter 8, 6.3 “Estimate of mirror cost”|
|Timestamps (at C2PA signing)|Free TSAs (DigiCert, etc.)|Research Materials “Technical Elements of Signing”|
|Hardware token (for the reference-information signing key; one with PIV ECC P-256)|58 USD each for the YubiKey 5 series (Yubico's sales page, September 2026). Two including a spare|Yubico's sales page, Chapter 8, 3.4 “Reference information package”|
|Offline PC for the update signing key|An existing PC is used (buying a new one is NRSD's decision)|Chapter 9, DD-9-4“The update signing key stays outside CI; code-signing credentials are CI secrets”|
|Expert review of Rights Document wording|[Estimate required] per country, each time the wording is revised|Chapter 5|

- Borne by: NRSD (the System is free of charge. As in Legal Research L-10“Handling of legal business (unauthorized practice of law) (Japan)”, not receiving consideration from Users is the premise for staying clear of unauthorized practice of law).

### 18.3 Incidents involving NRSD's keys

|Step|Action|
|---|---|
|1 Notice|Loss of the key file, paper backup, or hardware token, suspected takeover of a PC, or update signatures or reference packages one does not recognize|
|2 Stop|Stop publishing the update manifest and reference packages in the Public Repository (same as step 1 of Chapter 9, 6.5 “When a version with errors has been released”)|
|3 Inform|On the public page, announce the date and time of the incident, the impact (which period's updates and reference information are suspect), and what Users should do (reinstall from the README). App notifications (G-21“Notifications”) are carried in reference packages (Chapter 8, 3.4 “Reference information package”), so during a reference-information signing key incident they are not used; instead the release notes of the app version containing the new key (shown at update time; Chapter 9, 4.1 “Flow and states” step 6) are used. For an update signing key incident, notify via notifications in a new reference package|
|4 Replace|Create a new key and release an app version containing the new public key. For an update signing key incident, existing Users reinstall manually from the README (Chapter 9, 4.5 “Failures and hijacking”). For a reference-information signing key incident, distribute the app version containing the new public key with the update signing key and publish packages signed with the new key (version numbers count from 1 under the new key). The new app version does not accept packages of the old key (Chapter 8, 3.4 “Reference information package”). The procedure of raising the “minimum version” with the old key is not used (it cannot be signed when the key is lost, and if leaked an attacker can pre-empt with a large version number)|
|5 Record|Record the course of the incident and the response in the operations record|

- Contact during failures: announce on the public page and in app notifications (Chapter 10, G-21“Notifications”).

## 19. Operator Tool

- A separate executable built on the same base as the Client App, not distributed to Users.

|Number|Screen|What it does|
|---|---|---|
|M-01|Reference Information|Editing versions of contact points, texts, each country's laws, and Enclosed Document wording; checking differences from the previous version; record signing and publishing|
|M-02|Release Signing|Checking distribution hashes and applying the update signature (on the operator's offline PC, using the passphrase-encrypted update signing key; Chapter 9, DD-9-4“The update signing key stays outside CI; code-signing credentials are CI secrets”)|

- Operations using keys are done only on the operator's PCs. M-01“Reference Information” runs on the operator's management PC (network-connected; the reference-information signing key is inside the hardware token, and a PIN is entered at each signing), and M-02“Release Signing” runs on the operator's offline PC (update signing key). CI holds neither key (R-1-10-2“The operator functions themselves do not become a path for fake updates or fake reference information”). Feedback is received by e-mail and not handled in the operator tool (Chapter 10, 9 “Feedback Channel”).

## 20. Chapter Dependencies

- As in the dependency table of Outline Design Document 1-9 “Figures and mapping”. This chapter's DD-1-3 (certificates created on the device) and DD-1-4 (records kept on the device) constrain Chapter 2 “Signing Information and Matching” and Chapter 8 “Repository and Data Management” together. DD-1-8 (division between screen and core) constrains Chapter 10 and Chapter 11.

## 21. Gaps Declared in This Chapter

- Prior C2PA signing and replacement by persons posing as the Rights Holder (T-1“Posing as the Rights Holder to C2PA-sign first or replace the C2PA signature”) cannot be prevented. The System only shows matching clues.
- If the device is fully compromised, C2PA signing until the signing key is remade cannot be prevented (T-4“Stealing a User's signing key to C2PA-sign”).
- A User who has not created backup files loses records and evidence if they lose the device.
- Fake apps obtained through routes other than the README cannot be prevented (T-3“Distributing a fake app”).
- If NRSD's keys (update, reference information) are stolen, fake updates or reference information may be distributed until the keys are replaced (T-8“Stealing NRSD's keys”, Chapter 13, H-22“If NRSD's keys (update, reference information) are stolen, fake updates or reference information may be distributed”).
- If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases (T-22“Taking over CI or the GitHub organization to place correctly code-signed fake builds in official releases”, Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”).
- While the GitHub organization has one owner, if that person loses all authentication and recovery methods, the organization cannot be recovered (Chapter 13, H-45“While the GitHub organization has one owner, if that person loses all authentication and recovery methods, the organization cannot be recovered”).
- Taking a record of the repost page (the Registration default) shows the User's IP address to the other site (T-13“Fetching repost pages shows the User's IP address to the other site”).

## 22. Corrections to Other Chapters and the Outline Design Document

- Chapter 4, 11.8 “Where kept”: thumbnails are placed not in `sessions/<work session number>/thumbs/` but in the OS cache location (DD-1-10“Data is kept in a per-user location that is not synchronized”).
- Chapter 8, 2.1 “Arrangement”: the location is app_local_data_dir (not Roaming).
- Chapter 10: the division between screen and core in DD-1-8“The screen handles only display and input; processing happens in the Rust core” is a premise of the screen design.
- Chapter 11: tauri-plugin-single-instance is added to the components.
- Chapter 13: the threats are the 23 from T-1“Posing as the Rights Holder to C2PA-sign first or replace the C2PA signature” to T-23“Rewriting the public page's “Check notice code” page to show a fake notice code”.
