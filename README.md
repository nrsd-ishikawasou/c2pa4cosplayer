<div align="center">

# c2pa4cosplayer

**Anti-Repost Tool for Cosplay Photos**

*Put your rights on your photos. Keep everything on your own machine.*

[![Status](https://img.shields.io/badge/status-design%20phase-blue)](#project-status)
[![Docs](https://img.shields.io/badge/docs-%E6%97%A5%E6%9C%AC%E8%AA%9E%20%7C%20English%20%7C%20%E4%B8%AD%E6%96%87-informational)](#documents)
[![Platforms](https://img.shields.io/badge/platforms-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey)](#technology)
[![License](https://img.shields.io/badge/license-Apache--2.0-green)](#license)

</div>

---

## What this is

A desktop application, currently in the design stage, that lets cosplayers and photographers (the **Rights Holders**) show their rights on the photos they sell or post, and gives them the means to respond when a photo is reposted against their will.

The tool does not try to stop reposting by force. It is built for three things:

| | Goal | How |
|---|---|---|
| 1 | **Deterrence** | Every image carries a verifiable rights statement and a Permitted Scope. Showing that the Rights Holder has both the intent and the means to act is the main value of the System. |
| 2 | **Proof** | C2PA signatures, an invisible watermark, a perceptual hash, and a trusted timestamp are attached or recorded for each work, so that the Rights Holder can show "this is mine, and this is what I allowed." |
| 3 | **Response** | When a repost is found, the tool records it, preserves evidence, and points the Rights Holder to the right counter (platform takedown, notice, or a lawyer) with ready-to-use wording. |

## What it does

1. **Sign** — The Rights Holder enters a handle name, role, and Notice accounts. The app creates a signing key and certificate on the device and signs each photo with C2PA (manifest, invisible watermark, identification number).
2. **Edit and batch-apply** — Add a visible signature or overlay, set the Permitted Scope, and apply the same settings to a whole shoot at once. Work is saved in layers and can be resumed.
3. **Attach rights documents** — Generate the Rights Document and Enclosed Documents that go with delivery images, in the buyer's language.
4. **Register and preserve evidence** — When a repost is found, register it in the ledger, capture evidence, and produce a timestamped record.
5. **Get guidance** — See which route applies (platform report, notice to the reposter, legal action) and what to prepare, with reference information maintained in the Public Repository.

Everything above runs on the Rights Holder's own computer. There is no account, no server, and no upload.

## What it does not do

These are deliberate boundaries, not gaps:

- **It does not collect anything.** Rights Holder information, signing keys, work data, the ledger, and evidence stay on the User's device. NRSD never receives them.
- **It is not a certificate authority.** The certificate for C2PA signatures is generated on the device from the information the User enters. No identity verification, no email address.
- **It cannot prevent impersonation.** If someone overwrites a signature, the tool lays out the clues for matching (watermark, timestamp, material history, original files, Notice) and leaves the judgment to the people involved.
- **It does not perform matching for you.** It provides the algorithms and shows the available options.
- **It does not give legal advice or act on anyone's behalf.** Legal research is compiled with sources, but NRSD is not a law firm and guarantees no legal judgment.
- **It does not concern itself with the original works** (characters, franchises) that cosplay is based on. The System protects only the rights that the Rights Holders have in their own photographs.
- **No automated detection, no sales management.** Automated detection is a separate, later design. Sales management is out of scope; fork it if you need it.

## Technology

Decisions made in the Basic Design (Chapter 11, "Development Base"):

| Area | Choice |
|---|---|
| App framework | Tauri 2 (screens in HTML, CSS, TypeScript with Svelte 5; processing in Rust) |
| Platforms | Windows, macOS, Linux (desktop) |
| Content credentials | C2PA 2.4 via `c2pa-rs` |
| Invisible watermark | TrustMark (Rust crate, ONNX runtime) |
| Perceptual hash | PDQ |
| Signing keys | ECDSA P-256, kept in the OS keystore |
| Timestamps | RFC 3161 time-stamping authorities |
| Distribution | GitHub Releases with code signing, SBOM, and provenance attestations; self-update |
| Languages | Japanese, English, Chinese (others on request) |

## Documents

All documents live under `docs/`, one folder per language, in two formats with the same name: Markdown (readable in the browser) and Word. The Japanese original prevails where translations differ.

| | 日本語 | English | 中文 |
|---|---|---|---|
| Draft Project Proposal | [企画草案](docs/jp/コスプレ写真の無断転載対策ツール%20企画草案.md) ([Word](docs/jp/コスプレ写真の無断転載対策ツール%20企画草案.docx)) | [Draft Project Proposal](docs/en/Anti-Repost%20Tool%20for%20Cosplay%20Photos%20-%20Draft%20Project%20Proposal.md) ([Word](docs/en/Anti-Repost%20Tool%20for%20Cosplay%20Photos%20-%20Draft%20Project%20Proposal.docx)) | [企划草案](docs/cn/Cosplay照片防盗图工具%20企划草案.md) ([Word](docs/cn/Cosplay照片防盗图工具%20企划草案.docx)) |
| Design Plan (Edition 2) | [設計計画書](docs/jp/設計計画書.md) ([Word](docs/jp/設計計画書.docx)) | [Design Plan](docs/en/Anti-Repost%20Tool%20for%20Cosplay%20Photos%20-%20Design%20Plan.md) ([Word](docs/en/Anti-Repost%20Tool%20for%20Cosplay%20Photos%20-%20Design%20Plan.docx)) | [设计计划书](docs/cn/Cosplay照片防盗图工具%20设计计划书.md) ([Word](docs/cn/Cosplay照片防盗图工具%20设计计划书.docx)) |
| Outline Design | [00 概要設計書](docs/jp/design/00_概要設計書.md) ([Word](docs/jp/design/00_概要設計書.docx)) | [00 Outline Design](docs/en/design/00_Outline%20Design.md) ([Word](docs/en/design/00_Outline%20Design.docx)) | [00 概要设计书](docs/cn/design/00_概要设计书.md) ([Word](docs/cn/design/00_概要设计书.docx)) |
| Basic Design, Chapters 1 to 13 | [docs/jp/design](docs/jp/design) | [docs/en/design](docs/en/design) | [docs/cn/design](docs/cn/design) |
| Research Materials | [90 調査資料](docs/jp/design/90_調査資料.md) ([Word](docs/jp/design/90_調査資料.docx)) | [90 Research Materials](docs/en/design/90_Research%20Materials.md) ([Word](docs/en/design/90_Research%20Materials.docx)) | [90 调查资料](docs/cn/design/90_调查资料.md) ([Word](docs/cn/design/90_调查资料.docx)) |
| Legal Research | [法務調査](docs/jp/legal/法務調査.md) ([Word](docs/jp/legal/法務調査.docx)) | [Legal Research](docs/en/legal/Legal%20Research.md) ([Word](docs/en/legal/Legal%20Research.docx)) | [法务调查](docs/cn/legal/法务调查.md) ([Word](docs/cn/legal/法务调查.docx)) |

### Basic Design chapters

| Ch. | Title | Ch. | Title |
|---|---|---|---|
| 01 | Overall Architecture | 08 | Repository and Data Management |
| 02 | Signing Information and Matching | 09 | Distribution and Updates |
| 03 | Signing | 10 | Screens and Design |
| 04 | Image Editing and Batch Application | 11 | Development Base |
| 05 | Rights Documents | 12 | Interface with Legal |
| 06 | Registration and Evidence Preservation | 13 | Design Verification |
| 07 | Legal Action Guidance | | |

The documents are organized in order: Draft Project Proposal, then Design Plan, then Outline Design, then Basic Design. Each later document takes the earlier one as its premise. Chapter 13 lists the known limitations of the design openly rather than claiming there are none.

## Project status

| Stage | State |
|---|---|
| Draft Project Proposal | Done (2026-09-24) |
| Design Plan | Edition 2, draft (2026-09-29) |
| Outline Design | Done (150 requirements) |
| Basic Design, Chapters 1 to 13 | Done, audited, in three languages (2026-09-30) |
| Detailed design | Not started |
| Implementation | Not started |
| Test documents | Not started |

The User community will be asked to try a working preview build rather than read the documents.

## About

Developed by **Nagareyama Software Development Co., Ltd. (NRSD)**, lead developer Ishikawa Sou.
No fees, no sales, no donations. NRSD keeps the copyright but wants the software used freely.

## License

This repository is licensed under the **Apache License 2.0** (see [LICENSE](LICENSE)). Copyright in the design documents and the software belongs to NRSD; the rights in the third-party standards, algorithms, and software referenced belong to their respective holders.
