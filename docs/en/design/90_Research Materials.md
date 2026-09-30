# Research Materials

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [90_Research Materials.docx](90_Research%20Materials.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

## Research I-01 and I-02: C2PA Certificates, Identity, and Entitlement (2026-09-29, revised 2026-09-30)

- This material records facts researched at the stage of Design Plan Edition 1. Design Plan Edition 2 (2026-09-29) decided that the System does not verify Users, does not collect Users' information, and makes certificates for C2PA signatures on the device from the information Users enter (requiring no certificate authority) (D-5-1“The System does not confirm Users and collects no User information. No registration is required in order to use the System”, D-5-2“The certificate for C2PA signatures is created inside the Client App from information entered by the User. No certificate authority is required”). Therefore, the options at the Edition 1 stage in Section 2 “Consequences for the Design” (NRSD's private certificate authority, obtaining conformance and receiving device certificates, an identity aggregator) are not adopted. The adopted design is in Basic Design Chapter 2, 3 “Certificates”.

### 1. Facts Confirmed (with Sources)

#### 1.1 C2PA claim signing certificates are issued to “generator products”

- The C2PA Trust List is a list of “certificate authorities that issue certificates to conforming generator products”. Validator products judge by this list whether a signature is by a valid certificate. [S1] [S2]
- Conforming generator products must purchase signing certificates from certificate authorities on the Trust List. [S3]
- SSL.com: “issued only to tools and devices that have passed C2PA conformance”. Free tier: one Level 1 certificate (1 year) and 2,500 timestamps per year. Paid tiers include device certificates and API. [S4]
- Trufo: C2PA certificates USD 600 per year. OV (organization validation) and PV (product validation) required. Test certificates are free but not trusted by validator products. [S5]
- The certificates in the Trust List (C2PA-TRUST-LIST.pem) numbered 30 when obtained on 2026-09-30, and the issuing organizations are 17: Adobe, Castlabs, DigiCert, Encypher, Google, Huanyu Trust, Huawei, Irdeto, SSL.com, Snowball Technology, Tauth Labs, Trufo, TrustAsia, Verimago, Whole Earth Labs, Xiaomi, vivo (the description as of April 2026 listed four: DigiCert, SSL.com, Tauth Labs, Trufo). [S3] [S12]
- The Interim Trust List (ITL) was frozen on January 1, 2026. No new additions. [S2]
- Signatures not on the Trust List are shown as “Valid (cryptographically correct, but a signer not vouched for by the ecosystem)” and do not become “Trusted”. [S6] Adobe's verification site shows “unknown issuer”. [S2]

#### 1.2 Conformance procedure and the level of key protection

- Procedure: expression of interest, then a legal agreement with C2PA, then technical and security evaluation (architecture diagrams, documents), then issuance of certificates. Costs and duration are not stated in public information. [S3] [S7]
- The assurance level is recorded in an extension of the certificate. [S3]
- Level 1: signing keys stored encrypted. The OS keystore (macOS Keychain, Linux kernel keyring, etc.) may be used. “Distributed binaries must not contain secrets used for authentication.” [S8]
- Level 2: keys are generated, stored, and used in an environment with higher privilege than the claim generator (the OS's keystore service or a KMS), wrapped by hardware-derived keys. For remote signing, attestation by a hardware root of trust is verified before signing. [S8]
- There are categories: Edge (products running on the user's device), Backend (signing on a server), and Distributed. [S8]

#### 1.3 Personal identity is carried by the CAWG identity assertion (a signal separate from the C2PA signature)

- The current version of the CAWG identity assertion is 1.3 (checked at cawg.io on 2026-09-30; the research in this material is based on 1.2). The current version of metadata (cawg.metadata) is 1.1.
- The CAWG identity assertion proves that a credential holder controls a digital identity and binds it to C2PA assertions. It need not be the same as the claim signer and is an independent, additional trust signal. [S9]
- Credential types: X.509 certificates (cawg.x509.cose) and identity claims aggregation (W3C verifiable credentials). [S9]
- Aggregation types: cawg.social_media (control of an account; username required, uri recommended), cawg.document_verification (official ID; name required), cawg.web_site, cawg.affiliation, cawg.crypto_wallet. [S9]
- The aggregator creates and signs credentials per asset (per-asset signatures). [S9] Therefore the aggregator is involved in every signature (judged to require online involvement; not stated explicitly in the specification).
- Roles (role): cawg.creator, cawg.contributor, cawg.producer, etc. Custom values are also allowed. [S9]
- The specification does not require the X.509 subject to be a legal name. [S9]
- SSL.com's CAWG certificates: also issued to individuals. After identity verification, a “verified name” goes in. Individual USD 49 per year, individual and organization USD 69 per year, organization USD 99 per year. [S10]

#### 1.4 Existing similar services

- Adobe Content Authenticity (free, beta): name verification via LinkedIn, showing Behance, Instagram, LinkedIn, and X accounts in Content Credentials. Batch application to up to 50 JPG/PNG files of up to 20 MB each. [S11]

### 2. Consequences for the Design (to Chapters 2, 3, and 8)

#### 2.1 Replacement in Design Plan Edition 2 (the adopted design)

- A personal root and signing certificate are made on the User's device and used for C2PA signing. They are not on the Trust List, and on general verification sites they are “Valid”, not “Trusted” (Basic Design Chapter 2, 3 “Certificates”, 3.3 “How validators show it”).
- The clue to Entitlement is the notice code (made from the fingerprint of the personal root's public key) posted on the Notice account (Basic Design Chapter 2, DD-2-3“The clue to Entitlement is the notice code posted on the Notice account”, 2.4 “How the notice code is made”).
- The identity record is attached in `cawg.identity` as X.509 (`cawg.x509.cose`) with the signing certificate. No third-party identity verification is done (Basic Design Chapter 3, 3.1 “Fields recorded”).

#### 2.2 Study at the Edition 1 stage (kept as a record; not adopted)

- A configuration in which “the User themselves holds a C2PA claim signing certificate” is not possible. Claim signing certificates are issued to products.
- Options for claim signing
  - A: signatures outside the Trust List (NRSD's private certificate authority or self-issued per User). No cost, no conformance procedure. “Valid” on verification sites but not “Trusted”.
  - B: this app obtains conformance as a Level 1 Edge generator product, and receives a device certificate per installation from a certificate authority on the Trust List. Requires a legal agreement, evaluation, certificate costs, and an issuance mechanism (authentication at registration).
  - C: NRSD sets up a signing server (Backend). The server becomes an attack target and runs against the intent of the policy “not put on the web” (Design Plan D-6-2“The System shall not be built as a website”). Not adopted.
- Options for identity
  - X.509 CAWG certificates (individual): a verified name goes in. Therefore it runs directly into audit item A-15“Risk of real names appearing in C2PA signatures”.
  - Identity claims aggregation (social_media): the handle name (username) and Notice account can be included. However, the aggregator signs per asset. Therefore, if NRSD became the aggregator, an always-running service would be needed.
  - The System's own matching: the key fingerprint is posted on the Notice account and checked with the System's matching means and the public page (the proposal of Design Plan 5.3). A signal outside the C2PA ecosystem.
- At the Edition 1 stage, a staged configuration (first A and own matching, later B) was the leading proposal. In Edition 2, of A, “self-issued per User” and own matching were adopted, and NRSD's private certificate authority, B, and C were not adopted (2.1).

### 3. Sources

- S1 https://opensource.contentauthenticity.org/docs/conformance/trust-lists/
- S2 Same as above (freezing of the ITL, display on verification sites)
- S3 https://opensource.contentauthenticity.org/docs/signing/get-cert/ , https://opensource.contentauthenticity.org/docs/conformance/
- S4 https://www.ssl.com/products/content-authenticity/content-credentials/c2pa/
- S5 https://trufo.ai/tca
- S6 https://chronoverify.com/blog/c2pa-trust-list-explained
- S7 https://www.ssl.com/article/how-to-submit-to-the-c2pa-conformance-program/
- S8 https://github.com/c2pa-org/conformance-public/blob/main/docs/v0.2/C2PA%20Generator%20Product%20Security%20Requirements.md
- S9 https://cawg.io/identity/1.2/
- S10 https://www.ssl.com/products/content-authenticity/content-credentials/cawg/
- S12 https://github.com/c2pa-org/conformance-public/blob/main/trust-list/C2PA-TRUST-LIST.pem (obtained and counted on 2026-09-30)
- S11 https://helpx.adobe.com/creative-cloud/apps/adobe-content-authenticity/customization.html , https://blog.adobe.com/en/publish/2025/04/24/adobe-content-authenticity-now-public-beta-helps-creators-secure-attribution

## Research I-03: Retention of C2PA on Posting Sites and Cloud (2026-09-29)

### 1. Facts Confirmed

|Service|Handling of C2PA and EXIF|Source|
|---|---|---|
|Instagram, Facebook, Threads|Reads C2PA at upload and uses it for the “AI info” label; removes both EXIF and C2PA from delivered and re-downloaded files|[S1] [S2]|
|X|Removes EXIF and C2PA. No provenance display|[S1] [S2]|
|TikTok|Reads C2PA and labels automatically. Adds its own watermark to AI content|[S1] [S2]|
|LinkedIn|Reads C2PA and shows the CR icon and provenance panel. Retention as a file is not certain|[S1] [S2]|
|YouTube|Embedded information in videos is lost by re-encoding|[S1]|
|WhatsApp|Normal sending removes all metadata. “Send as file” keeps the original bytes|[S1] [S2]|
|Pinterest|Reads IPTC and shows AI labels. Retention not published|[S2]|
|Weibo|Single images over 30 MB are compressed even when the original is specified. Image formats are optimized (file size drops greatly). The official help does not mention EXIF. Reports and commentary say EXIF is removed|[S3] [S4]|
|Xiaohongshu|Reads the EXIF of uploaded images (for screenshot detection, etc.). No public information on retention in delivered files|[S5]|
|Patreon|Post images are delivered in several resolution versions (possibly re-encoded). Attachments are delivered in the original format the creator uploaded|[S6]|
|Google Drive|Downloads return the original file unchanged. Folder zips also do not change the content (secondary information). Google Photos may remove GPS in shared links (separate from Drive)|[S7] [S8]|

### 2. Consequences for the Design

- For major social media (Instagram, Facebook, X, TikTok, YouTube, Weibo), the design assumes C2PA does not remain on posted images. The social media flow is protected by the Visible Signature and the invisible watermark (consistent with Design Plan 7.4).
- LinkedIn displays it, but retention is not certain. It is of low importance as a posting site targeted by the System.
- Patreon is highly likely to handle “post images” and “attachments” differently. When Patreon attachments are used for delivery, the assumption is that the original bytes are kept. Therefore it is actually uploaded and checked (cannot be known without sending and receiving; assumption: attachments are kept, post images are not).
- All sources in this table are secondary (commentary articles, user reports), and no statement on the handling of C2PA could be confirmed in each service's official documents.
- For Google Drive, there are several pieces of secondary information that it returns the original bytes. Therefore it is determined by uploading to NRSD's Drive, downloading, and comparing SHA-256 (assumption: kept).
- The watermark's strength must withstand the re-encoding (reduction, JPEG recompression, conversion to WebP) of the above social media (requirements of Chapter 3).

### 3. Sources

- S1 https://aimetadataremover.org/blog/does-instagram-keep-c2pa/
- S2 https://www.lumethic.com/en/articles/content-credentials-social-media-platforms
- S3 https://kefu.weibo.com/faqdetail?id=21265
- S4 http://www.news.cn/tech/20220614/fea229915dfb41368bc81f7040d2bfe5/c.html
- S5 https://www.v2ex.com/t/1226436
- S6 https://github.com/mikf/gallery-dl/issues/2257 , https://github.com/mikf/gallery-dl/discussions/6569
- S7 https://metaclean.app/blog/does-google-drive-dropbox-remove-exif-data
- S8 https://www.experts-exchange.com/questions/29107039/How-to-Preserve-File-Metadata-when-Downloading-from-Google-Drive.html

## Research I-04, I-05, and Others: GitHub Limits, Connections from Mainland China, Code Signing (2026-09-29, revised 2026-09-30)

- Part of this material was researched at the stage of Design Plan Edition 1 (a configuration in which images and evidence were placed in a private repository and the app wrote to it via the REST API). Design Plan Edition 2 (2026-09-29) decided not to set up a private repository and to place records and evidence on the User's device (D-12-1“The Public Repository is provided. No Private Repository is provided”, D-12-3“Rights Holder Information, the signing key, work data, the ledger, and evidence are kept on the User’s device; NRSD does not collect them”). Therefore, the facts about the REST API and personal access tokens in Section 2, and the consequences for a configuration accumulating images and evidence in one repository, are not used in the current design (kept as a record). The replacement in Edition 2 is written at the end of each section.

### 1. GitHub from Mainland China (I-04)

- In GreatFire's observations, as of August 3, 2026, https://github.com was 60% obstructed in mainland China (interference in 60% of the last 35 tests). [S1]
- raw.githubusercontent.com is blocked or DNS-poisoned at some ISPs. Downloads of large files and binary releases are slow. Even when HTTPS fails, git over SSH sometimes works. [S1] [S2] [S3]
- Consequences for the design (Edition 1 stage): the design assumes that Users in mainland China cannot stably obtain, update, or register via GitHub (“registration” no longer exists in Edition 2; next item). Alternative routes are set in Chapters 8 and 9. The effectiveness of alternative routes is checked by actually fetching from lines in mainland China (cannot be known without sending and receiving; assumption: GitHub alone is insufficient).
- Replacement in Edition 2: the alternative route is object storage in the Hong Kong region (Alibaba Cloud OSS; no ICP filing required), used for obtaining updates and reference information and for direct download from the Chinese section of the README (Basic Design Chapter 8, DD-8-5“A Hong Kong mirror is set up for mainland China”, Chapter 9, 5 “Download Routes”). The “registration” listed at the Edition 1 stage (the app writing to GitHub) no longer exists, because in Edition 2 registration is completed within the device.

### 2. GitHub Limits (Chapter 8)

- Repository size: ideally under 1 GB, strongly recommended under 5 GB. [S4]
- File size: warning over 50 MiB, rejected over 100 MiB. Up to 25 MiB from the browser. Up to 2 GB per push. 3,000 entries per directory, depth up to 50. [S4] [S5]
- Git LFS: Free and Pro include 10 GiB of storage and 10 GiB of bandwidth; Team and Enterprise include 250 GiB each. Overages are metered. [S6]
- GitHub Actions: Free organizations get 2,000 minutes per month and 500 MB of artifacts for private repositories; Team gets 3,000 minutes per month and 2 GB. Standard runners for public repositories are free. [S7]
- REST API: personal access tokens and GitHub App installation tokens get 5,000 requests per hour. Actions' GITHUB_TOKEN gets 1,000 per hour per repository. Content-creating requests are limited to 80 per minute and 500 per hour (secondary limits). [S8]
- Fine-grained personal access tokens: can be limited to specific repositories, with fine-grained permissions. Organizations can require approval and impose a maximum lifetime. [S9]
- Releases: up to 1,000 files per release, each file under 2 GiB. There is no limit on the total size of releases or on bandwidth. [S14]
- Immutable releases: after publication, assets and tags cannot be changed or deleted (the title and description can be changed). Release attestations (tag, commit, assets) are created automatically. [S15]
- GitHub Pages: published sites up to 1 GB, with a guideline of 100 GB bandwidth per month (a notice may come if exceeded). [S16]
- Consequences at the Edition 1 stage (record): a configuration that keeps accumulating images and evidence in one repository would soon hit the recommended capacity (1 to 5 GB).
- Replacement in Edition 2: the Public Repository holds only the source, releases (distributables), the public page, and reference information, and does not hold Users' images or evidence (Basic Design Chapter 8, 3 “Public Repository”). The REST API and personal access tokens are not used. Distributables are placed in releases (about 200 to 300 MB each; within the 2 GiB limit), and reference information and the update manifest on the public page (within 1 GB).

### 3. Code Signing (I-05)

- macOS: the Apple Developer Program is USD 99 per year. Distribution outside the App Store uses a Developer ID certificate and notarization. [S10] From macOS Sequoia on, unsigned and unnotarized apps cannot be opened with Control-click and must be allowed from “Privacy & Security” in System Settings. [S11]
- Windows: Azure Artifact Signing (formerly Trusted Signing) is USD 9.99 per month for up to 5,000 signatures. Public trust certificates for organizations are available in countries including Japan. Individuals only in the United States and Canada. A paid Azure subscription is required. EV certificates are not issued. SmartScreen warnings appear even when signed until a download track record accumulates. [S12] [S13]
- Consequences for the design: NRSD (a Japanese corporation) can use Artifact Signing as an organization. macOS requires the Developer Program. Linux has no OS signing mechanism, so signing of distributables (a separate verification means) is set in Chapter 9. Assuming that SmartScreen shows warnings initially, this is explained in the first-run guidance (Chapter 10).

### 4. Sources

- S1 https://en.greatfire.org/https/github.com
- S2 https://en.wikipedia.org/wiki/Censorship_of_GitHub
- S3 https://github.com/MichaIng/DietPi/issues/7555
- S4 https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github
- S5 https://docs.github.com/en/repositories/creating-and-managing-repositories/repository-limits
- S6 https://docs.github.com/en/billing/concepts/product-billing/git-lfs
- S7 https://docs.github.com/billing/managing-billing-for-github-actions/about-billing-for-github-actions
- S8 https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api
- S9 https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens
- S10 https://developer.apple.com/programs/
- S11 https://developer.apple.com/news/?id=saqachfa
- S12 https://learn.microsoft.com/en-us/azure/artifact-signing/faq , https://azure.microsoft.com/en-us/pricing/details/artifact-signing/
- S13 https://learn.microsoft.com/en-nz/answers/questions/5810735/cant-create-a-new-trusted-signing-individual-ident
- S14 https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases
- S15 https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/immutable-releases
- S16 https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits

## Research: Technical Elements of Signing (Premises of Chapter 3) (2026-09-29, revised 2026-09-30)

- Revision of 2026-09-30: following an external audit, the descriptions of c2pa.metadata, the sentence on PDQ, and the handling of trust in timestamps were revised. The adopted design is in Basic Design Chapter 3 “Signing”.

### 1. C2PA Assertions (Specification 2.4) [S1]

- Binding to content (hard binding): c2pa.hash.data, c2pa.hash.boxes, c2pa.hash.bmff.v3
- Soft binding: c2pa.soft-binding (identifiers of watermarks and fingerprints)
- History of actions: c2pa.actions.v2. c2pa.created or c2pa.opened is required. Others include c2pa.edited, c2pa.watermarked, c2pa.resized, c2pa.published, c2pa.placed, c2pa.converted, etc.
- Ingredients: c2pa.ingredient.v3. Metadata: c2pa.metadata in specification 2.x is limited to the permitted fields of Appendix B.2 (fixed fields such as shooting date/time, device, location) and cannot hold fields for author name, copyright, or terms of use. These are placed in CAWG's `cawg.metadata` (Basic Design Chapter 3, 3.1 “Fields recorded”). Thumbnail: c2pa.thumbnail.claim (a reduction of the output). Time: c2pa.time-stamp
- Added in 2.4: c2pa.repository-receipt, c2pa.ai-disclosure, etc.
- The specification strongly recommends letting the creator control which provenance data is included (consideration of personal information).
- The latest version is 2.4 (confirmed in Design Plan Chapter 17).

### 2. Invisible Watermark (TrustMark) [S2] [S3] [S4] [S5]

- The payload is 100 bits. Data bits available per error correction scheme: BCH_SUPER 40 bits (corrects 8-bit errors), BCH_5 61 bits (5), BCH_4 68 bits (4), BCH_3 75 bits (3). [S3: datalayer.py]
- TrustMark variants Q, C, and P are on C2PA's list of approved soft bindings (B is not). [S5]
- Usage: put a random identifier in the watermark and record the same identifier in the C2PA soft binding assertion. Even if C2PA is lost, a copy of the manifest can be looked up using the watermark's identifier as the key (Durable Content Credentials). [S4]
- Rust version: supports embedding and reading in binary mode for all variants. Text mode and watermark removal are not implemented. Uses the ONNX runtime (ort), and models are obtained separately. Supported environments: Windows 10/11, glibc 2.35 or later (Ubuntu 22.04 or later, etc.), macOS 10.15 or later. [S6]
- License: MIT (code and models). [S2]
- PSNR of about 50.4 dB on sample images. [S2]
- Figures for execution speed and robustness to recompression are not in the public README. Therefore it is judged by measuring on target images (tens of millions of pixels) and the recompression of target social media (cannot be known without running the code; assumption: within a few seconds per image on CPU, and readable with BCH_5 after social media reduction and recompression).

### 3. Matching Hash (PDQ) [S7] [S8]

- There are pdqhash (Rust, Apache-2.0, pure Rust), pdq-rs (wrapping the original C++), and yume-pdq (speed first).
- The guide for PDQ match determination is a Hamming distance of 31 bits or less (out of 256 bits). [S8]
- PDQ is weak against cropping and rotation (a property of the algorithm; the original document hashing.pdf [reference 12 of the Draft Project Proposal]). Robustness to cropping is supplemented by the watermark. Rotations in 90-degree steps and flips can be handled by computing and comparing 8 hashes (Basic Design Chapter 3, 6 “Matching Hash”).

### 4. Trusted Timestamps (RFC 3161)

- Free TSAs: DigiCert (timestamp.digicert.com), GlobalSign, FreeTSA (no guarantee). [S9]
- SSL.com's free C2PA tier includes 2,500 timestamps per year. [Note on design: S4 of Research Materials “C2PA Certificates and Identity”]
- C2PA has a separate TSA trust list. Validators can confirm validity at the time of signing, even after the certificate is revoked or expired, through timestamps that chain to the TSA trust list. [S2 of Research Materials “C2PA Certificates and Identity”] However, the free TSAs above (DigiCert's general-purpose one, GlobalSign, FreeTSA) are not on the C2PA TSA trust list, and C2PA 2.x validators ignore their timestamps and check the certificate's validity against the time of checking (specification 2.4, 15.8.2; confirmed in the original on 2026-09-30). Therefore the certificate's validity is made long (Basic Design Chapter 2, DD-2-11“Certificate validity is long enough that every image is shown valid for 20 years from signing”).
- China: United Trust Time Stamp Service Center (built by the National Time Service Center and United Trust; RFC 3161, GB/T 20520-2006). As of April 2025, over 100,000 court documents at 1,221 courts used trusted timestamps as evidence. [S10]
- Japan: timestamp accreditation system by the Minister for Internal Affairs and Communications (started 2021). Amano Timestamp Service 3161 received the first accreditation in February 2023, renewed in February 2025. [S11]
- Fees: prices for United Trust and Japanese accredited providers could not be confirmed in public information. Therefore a quote is obtained before deciding (assumption: paid; a configuration is considered in which they are used only for timestamps at registration (evidence preservation) and free TSAs are used at signing).

### 5. Self-Update (If Tauri Is Adopted) [S12]

- Verification of update signatures is mandatory and cannot be disabled. A public key is embedded in the app, and artifacts are signed with the private key. If the private key is lost, updates cannot be distributed to existing users.
- Several update sources can be specified, moving to the next on non-2xx responses (alternative routes can be held).
- Supports static JSON placed in GitHub Releases.

### 6. Sources

- S1 https://spec.c2pa.org/specifications/specifications/2.4/specs/C2PA_Specification.html
- S2 https://github.com/adobe/trustmark
- S3 https://raw.githubusercontent.com/adobe/trustmark/main/python/trustmark/datalayer.py
- S4 https://github.com/adobe/trustmark/blob/main/c2pa/README.md
- S5 https://github.com/c2pa-org/softbinding-algorithm-list (softbinding-algorithm-list.json)
- S6 https://github.com/adobe/trustmark/blob/main/rust/README.md
- S7 https://github.com/darwinium-com/pdqhash , https://crates.io/crates/pdq-rs
- S8 https://crates.io/crates/yume-pdq/0.1.0
- S9 https://gist.github.com/Manouchehri/fd754e402d98430243455713efada710 , https://www.freetsa.org/index_en.php , https://knowledge.digicert.com/general-information/rfc3161-compliant-time-stamp-authority-server
- S10 https://www.tsa.cn/ , https://www.chinaiprlaw.cn/index.php?id=5357
- S11 https://www.e-timing.ne.jp/news/detail/145 , https://www.soumu.go.jp/main_sosiki/joho_tsusin/top/ninshou-law/timestamp.html
- S12 https://v2.tauri.app/plugin/updater/
