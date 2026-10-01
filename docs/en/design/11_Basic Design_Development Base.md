# Basic Design Document Chapter 11: Development Base

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [11_Basic Design_Development Base.docx](11_Basic%20Design_Development%20Base.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

## Position of This Document

- Details Chapter 11 of the Outline Design Document. This chapter sets the framework for screens and processing, the list of components (dependencies) and the policy on them, the repository structure, build and continuous integration (CI), the software bill of materials (SBOM) and provenance attestations, coding conventions, versioning, preparation for when a component's maintenance stops, and NRSD's license (decided as Apache-2.0).
- This chapter is based on the study memo “Study of Chapter 11 Development Base” (NRSD's internal record of study, not distributed with this document; the facts underlying the design are written with their sources in each section of this document). The third draft revised the minimum OS and CPU supported by the ONNX runtime, how the ONNX runtime is obtained, and how c2pa-rs's cryptography implementation is chosen, to fit the facts researched (the second-round section of the study memo).
- The decisions received are as in the following table. The texts of the Design Plan's decisions, items to be investigated, and open items are per Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” and the destination table of the Outline Design Document (not reproduced in this chapter; only the numbers and the omissions found in the item breakdown are listed).

|Number|Type|Content|
|---|---|---|
|A-11|Omission found in the item breakdown|Font licenses|
|A-18|Omission found in the item breakdown|Export controls on software containing cryptography (handled by Legal)|

## 1. List of Design Decisions

|Number|Decision|Reason|Rejected alternatives|
|---|---|---|---|
|DD-11-1|The Client App and operator tool are built with Tauri 2. Screens in HTML, CSS, and TypeScript; processing in Rust|Meets the conditions of Design Plan 6.2 “Development Base” (three OSes, freedom of look, image processing, C2PA, watermark, and hash, self-update) (2). Uses the OS's WebView, so distributables are small|Electron (bundles Chromium and Node.js; large distributables). Qt (requires judgment on freedom of look and on LGPL/commercial licensing). Flutter (C2PA and watermark components would need to be called separately from Rust)|
|DD-11-2|C2PA uses c2pa-rs (crate name c2pa)|The public SDK of the Content Authenticity Initiative (CAI) (c2pa-rs README: “part of the Content Authenticity Initiative open-source SDK”). The core of c2pa-python|Calling c2pa-python as a separate process (requires bundling Python)|
|DD-11-3|The invisible watermark uses TrustMark's Rust crate (trustmark) and the ONNX runtime (ort). Variant Q is the default, with P or C as candidates decided by measurement (Chapter 3, DD-3-2“The watermark is TrustMark and carries the identification number”)|A C2PA-approved watermark (Q, C, P). MIT. The Rust version supports embedding and reading|Bundling the Python version (large distributables with PyTorch)|
|DD-11-4|PDQ uses pdqhash (pure Rust, Apache-2.0), adopted after confirming agreement with the test values of the original reference implementation. If they do not agree, switch to pdq-rs (wrapping the original C++)|Pure Rust is easy to build|Wrapping C++ from the start (build effort on three OSes)|
|DD-11-5|Images are read and written with the image crate. Supported formats: JPEG, PNG, TIFF, WebP. Pixels of HEIC and RAW are not read (for Originals such as RAW, only the SHA-256 fingerprint is recorded)|Decoding HEIC carries a heavy burden of patents and components. Pixels are not needed to prove possession of the Original (the file's fingerprint suffices)|Bundling libheif and libraw|
|DD-11-6|Keys are kept in the OS keystore (Credential Manager / DPAPI on Windows, Keychain on macOS, Secret Service on Linux) via the keyring crate|Fits C2PA's Level 1 requirements (stored encrypted, using the OS keystore) (Research Materials “C2PA Certificates and Identity”)|App-specific encrypted files|
|DD-11-7|Communication uses reqwest (TLS by rustls), limited to the parties and limits of Chapter 1, 4.2 “Communication with the outside”|Limits outbound sending (quality goal 3 of Chapter 1). rustls avoids differences between OS TLS implementations|Using the OS's TLS (behavior differs across three OSes)|
|DD-11-8|Internationalization uses screen text files (JSON in ICU MessageFormat)|Languages can be added later to Japanese, Chinese, and English (Design Plan D-9-8“The display languages are Japanese, Chinese, and English, with others added on request”)|—|
|DD-11-9|Licenses of third-party components are listed, collected automatically per release, and bundled (cargo-about for Rust, npm dependencies for screens)|Design Plan Chapter 17 “Third-Party Rights and Licenses”, R-9-5-1|A manual list|
|DD-11-10|The screen framework is Vite, TypeScript, and Svelte 5|The screen is one window and state management is simple. Svelte converts screen components to JavaScript at build time, and the runtime is small|React (many components; heavy for this scale). Vue (comparable; no deciding factor). Fitting a commissioned designer (there is no designer to commission; Chapter 10, DD-10-2“Design is done and judged by NRSD's Lead Developer”)|
|DD-11-11|Dependencies are limited to permitted licenses and official registries (crates.io, npm), and checked every time in CI with cargo-deny and npm audit|Keeps room for NRSD's licensing decision (no GPL family). Limiting sources prevents tampered components from getting in|Checking by hand|
|DD-11-12|For each release, an SBOM (CycloneDX) accompanies the distributables, the dependency list is embedded in executables (cargo-auditable), and provenance attestations (GitHub artifact attestations) are attached|When a vulnerability is published, anyone can check which versions contain it. It can be confirmed that distributables were built from the public source|Not making an SBOM|
|DD-11-13|The update signing key and the reference-information signing key are not placed in CI. Only code signing credentials are placed in CI secrets. Of the TUF role keys, root and targets are in the same location as these two keys (offline); snapshot and timestamp, as keys that cannot forge content, are placed in a CI environment `tuf-online` separate from the release environment (usable only by the periodic run and the snapshot update after a release), and the re-signing of timestamp and snapshot (every 3 days; half of the 7-day expiry; Chapter 9, 4.3) and their placement in `tuf/` on the public page and the mirror are done by that periodic run (the division recommended by TUF specification 5.1 “Key management and migration”: timestamp and snapshot as online keys, root and targets as offline keys. The same as the TUF operation of PyPI (Warehouse) and Sigstore. Chapter 9, 4.3)|Even if CI is taken over, fake updates and reference information cannot be distributed (threat T-8“Stealing NRSD's keys” of Chapter 1, 18.3)|Doing all signing in CI|
|DD-11-14|The ONNX runtime uses ort's default acquisition (prebuilt components distributed by pyke; ort-sys holds a list of URLs and SHA-256 per version, with GitHub attestations). The SHA-256 of the obtained components is kept in the release record and SBOM, and the minimum OS version of the bundled components (`minos` on macOS) is checked in CI. At every start the app checks the SHA-256 of the bundled ONNX runtime and models against embedded values, and if they do not match it stops as an anomaly detection (2026-09-30, NRSD's requirement)|Building the ONNX runtime from source is a lot of effort on three OSes. pyke's distribution can be checked by per-version hashes and attestations (ort's “Prebuilt binaries” guide)|Building the ONNX runtime from source every time (build time and effort; kept as a preparation for when maintenance stops; 12)|
|DD-11-15|c2pa-rs is used with default features off, using only the Rust cryptography implementation (`rust_native_crypto`) and the reqwest implementation for communication. OpenSSL is not brought in|c2pa-rs's defaults include OpenSSL and several HTTP implementations (c2pa-rs's Cargo.toml). Communication is aligned on reqwest and rustls (DD-11-7“Networking uses reqwest and rustls, with limited destinations and caps”), avoiding building C components and OpenSSL differences per OS|Using the default features as is (requires bundling OpenSSL)|

## 2. Basis for Selection (Conditions of Design Plan 6.2)

|Condition (Design Plan 6.2 “Development Base”)|How Tauri 2 meets it|
|---|---|
|Runs on three OSes|Runs on Windows (WebView2), macOS (WKWebView), and Linux (webkit2gtk 4.1) (Tauri's official prerequisites guide)|
|The look can be made freely|Screens are made with HTML and CSS (Chapter 10)|
|Image editing is easy to handle|Drawing is done in Rust, and screens only handle display (Chapter 1, DD-1-8“The screen handles only display and input; processing happens in the Rust core”, Chapter 4, DD-4-5“Text shaping and rendering use the same Rust mechanism for preview and export”)|
|Can handle C2PA signatures, invisible watermarks, and matching hashes|c2pa-rs, trustmark, and pdqhash are used directly from Rust|
|Has or can incorporate a self-update mechanism|Tauri's update mechanism (verification of update signatures cannot be disabled; Chapter 9)|

- The comparison of distributable sizes (Electron bundles Chromium and Node.js at 80 to 150 MB, Tauri under 10 MB) comes from third-party articles and is not an official figure. The size of the System's distributables is determined by fonts and models (4.3 “Assets”).

## 3. Development Environment and Versions

|Item|Version / content|
|---|---|
|Rust|The stable release, pinned to a specific version in `rust-toolchain.toml` and raised every 3 months (initial value). The c2pa crate requires Rust 1.96.0 or later (c2pa-rs README). trustmark requires 1.88 or later. MSVC Rust on Windows|
|Windows build prerequisites|Microsoft C++ Build Tools, WebView2 (Tauri's official prerequisites)|
|macOS build prerequisites|Xcode|
|Linux build prerequisites|webkit2gtk 4.1 and development packages. Built on Ubuntu 22.04 (to make glibc 2.35 the minimum; 3.1)|
|Screens|Node.js long-term support version, Vite, TypeScript, Svelte 5. Dependencies pinned with package-lock|

### 3.1 Minimum supported OS versions

|OS|Minimum|CPU|Basis|
|---|---|---|---|
|Windows|Windows 10|x86_64 supporting x86-64-v3 (with AVX2 and so on: for Intel, Core and the like from Haswell (2013), though some Pentium and Celeron after 2013 lack it; for AMD, Excavator and later)|Support of TrustMark's Rust version (Research Materials “Technical Elements of Signing”, Section 2). ort's prebuilt x86_64 components require x86-64-v3 at minimum (ort 2.0.0-rc.11 guide)|
|macOS|macOS 14 or later (due to `minos` of the bundled ONNX runtime; the ONNX runtime 1.27.1 as of 2026-09 is 14.0)|Apple CPUs only|The ONNX runtime stopped distributing for Intel macOS from 1.24. ort dropped Intel macOS in 2.0.0-rc.11 and raised the macOS minimum to 13.4. The components of ONNX runtime 1.27.1 are built with macOS 14.0 as the minimum (ort's guide, onnxruntime issues, other users' reports)|
|Linux|glibc 2.35 or later (Ubuntu 22.04 or later, Debian 12 or later), libstdc++ 12 or later|x86_64 supporting x86-64-v3|TrustMark's Rust version README, ort's guide|

- The README of TrustMark's Rust version says macOS 10.15 or later, which conflicts with the guide of the ort it depends on (raised the macOS minimum to 13.4 and dropped Intel macOS). The guide of the depended-on component is adopted.
- Macs with Intel CPUs are not supported (14 “Gaps Declared in This Chapter”). Staying on the ONNX runtime 1.23 series might work on Intel Macs, but that series receives no new fixes, so it is not adopted.
- [To be measured] That the app does not crash at start on PCs whose CPUs do not support x86-64-v3. Before crashing, the CPU support is checked and “The CPU of this PC is not supported” is shown (CPU features are examined at start).

## 4. List of Components

### 4.1 Rust components

|Component|Version (as of 2026-09-29)|License|Role|Chapter|
|---|---|---|---|---|
|tauri|2 series|MIT OR Apache-2.0|App framework|Chapter 1|
|tauri-plugin-single-instance|2 series|MIT OR Apache-2.0|Single instance|Chapter 1, 9.3 “Single instance”|
|c2pa|0.91.1 (2026-09-27). Default features off, using `rust_native_crypto`, `file_io`, `http_reqwest` (DD-11-15“c2pa-rs does not bring in OpenSSL”)|MIT OR Apache-2.0|C2PA manifests, C2PA signatures|Chapter 3|
|trustmark, ort|trustmark per 4.4 “Taking in TrustMark”. ort is the latest version on crates.io (2.0.0-rc.13 as of 2026-09-30)|trustmark MIT. ort MIT OR Apache-2.0. ONNX runtime MIT|Invisible watermark, running the auto-placement model|Chapter 3, Chapter 4|
|tauri-plugin-updater|2.12.0 or later (the first version with `requireSignedVersion`; confirmed in the crates.io distribution)|MIT OR Apache-2.0|Self-update (Chapter 9, 4 “Updates”)|Chapter 9|
|tauri-plugin-process|2 series|MIT OR Apache-2.0|Restart after updating (“Restart now to update”; Chapter 9, 4.1 “Flow and states”)|Chapter 9|
|tauri-cli (development tool)|2.11.5 or later (the first version with `tauri signer sign --app-version`; confirmed in the crates.io distribution)|MIT OR Apache-2.0|Build, update signatures (operator's PC)|Chapter 9, 6.1 “Procedure”|
|pdqhash|Latest stable|Apache-2.0|Matching hash|Chapter 3|
|image|0.25.10|MIT OR Apache-2.0|Reading and writing images (except JPEG export)|Chapter 3, Chapter 4|
|jpeg-encoder|0.7.1|(MIT OR Apache-2.0) AND IJG|JPEG export|Chapter 4, 14.3 “Resizing, color conversion, and JPEG export”|
|fast_image_resize|6.1.0|MIT OR Apache-2.0|Resizing|Chapter 4, 14.3 “Resizing, color conversion, and JPEG export”|
|moxcms (candidate), lcms2 (alternative)|0.9.1, 6.2.0|BSD-3-Clause OR Apache-2.0, MIT|Color conversion (decided by measurement and switched by a build feature; Overall Implementation Design 2.1)|Chapter 4, 14.3 “Resizing, color conversion, and JPEG export”|
|harfrust|0.13.3|MIT|Text shaping (including vertical)|Chapter 4, 3 “Text Layers”|
|skrifa|0.47.0|MIT OR Apache-2.0|Glyph outlines, color glyphs|Chapter 4, 3 “Text Layers”|
|tiny-skia|0.12.0|BSD-3-Clause|Fill, outline, transformation|Chapter 4, 3.5 “Drawing”|
|keyring|4.2.0 (windows-native-keyring-store 1.1.0 on Windows)|MIT OR Apache-2.0|OS keystore|Chapter 2, 7.5 “Storage method per OS”|
|p256, sha2|Latest stable|MIT OR Apache-2.0|ECDSA P-256, SHA-256|Chapter 2, Chapter 3|
|rcgen|Latest stable|MIT OR Apache-2.0|Creating certificates|Chapter 2, 3 “Certificates”|
|age|0.12.1|MIT OR Apache-2.0|Encrypting backup files (passphrase, scrypt), key storage on Linux without Secret Service|Chapter 8, 5 “Backup Files”, Chapter 2, 7.5 “Storage method per OS”|
|zeroize|Latest stable|MIT OR Apache-2.0|Erasing keys from memory after use|Chapter 2, 7.6 “Handling during use”|
|serde, serde_json|Latest stable|MIT OR Apache-2.0|Reading and writing records|Chapter 1, 8.3 “Formats and versions”|
|json-patch|Latest stable|MIT OR Apache-2.0|Undo differences (RFC 6902)|Chapter 4, 10.3 “Contents of the history”|
|uuid|Latest stable|MIT OR Apache-2.0|Numbers (UUID v7)|Chapter 1, 8.2 “Identifier scheme”|
|reqwest (rustls)|Latest stable|MIT OR Apache-2.0|Communication|Chapter 1, 4.2 “Communication with the outside”|
|hickory-resolver|Latest stable|MIT OR Apache-2.0|Name resolution (DNS queries; raw responses are kept as evidence)|Chapter 1, 4.2; Chapter 6, 5|
|zip|Latest stable|MIT|Template files, backup files|Chapter 4, 8.3 “Passing on (export and import)”, Chapter 8, 5 “Backup Files”|
|warc|0.4.0|MIT|Saving fetches of reposted pages (WARC)|Chapter 6, 2.2.1 “Saving in WARC and WACZ”|
|img-parts|0.4.0|MIT OR Apache-2.0|Writing XMP and EXIF segments of JPEG, PNG, WebP|Chapter 3, 3.6 “The file's rights statement (IPTC photo metadata)”|
|little_exif|0.6.23|MIT OR Apache-2.0|Creating EXIF Artist and Copyright|Chapter 3, 3.6 “The file's rights statement (IPTC photo metadata)”|
|(In-house) Creation of RFC 3161 requests and verification of responses|—|—|Timestamps (Chapter 3, 4). Uses RustCrypto's der, x509-cert, and cms, not depending on the maintenance of an external component (2026-09-30, NRSD's requirement)|Chapter 3, Chapter 6|
|serde_json_canonicalizer|0.3.2|MIT|JCS (RFC 8785). Its output is compared against our own minimal implementation (Chapter 1, 8.3)|Chapter 1|
|icann-rdap-client|1.0.0|MIT OR Apache-2.0|RDAP queries (Chapter 6, 5). Bootstrap from the reference information|Chapter 6|
|tauri-plugin-dialog, tauri-plugin-opener, tauri-plugin-notification|2.8.0, 2.7.0, 2.5.0|Apache-2.0 OR MIT|File selection, opening the default browser, mailto, and the log location, OS notifications. Official plugins only, with permissions minimal per screen and per command (Chapter 1, 6.2)|Chapter 1, Chapter 10|
|tauri-plugin-deep-link|2 series|Apache-2.0 OR MIT|Receiving files from the browser extension “Save evidence” (the URL scheme registered with the OS; Chapter 6, 2.2.2)|Chapter 6|
|yubikey|0.8.0|BSD-2-Clause|PIV signing in the operator tool (Chapter 8, 3.4)|Chapter 1, 19|
|zxcvbn|3.1.1|MIT|Estimating the strength of backup passphrases (Chapter 8, 5.2)|Chapter 8|
|iroh|1.3.0|MIT OR Apache-2.0|Direct connections between devices and with verified counterparties (QUIC addressed by public key; NAT hole punching via public relays; adopted by Delta Chat; Chapter 8, 5.4). Early in implementation, reachability from the three OSes and from mainland China is checked|Chapter 8|
|qrcode, rqrr (candidate)|Latest stable|MIT OR Apache-2.0, Apache-2.0|Generating and reading QR codes (Chapter 1, 10.10 “Forms of URLs and QR codes”)|Chapter 1|
|zstd|Latest stable|MIT|Compression of reference information (Chapter 8, 3.4)|Chapter 8|
|trash|5.2.9|MIT|Moving the evidence of withdrawn cases to the OS trash (Chapter 6, 7)|Chapter 6|
|tough|0.24.0|MIT OR Apache-2.0|Verification of TUF metadata (Chapter 9, 4.3, Chapter 8, 3.4)|Chapter 9, Chapter 8|
|josekit (candidate)|Latest stable|MIT|Creating and verifying JWS (RFC 7515) (Chapter 1, 8.3). The JAdES headers are added in-house|Chapter 1|
|unicode-security|Latest stable|MIT OR Apache-2.0|UTS #39 skeleton and mixed-script detection (Chapter 2, 2.3, 4.1)|Chapter 2|
|url, idna|Latest stable|MIT OR Apache-2.0|WHATWG URL Standard and UTS #46 (Chapter 1, 10.10 “Forms of URLs and QR codes”)|Chapter 1|
|rustls-platform-verifier|Latest stable|MIT OR Apache-2.0|Delegating the TLS trust anchors to the OS verifier (Chapter 1, 4.2)|Chapter 1|
|sysproxy|Latest stable|MIT|Reading the OS proxy settings (Chapter 1, 4.2)|Chapter 1|
|fd-lock|Latest stable|MIT OR Apache-2.0|In-use mark of a folder (OS file lock; Chapter 1, 10.2 “Storage”)|Chapter 1|
|unicode-segmentation, unicode-bidi|Latest stable|MIT OR Apache-2.0|Grapheme segmentation (UAX #29) and bidirectional text (UAX #9) (Chapter 4, 3.5)|Chapter 4|
|html5ever|Latest stable|MIT OR Apache-2.0|Parsing the HTML of reposted pages (Chapter 6, 2.2)|Chapter 6|
|iroh-blobs|Latest stable|MIT OR Apache-2.0|Transfer of large files between devices (Chapter 8, 5.4)|Chapter 8|
|pkarr (iroh's `discovery-pkarr-dht`)|Latest stable|MIT OR Apache-2.0|For connections by passphrase, publishing and looking up the device number in the public DHT for 10 minutes only (Chapter 8, 5.4)|Chapter 8|
|spake2|Latest stable|MIT OR Apache-2.0|Key agreement by passphrase (SPAKE2; RFC 9382; Chapter 8, 5.4)|Chapter 8|
|argon2|Latest stable|MIT OR Apache-2.0|Deriving the seed of the DHT key pair from the passphrase (Argon2id; Chapter 8, 5.4). Not used for signing key storage (age; Chapter 2, 7.5)|Chapter 8|
|tauri-plugin-window-state|2 series|Apache-2.0 OR MIT|Saving window size and position (Chapter 10, 10)|Chapter 10|
|krilla (PDF generation; export with validation for PDF/A-2u and 3u and PDF/UA-1, tagged PDF, embedded files; Chapter 1, 10.11 “PDF documents”; also checked with veraPDF in CI)|0.8.2|MIT OR Apache-2.0|Emergency kit, reports (Chapter 8, 5.1, Chapter 6, 3.4)|Chapter 8, Chapter 6|
|XLIFF 2.1 conversion (a CI tool; candidate: translate-toolkit)|—|GPL-2.0 (used only as a CI tool, not included in distributables)|Exchange of text translations (Chapter 10, 5)|Chapter 10|
|iscc-lib|0.6.0|Apache-2.0|Computing ISCC (ISO 24138:2024) (Chapter 3, 6)|Chapter 3|
|(In-house) BagIt and WACZ export and RFC 4998 ERS|—|—|The evidence package bag, the WACZ package and its signature, creating and renewing EvidenceRecords (Chapter 6, 2.2.1, 3.2, 3.4). Uses RustCrypto's der and cms|Chapter 6|
|objc2-local-authentication (macOS; the LocalAuthentication framework. `security-framework` wraps Security.framework and has no LAContext), windows (Windows)|Latest stable, 0.62.2|MIT, MIT OR Apache-2.0|OS user verification (LAContext, UserConsentVerifier; Chapter 2, 7.4). Linux calls polkit over D-Bus (components revised on 2026-10-01)|Chapter 2|
|minidumper (uses rust-minidump's minidump-writer inside)|Latest stable|MIT OR Apache-2.0|Minidumps on crash (the crash report of Chapter 1, 10.3 “Errors”)|Chapter 1|
|The Wayland color management protocol and reading X11's `_ICC_PROFILE` (candidates: wayland-client, x11rb)|Latest stable|MIT|Display ICC on Linux (Chapter 4, 14.3 “Resizing, color conversion, and JPEG export”)|Chapter 4|
|thiserror, tracing, tracing-appender|Latest stable|MIT OR Apache-2.0|Error types, logging|Chapter 1, 10.3 “Errors”, 10.4 “Operation log”|
|ciborium|Latest stable|Apache-2.0|Messages of device-to-device sync (CBOR; Chapter 8, 5.4)|Chapter 8|
|jsonschema|Latest stable|MIT|Validation of records against JSON Schema (Chapter 1, 8.3)|Chapter 1|
|semver|Latest stable|MIT OR Apache-2.0|Version comparison (the minimum version, the pre-release of the preview version; Chapter 9, 4.2)|Chapter 9|
|(In-house) Export of iCalendar (RFC 5545) VTODO and VALARM|—|—|The `.ics` of procedural deadlines (Chapter 7, 6.3). No reading is done, so no component is brought in|Chapter 7|

- For “latest stable”, the version is decided at the start of implementation and pinned in Cargo.lock.
- Versions stated explicitly are those confirmed in the research for Chapter 4.

### 4.2 Screen components

|Component|License|Role|
|---|---|---|
|Svelte 5 (5.57.1), Vite (8.3.1), TypeScript (7.0.2)|MIT, MIT, Apache-2.0|Screen framework|
|intl-messageformat (FormatJS, 12.1.2)|BSD-3-Clause|Formatting ICU MessageFormat texts (screen)|
|formatjs_icu_messageformat (Rust; 0.1.3, 2026-09-12; depends on icu 2.1)|BSD-3-Clause|Composes the fixed texts composed by the core (the `core.*` keys of Chapter 10, 5) from the same message files. The Rust port of FormatJS. [To be measured] that plural, select, and date formatting give the same output as the screen's intl-messageformat (CI composes all texts in three languages with both and compares)|
|@tauri-apps/api (2.12.0)|Apache-2.0 OR MIT|Calling core commands|

- Versions and licenses are as listed for the latest versions in the npm registry (2026-09-30). At the start of implementation, versions are decided and pinned with package-lock, and checked every time by CI's license check (DD-11-11“Dependencies are limited to permitted licenses and official registries”).

### 4.3 Assets

|Asset|License|Size|Chapter|
|---|---|---|---|
|Caveat, Klee One, Yomogi, LXGW WenKai Lite, Noto Color Emoji (COLRv1)|SIL OFL 1.1|About 52 MB (including Noto Color Emoji 25.3 MB and LXGW WenKai Lite 13.9 MB)|Chapter 4, 6.1 “Bundled fonts”, 6.3 “Emoji”|
|Zen Maru Gothic, Resource Han Rounded CN (screen text)|SIL OFL 1.1|About 18 MB|Chapter 10, 2.2 “Fonts”|
|TrustMark model (Q)|MIT|About 65 MB (`encoder_Q.onnx` for embedding 17.3 MB, `decoder_Q.onnx` for reading 47.4 MB; response sizes from Adobe's distribution source, 2026-09-30). trustmark's fetch command (`cargo xtask fetch-models`) fetches from Adobe's distribution source (cai-watermark.adobe.net) at fixed URLs and does not check hashes, so the SHA-256 of the fetched models is recorded and they are placed in `assets/models/` in the repository, and the repository's copies are used thereafter|Chapter 3|
|u2netp (auto-placement model)|Apache-2.0|About 4.6 MB (ONNX)|Chapter 4, 7.2 “Auto-placement”|
|ONNX runtime (shared library)|MIT|About 14 MB (reported for the 1.18 Linux version; varies by version and build)|—|

### 4.4 Taking in TrustMark

- Facts (checked 2026-09-30): the latest trustmark on crates.io is 0.2.2 (2025-09-17), which pins ort to 2.0.0-rc.8 (ONNX runtime 1.19.2). Of the download locations of rc.8's prebuilt components (1.19.2 at parcel.pyke.io), those for Windows (x86_64, aarch64) and Intel macOS do not respond (404). Those for Linux and Apple-CPU macOS can be obtained. Windows is a target OS, so 0.2.2 cannot be used as is. The upstream changelog also says, in the 0.3.0 entry, “fix for 0.2.2 no longer building (the prebuilt components fetched by ort-sys 2.0.0-rc.8 are no longer hosted)” (rust/CHANGELOG.md of adobe/trustmark). 0.3.0 exists only on GitHub's main (rust/Cargo.toml).
- Decision: if 0.3.0 or later is on crates.io when implementation begins, it is adopted. If not, the necessary parts of TrustMark's Rust implementation (MIT; rust/ of adobe/trustmark) (preprocessing, inference calls, BCH codes) are taken into the System's repository with the source commit hash and MIT notice, and run with the latest ort on crates.io. The taken-in source undergoes CI checks and review as the System's own source. Direct git references are not used (5.2).
- Reason: 0.2.2 has no download location for its prebuilt components, and would require building the 2024 ONNX runtime ourselves. Taking it in is allowed by MIT and keeps the intent of the source policy (5.2) (not bringing in unreviewed source). When it is published on crates.io, the taken-in source is replaced with the crates.io version.
- The minimum supported OS (3.1) follows the latest ort's prebuilt components in either case.

- The source, version, hash, and license of assets are placed in `assets/manifest.json` (6).
- Sources: font sizes are measurements from the research for Chapter 4. The size of the ONNX runtime is from a report in microsoft/onnxruntime issues (14 MB for version 1.18).

## 5. Dependency Policy

### 5.1 Permitted licenses

|Permission|Conditions|
|---|---|
|MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC, Zlib, Unicode-3.0, IJG|Permitted as is|
|OFL-1.1|Fonts only|
|MPL-2.0|Permitted individually after checking the per-file disclosure obligation|
|GPL, LGPL, AGPL|Not permitted (to keep room for NRSD's licensing decision; 9). Exception: OS-derived shared components bundled by the Linux AppImage (WebKitGTK, GTK, GLib, etc.; LGPL) are bundled with dynamic linking kept, following the terms of LGPL-2.1: ① bundle the license text and a notice that the components are used (Section 6); ② keep them in a form the User can replace (as shared libraries) (Section 6(b)); ③ make the corresponding source of the bundled components (the source packages of the distributor, Ubuntu 22.04) equally obtainable from the same place as the distributables (GitHub releases and the Hong Kong mirror) (Section 4, Section 6(d); because distributables are obtained from designated places). They are not combined with the System's source|

- The table above applies to components that go into distributables (Rust and screen dependencies, bundled assets). CI and development tools that do not go into distributables (translate-toolkit for XLIFF conversion (GPL-2.0), veraPDF, reuse, etc.) are outside its scope and are held in 4.1 marked explicitly as “CI tool”.
- If a component outside the table above enters in cargo-deny's license check, CI stops. Components bundled in the AppImage are outside cargo-deny's scope, and at release the list and licenses of the shared components inside the AppImage are put in the SBOM and the list of licenses (Chapter 9, 6.2 “Bundled license notices”).

### 5.2 Sources and prohibitions

- Sources are limited to the official registries crates.io and npm. Direct git references and obtaining from personal distribution locations are prohibited (cargo-deny's source check).
- Exceptions (obtained from outside component registries; the SHA-256 of what is obtained is recorded for each): ① prebuilt components of the ONNX runtime (pyke's distribution; ort-sys holds URLs and SHA-256 per version; DD-11-14“Hashes and minimum versions of ONNX Runtime components are checked”); ② TrustMark models (Adobe's distribution source; the fetch command does not check hashes, so the SHA-256 is recorded and they are placed in the repository; 4.3); ③ the AppImage runtime and linuxdeploy plugins (obtained by Tauri's build components; the version cannot be pinned, so the SHA-256 of what is obtained is recorded; Chapter 9, DD-9-8“The hash of the AppImage runtime is kept”); ④ taking in TrustMark's Rust implementation (4.4; taken in as source).
- Multiple versions of the same component are only warned about (sometimes unavoidable due to component dependencies).

### 5.3 Vulnerabilities

- In CI, cargo-deny (vulnerability advisories) and npm audit run on every change.
- If a component with a critical vulnerability (CVSS 7 or more; initial value) is included, the release is stopped. Lower ones are fixed at the next periodic update.
- On receiving notice of a published vulnerability, the impact is checked with the distributables' SBOM (6), and if affected, a fixed version is released.
- Whether there is an impact is published on the public page as a VEX of OASIS CSAF 2.0 (or CycloneDX's VEX; not_affected, affected, fixed, under_investigation, with the reason), linked to the SBOM, put into the reference information package, and notified in G-21“Notifications” (2026-10-01, NRSD's requirement).

### 5.4 Updates

- Updated once a month (initial value) and upon vulnerability notices. Updates are done one at a time, checking the build and main behavior.
- Drawing, color, and watermark components (harfrust, tiny-skia, moxcms, trustmark) may change output, so test image outputs are compared before and after updating (Chapter 4, 3.5 “Drawing”).
- Risk: components such as HarfRust, krilla, and moxcms are young. Impact: breakage when versions are raised. Provision: pin the versions and test when raising them (Chapter 11)

## 6. SBOM and Provenance Attestations

- SBOM: for each release, an SBOM in CycloneDX format is made and accompanies the distributables. Rust uses cargo-cyclonedx (made from Cargo.lock and cargo metadata), and screens are made from npm dependencies. CycloneDX 1.6 is an Ecma standard.
- The taken-in TrustMark source (4.4 “Taking in TrustMark”) does not appear in Cargo.lock, so it appears in none of cargo-cyclonedx's SBOM, cargo-auditable, or cargo audit. It is listed in the asset manifest (below) and added to the SBOM automatically (name, source commit hash, MIT), and NRSD checks upstream (adobe/trustmark) advisories and changes quarterly.
- Asset manifest: the source, version, hash, and license of the taken-in source (TrustMark) and of the bundled assets (fonts, models, the passphrase list, the ONNX runtime components, the AppImage runtime) are placed in one manifest in the repository (`assets/manifest.json`), from which CI adds them automatically to the SBOM and the list of licenses. The SBOM is not touched by hand (2026-09-30, NRSD's requirement).
- The repository conforms to the REUSE Specification 3.3 (FSFE): `SPDX-License-Identifier` and `SPDX-FileCopyrightText` headers in every file, images, fonts, and models declared in `REUSE.toml`, license texts under SPDX names in `LICENSES/`, and `reuse lint` in CI. The asset manifest is integrated into `REUSE.toml` (2026-10-01, NRSD's requirement).
- Embedding in executables: cargo-auditable embeds the dependency list in executables. Even after distribution, dependencies can be read from executables with cargo audit, trivy, and the like.
- Provenance attestations: GitHub artifact attestations attach proof of which source of the Public Repository and which CI built the distributables. Users and experts can check with `gh attestation verify` (the procedure is in the README; Chapter 9).
- Converting u2netp: the procedure for PyTorch → ONNX conversion (`torch.onnx.export`, a fixed opset, the source commit) is placed in `tools/models/`, and the SHA-256 and source of the converted model are listed in the asset manifest. The conversion is done once by the Lead Developer, and CI checks only the bundled hash.

## 7. Repository Structure

|Place|Content|
|---|---|
|`crates/`|Rust components (7.1)|
|`apps/client/`|The Users' app (Tauri)|
|`apps/operator/`|Operator tool (Tauri; Chapter 1, 19 “Operator Tool”)|
|`ui/`|Screens (Svelte)|
|`i18n/`|Text files (Japanese, Chinese, English)|
|`assets/fonts/`|Bundled fonts and license documents|
|`assets/models/`|TrustMark and u2netp models|
|`reference/`|Drafts of reference information (`reference/src/`; Chapter 1, 19). The package and the TUF metadata are made by CI from the drafts and placed in `reference/` and `tuf/` on the public page (Chapter 8, 3.4, Chapter 9, 4.3)|
|`extension/`|The browser extension “Save evidence” (Chapter 6, 2.2.2; versioned separately from the app)|
|`docs/`|Design documents (docx)|
|`.github/workflows/`|CI|
|`tools/`|Operating tools (copying to and comparing with the mirror, release helpers, generation of the reference information package and the public page `tools/refbuild`; Chapter 9, 6.1 “Procedure”, Chapter 8, 3.3, 3.6)|
|`dev/`|Developer-side documents (the Overall Implementation Design, the table of bridges, the types and permission lists generated from the list of boundaries, JSON Schemas and test vectors `dev/schemas/`, the error ledger `dev/errors.yaml` (Chapter 1, 10.3), the extension–app compatibility table `dev/compat.yaml` (Chapter 6, 2.2.2))|

### 7.1 How the Rust components are divided

- Revised 2026-10-01: in line with the division of the Overall Implementation Design (dev/docs/jp/全体実装設計書.md; English in dev/docs/en, Chinese in dev/docs/cn), there are 18 crates. The specification sections each crate takes charge of are in “Sections taken” of the block's page, the boundaries (bridges) between crates are in the “Bridges” table of the page, and the table in Overall Implementation Design 2 is authoritative for the dependency edges.

|Component|Role|Main external components|
|---|---|---|
|core-common|Error types and codes (normal paths and anomaly detection; Chapter 1, 10.3 “Errors”), logging and crash reports, common values, Base32 and CRC-4, the character sequences of identification numbers and notice codes, time and skew, URL normalization (WHATWG URL, UTS #46), QR encoding and decoding, posting site detection (the table is passed by the caller), fixed texts composed by the core|thiserror, tracing, tracing-appender, url, idna, unicode-security (mixed-script host names), qrcode, rqrr, minidumper, zeroize (`Secret<T>`), formatjs_icu_messageformat|
|core-store|Saving records (temporary file and replacement; Chapter 1, 10.2 “Storage”), record signatures (JWS, JAdES, JCS, chains), JSON Schema validation, migration of format versions, settings, locations, safe reading and writing of ZIP, the running mark, the index of work data, retention and automatic deletion|serde, serde_json, serde_json_canonicalizer, jsonschema, josekit, p256, zip|
|core-net|Common communication (timeouts, size limits, redirect limits; Chapter 1, 4.2 “Communication with the outside”), the order of sources, HTTP for RFC 3161, DNS queries (with raw responses), detection of line recovery and metered connections, the queue for background jobs (Chapter 1, 10.5)|reqwest (rustls), rustls-platform-verifier, hickory-resolver, windows (INetworkListManager, GetConnectionCost), the objc2 Network framework (NWPathMonitor; the wrapper crate is unverified), zbus (NetworkManager's D-Bus)|
|core-hash|SHA-256, PDQ, ISCC|sha2, pdqhash, iscc-lib|
|core-image|Reading and writing images, orientation normalization, resizing, color conversion, encoding, removing shooting information, rights statements in XMP and EXIF, reduced images, display ICC on Linux|image, fast_image_resize, moxcms, jpeg-encoder, img-parts, little_exif, wayland-client, x11rb|
|core-identity|Input validation (UTS #39), keys, the personal root and signing certificate, notice code, keystore and OS user verification, past personal roots, authorizations, revocations, and joint-rights documents|rcgen, keyring, p256, sha2, age, zeroize, unicode-security, objc2-local-authentication, windows|
|core-mark|Embedding and reading watermarks, loading and sharing the ONNX runtime (only the model in use is loaded, managed under a memory limit; 2026-09-30, NRSD's requirement)|trustmark, ort|
|core-render|Fonts, text shaping (including vertical), drawing, shadow blur, placement, compositing, PDF/A generation|harfrust, skrifa, tiny-skia, krilla|
|core-rights|Permitted Scope, Enclosed Documents (TXT, ODRL), corrected versions, sample Notice texts, rights statement texts (Chapter 5)|serde_json|
|core-work|Templates, work sessions, undo history, auto-placement (ONNX execution uses core-mark), export presets, rules for merging with backups and other devices|json-patch|
|core-sign|Verification of existing C2PA signatures, creating C2PA manifests and C2PA signatures, timestamps (in-house RFC 3161) and ERS, the export flow, work data, read-back verification|c2pa, der, x509-cert, cms|
|core-case|Registration, fetching evidence (fetching without running scripts), WARC and WACZ, matching, RDAP and DNS, case records, evidence packages (BagIt), procedural deadlines (VTODO), trash|warc, icann-rdap-client (interpreting RDAP responses only; communication is in core-net), trash|
|core-guidance|Contact points, the standing of the complainant, filling in model texts, procedural deadlines, references to each country's law (Chapter 7)|—|
|core-ref|Obtaining the reference information package and the update manifest and TUF verification, definition of the package format, consent state, the minimum version|tough, zstd, semver|
|core-backup|Creating and importing backup files, merging, the emergency kit, opening as a handover|age, zxcvbn|
|core-sync|Direct connections between devices and with verified counterparties and their verification (safety numbers), aligning records, exchange of documents and reference information (Chapter 8, 5.4, 3.4, Chapter 2, 5.5; 2026-09-30, NRSD's requirement)|iroh, iroh-blobs, pkarr, spake2, argon2, ciborium|
|app-client|The Users' app (definition and permission of commands; Chapter 1, 6.2 “Between the screen and the core”), start-up order and states, update replacement, notifications, drag and drop, the receiving point for the extension|tauri, tauri-plugin-single-instance, updater, process, dialog, opener, notification, deep-link, all of the above|
|app-operator|Operator tool (M-01“Reference Information”, M-02“Release Signing”)|tauri, core-store, core-ref, yubikey, tauri-cli|

- Dependencies between components are one-way, with core-common, core-store, and core-net at the lowest layer. There are four dependency inversions (a lower crate defines a trait, and an upper crate implements and passes it): `RecordSigner` (defined by core-store, implemented by core-identity), `BundleSource` (defined by core-ref, implemented by core-sync), `BundleSigner` (defined by core-ref, implemented by app-operator), and screen events (emitted by app-client and subscribed to by the screens). Only the two apps depend on tauri (separating the core components from the screen framework so they can be tested alone).

## 8. Build and Continuous Integration (CI)

### 8.1 On every change

|Step|Content|
|---|---|
|Formatting|Check rustfmt and prettier (stop if different)|
|Static checks|clippy (warnings treated as errors), eslint, TypeScript type checks|
|Dependency checks|cargo-deny (vulnerabilities, bans, licenses, sources), npm audit|
|Pinning Actions|Third-party Actions are specified by full-length commit hash rather than version name (GitHub's “Security hardening for GitHub Actions”: pinning to a commit hash is the only way to use an action as an immutable release)|
|Build|Windows, macOS, Linux|
|Reproducible builds|Fixing `SOURCE_DATE_EPOCH` and matching the SHA-256 of two builds (Chapter 9, 3.2). `reuse lint` (9)|
|Repository health|Run OpenSSF Scorecard in CI and publish the score and items on the vulnerability policy page of the public page (branch protection, dependency pinning, signed releases, SECURITY.md, and other items) (2026-10-01, NRSD's requirement)|
|Output comparison|For changes that update drawing or color components, compare test image outputs (5.4)|
|Checking permissions and boundaries|Agreement between Tauri's capabilities and permissions and the permission list, agreement between the types generated from the list of boundaries (Chapter 1, 6.2) and the implementation, test vectors for RFC 8785, RFC 3161, and notice codes, agreement between the asset manifest (6) and the hashes of bundled items (2026-09-30, NRSD's requirement), automated readability checks (all screens; the axe-core family), the plain-language check of texts (Chapter 10, 2.7), XLIFF round-trip agreement, OpenACR YAML validation, agreement between the error ledger `dev/errors.yaml` and the types, the three-language texts, and failing tests (Chapter 1, 10.3), judgment by the test URLs of every row of `platforms.json` (Chapter 8, 3.4), the form of the extension–app compatibility table `dev/compat.yaml` (Chapter 6, 2.2.2), PDF/A and PDF/UA validation of PDF test outputs with veraPDF (Chapter 1, 10.11), and the consistency of the published TUF metadata (agreement between the hashes in `targets.json` and the release's distributables and packages, expiries) (added 2026-10-01)|

### 8.2 At release

|Step|Content|
|---|---|
|Fixing the version|Assign the version number (10) and write the changelog|
|Build|Distributables for the three OSes (Chapter 9, DD-9-1“One distribution per OS, code-signed”). macOS is Apple CPU only (3.1). Linux is built in an Ubuntu 22.04 environment. The Flatpak is not built in NRSD's CI; Flathub's CI builds it from the manifest (Chapter 9, 2.1, 6.1)|
|Checking bundled components|Record the SHA-256 of the ONNX runtime components (DD-11-14“Hashes and minimum versions of ONNX Runtime components are checked”). For macOS, check that `minos` of bundled dynamic libraries is at or below the app's minimum (`LSMinimumSystemVersion`), and stop if exceeded|
|AppImage runtime|Record the SHA-256 of the runtime (type2-runtime) fetched at build time and put it in the SBOM (Chapter 9, DD-9-8“The hash of the AppImage runtime is kept”)|
|Code signing|Artifact Signing for Windows, Developer ID and notarization for macOS|
|SBOM and attestations|SBOM, embedding the dependency list, provenance attestations (6)|
|List of licenses|Make and bundle the list of DD-11-9“The license list is bundled automatically”|
|Draft release|Create the GitHub release as a draft|
|Update signatures|CI sets `createUpdaterArtifacts` to false and does not make update distributables (if true, the update signing key is needed at build time). Making update distributables and update signatures is done outside CI, with the operator tool (Chapter 1, M-02“Release Signing”) (DD-11-13“The update signing key and the reference-information signing key are not kept in CI”, Chapter 9, 6.1 “Procedure”)|
|TUF|Signing the `targets.json` of the delegation `release` is outside CI (M-02; step 4 of Chapter 9, 6.1). Re-signing `snapshot.json` and `timestamp.json` and placing them in `tuf/` on the public page and the mirror is done by CI in the `tuf-online` environment (Chapter 9, 4.3, Chapter 8, 3.6)|
|Publication|Publish the draft and copy to the mirror (Chapter 8)|

### 8.3 Handling of secrets

- Where each secret is kept is per Chapter 1, 8.5 “List of secrets” (not reproduced in this chapter). What is placed in CI secrets is only the code signing credentials (Windows, macOS), the notarization credentials, the TUF snapshot and timestamp keys, and the key that can write only to `tuf/` on the mirror (Chapter 8, 3.6); the update signing key, the reference-information signing key, the TUF root and targets keys, and the key for writing distributables and packages to the mirror are not placed in CI (DD-11-13, Chapter 9, DD-9-4).
- CI secrets are set so they can be used only in the release CI environment (GitHub environment) that runs when a `v*` tag is created. Only the TUF snapshot and timestamp keys are placed in the separate environment `tuf-online` as described above. Creating tags is limited to owners (Chapter 8, 3.5 “Protection of the GitHub account and repository”).
- If CI secrets leak, or the CI or GitHub organization is taken over, fake distributables with valid code signatures may be built (threat T-22“Taking over CI or the GitHub organization to place correctly code-signed fake builds in official releases” of Chapter 8, 3.5, Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”).
- Secret: code signing credentials (Windows). Location: the signing permission of Artifact Signing (a credential holding the signer role of Azure Artifact Signing; CI secrets, limited to the release environment; Chapter 11, 8.3 “Handling of secrets”). The certificate itself is inside the service and is not handed to NRSD. If leaked: fake distributables with valid code signatures may be built. Revoke the credential and report to Artifact Signing's contact point (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”)
- Secret: code signing credentials (macOS). Location: the Developer ID Application certificate and private key (the .p12 and its passphrase), and the App Store Connect API key used for notarization. CI secrets (limited to the release environment). The original on the operator's management PC. If leaked: fake distributables with valid signatures and notarization may be built. Ask Apple to revoke the certificate (Chapter 13, H-44“If CI or the GitHub organization is taken over, correctly code-signed fake builds may be placed in official releases”)

## 9. NRSD's License (Decision on O-09“NRSD’s license type”)

- Decision (2026-09-30, NRSD): the license of the System's source code and program is Apache-2.0. All depended-on components are under the permitted licenses of 5.1 “Permitted licenses” (MIT, Apache-2.0, BSD, ISC, Zlib, Unicode-3.0, IJG, OFL, etc.) (with the exception of OS-derived LGPL shared components bundled by the Linux AppImage with dynamic linking kept), and are compatible.
- Intent: NRSD charges no usage fees for the System and does not sell it. It does not waive copyright or the license, but aims for the System to be used freely. The aim is to keep it open enough that sales venues (Patreon, etc.) could take in the System's matching means and design as they are. Distributing modified versions closed is also not prevented (Design Plan D-2-5“Sales management is not handled. Anyone who needs it shall fork and modify the System, and NRSD takes no part in the results”, D-12-2“The software is published, and its being taken away is not prevented”).

|Candidate compared|Taking out and modifying|Handling of names|Patents|Adopted or not|
|---|---|---|---|---|
|MIT|Not prevented. Modified versions can be distributed closed|No provision|No explicit grant|Not adopted (no provisions on names or patents)|
|Apache-2.0|Not prevented. Modified versions can be distributed closed. Changes must be stated|Section 6 states that use of trademarks (names, marks) is not permitted, which is one basis for restraining modified versions from calling themselves “official”|Explicit grant|Adopted|
|GPLv3, AGPL-3.0|Requires those who distribute modified versions to publish source. Modifications adding sales management cannot be distributed closed|No provision|Explicit grant|Not adopted (conflicts with D-2-5“Sales management is not handled. Anyone who needs it shall fork and modify the System, and NRSD takes no part in the results” and D-12-2“The software is published, and its being taken away is not prevented”)|

- Reasons: ① fits the Plan's policy of not preventing forks and modifications. ② Posting sites, verification sites, and other software can easily take the notice code computation and matching procedure into their products, increasing the places where matching is possible. ③ Section 6 explicitly states that use of names is not permitted (supplementing Chapter 9, DD-9-3“Official builds are identified by the code-signing publisher and SHA-256”; protection of names is by trademark law). ④ It is the same family as surrounding components such as c2pa-rs and Tauri.
- Notices: `LICENSE` (the Apache-2.0 text) and `NOTICE` (copyright notice: © 2026 Nagareyama Software Development Co., Ltd.) are placed at the top of the repository, and the same are included in distributables and the app (G-23“About This App”).
- Donations and support: not accepted (as of 2026-09-30). If accepted, because the arrangement that the legal action guidance of Chapter 7 does not constitute unauthorized practice of law presumes it is provided free of charge (Legal Research L-10“Handling of legal business (unauthorized practice of law) (Japan)”: “without receiving any benefit whatsoever, including usage fees”), they are accepted only after confirming with experts that donations do not touch this premise.
- Export control (Legal Research L-14“Export control of software containing cryptography (Japan, US)”): by using only public standard cryptography, publishing the source, and distributing free of charge, it is treated as a transaction not requiring permission in Japan and as not requiring notification in the United States. The conditions are checked at each release (Chapter 9, 6.3 “Export control of software containing cryptography”).

## 10. Versioning and Branch Operation

- The app version is “year.month.serial” (e.g., `2026.10.1`). This form fits the form of Semantic Versioning (major.minor.patch) and meets the requirement of Tauri's update mechanism (valid SemVer) (Chapter 9, 6.4 “Versioning”). Compatibility of record formats is expressed not by the app version but by the record format version (`schema`).
- The record format version (`schema`), the C2PA specification version, and the reference information version are held separately from the app version (Chapter 1, 10.9 “Version compatibility”).
- Direct pushes to the main branch are prohibited; changes go through pull requests and CI checks (Chapter 8, 3.5 “Protection of the GitHub account and repository”). While there is one developer, review is the developer's own check. Commits are signed.
- Releases are built by CI when a version tag is created.

## 11. Coding Conventions

|Item|Convention|
|---|---|
|Formatting|Defaults of rustfmt and prettier|
|Static checks|clippy warnings treated as errors. eslint recommended rules|
|Errors|Define error types with thiserror, carrying the categories and codes of Chapter 1, 10.3 “Errors”. Texts for Users are placed in text files|
|Logging|tracing. Keep the items not to be written (Chapter 1, 10.4 “Operation log”)|
|Dangerous processing|Rust unsafe is limited to within components, with the reason for use written alongside|
|Texts|Do not write text directly in screens; place it in text files (DD-11-8“Internationalization uses message files”)|
|Algorithms|Things whose method can be chosen, that are switched by measurement, or that hold thresholds are placed behind a trait, and the choice is made in one place in each crate (`algo.rs`). Do not scatter ifs (Overall Implementation Design 2.1 “Algorithm blocks”)|

## 12. Preparation for When a Component's Maintenance Stops

|Component|Replacement if it stops|
|---|---|
|c2pa-rs|It is CAI's public SDK, maintained by Adobe, which is involved in developing the C2PA specification. It is unlikely to stop. If it stops, our own implementation following the specification (JUMBF, COSE)|
|Prebuilt ONNX runtime components (pyke's distribution)|Build the ONNX runtime from source and pass it to ort with `ORT_LIB_PATH` (ort's “Linking” guide). ort's maintainer said in the 2.0.0-rc.11 guide that little macOS support can be expected (“expect little to no macOS support”); if ort stops working on macOS, the ONNX runtime is built from source and passed|
|trustmark|Switch to another C2PA-approved watermark. The identification number system is not changed|
|pdqhash|pdq-rs (the original C++). pdqhash's last release was 0.1.1 (2022-07-08), which cannot be said to satisfy continued maintenance. Before adopting, check agreement with the original's test values (DD-11-4“PDQ uses pdqhash”), and switch to pdq-rs if they do not agree or a vulnerability is found|
|harfrust|A component wrapping HarfBuzz's C implementation|
|tiny-skia|vello_cpu (has shadows and blur, but its README notes limitations)|
|moxcms|lcms2 (Little CMS)|
|krilla|printpdf (PDF/A conformance checked with veraPDF). The switch is behind the `PdfWriter` trait (Overall Implementation Design 2.1)|
|iroh (direct connections)|libp2p's QUIC and relays. The device number (Ed25519 public key) and the ALPN and CBOR procedure are not changed|
|tough|In-house verification following TUF specification 1.0 (the metadata format is not changed)|
|Tauri|Keep the screens (HTML, CSS) and move the shell to another framework (Electron, etc.)|

## 13. Mapping to Requirements

|Requirement number (text in the Outline Design Document)|Sections in this chapter|
|---|---|
|R-11-1-1|2 “Basis for Selection (Conditions of Design Plan 6.2)”|
|R-11-2-1|4 “List of Components”, 5 “Dependency Policy”, 12 “Preparation for When a Component's Maintenance Stops”|
|R-11-3-1|4 “List of Components”, 5.1 “Permitted licenses”|
|R-11-3-2|9 “NRSD's License (Decision on O-09“NRSD’s license type”)” (Apache-2.0)|
|R-11-4-1|3 “Development Environment and Versions”, 8 “Build and Continuous Integration (CI)”, DD-11-8“Internationalization uses message files”|
|R-11-5-1|5 “Dependency Policy”, 6 “SBOM and Provenance Attestations”|

## 14. Gaps Declared in This Chapter

- The gaps of this chapter follow the table in Chapter 13, 4.1 “Gaps in the mechanism” (the rows whose chapter column is this chapter; with why they cannot be closed, the extent addressed, the remaining risks, and who bears them) (not reproduced in this chapter).

## 15. Corrections to Other Chapters and the Outline Design Document

- Set the supported formats of Outline Design Document 3-1 “Input” to “JPEG, PNG, TIFF, WebP. RAW, HEIC, and the like: file fingerprint only”.
- The first draft's Section 9 “placing evidence as release attachments” was deleted, because Design Plan Edition 2 placed evidence on the User's device.
- The first draft's 3.3 “where evidence is kept” was consolidated into Chapter 8, 2.1 “Arrangement”.
