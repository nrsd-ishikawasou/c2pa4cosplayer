# Basic Design Document Chapter 9: Distribution and Updates

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [09_Basic Design_Distribution and Updates.docx](09_Basic%20Design_Distribution%20and%20Updates.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

## Position of This Document

- Details Chapter 9 of the Outline Design Document.
- The decisions received are as in the following table (the decisions, items to be investigated, and open items of Design Plan Edition 2, and omissions found in the item breakdown).

|Number|Type|Content|
|---|---|---|
|D-3-3|Decision|Users shall be able to use the System without looking at GitHub|
|D-12-6|Decision|Distribution is from a GitHub link, with an easy-to-understand README|
|D-12-7|Decision|The Client App receives binaries from the Public Repository and updates itself|
|I-04|To Be Investigated|Access to GitHub from mainland China|
|I-05|To Be Investigated|Code signing|
|A-6|Omission found in the item breakdown|Preparation against fake apps|
|A-18|Omission found in the item breakdown|Export controls on software containing cryptography (handled by Legal)|

- The references are as in the following table.

|Reference|What is referred to|
|---|---|
|Research Materials “GitHub and Distribution”|Connections from mainland China, code signing|
|Research Materials “Technical Elements of Signing”, Section 5|Tauri's update mechanism|
|Section 2 of the study memo “Study of Chapter 7 Legal Action Guidance, Chapter 9 Distribution and Updates, Chapter 10 Screens and Design” (NRSD's internal record of study, not distributed with this document; the facts underlying the design are written with their sources in each section of this document)|Facts researched and considerations for Chapter 9 (including the second round)|
|Chapter 1 “Overall Architecture”|Where secrets are kept, threats (fake apps, fake reference information and updates)|
|Chapter 8 “Repository and Data Management”|Public Repository, Hong Kong mirror, location of device data|
|Chapter 11 “Development Base”|Build, minimum supported OS versions, components|
|Legal Research L-14“Export control of software containing cryptography (Japan, US)”|Export control|

## 1. List of Design Decisions

|Number|Decision|Reason|Rejected alternatives|
|---|---|---|---|
|DD-9-1|One distributable per OS: for Windows, an NSIS installer (per-user installation; code-signed with Artifact Signing); for macOS, one DMG for Apple CPUs (code-signed with Developer ID and notarized); for Linux, an AppImage (with an update signature by the update signing key in an accompanying file)|Formats non-technical people can install with one operation. Minimizes each OS's warnings (I-05“Code signing”). Because the ONNX runtime stopped distributing for Intel macOS, macOS is Apple CPU only (Chapter 11, 3.1 “Minimum supported OS versions”)|Separating Linux into deb and rpm (one format suffices for non-technical people). Making macOS a form containing both Intel and Apple (Universal) (second draft; there is no ONNX runtime for Intel). MSI (has no per-user installation setting, and the major value of the version number goes only up to 255, which does not fit “year.month.serial”)|
|DD-9-2|Self-update uses Tauri's update mechanism, and the update manifest is obtained in the order “`update/latest.json` on the public page (GitHub Pages)”, then “`update/latest.json` on the Hong Kong mirror”. Distributables are taken from GitHub releases for the GitHub manifest and from the mirror for the mirror manifest. Verification of update signatures cannot be disabled. The version is put into the update signature (`tauri signer sign --app-version`), and `requireSignedVersion` of the update component is set to true, rejecting updates whose manifest version does not match the signature's version|Verification of update signatures is mandatory (Research Materials “Technical Elements of Signing”, Section 5). Several sources can be held (mainland China). The update manifest (JSON) has no signature, and someone who can rewrite it could distribute a past, correctly signed distributable with a large version number. If `requireSignedVersion` is left false, the check is skipped for signatures without a version record (verify_signed_version in updater.rs of tauri-plugin-updater, `--app-version` of signer sign in tauri-cli; checked 2026-09-30)|Having Users install updates by hand. Trusting only the manifest's version (old versions could be forced on Users, and DD-9-6“No updates that roll back the version are distributed” could not be kept)|
|DD-9-3|The official version is identified by the publisher name of the OS code signature (Nagareyama Software Development Co., Ltd.) and the SHA-256 of distributables listed in the README and on the public page. The app's G-23“About This App” shows the publisher of its own code signature and its version|Protection against fake apps (T-3“Distributing a fake app”). Provides means Users can check|—|
|DD-9-4|The update signing key is Tauri's signing key (a key file made with `tauri signer generate`, encrypted with a passphrase), kept on an operator's PC not connected to the network, with a paper backup in a safe. Update signatures are made with the operator tool on that PC. CI does not make update distributables (`createUpdaterArtifacts` set to false). The macOS `.app.tar.gz` for updates is made on the operator's PC from the `.app` whose notarization CI has completed, and signed (6.1). Code signing credentials are the cloud code signing service (Artifact Signing) for Windows and Developer ID for macOS, and are placed in CI secrets. The risk of having code signing credentials in CI is dealt with in Chapter 1, T-22“Taking over CI or the GitHub organization to place correctly code-signed fake builds in official releases” and Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”|Protection against hijacking of the update path (Chapter 1, T-7“Distributing fake reference information or updates”). Tauri's update signatures cannot be disabled, and losing the key means updates cannot be distributed (the official guide to Tauri's update mechanism). Tauri's signing key is designed to be handled as a key file, and no mechanism for using it inside a hardware token could be confirmed|Keeping it in a hardware token (first draft; does not fit how Tauri's signing key is handled). Keeping the update signing key in CI secrets (a CI takeover could distribute fake updates). Making update distributables in CI (with `createUpdaterArtifacts` true, `TAURI_SIGNING_PRIVATE_KEY` is needed at build time, and the build stops without it; bundle.rs of tauri-cli)|
|DD-9-5|Updates are downloaded and verified in the background, and replacement happens when the User closes the app or presses “Restart now to update”. Replacement does not happen in the middle of batch processing, registration, or backup creation|On Windows the app exits automatically at the replacement step (the official guide to Tauri's update mechanism). The app is not terminated in the middle of editing. Saving of work sessions is finished before replacement (Chapter 4, 11 “Work Sessions (the State in the Middle of Editing)”)|Replacing as soon as downloaded (would exit in the middle of editing)|
|DD-9-6|Updates that go back to an earlier version are not distributed. If a version with errors is released, a fixed newer version is released (fixing forward). Spread to Users who have not yet installed it is stopped by reverting the update manifest to the previous version|If the record format version has gone up, going back to an old version would leave the old version unable to read the new records (4.4). By default Tauri installs only newer versions|Allowing version rollback (record format inconsistency)|
|DD-9-7|The operation to erase the device's records is placed in the app's settings (G-20“Settings”), and erases including the signing keys in the OS keystore. The “delete data” checkbox of the Windows uninstaller has its text replaced so that it is clear that evidence will be lost and cannot be restored|The macOS DMG and Linux AppImage have no uninstaller. With the default checkbox name of the Windows uninstaller (“Delete the application data”), it is not conveyed that evidence will be lost, and the signing keys (Credential Manager) are not erased (Tauri's NSIS template)|Leaving it only to the uninstaller (cannot erase on macOS and Linux; signing keys remain even on Windows)|
|DD-9-8|For the Linux AppImage runtime (type2-runtime, fetched at build time) and linuxdeploy plugins, the hashes of what was fetched are kept in the release record and the SBOM|Tauri fetches the runtime without pinning its version (a report of 2026-09). It becomes possible to check afterwards what went in|Doing nothing (the runtime in the distributable cannot be identified afterwards)|

## 2. Installation

### 2.1 Distributables per OS

|OS|Format|CPU|Install location and privileges|Prerequisite components|First-run warnings and guidance|
|---|---|---|---|---|---|
|Windows 10/11|NSIS installer (`.exe`). installMode is currentUser (Tauri's default)|x86_64 supporting x86-64-v3 (Chapter 11, 3.1 “Minimum supported OS versions”). ARM Windows runs via x64 emulation ([To be measured] start-up and watermarking on Windows 11 on Snapdragon; whether x64 emulation handles x86-64-v3 instructions)|`%LOCALAPPDATA%\<product name>`. No administrator privileges needed|WebView2: downloadBootstrapper (Tauri's default; Windows 10 (April 2018 and later) and 11 have it as part of the OS; installed from the network only if absent)|SmartScreen may show an “unrecognized app” warning even with code signing until reputation accumulates. The publisher name is displayed, so the README and first-run guidance (the README steps before Chapter 10, G-01“Welcome”) show with images: “Check that the publisher is Nagareyama Software Development Co., Ltd., then [More info], then [Run anyway]”|
|macOS 14 or later (due to the minimum of the bundled ONNX runtime; Chapter 11, 3.1 “Minimum supported OS versions”)|DMG|Apple CPUs (aarch64) only. Intel Macs are not supported (8 “Gaps Declared in This Chapter”)|The User drags it into “Applications”|None|Notarized, so only the usual confirmation. If started directly from the DMG (a read-only location), updates are not possible, so at start the User is told to move it to “Applications” ([To be measured] how to detect this)|
|Linux (glibc 2.35 or later; Ubuntu 22.04 or later, Debian 12 or later)|AppImage|x86_64 supporting x86-64-v3 (ARM can only be built on ARM machines, so not released in the first edition)|Where the User places it. Execute permission is given|None. The AppImage bundles the shared components it depends on (WebKitGTK 4.1, GTK, GLib, etc.; LGPL) (Tauri's AppImage guide: “bundles all dependencies”). Bundled components are covered by the exception in Chapter 11, 5.1 “Permitted licenses” and by the license notices of 6.2. A runtime that does not require FUSE 2 is used (tauri-bundler 2.6.0 or later; the source is written as an expectation, “should remove”, so [To be measured] it starts on a fresh Ubuntu 24.04 environment)|No OS warning. How to give execute permission is shown in the README|

- The minimum versions follow the strictest of TrustMark, ort, and the ONNX runtime (Chapter 11, 3.1 “Minimum supported OS versions”). Linux distributables are built on the minimum OS (Ubuntu 22.04) (Tauri's guide: build on the oldest OS you want to support).
- Size of distributables: the app itself, on Linux the bundled WebKitGTK and so on (about 70 MB or more; [To be measured]), the ONNX runtime (about 14 MB or more; Chapter 11, 4.3 “Assets”), the TrustMark model (about 65 MB), u2netp (about 4.6 MB), fonts (about 71 MB; the sum of the bundled fonts of about 52 MB and the screen text fonts of about 18 MB in Chapter 11, 4.3 “Assets”; includes emoji at 25.3 MB). [To be measured] the total. Assumption: around 200 MB for each OS. Criterion: 300 MB or less.
- PCs not connected to the network: the installer is self-contained except for WebView2. Windows without WebView2 (fixed corporate versions, etc.) that is not connected to the network cannot install it (8 “Gaps Declared in This Chapter”).

### 2.2 Uninstallation and device records

|OS|How to remove the app|What remains (by default)|How to erase records|
|---|---|---|---|
|Windows|From “Settings, then Apps”, or the uninstaller in the install location|Records in `%LOCALAPPDATA%\<identifier>` (app data, reduced images, operation logs), signing keys in Credential Manager|Before uninstalling, “Erase this device's records” in G-20“Settings”. Or choose the uninstaller checkbox (2.3) (in this case the signing keys remain)|
|macOS|Move the `.app` from “Applications” to the Trash|Records in `~/Library/Application Support/<identifier>`, signing keys in the Keychain|Before uninstalling, “Erase this device's records” in G-20“Settings”|
|Linux|Delete the AppImage file|Records in `~/.local/share/<identifier>`, signing keys in Secret Service (or the key file)|Before uninstalling, “Erase this device's records” in G-20“Settings”|

- By default records remain. Records exist only on this device (Chapter 8 “Repository and Data Management”), and can be used as they are after reinstalling.
- The README's “How to remove the app” gives the table above and “To erase records, erase them in the app first” in three languages.

### 2.3 Procedure for “Erase this device's records”

1. Placed in the “Data” section of G-20“Settings”. Pressing it shows a confirmation dialog.
2. The dialog lists what will be erased (signing information and signing keys, work data, case records and evidence, templates, work sessions, settings, operation logs) and what will not (images and backup files the User exported, the originals of the User's photos). It shows the date/time of the last backup, and if there is no backup or records have grown since the last backup, “Create a backup” is shown first.
3. It can be executed only after the User types a confirmation word (“erase” in the screen language). The default button is “Cancel”.
4. Execution: erase the signing keys in the OS keystore, then the app's data location (app_local_data_dir; on Windows including the WebView2 data `EBWebView/`), then the reduced image location (app_cache_dir), then the operation log location (app_log_dir), then on macOS the WKWebView data (`~/Library/WebKit/<identifier>`) (Chapter 1, 9.1 “Locations per OS”, Chapter 8, 2.1 “Arrangement”; on macOS these are under ~/Library/Application Support, ~/Library/Caches, and ~/Library/Logs respectively; on Windows and Linux, operation logs are in `logs/` under the app's data location). Items that failed midway are shown in a list.
5. After execution the app exits. The next start is the same as the first (G-01“Welcome”).
- The text of the Windows uninstaller checkbox is replaced by replacing Tauri's language file with text meaning “Erase records (works, cases, evidence, templates). They cannot be restored without a backup. Signing keys remain. To erase signing keys too, erase them first in the app's settings” (Japanese, English, Chinese). It is replaced with Tauri's NSIS setting `customLanguageFiles` (specifying per language a `.nsh` file with Tauri's own texts; the language is also put in `languages`). The text of this checkbox is the `deleteAppData` string in the `.nsh` (Tauri's config.rs, installer.nsi, and language files; checked 2026-09-30).

## 3. Proof of the Official Version

### 3.1 Code signing

|OS|Credential|Procedure and cost of obtaining (researched 2026-09-29)|Scope of signing|
|---|---|---|---|
|Windows|A public trust certificate of Artifact Signing (formerly Trusted Signing). The certificate name is the verified legal entity name (Nagareyama Software Development Co., Ltd.)|Organizations in Japan are eligible (individuals only in the United States and Canada). A paid Azure subscription (from USD 9.99 per month). Organization identity validation takes 1 to 20 business days. The official guide states no minimum number of years since incorporation (a Microsoft Q&A answer says there is none). EV is not issued|The installer (`.exe`) and the executables and DLLs inside it|
|macOS|Developer ID Application certificate and notarization|Apple Developer Program (USD 99 per year)|Sign all executables and dynamic libraries in the `.app` (including the ONNX runtime), enable Hardened Runtime, notarize, and staple the notarization record to the DMG. [To be measured] that the ONNX runtime's dynamic library can be loaded (library validation)|
|Linux|The OS has no code signing mechanism|—|Update signature (`.AppImage.sig`), SHA-256 in the README, and GitHub provenance attestations (Chapter 11, 8 “Build and Continuous Integration (CI)”)|

- Prerequisites for obtaining them (checked in each company's guides on 2026-09-30):
  - Artifact Signing's organization identity validation asks for the company website URL, e-mail addresses at a domain the company owns (primary and secondary), a business identifier, the address, and personal identity verification of the representative, and may ask for additional documents (registration, proof of domain registration, etc.) (Microsoft “Quickstart: Set up Artifact Signing”). A paid Azure subscription is required (free and trial subscriptions cannot be used; same FAQ). Issued certificates rotate over short periods (3 days per the ELAM guide), and signatures carry a timestamp (timestamp.acs.microsoft.com) (same FAQ). The certificate name (CN, O) is the verified legal entity name and cannot be changed.
  - Apple Developer Program organization enrollment asks for a D-U-N-S number, a public and functioning company website (company domain), and a business e-mail at that domain (Apple “Enrollment”).
  - Therefore, NRSD's company website and domain (nrsd.jp; confirmed on 2026-09-30 to be the website of Nagareyama Software Development Co., Ltd.) and e-mail at that domain are prerequisites. The public page (github.io of GitHub Pages) does not meet this prerequisite.
  - Whether the publisher name is in kanji or Latin letters is recorded after obtaining it (the certificate name follows the registered legal name).
- SmartScreen reputation accumulates on two things: the publisher's certificate and the file's hash. If signing continues with the same publisher's certificate, new versions may inherit the certificate's reputation (Microsoft “SmartScreen reputation for Windows app developers”). Therefore the publisher (legal name) is not changed.
- Windows 11's Smart App Control blocks unsigned executables, so all executables and DLLs inside the installer are also signed.

### 3.2 How to tell

- DD-9-3“Official builds are identified by the code-signing publisher and SHA-256”. The README and public page list the SHA-256 of each distributable, the code signing publisher name, and the Developer ID team name.
- The official source is the README of the Public Repository, and only the sources listed in the README (GitHub releases and NRSD's Hong Kong mirror listed in the Chinese section of the README) are official (5). The README states clearly: “The official sources are GitHub releases and the NRSD mirror listed here. Anything distributed on other sites is not official.” Distributables from either source can be identified by the same SHA-256 and code signing publisher.
- Keys used for releases (R-9-2-2“The keys used for releases are protected”): DD-9-4“The update signing key stays outside CI; code-signing credentials are CI secrets”. The update signing key is Tauri's signing key (losing it means updates cannot be distributed to existing Users), and it is kept in two ways: the key file (encrypted with a passphrase) on the operator's PC not connected to the network, and a paper backup (safe).

## 4. Updates

### 4.1 Flow and states

|Step|What is done|What the User sees|On failure|
|---|---|---|---|
|1 Check|At start and every 24 hours while running, obtain the update manifest (4.3) from GitHub, then the mirror|Nothing|Try at the next check|
|2 Download|If there is a new version, download the distributable in the background (temporary location)|“Preparing an update” in G-21“Notifications”|If the download fails (no response, cut off midway, not 2xx), the order of sources is changed to mirror first from the next check (4.3). If the mirror also fails three times in a row, the User is told and guided to reinstall manually|
|3 Verify|Verify the update signature with the public key embedded in the app|Nothing|If verification fails, discard it. Record it in the operation log and tell the User “The update could not be verified”|
|4 Wait|Wait ready for replacement|“The update is ready. It will be applied when you close the app” and “Restart now to update” in G-21“Notifications” and at the top of the screen|—|
|5 Replace|When the app is closed, or when “Restart now to update” is pressed. It cannot be pressed in the middle of batch processing, registration, or backup creation. Saving of work sessions is finished before replacement|Windows: a small progress window (passive). macOS, Linux: restart|Start with the old version (4.5)|
|6 Begin|At the first start of the new version, perform record format migration (4.4) and show the changes in G-21“Notifications”|Summary of changes (three languages)|Rollback of 4.4|

- No operation by the User is needed (R-9-3-1“It is updated without User operation, and what is received is verified”). Only the timing of replacement is matched to the User's operation (DD-9-5“Replacement happens on close or when the User presses the button”).
- All that is sent when checking for updates is the download request, and the other party can see the IP address. The request's User-Agent is the update component's name and version (`tauri-plugin-updater/<version>`), and the app version is not sent (updater.rs of tauri-plugin-updater) (Chapter 12, 5.2 “List of what is sent outside the device”).

### 4.2 Users of old versions (forced updates)

- The reference information holds a “minimum version”, and versions older than that stop C2PA signing and registration and ask for an update (to follow changes in record and reference information formats). Viewing (the Verify screen), exporting evidence packages, and creating backups can be used even on old versions (so that Users are not prevented from getting their records out).
- The minimum version is raised only when ① old versions cannot correctly handle the formats of records or reference information, or ② old versions have errors or vulnerabilities that damage Users' records.

### 4.3 Form of the update manifest (static JSON)

- Follows the static JSON form of Tauri's update mechanism (Tauri's official guide).

|Field|Content|
|---|---|
|`version`|Valid SemVer (the version number of 6.4)|
|`notes`|Summary of changes (texts in three languages combined into one)|
|`pub_date`|RFC 3339 date/time|
|`platforms.windows-x86_64`|`signature` (the content of `.exe.sig`) and `url` (the NSIS installer)|
|`platforms.darwin-aarch64`|`.app.tar.gz` and the content of its `.sig`. `darwin-x86_64` is not listed (Intel Macs are not supported)|
|`platforms.linux-x86_64`|`.AppImage` and the content of its `.sig`|

- Two manifests are placed. In the GitHub manifest the distributable URLs point to GitHub releases, and in the mirror manifest the URLs point to the mirror. The version and update signatures are the same (revising the second draft's “the same JSON”; URLs pointing to GitHub may not be downloadable in mainland China).
- Location: the manifest for GitHub is `update/latest.json` on the public page (outside immutable releases; Chapter 8, 3.1 “Structure”, 3.5 “Protection of the GitHub account and repository”), and the manifest for the mirror is `update/latest.json` on the mirror. Distributables and update signatures are placed in GitHub releases (immutable) and on the mirror.
- Tauri moves on to the next source only when the response is not successful (2xx) (official guide). If GitHub returns success but the content is broken, it fails update signature verification and is not applied. In this case the mirror is tried first at the next check. The order of sources can be passed at run time via the update component's `endpoints` (updater.rs of tauri-plugin-updater), so the app counts the times downloads of distributables from GitHub failed and the times verification failed, and after a failure checks again with the mirror first. Tauri moves to the next source only at the manifest download step, and distributables are taken from the manifest's `url`, so the app changes the order itself in preparation for cases in mainland China where the manifest arrives but distributables are blocked.
- The Windows installation mode is passive (only a small progress window; no operation needed; the official default).
- Version check: set the update component's setting `plugins.updater.requireSignedVersion` to true. Update signatures are made on the operator's PC with `tauri signer sign --app-version <version>`, putting the version in the signature's trusted comment. Updates whose manifest `version` does not match the signature's version are not applied (DD-9-2“Updates are fetched via Tauri's mechanism, checking the version in the manifest against the signature”).

### 4.4 Record format migration

- In updates that raise the `schema` version of records, migration is performed at the first start of the new version.
- Procedure: ① before migration, copy the records whose format changes (excluding evidence files, whose format does not change) to `migration-backup/<original version>/` in the same place. Before copying, check the free space, and if insufficient, notify without starting migration (Chapter 8, 6.1 “Guide to device capacity”) ② perform the migration ③ verify the migrated records (record signatures, counts, `schema` versions) ④ if successful, keep the copy for 30 days (initial value) and then delete it.
- On failure: restore from the copy and show “Preparation after the update failed. Your records are as they were”. The new version starts with records opened read-only (no writing), and shows how to send feedback (Chapter 10, 9 “Feedback Channel”). Reinstalling the old version is not suggested (no version rollback; DD-9-6“No updates that roll back the version are distributed”).

### 4.5 Failures and hijacking

- If stopped in the middle of replacement: Windows NSIS may stop with old files left in place. If the old version starts at the next start, retry from step 2 of 4.1. If it does not start, follow the reinstall procedure in the README (records are not lost; 2.2).
- Hijacking of the update path (R-9-3-3“There is a provision for when the update route is taken over”): the update signing key is outside the repository, so even if the Public Repository account is taken over, updates without a valid update signature are not applied. If the key itself leaks, Users are asked to manually reinstall, from the README, a version with a new key (Tauri's update public key is embedded in the app and cannot be replaced automatically; declared as a gap).

## 5. Download Routes

- README (Japanese, Chinese, English): ① what it can do (one paragraph), ② download (button-like links per OS), ③ how to install (up to three images per OS: the SmartScreen screen for Windows, the drag to “Applications” for macOS, execute permission for Linux), ④ how to tell the official version (publisher name, SHA-256, how to check them), ⑤ the matching procedure (how to check the signer of an image and the notice code), ⑥ how to remove the app and the handling of records (2.2), ⑦ not being involved in the rights of the original works and not collecting Users' information. Only these are placed at the top of the README so that the rest of the GitHub screen does not have to be looked at.
- Mainland China: the Chinese section of the README carries direct links to the latest distributables on the Hong Kong mirror and their SHA-256. The links are inside the README on GitHub, so this stays within the scope of Design Plan D-12-6“Distribution is from a GitHub link, with an easy-to-understand README”. The mirror is used both for obtaining updates (4) and for first download.
  - Reason: in mainland China, connections to github.com are often obstructed, and downloading large release files is slow (Research Materials “GitHub and Distribution”). The Design Plan makes Chinese Users one of the main targets, and to meet Outline Design Document R-8-5-2“Distributions and reference information of the Public Repository can be obtained from mainland China as well”, a route outside GitHub is needed for first download too.
  - Precedents: TrafficMonitor's README guides domestic users to a domestic copy (Gitee) with “国内用户如果遇到Github下载缓慢的问题，可以点击此处” (checked 2026-09-30). Pot's official site uses its own relay (dl.pot-app.com) from the first download.
  - Sample Notice texts and Enclosed Documents do not carry the mirror URL, but point to the README (Public Repository). Old versions on the mirror are deleted (Chapter 8, 3.6), so version URLs are not placed in texts that remain long.
  - The default domain of Alibaba Cloud OSS in Hong Kong returns distributables (executables, DMG, AppImage) as downloads, so it can be used for direct links. No HTML guide page is placed. Alibaba Cloud itself says that connections from mainland China to regions outside may be unstable, and reachability is not guaranteed (Chapter 13, H-17“In mainland China the README may be unreachable”).
- [To be measured] Whether GitHub's README and releases and the mirror can be reached from mainland China (Chapter 8, 6.2 “Mainland China”). Users who cannot reach the README (GitHub) cannot obtain it (8 “Gaps Declared in This Chapter”, Chapter 13, H-17“In mainland China the README may be unreachable”). If university mirrors (TUNA, USTC, SDU github-release) copy the System, they are introduced in the Chinese section of the README with the note “This is a third-party mirror; check the SHA-256” (the same treatment as Flutter's guide).

### 5.1 Preview version (version for User trials)

- User trials (Chapter 10, 2.4 “Judgment”) are done with a working version. That version is the “preview version”; it is not listed in official distribution (README and update manifest) and is given only to trial participants.
- The preview version is code-signed by the same procedure as the official version, and C2PA signatures and record formats are the same as production (images and records made in trials can be used as they are in the official version). The version number is `year.month.serial-preview.serial` (the SemVer pre-release form; e.g., `2026.11.1-preview.1`), treated as a version before the official one.
- The preview version's update manifest is placed at a URL separate from the official manifest, and only trial participants' apps look there. When the official version is released after the trial, they move to the official manifest.

## 6. Release

### 6.1 Procedure

1. Decide the version number (6.4). Check the export control conditions (6.3).
2. Build artifacts for the three OSes with Actions. Linux is built in an Ubuntu 22.04 environment, and the hash of the AppImage runtime is recorded (DD-9-8“The hash of the AppImage runtime is kept”). Make the list of licenses (Chapter 11, DD-11-9“The license list is bundled automatically”) and the SBOM.
3. Perform Windows and macOS code signing and notarization in CI. CI builds with `createUpdaterArtifacts` false and does not make update distributables or update signatures.
4. The person in charge moves the artifacts to the operator's PC and checks their hashes. For macOS, the notarized `.app` is packed into `.app.tar.gz` (the same form as the update distributable Tauri makes). Each OS's update distributable (Windows `.exe`, macOS `.app.tar.gz`, Linux `.AppImage`) is given an update signature with the update signing key by `tauri signer sign --app-version <version>` (DD-9-4“The update signing key stays outside CI; code-signing credentials are CI secrets”).
5. On fresh environments of the minimum OSes (Windows 10, a macOS 14 machine with an Apple CPU, Ubuntu 22.04 and 24.04), check ① new installation, ② update from the previous version, ③ record migration, ④ start-up and export ([Estimate required] preparing virtual machines). The content of the tests is set in the test documents.
6. Place them in GitHub releases and copy to the Hong Kong mirror. Place the corresponding source packages (of the Ubuntu 22.04 versions used for the build) of the LGPL shared components (WebKitGTK, GTK, GLib, etc.) bundled in the Linux AppImage in the same release and mirror (the exception of Chapter 11, 5.1 “Permitted licenses”; Sections 4 and 6(d) of LGPL-2.1). Update the SHA-256 in the README, the direct links to the mirror in the Chinese section, and the changes (three languages). Replace the update manifests (for GitHub, `update/` on the public page; for the mirror, on the mirror; Chapter 8, 3.1 “Structure”) last (so the manifest does not appear before the distributables; the manifest is placed outside immutable releases, so it can be replaced).
7. Raise the “minimum version” in the reference information as needed (4.2).

### 6.2 Bundled license notices

- License documents and copyright notices of third-party components (Tauri, c2pa-rs, trustmark and its model, ort and the ONNX runtime, pdqhash, image, u2netp, fonts, the AppImage runtime, OS-derived shared components bundled in the AppImage (WebKitGTK, GTK, GLib, etc.; LGPL; the exception of Chapter 11, 5.1 “Permitted licenses”)). For LGPL components, the corresponding source is also placed at the same location as the distributables (step 6 of 6.1) (Design Plan Chapter 17 “Third-Party Rights and Licenses”, Chapter 11, 4 “List of Components”).
- For OFL fonts, the copyright notice, license notice, and license text are bundled (Legal Research L-19“Font licenses”).

### 6.3 Export control of software containing cryptography

- The System's uses of cryptography are C2PA signatures and record signatures (ECDSA P-256), hashes (SHA-256), encryption of backup files (age format: scrypt, ChaCha20-Poly1305, HMAC-SHA-256), update signatures (Ed25519; the format of Tauri's signing key), and communication (TLS), all of which are public standard cryptography. No custom cryptography is implemented (Chapter 12, DD-12-8“Only public standard cryptography is used, distributed openly and free of charge”).
- Japan (where NRSD is located): providing programs whose source code is published does not require permission under Article 9(2)(ix)(d) of the Ministerial Order on Trade-Related Transactions and Other Non-Trade Transactions. The distributables (executables) may also fall under item (xiv)(a) of the same paragraph, as provided free of charge without restriction on acquisition and designed not to require technical support.
- United States: publicly available encryption source code is not subject to the EAR, and notification is required only when implementing “non-standard cryptography” (15 CFR 742.15(b); amendment of March 29, 2021). The System does not require notification.
- In the release procedure (step 1 of 6.1), the following is checked every time: ① no newly added component implements non-standard cryptography (checked with the SBOM and component descriptions) ② the source code of that version is published in the Public Repository ③ distributables are distributed free of charge without restriction. If even one is not met, the release is stopped and Legal Research L-14“Export control of software containing cryptography (Japan, US)” is reviewed.

### 6.4 Versioning

- “year.month.serial” (e.g., the first in October 2026 is `2026.10.1`). SemVer does not allow leading zeros in numbers, so months are written as `9`, not `09`. The second in the same month is `2026.10.2`.
- Compatibility of record formats is expressed not by version numbers but by the records' `schema` versions (Chapter 1, 8.3 “Formats and versions”).
- If MSI is added, the upper limit of MSI version numbers (major and minor values up to 255) applies, so the versioning is reviewed (Microsoft “ProductVersion property”).

### 6.5 When a version with errors has been released

|Step|What is done|Deadline (initial value)|
|---|---|---|
|1|Revert the update manifests (for GitHub and the mirror) to the previous version, stopping the spread to Users who have not yet installed it|Within 1 hour of noticing|
|2|Investigate the impact on Users who installed the version with errors, and announce on the public page and in the reference information (G-21“Notifications”)|Within 24 hours|
|3|Release a fixed version (the version number larger than the version with errors; DD-9-6“No updates that roll back the version are distributed”)|Depends on the severity of the error|
|4|For errors that damage records, raise the minimum version in the reference information to the fixed version (4.2)|At the same time as the fixed version|

- The version with errors in GitHub releases is not deleted, and “Do not use” is added to its title (kept so Users who have downloaded it can check the SHA-256).

## 7. Mapping to Requirements

|Requirement number|Requirement|Sections in this chapter|
|---|---|---|
|R-9-1-1|Non-technical people can install it on the three OSes|2.1 “Distributables per OS”|
|R-9-1-2|The handling of device data at uninstallation is determined|2.2 “Uninstallation and device records”, 2.3 “Procedure for “Erase this device's records””|
|R-9-2-1|Can be distinguished from fake apps|3 “Proof of the Official Version”|
|R-9-2-2|The keys used for releases are protected|3.2 “How to tell”|
|R-9-3-1|Updated without operation, and what is received is verified|4.1 “Flow and states”|
|R-9-3-2|Does not break even on failure, and data is migrated|4.4 “Record format migration”, 4.5 “Failures and hijacking”|
|R-9-3-3|There is preparation for when the update path is hijacked|4.5 “Failures and hijacking”|
|R-9-4-1|Can be obtained from the README without getting lost. Can also be obtained from mainland China|5 “Download Routes”|
|R-9-5-1|Necessary license notices are bundled|6.2 “Bundled license notices”|

## 8. Gaps Declared in This Chapter

- Fake apps obtained through routes other than the README cannot be prevented (T-3“Distributing a fake app”).
- If the update signing key leaks, existing Users need to reinstall manually. If NRSD's keys are stolen, fake updates may be distributed until the keys are replaced (Chapter 13, H-22“If NRSD's keys (update, reference information) are stolen, fake updates or reference information may be distributed”).
- If the CI or GitHub organization is taken over, fake distributables with valid code signatures may be placed in the official release (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”).
- Windows SmartScreen may show warnings initially even with code signing.
- Users in mainland China who cannot reach the README (GitHub) itself cannot obtain it. If they reach the README, they can obtain it via the direct links to the mirror (5). The mirror may also be unreachable.
- It cannot be installed on Windows without WebView2 and not connected to the network.
- It cannot be used on Macs with Intel CPUs, or on PCs with CPUs lacking AVX2 and so on (x86-64-v3) (Intel before 2013, and some Pentium and Celeron after 2013, etc.) (Chapter 11, 3.1 “Minimum supported OS versions”).
- Removing the app alone leaves records and signing keys on the device (a design that keeps them by default). To erase them, they must first be erased in the app, and the records of Users who removed the app without knowing this remain on the device until reinstalled.

## 9. Corrections to the Outline Design Document and Other Chapters

- Add to 9-5 “check the export control conditions (only public standard cryptography, published, free of charge) in the release procedure”.
- Add “Erase this device's records” (2.3) to the “Data” section of Chapter 10, G-20“Settings”. Add the update preparation state and “Restart now to update” to G-21“Notifications”.
- Chapter 1, T-7“Distributing fake reference information or updates” and Chapter 11, 8 “Build and Continuous Integration (CI)”: add recording the hash of the AppImage runtime (DD-9-8“The hash of the AppImage runtime is kept”) to the CI procedure.
- Chapter 13, 4.1 “Gaps in the mechanism”: add the gaps “Windows without WebView2 and not connected to the network” and “removing the app alone leaves records and signing keys”.
