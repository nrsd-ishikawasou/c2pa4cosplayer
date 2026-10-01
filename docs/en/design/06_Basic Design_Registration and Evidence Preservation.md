# Basic Design Document Chapter 6: Registration and Evidence Preservation

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [06_Basic Design_Registration and Evidence Preservation.docx](06_Basic%20Design_Registration%20and%20Evidence%20Preservation.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

## Position of This Document

- Details Chapter 6 of the Outline Design Document. Goes back and forth with Chapter 8 “Repository and Data Management” on capacity and storage location (Chapters 6 and 8 depend on each other).
- The decisions received are as in the following table (the decisions, items to be investigated, and open items of Design Plan Edition 2, and omissions found in the item breakdown). The texts of the Design Plan's decisions, items to be investigated, and open items are per Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” and the destination table of the Outline Design Document (not reproduced in this chapter; only the numbers and the omissions found in the item breakdown are listed).

|Number|Type|Content|
|---|---|---|
|A-4|Omission found in the item breakdown|Correcting wrong Registrations and relieving the impersonated true Rights Holder|
|A-5|Omission found in the item breakdown|Content of status checking (automatically visiting URLs would touch the boundary with Phase 2)|
|A-20|Omission found in the item breakdown|Safety when fetching repost pages|

- The references are as in the following table.

|Reference|What is referred to|
|---|---|
|Chapter 1 “Overall Architecture”|Case numbers, trust boundaries, threats (complaining about a true Rights Holder's post as a repost; touching malicious code when fetching a reposted page)|
|Chapter 2 “Signing Information and Matching”|Matching clues|
|Chapter 3 “Signing”|Watermark, PDQ, work data|
|Chapter 5 “Rights Documents”|Permitted Scope|
|Chapter 8 “Repository and Data Management”|Device data, export|
|Legal Research L-7“Takedown request procedures (Japan)” to L-9“Takedown request procedures (US)”, L-15“Treatment of timestamps as evidence (China)”, L-16“The timestamp system (Japan)”, L-20“Limitation periods for damages claims (Japan, China, US)”, L-21“Copying for evidence (Japan, China, US)”|Procedures for takedown requests, treatment of timestamps, periods for claims, copying for evidence|

- The decisions received follow Design Plan Edition 2 (D-10-7“Evidence is stored on the User’s device and not published. The System does not collect evidence”: evidence is stored on the User's device and the System does not collect it. D-10-9“Information on registered reposts is listed on the User’s device, and the options for action are presented. No sharing among Users takes place (dealt with in Phase 2)”: registered reposts are listed on the device with options for action, and are not shared among Users).

## 1. List of Design Decisions

|Number|Decision|Reason|Rejected alternatives|
|---|---|---|---|
|DD-6-1|The app takes in reposted pages only by “fetching without running scripts” (the HTTP response is saved as is and not rendered on screen). For a record of how the page looks, the User takes a screenshot in their own browser and imports it|The device is not exposed to scripts and vulnerabilities of malicious sites (T-9“Touching malicious content when fetching repost pages”, A-20“Safety when fetching repost pages”). For pages requiring login, only what the User saw can be kept (D-10-3“Pages requiring login are obtained by the User personally logging in”)|Rendering pages in a browser built into the app|
|DD-6-2|Status checking is updated only by the User's operation (the “Check current status” button); the app does not visit URLs automatically on a schedule|Scheduled automatic crawling belongs to Phase 2 (automated detection of license violations) (A-5“Content of status checking (automatically visiting URLs would touch the boundary with Phase 2)”, D-13-1“Phase 2 shall be a separate design plan”)|Scheduled checking|
|DD-6-3|An RFC 3161 timestamp is attached to the evidence package (the list of hashes of what was obtained). By default a free TSA; if the User chooses, a Japanese accredited provider or Chinese trusted timestamp (paid) can also be attached|Section 7 of the Draft Project Proposal (git dates have no probative value). Trusted timestamp evidence is widely used in Chinese courts (L-15“Treatment of timestamps as evidence (China)”)|Always using a paid TSA (cost)|
|DD-6-4|Registered reposts are listed on the device, grouped by reposting site (domain), with their subsequent status and options for action (Chapter 7 “Legal Action Guidance”). They are not shared with other Users|D-10-9“Information on registered reposts is listed on the User’s device, and the options for action are presented. No sharing among Users takes place (dealt with in Phase 2)”. Lightens the burden on the finder of tracking the outcome alone (Design Plan 10.6 “Sharing the Burden”)|Sharing reposting sites among Users (Edition 1; would mean collecting Users' information)|
|DD-6-5|Case records are append-only; corrections and withdrawals are also appended as new status records (the evidence files of a withdrawn case are deleted, and the record of withdrawal is kept; 7)|Prevents tampering and shows the history. To show the chain of evidence handling later (Berkeley Protocol paragraph 167), records must be accumulated without rewriting. Records are given a record signature with the User's key (Chapter 1, DD-1-5“Device records carry record signatures and are verified”)|Overwriting|
|DD-6-6|For each registration, a “collection record” (who collected, when, with which tool and version, by which procedure, and the device clock offset) is kept in the case record. Its form follows Annex IV “Online data collection form” of the Berkeley Protocol of the UN Office of the High Commissioner for Human Rights and the University of California, Berkeley|The origin of evidence and its chain of custody can be shown later. The Berkeley Protocol calls for recording the collector, the collection device, the time, and hashes (paragraphs 155 and 167)|Not keeping a collection record|
|DD-6-7|Responses of pages fetched by the app are saved in WARC (ISO 28500:2017), the international standard for web records. The HTML source goes inside the WARC|Can save including the control information of requests and responses (headers, etc.); it is the form widely used by web archiving institutions, and experts can open it with common tools|Saving HTML and headers separately in a custom form (first draft)|
|DD-6-8|Requests for fetching reposted pages do not identify the System. The request headers (User-Agent, etc.) are aligned with the values of the OS's default browser screen component (WebView) on that device, and all headers sent are kept in the WARC request record|If the fetch request tells the Reposter that evidence is being collected, the repost may be deleted or rewritten. The Berkeley Protocol (paragraphs 99 to 102) mentions concealing the investigator's identity (the principles of anonymity and non-attribution), that leaks of browser information can alert the subject, and changing headers that reveal user information (user agent, etc.). Headers sent are recorded so the conditions of fetching can be shown later|Identifying the System's name and version (would alert the Reposter)|

## 2. Registration

### 2.1 Procedure

|Step|User|App|
|---|---|---|
|1|Press “Register a repost” and paste the URL of the reposted page|Normalize the URL and determine the platform (Chapter 1, 10.10 “Forms of URLs and QR codes”; the URL as entered is also kept in the record). The determination corresponds to the list of contact points in the reference information (Chapter 7, 2.1 “List of contact points in the first edition (initial content of the reference information)”). If the same domain or URL is already in the User's list on the device, the User is told|
|2|Specify the reposted image as a file saved from the User's own browser, or by the image URL (optional)|A file is imported at once; the watermark is read and PDQ computed, and matched against the User's own work data (4). For a URL, only the record is made, and fetching is done in step 6 (after the notice and choice of step 4)|
|3|Take a screenshot of the page in the User's own browser and put in the image (required for pages requiring login; recommended otherwise). Putting in a file made with the browser extension “Save evidence” (2.2.2) satisfies this step and step 4 at once|Compute the hash of the screenshot. A file from the extension is imported after its internal hashes are checked|
|4|(Done by default) Take a record of the page. When the URL of step 1 is entered, “A record of the page is taken together with registration. The other site can see this PC's IP address” is shown, with the option “Do not take it for this registration” (the default is to take it)|When step 6 is pressed, save the page's HTML and main images by fetching without running scripts (2.2) (except when “Do not take” was chosen)|
|5|(Optional) Fill in the input fields for evidence that should be kept (2.3)|—|
|6|Press “Register” (one button)|Preserve evidence (3), then create the case record with a record signature, then save on the device, then the timestamp (attached later if there is no connection; the timestamp response is not added to the case record but added as a separate appended record `status/<device number>.jsonl`; 3.2) (Chapter 1, 7 “Runtime Scenarios”)|
|(Before)|To search for reposts by hand, “Search by image” in G-12“Works” opens the pages of Google Lens (`lens.google.com`; `uploadbyurl` is not used because it requires a public URL) and TinEye (`tineye.com/search`) in the default browser, and the User uploads a copy of the export to the page|The app sends no photo; it only opens the page and shows where the export is. No automatic crawling (Phase 2) (NRSD's request, 2026-10-01)|

- For the processing of step 6 (evidence preservation, creation of the case record and record signature, saving on the device, timestamp), a done mark is placed per stage in `state/` (Chapter 8, 2.1 “Arrangement”), and if it stops midway it continues from the recorded stage at the next start (Chapter 1, 7.8 “Starting after an abnormal exit”).
- One button (R-6-1-1): registration is possible with only the URL and step 6, and the record of the reposted page (step 4) is also taken by default. Design Plan 10.2 says “Evidence is preserved at the same time as registration. Users do not need to know what is needed as evidence”, and 10.3 calls for keeping the record of “when and where it was published (②)” at the time of registration, so the page record does not depend on the User's operation. Steps 2, 3, and 5 are optional additions; if omitted, it is shown that the evidence will be less.
- Balancing the IP address: fetching lets the other site see the User's IP address (Chapter 13, H-26“Taking a record of the repost page (the Registration default) shows the User's IP address to the other site”). It is taken by default, this visibility is shown before registration, and it can be turned off for that registration only. Registrations with it turned off are marked “no page record (② only by screenshot)”.
- Recorded on the device (R-6-1-3): the case record is placed on the User's device and given a record signature with the User's key. It does not affect other Users' records.
- Threat T-13 (information disclosure): fetching repost pages shows the User's IP address to the other site. Attacker: the operator of the repost site. Target asset: the User's location. Remaining gap: visible when fetched (Chapter 13, H-26“Taking a record of the repost page (the Registration default) shows the User's IP address to the other site”)

### 2.2 Fetching without running scripts

- HTTP GET (the timeouts, size limits, and redirect limits follow the table in Chapter 1, 4.2 “Communication with the outside”). The response body, headers, final URL, time of fetching (UTC), and the IP address of the responding server are saved.
- Among `og:image` and `<img>` in the HTML, the top 5 largest are fetched by the same method (the HTML is parsed with html5ever; relative URLs are resolved against the final URL, and for `srcset` the candidate with the largest descriptor is taken). JavaScript, CSS, and fonts are not fetched.
- What is fetched is not rendered on screen. Images are read with the same image foundation as Chapter 3 “Signing” (Rust's image crate) (those that cannot be read get only a hash).
- If redirected to a login screen, that response is recorded as is, and “Login is required, so a screenshot (step 3) is needed” is shown.
- Request headers (DD-6-8“Fetch requests do not identify the System”) follow the repost page row of Chapter 1, 4.2 “Communication with the outside”.
- Notice before fetching (Chapter 1, 4.2 “Communication with the outside”): shows that the other site can see the User's IP address, and that a VPN can be used if this is a concern (Berkeley Protocol paragraph 101). The System does not provide a VPN.
- Threat T-9 (elevation of privilege): touching malicious content when fetching repost pages. Attacker: malicious sites. Target asset: the User's device. Remaining gap: viewing in the User's own browser
- Threat T-21 (information disclosure): fetch requests for repost pages reveal to the Reposter that evidence is being collected. Attacker: the operator of the repost page. Target asset: case (evidence preservation). Remaining gap: the IP address is visible to the other party (H-26“Taking a record of the repost page (the Registration default) shows the User's IP address to the other site”)

### 2.2.1 Saving in WARC and WACZ

- One fetch (the page and main images) is put in one WARC file (`capture-<serial>.warc.gz`; WARC/1.1). The record types are warcinfo (`software`, `format`, the fetching tool and its version), request (the request sent; each stage's request if redirected), response (the response received; each stage's response of redirects and the final response), and metadata (the time of fetching, final URL, IP address of the responding server, and the subject and issuer of the TLS certificate), and each record carries `WARC-Record-ID` (`urn:uuid:`), `WARC-Target-URI`, `WARC-Date`, and `WARC-Block-Digest` (sha256) (the record types and mandatory headers of ISO 28500:2017). A copy of the DOM made by the extension (not an HTTP response) goes in as a resource record. The WACZ `datapackage.json` has `wacz_version` 1.1.1, `created`, `software`, and `resources` (the sha256 of each file).
- The WARC is wrapped in WACZ (Webrecorder's Web Archive Collection Zipped; Library of Congress format registration fdd000586; the WARC, indexes, `pages.jsonl`, and screenshots in one ZIP), and in the form of WACZ Signing and Verification 0.1.0 a signature with the User's signing certificate and an RFC 3161 timestamp are placed in `datapackage-digest.json`. Anyone can verify it with Webrecorder's tools (py-wacz, ReplayWeb.page) and Starling Lab's tools. The output of the browser extension (2.2.2) is the same WACZ (NRSD's request, 2026-10-01).
- Signing the extension's output: because the extension has no key, it writes only hashes in `datapackage-digest.json` (leaving `signedData` of WACZ Signing 0.1.0 empty). At import, the app checks the hashes, attaches an anonymous signature (`hash`, `created`, `software`, `signature`, `publicKey`) with the User's signing certificate, and puts an RFC 3161 timestamp in `timeSignature`. The meaning of the signature is “this was the content when the app received it”. The same form as a signature by an archiving tool in the specification.
- The records of RDAP queries are added to the same WARC (5).
- Component: the warc crate (0.4.0, MIT, updated 2024-11, about 2.37 million cumulative downloads on crates.io). It has reading and writing of WARC 1.0 and 1.1 records. [To be measured] that the written WARC can be opened with common tools (pywb, etc.).

### 2.2.2 Guidance on taking screenshots

- The Berkeley Protocol (paragraph 155) lists a full-page screen capture (with date and time) as a minimum standard alongside the URL and the HTML source. Users are guided as follows.

  1. Capture with the browser's “full-page screenshot” function (in Firefox, “Save full page” in the screenshot function; in Chrome, “Capture full size screenshot” in the developer tools). The extension captures the whole page: in Firefox with the whole of `tabs.captureTab`, and in the Chromium family by stitching while scrolling the screen (the same as full-page extensions such as GoFullPage; the `debugger` permission is not used).

  2. Also take one screenshot showing the URL bar at the top and the OS clock (a full-page capture does not show the URL or date/time).

  3. For pages requiring login, capture while logged in.

- The app reads the capture date/time of the imported screenshot. Full-page captures from browsers are mostly PNG without a capture date/time, so in that case the file's modification date/time is used and recorded distinguished as “file date/time (not a record of the capture date/time)”. If either differs greatly from the registration date/time (24 hours or more; initial value), confirmation is requested. If the date/time is unknown, it is recorded as “no date/time”.
- The browser extension “Save evidence” (Chromium family and Firefox; placed in the public extension stores, with the source in `extension/` of the Public Repository; Chapter 9, 2): like SingleFile, with one button it gathers the page's URL, date/time, a full-page screenshot, and the HTML and images being displayed into one WACZ file (`.wacz`; the same form as 2.2.1), with each hash attached inside the extension, saves it with the browser's downloads API, and passes its location to the app through the URL scheme registered with the OS (`c2pa4cosplayer://import?path=…`) (if the app is not running, the OS starts it. The location accepted is only under the User's Downloads or the watched folder; other locations and locations containing `..` are not imported, and the User is asked to choose the file. In case it does not get through, the app also watches the save folder the User chose). Even a page requiring login can be kept as the User's own screen, in the login state of the User's browser. The import limits are 200 files inside the ZIP and 400 MB after extraction (within the limit per case), with no `..` in names. The extension does not identify the System and sends nothing to sites. The extension's manifest is MV3, and its permissions are only `activeTab`, `scripting`, and `downloads` (no `host_permissions` or `storage`; the same manifest for Firefox). Compatibility between the extension and the app is held in `dev/compat.yaml` (a row is the extension version, the minimum app version that accepts it, `wacz_version`, and the major version of `software`), and the app refuses output from an extension that is too new by this table at import. The app's own saving is WARC and PNG as at Perma.cc (2.2.1). The guidance above remains for Users without the extension (NRSD's request, 2026-09-30).

- Extension versions and when the app is not running: the extension writes `nrsd-capture/<version>` and `wacz_version` in `software` of `datapackage.json`. The app accepts up to the `wacz_version` and the extension's major version it knows; if newer, “Please update the app” (the same handling as a newer record format). The URL scheme (deep link) is handed over by the OS launching the app (tauri-plugin-deep-link; on Linux, AppImage registers with `register()` at start). Watching the save folder happens only while running, and at start any `.wacz` not yet imported in that folder is picked up.

### 2.3 Input fields for evidence that should be kept

|Field|Purpose|
|---|---|
|Reposter's account name and display name|To identify the other party in complaints and investigations|
|Date/time of the post (as shown on the page)|Since when it has been published|
|Number of views, likes, etc. (if shown)|Material on the extent of damage|
|Kind of commercial use (multiple choice: paid distribution or sale, advertising, merchandising, AI training data, none or unknown)|Priority for commercial use (Chapter 7, 6.2 “Joint action and priority”, Legal L-5“Portrait rights and publicity rights (Japan)”)|
|URLs of other posts by the same Reposter|Material on continuity|
|Free description|—|

- The screen says “as far as you know”, and registration is possible with fields left blank.

### 2.4 Fields of registration data

|Field|Content|
|---|---|
|schema|`nrsd.case/1`|
|case_id|Case ID|
|source|`manual` (this chapter). `auto` is reserved for Phase 2 (R-6-1-4)|
|url, url_normalized, final_url, platform|The reposted page (as entered, the canonical form, the final URL)|
|work_refs|Identification numbers of works found by matching and match levels (4)|
|evidence|List of evidence files (original name, kind, SHA-256, size, how obtained (provided by the User / fetched by the app), time obtained). Files are placed in `evidence/` under their SHA-256 names (with extension), and the original names of screenshots the User put in are kept only in the list (they may contain real names and the like; the same as browsertrix and Perma.cc)|
|whois|Results of the investigation for ④ (5)|
|user_notes|Input of 2.3|
|rights_ref|The ID and version of the Permitted Scope of the matched work (Chapter 5 “Rights Documents”)|
|violations|Determination of what is violated (4.2)|
|(Timestamp)|Not included in the case record. The timestamp response is added, together with the list of evidence, as a status addition (`status/<device number>.jsonl` and `.sig`) (case records are append-only; DD-6-5“Case records are append-only”)|
|signer|Serial number of the signing certificate and notice code (Chapter 2)|
|created_at|UTC|
|clock|The device clock value, the clock skew at that time, the monotonic number (Chapter 1, 10.7)|
|chain|The hash of the previous row and this row's record signature (the chain of case records and status additions; Chapter 1, 8.3)|
|collection|Collection record (2.5)|

- Where saved: `cases/<case ID>/case.json` with a record signature on the device, and evidence files in `cases/<case ID>/evidence/` (Chapter 8, 2.1 “Arrangement”). The total evidence of one case is up to 1 GB (initial value; the sum of the fetch limits of 2.2 plus screenshots and fetches by “Check current status”). When exceeded, import is refused and the User is told to register it as a separate case.

### 2.5 Collection record

- The form follows Annex IV “Online data collection form” of the Berkeley Protocol.

|Item of Annex IV|What the System records|
|---|---|
|Collector|The signer's handle name and notice code (names are not recorded; the User can add them to the report at export; 3.4)|
|IP address of the collection device|Not recorded (the IP address visible from outside the device cannot be known without querying outside). The User can add it to the report|
|Start and end date/time of collection|The time the registration screen was opened and the time Register was pressed (UTC)|
|Target URL|The input URL and the final URL of the fetch|
|HTML source|Inside the WARC (2.2.1)|
|Screenshots|Imported screenshots (2.2.2)|
|Data obtained|List of fetched images|
|IP address|IP address of the responding server and DNS results (5)|
|List of hashes of the collection package, and the hash of that list|List of SHA-256 per evidence file, and the SHA-256 of the list (the target of the timestamp in 3.2)|

- Also recorded: the app version, the version of the fetching tool, and the device clock offset. The clock offset is recorded as the difference between the time in the timestamp response and the device clock (Berkeley Protocol paragraph 155(g) calls for synchronizing the device clock; the System does not correct the device clock but records the offset).
- When captured with the extension, the extension's version, the browser's name and version, and the date/time the extension recorded are also written in the collection record.

### 2.6 Intake from Phase 2

- Registrations with `source: auto` are added to the case records on the device, with a record signature, only after the User checks and approves the content in the app (not registered automatically). Details are set in the design plan of Phase 2.

## 3. Evidence Preservation

### 3.1 Correspondence with the items to be shown later (Design Plan 10.3)

|Number|Item to be shown|What the System keeps|
|---|---|---|
|①|That it is one's own work|Work data (the Original's fingerprint, the output's SHA-256 and PDQ, the copy of the manifest, the timestamp) referenced from the case record|
|②|When and where it was published|Screenshots, the record of fetching without running scripts (HTML, headers, final URL, time of fetching), the input of 2.3, the timestamp on the evidence package|
|③|That it is the same image|The identification number read from the invisible watermark, the PDQ distance (4)|
|④|Who published it and where|DNS, IP address, ASN, country, domain registration information (5), the account name of 2.3|
|⑤|That the use is outside the Permitted Scope|The Permitted Scope of the matched work and the hash of the Enclosed Document (Chapter 5 “Rights Documents”), the posting of the Notice (Chapter 2 “Signing Information and Matching”)|

### 3.2 Trusted timestamps

- An RFC 3161 timestamp is obtained on the SHA-256 of a list (JSON) of the SHA-256 of each evidence file (the timestamp here has nothing to do with C2PA validators and is used as evidence of time). The responses of all TSAs that responded are saved (Chapter 3, 4) (NRSD's request, 2026-09-30). The list and the timestamp response are added to the case as a status addition (`status/<device number>.jsonl`).
- External means of reinforcing evidence (those for which Design Plan 10.3 sets references): Chinese internet courts accept electronic data whose authenticity can be proven by technical means such as electronic signatures, trusted timestamps, hash value verification, and blockchain, or by certification from electronic evidence-collection and preservation platforms (Provisions of the Supreme People's Court on Several Issues Concerning the Trial of Cases by Internet Courts, Article 11). In Japan, notarial deeds of factual experiments, in which a notary records facts experienced with the five senses, are used to preserve the display state of web pages. The System does not perform these procedures, and shows their names and links under “To prepare further” in the action guide (G-17“Action Guide”). The export of the evidence package (3.4) is in a file form that can be brought as is into these procedures (Legal Research L-24“Treatment of electronic signatures and electronic data as evidence (Japan, China, US)”).
- Paid TSAs (Japanese accredited providers, Chinese trusted timestamps) can be chosen by the User per case. The cost is borne by the User (NRSD does not act as an intermediary; the User sets the account of the TSA they contracted with in the app). The connection and where the account is kept follow Chapter 3, 4 “Trusted Timestamps”. [Estimate required] Each provider's offerings and fees for individuals.
- The bundled timestamp of the heads of all record chains on the device and the ERS (Chapter 3, 4 “Trusted Timestamps”) also prove the existence of case records and status additions. A case registered offline is proved at the next connection in one go. The evidence package bag includes the EvidenceRecord extract leading to that case (3.4).
- Secret: paid TSA accounts (if the User set them). Location: the OS keystore. Included in backups. Not synchronized between devices. If leaked: timestamps could be requested under that account. Disable it with the provider

### 3.3 Fetch failures

|Situation|Handling|
|---|---|
|The URL has already disappeared (404, 410)|Record that response; can be registered if there is a screenshot|
|Cannot connect (blocked, timeout)|Record the failure and register with the screenshot. Retry later with “Check current status” (6)|
|Redirected to login|As in 2.2|
|Response too large (over 50 MB)|Record the first 50 MB, the hash, and that it was cut off|

### 3.4 Export

- `bag-info.txt` holds, among the reserved fields of RFC 8493, `Bagging-Date`, `Bag-Software-Agent` (this app's name and version), `Payload-Oxum`, `External-Identifier` (the case number), and `Source-Organization` (the signer's handle name), and no real name (the User adds it in the report's fields).
- “Export evidence package” makes, per case, a BagIt (RFC 8493; the packaging form of the Library of Congress and the California Digital Library) bag, serialized per RFC 8493, 4 “Serialization” into one ZIP (named `evidence-<case number>-<date and time>.zip`, containing one bag folder) (`data/` holds the content below, `manifest-sha256.txt` the SHA-256 of all files, `bag-info.txt` a summary of the collection record, and `tagmanifest-sha256.txt`; experts can verify it with ordinary bagit tools) (NRSD's request, 2026-10-01). The content of the bag is: the case record with its record signature, evidence files (WARC, screenshots, fetched images), timestamp responses, matched work data, a copy of the Enclosed Document, the collection record (2.5), verification instructions (TXT; Japanese, Chinese, English), and a report (TXT; summary of registration, correspondence with ① to ⑤, list of hashes).
- The report has fields for the User to add (name, IP address of the collection device, etc.). What is added is not saved in the app (it goes only into the exported file).
- In addition to TXT, the report is also made as PDF/A-3u (ISO 19005-3; long-term preservation; text can be extracted; how it is made is in Chapter 1, 10.11 “PDF documents”), with the bag's `manifest-sha256.txt` and the ERS EvidenceRecord embedded. A single PDF can be handed to an expert (NRSD's request, 2026-10-01).
- The evidence package includes the extract of the chain leading to that case, the bundled timestamp, and instructions for walking the chain (recomputing hashes). Experts can confirm “this record existed at this date and time” without the System (NRSD's request, 2026-09-30).
- The verification instructions give how to compute hashes (standard OS commands), how to verify timestamps (OpenSSL commands), and how to validate records with the JSON Schema (Chapter 1, 8.3), and a copy of the Schema is included in the evidence package, so that experts can check without the System (NRSD's request, 2026-09-30).

## 4. Matching

### 4.1 How the match level is shown

|Result|Display|
|---|---|
|The identification number can be read from the invisible watermark and matches the User's work data|“Match (watermark): identification number XXXXX-XXXXX-XXX”|
|The watermark cannot be read, but the PDQ distance is 0 to 15 (initial value)|“Near match (visual fingerprint)”|
|PDQ distance 16 to 31|“Similar (visual fingerprint): possibly the same photo edited” (within the matching threshold of 31 or less in Chapter 3, 6 “Matching Hash”)|
|The identification number can be read from the watermark but is not in the User's work data|“Watermark with someone else's number”. The User is guided to the screen of clues about persons posing as the Rights Holder (Chapter 10, G-24“Compare Clues”, Chapter 2, 4.4 “Clues when a person posing as the Rights Holder appears”)|
|The watermark cannot be read and the PDQ distance is 32 or more, but the ISCC Content-Code (Chapter 3, 6) matches the User's work data|“Near match (content code)” (aligned with “matching uses both PDQ and ISCC” in Chapter 3, 6)|
|PDQ distance 32 or more, no watermark, and no ISCC match either|“A match cannot be confirmed”|

- When multiple works match: `work_refs` are ordered by watermark match → ascending PDQ distance → newest first. The display is grouped by the Original's SHA-256, shown as “Match: identification number X (delivery). N other exports of the same photo” (Chapter 1, 8.2: different presets of the same photo are linked by the Original's SHA-256).
- The judgment of a match and responsibility for registration lie with the User (D-10-6“The judgment on matching and the responsibility for Registration are borne by the User who registered”). Registration is possible even with “A match cannot be confirmed”, but it is marked “registration with no confirmed match” in the list, and the User is told to check before making a complaint.

### 4.2 What is violated

- From the Permitted Scope of the matched work and the manner of the repost (“paid distribution” in 2.3, being published), the candidates of violation are shown. The rules for the candidates follow Chapter 5, 2.3 “Linking to images”.
- Candidates are shown not as legal judgments but as the result of comparison with the Permitted Scope (Chapter 7, 7 “Where the Line Is Drawn”).

## 5. Who and Where (④)

- Look up A and AAAA records by DNS, and for the IP address obtain the ASN, organization, country, and abuse contact by the RIR's RDAP. For the domain, obtain the registrar and contacts by domain RDAP (the same mechanism as ICANN Lookup) (Draft Project Proposal [23]).
- Chinese sites: if an ICP filing number can be read from the bottom of the page, it is recorded. Lookup in the filing system of the Ministry of Industry and Information Technology (Draft Project Proposal [22]) is guided in Chapter 7 “Legal Action Guidance” as a procedure for the User (the lookup screen has an image check, so the app does not look it up automatically).
- RDAP queries are saved as evidence in the WARC (2.2.1) as the raw request and response (JSON, headers, the subject and issuer of the TLS certificate). Both the IP address and the domain, and the result of the reverse lookup of the IP address, are kept (NRSD's request, 2026-09-30). The RDAP bootstrap (IANA's list of which registry to query) is obtained from the reference information, and IANA is not queried (Chapter 8, 3.4).
- The real server behind a CDN may not be found by RDAP. Even then, the CDN provider's abuse contact becomes the place for complaints (Chapter 7 “Legal Action Guidance”).

## 6. Status Checking

### 6.1 Status records

|Status|Who records it|
|---|---|
|Registered|The app (at registration)|
|Complaint made (date/time, contact point, method, standing of the complainant (Chapter 7, 3.4 “Standing of the complainant”))|The User's input (from the flow of Chapter 7)|
|Provider's response (receipt, result of decision)|The User's input|
|Removal confirmed|The User's operation (6.2)|
|Reposting confirmed|The User's operation|
|Withdrawn|The User's input (7)|

- Kinds of status (`kind` of `nrsd.status/1`): `registered`, `filed` (complaint made), `response` (provider's response), `removed`, `reposted`, `checked` (current status checked), `timestamped` (timestamp response), `withdrawn`.
- Fields of the “complaint made” and “provider's response” rows: contact point (the number in the reference information), method (form or e-mail), date/time sent, receipt number (if any), the complainant's standing, the SHA-256 of the copy of the text sent (the copy of the complaint placed in `filings/` of Chapter 8, 2.1 “Arrangement”), date/time of the response, result (received, removed, refused, counter forwarded, other), notes. These are the fields of `nrsd.status/1`.
- Copy of the complaint: a copy of the text at the time the model text was copied, with the fields the User entered (name, address, phone, e-mail) replaced by `[REDACTED]`, is saved as `cases/<case number>/filings/<UUID v7>.txt`, and the “complaint made” status append holds its SHA-256. It is included in the evidence package (BagIt). The policy of not storing names (Chapter 7, 3.1) is kept.
- The calculation of procedural deadlines after a complaint, the export of `.ics`, and the notices 3 days before and on the day follow Chapter 7, 6.3 “Display of procedural deadlines” (shown on the case screen).

### 6.2 Checking the current status

- Only when the User presses the button, one fetch without running scripts is done on the registered URL. The fetched response is added to `evidence/` as a WARC of the same form as 2.2.1 (`capture-<serial>.warc.gz`; the serial continues from the fetch at registration) (it becomes evidence of reposting; the same as Perma.cc and the Wayback Machine keeping a new record each time they fetch again), and the HTTP status, hash of the body, presence of images, and the SHA-256 of the WARC are recorded in a status addition with a timestamp.
- No scheduled automatic checks. The list screen shows the “last checked date” so the User can remember.

## 7. Corrections

- Withdrawing a mistaken registration: the User chooses a reason (mistake, already permitted, other), and a withdrawal record is appended. The evidence folder is moved to the OS trash (freedesktop.org Trash Specification 1.0, the Windows Recycle Bin, the macOS Trash; the `trash` crate; no custom trash). The User can restore it from the OS trash. The withdrawal, reason, and date/time remain in the case record (Chapter 1, 8.4 “Retention and deletion”). It is excluded from the counts in the list (NRSD's request, 2026-10-01). Chain verification after a withdrawal covers only the record rows (JSON), and the presence of evidence files is checked separately. For a withdrawn case, “evidence deleted” is normal, and absence is not treated as an error (as a Git commit can point to a deleted blob: the hash remains in the record). If evidence is missing for a case that was not withdrawn, it is reported with a `CAS` number.
- When a repost of something not one's own work has been registered (R-6-6-2): the registration stays on the device and does not reach other Users or providers. Harm arises only if the User sends a complaint based on that registration; before a complaint, the matching result and clues (Chapter 2, 4.4 “Clues when a person posing as the Rights Holder appears”) are shown, together with the fact that responsibility lies with the User (D-10-6“The judgment on matching and the responsibility for Registration are borne by the User who registered”).

## 8. List and Means

- List: registered reposts are grouped by reposting site (domain), showing the count, first registration date, last checked date, status (6.1), and a mark of commercial use (Chapter 7, 6.2 “Joint action and priority”).
- Options for action: for each case, the possible actions (complaints to the posting site's contact point, notices to CDN and hosting providers, ICP filing lookup for Chinese sites, export in preparation for consulting experts) are shown. Which to take is up to the User (Chapter 7 “Legal Action Guidance”).
- Sharing: not shared with other Users (D-10-9“Information on registered reposts is listed on the User’s device, and the options for action are presented. No sharing among Users takes place (dealt with in Phase 2)”). Sharing information on reposting sites and notifications to authors from persons other than the author are handled in the design plan of Phase 2 (D-13-3“Registration of contact details for notifying authors is dealt with in the Phase 2 design plan”).

## 9. Mapping to Requirements

|Requirement number (text in the Outline Design Document)|Sections in this chapter|
|---|---|
|R-6-1-1|2 “Registration”|
|R-6-1-2|2 “Registration”|
|R-6-1-3|2 “Registration”|
|R-6-1-4|2 “Registration”|
|R-6-2-1|2.2 “Fetching without running scripts”, 3.1 “Correspondence with the items to be shown later (Design Plan 10.3)”, 3.3 “Fetch failures”|
|R-6-2-2|2.2 “Fetching without running scripts”, 3.1 “Correspondence with the items to be shown later (Design Plan 10.3)”, 3.3 “Fetch failures”|
|R-6-3-1|3.2 “Trusted timestamps”, 3.4 “Export”|
|R-6-3-2|3.2 “Trusted timestamps”, 3.4 “Export”|
|R-6-4-1|4 “Matching” (the rules for the candidates of violation are in Chapter 5, 2.3 “Linking to images”)|
|R-6-5-1|6 “Status Checking”|
|R-6-5-2|6 “Status Checking”|
|R-6-6-1|7 “Corrections”|
|R-6-6-2|7 “Corrections”|
|R-6-7-1|8 “List and Means”|

## 10. Gaps Declared in This Chapter

- The gaps of this chapter follow the table in Chapter 13, 4.1 “Gaps in the mechanism” (the rows whose chapter column is this chapter; with why they cannot be closed, the extent addressed, the remaining risks, and who bears them) (not reproduced in this chapter).

## 11. Corrections to the Outline Design Document

- O-06“Sharing of information on repost sites” was deleted in Design Plan Edition 2 (not shared).
- State in 6-5 (status checking) that “it is checked only on the User's operation, with no automatic crawling”.
