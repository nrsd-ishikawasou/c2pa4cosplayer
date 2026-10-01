# Outline Design Document

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [00_Outline Design.docx](00_Outline%20Design.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

This document receives the Design Plan, sets the overall structure and the outline of the methods of the whole system, and defines the scope. The details of each chapter are set in the Basic Design Documents (Chapter 1 “Overall Architecture” to Chapter 13 “Design Verification”).

How each chapter is written:

- Decisions received: decisions (D-), items to be investigated (I-), and open items (O-) of the Design Plan, and omissions found in the item breakdown (A-)
- What to define
- Scope: covered / not covered (“none” if there is nothing)
- What must be possible: requirements. Numbered R-chapter-function-sequence. Chapter 13 (verification) and the test documents refer to these numbers
- What to settle in the basic design
- Dependencies: chapters depended on and chapters depending on this one. Chapters that depend on each other are marked “mutually dependent”
- To Legal: points to be investigated in the legal document set and linked from here

## Position of This Document

### Role of this document

- Receives the Design Plan and defines the overall structure and the outline of the methods of the whole system. The scope is defined in this document.
- The details of each chapter are left to the Basic Design Documents. The whole is closed in this document before going into details.
- Tests are handled in a separate document set paired with this document and the Basic Design Documents.
- Legal matters are investigated in a separate document set and linked from this document.

### What the System protects and does not protect (premise for all chapters)

- Protects: the rights of Users (the Rights Holders of images and those authorized by them) in their own photographs [D-2-1“What the System protects shall be the rights (copyright and portrait rights) that the Rights Holders have in their own photographs”]
- Does not protect: copyright and other rights in the source works of cosplay (characters, etc.). The System does not resolve relationships with the rights holders of source works, or licensing problems Users may cause regarding source works.
- This line is stated to Users in the Terms of Use and in on-screen guidance (Chapter 10 “Screens and Design”).

### Terms

- The terms of Design Plan 1.5 “Definitions” are used as they are. Terms added in this document are as follows.

|Term|Meaning|
|---|---|
|Work data|The record for each work of its identification number, matching hashes, date and time, and link to the Permitted Scope|
|Ledger|The collection of records of registered reposts (kept on the User's device)|
|Public page|A view-only page placed in the Public Repository|
|Reference information|Information NRSD maintains and delivers to Users' apps, such as the list of contact points, example texts, references to each country's laws, and the wording of Enclosed Documents|
|Operator functions|The mechanisms NRSD uses for releases, maintenance of reference information, and receiving feedback|
|Requirement number|R-chapter-function-sequence|

### Premises

- Delivery, Visible Signatures, and rights relationships with photographers are built on the assumption that the cosplayer understands them [D-3-4“The System is built on the premise that delivery, Visible Signatures, and the rights relationship with the photographer are understood by the cosplayer personally”]
- No interviews are held; improvements follow feedback once the System is usable [D-3-5“No interviews are conducted; improvement is made based on comments after the System becomes usable”]
- Zero gaps is impossible; known gaps are declared in Chapter 13 “Design Verification”.

### Destination table

|Number|Content|Chapter resolving it|Result (Basic Design Chapter 13, 1.3 “Destination table (Outline Design Document “Destination Table”)”)|
|---|---|---|---|
|I-01|C2PA certificates|Chapter 2 “Signing Information and Matching”|Resolved|
|I-02|Matching with the Notice account|Chapter 2 “Signing Information and Matching”|Resolved|
|I-03|Retention of C2PA on posting sites and Cloud|Chapter 3 “Signing”|Resolved (measurement on Drive awaits NRSD's check)|
|I-04|Connection to GitHub from mainland China|Chapter 8 “Repository and Data Management”, Chapter 9 “Distribution and Updates”|Resolved|
|I-05|Code signing|Chapter 9 “Distribution and Updates”|Resolved|
|I-06|Differences in rights by country|Chapter 12 “Interface with Legal” (handled by Legal)|Resolved|
|I-07|Knowing the purchaser's country|Chapter 5 “Rights Documents”|Cannot be narrowed down (with a VPN the country cannot be known or determined). The design does not depend on the purchaser's country|
|I-08|Expert review of Rights Document wording|Chapter 5 “Rights Documents”, Chapter 12 “Interface with Legal”|Resolved (distributed only after review)|
|O-01|How to present matching clues (retitled in Design Plan Edition 2)|Chapter 2 “Signing Information and Matching”|Resolved|
|O-02|Development base|Chapter 11 “Development Base”|Resolved (Tauri 2. Basic Design Chapter 11, DD-11-1“Built with Tauri 2”)|
|O-03|Approach to governing law|Chapter 5 “Rights Documents”, Chapter 12 “Interface with Legal”|A proposal is in place (adoption depends on expert review)|
|O-04|Options for the Permitted Scope|Chapter 5 “Rights Documents”|Resolved|
|O-05|Content of the public page|Chapter 8 “Repository and Data Management”|Resolved|
|O-06|Sharing repost site information|Chapter 6 “Registration and Evidence Preservation”|Deleted in Design Plan Edition 2 (not shared)|
|O-07|Integration with the OS|Chapter 10 “Screens and Design”|Resolved (not provided)|
|O-08|Issuer of identification numbers|Chapter 3 “Signing”|Decided in Design Plan Edition 2 (D-7-7“Identification numbers are issued by the Client App and recorded automatically in the C2PA manifest. Whether they are appended to file names is chosen by the User when saving”)|
|O-09|NRSD's license|Chapter 11 “Development Base”|Decided: Apache-2.0 (2026-09-30)|
|O-10|Recording format of work data (retitled in Design Plan Edition 2; recorded on the User's device)|Chapter 3 “Signing”, Chapter 8 “Repository and Data Management”|Resolved|
|O-11|Visible Signatures on Delivery Images|Chapter 3 “Signing”|Resolved|
|B-2|Add Cloud (Google Drive, etc.) to the scope of I-03“Retention of C2PA signatures on posting sites and Cloud” (decided by NRSD)|Chapter 3 “Signing”|Reflected in I-03“Retention of C2PA signatures on posting sites and Cloud” of Design Plan Edition 2|

## 1. Overall Architecture

Decisions received: per the table at the head of Basic Design Chapter 1 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: the system boundary, responsibilities of components, where data resides and how it flows, common policies, operator functions, and chapter dependencies.

Scope

- Covered: the Client App, the Public Repository, data on the User's device, operator functions, and interfaces with external organizations.
- Not covered: Phase 2 (automated detection), sales management, rights in source works (“What the System protects and does not protect”), collection of User information, and matching on others' behalf (Design Plan D-5-1“The System does not confirm Users and collects no User information. No registration is required in order to use the System”, D-5-5“The System provides the means of matching but does not perform matching on anyone’s behalf”).

What must be possible

- 1-1 Boundary and components
  - R-1-1-1 What each component holds and does not hold is uniquely determined.
  - R-1-1-2 It is stated explicitly that Phase 2, sales management, rights in source works, and collection of User information are outside the boundary.
- 1-2 Data
  - R-1-2-1 For all data, the source, storage location, public/non-public status, and writer are determined.
  - R-1-2-2 The identifier scheme and the versions of data formats are determined.
  - R-1-2-3 The locations of secrets (Users' signing keys, NRSD's keys) are listed.
  - R-1-2-4 The retention and deletion policy is determined.
  - R-1-2-5 Multilingual file names and characters can be handled.
- 1-3 Data flows
  - R-1-3-1 Each flow—signing, sales, social media posting, Registration, updates, and distribution of reference information—runs uninterrupted from start to end.
  - R-1-3-2 Even after partial failure, no data inconsistency remains.
  - R-1-3-3 Work that can be done offline is distinguished from work that needs a connection, and work can be held while offline.
- 1-4 Trust and threats
  - R-1-4-1 The trust boundaries are shown in a figure.
  - R-1-4-2 There is a list of threats (impersonation, fake apps, key theft, tampering).
- 1-5 External dependencies
  - R-1-5-1 The behavior when the TSA, GitHub, or posting sites stop is determined.
- 1-6 Devices and people
  - R-1-6-1 In case of device loss or failure, Users can export records and import them on another device.
  - R-1-6-2 One person can use multiple devices.
  - R-1-6-3 Transfer and termination of rights (including revocation of authorization) can be handled.
- 1-7 Common policies
  - R-1-7-1 Scale and performance targets, handling of failures, records (operation logs), version compatibility, and handling of time are common to all chapters.
  - R-1-7-2 The screen language and the country of Rights Documents are handled separately.
- 1-8 Operations
  - R-1-8-1 NRSD's points of involvement, operating structure, and contact during failures are determined.
  - R-1-8-2 Operating costs and who bears them are determined.
- 1-9 Figures and mapping
  - R-1-9-1 There are a structure diagram, a data flow diagram, and a trust boundary diagram.
  - R-1-9-2 There is a mapping table from Design Plan decision numbers to this document's requirement numbers.
  - R-1-9-3 There is a chapter dependency table and an order for entering the basic design (below).
- 1-10 Operator functions
  - R-1-10-1 NRSD can make releases, maintain reference information, and receive feedback.
  - R-1-10-2 The operator functions themselves do not become a path for fake updates or fake reference information.

What to settle in the basic design

- Data schemas, identifier formats, values of retention periods, performance figures, and the form of the operator functions.

Decisions in the basic design: per Basic Design Chapter 1, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies

Chapter dependency table and order (R-1-9-3“There is a chapter dependency table and an order for entering the basic design (below)”)

|Chapter|Chapters it depends on|
|---|---|
|Chapter 2 “Signing Information and Matching”|Chapter 1 “Overall Architecture”, Chapter 8 “Repository and Data Management” (mutually dependent), Chapter 10 “Screens and Design” (mutually dependent), Chapter 12 “Interface with Legal” (mutually dependent)|
|Chapter 3 “Signing”|Chapter 1 “Overall Architecture”, Chapter 2 “Signing Information and Matching”, Chapter 5 “Rights Documents” (mutually dependent), Chapter 8 “Repository and Data Management”, Chapter 11 “Development Base” (mutually dependent), Chapter 12 “Interface with Legal” (mutually dependent)|
|Chapter 4 “Image Editing and Batch Application”|Chapter 1 “Overall Architecture”, Chapter 3 “Signing”, Chapter 10 “Screens and Design” (mutually dependent), Chapter 11 “Development Base” (mutually dependent)|
|Chapter 5 “Rights Documents”|Chapter 1 “Overall Architecture”, Chapter 2 “Signing Information and Matching”, Chapter 3 “Signing” (mutually dependent), Chapter 10 “Screens and Design” (mutually dependent), Chapter 12 “Interface with Legal” (mutually dependent)|
|Chapter 6 “Registration and Evidence Preservation”|Chapter 1 “Overall Architecture”, Chapter 2 “Signing Information and Matching”, Chapter 3 “Signing”, Chapter 5 “Rights Documents”, Chapter 8 “Repository and Data Management” (mutually dependent), Chapter 12 “Interface with Legal” (mutually dependent)|
|Chapter 7 “Legal Action Guidance”|Chapter 1 “Overall Architecture”, Chapter 5 “Rights Documents”, Chapter 6 “Registration and Evidence Preservation”, Chapter 10 “Screens and Design” (mutually dependent), Chapter 12 “Interface with Legal” (mutually dependent)|
|Chapter 8 “Repository and Data Management”|Chapter 1 “Overall Architecture”, Chapter 2 “Signing Information and Matching” (mutually dependent), Chapter 6 “Registration and Evidence Preservation” (mutually dependent)|
|Chapter 9 “Distribution and Updates”|Chapter 1 “Overall Architecture”, Chapter 8 “Repository and Data Management”, Chapter 11 “Development Base”|
|Chapter 10 “Screens and Design”|All chapters (the screens receive the User operations of Chapters 2 to 9; mutually dependent with Chapters 2, 4, 5, and 7), Chapter 12 “Interface with Legal” (mutually dependent)|
|Chapter 11 “Development Base”|Chapter 1 “Overall Architecture”, Chapter 3 “Signing” (mutually dependent), Chapter 4 “Image Editing and Batch Application” (mutually dependent)|
|Chapter 12 “Interface with Legal”|All chapters (receives the legal points each chapter covers; mutually dependent with Chapters 2, 3, 5, 6, and 7), Chapter 10 “Screens and Design” (mutually dependent)|
|Chapter 13 “Design Verification”|All chapters|

- Order for entering the basic design: Chapter 1 “Overall Architecture”; then the outline of Chapter 11 “Development Base”; then Chapter 2 “Signing Information and Matching” and Chapter 8 “Repository and Data Management” together; then Chapter 3 “Signing” (back and forth with Chapter 11 “Development Base”; with Chapter 5 “Rights Documents” in the second round); then Chapter 4 “Image Editing and Batch Application” (back and forth with Chapter 11 “Development Base”) and Chapter 5 “Rights Documents”; then Chapter 6 “Registration and Evidence Preservation” (with Chapter 8 “Repository and Data Management” in the second round); then Chapter 7 “Legal Action Guidance”; then Chapter 9 “Distribution and Updates”; then Chapter 10 “Screens and Design”; then Chapter 12 “Interface with Legal”; then Chapter 13 “Design Verification”
- Chapter 10 “Screens and Design” and Chapter 12 “Interface with Legal” receive the requirements of all chapters and are therefore designed last; the mutually dependent chapters (Chapters 2, 3, 4, 5, 6, and 7) are revisited in a second round after Chapters 10 and 12 are designed, to fix inconsistencies (Basic Design Chapter 13, 3.2 “Dependency table”).

To Legal: NRSD's position, Terms of Use, and Privacy Policy.

## 2. Signing Information and Matching

Decisions received: per the table at the head of Basic Design Chapter 2 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: how the information used for C2PA signatures is created, the matching clues when someone posing as the Rights Holder appears, and the handling of Rights Holder Information and signing keys.

Scope

- Covered: input of information for C2PA signatures and creation of certificates, the means of matching and how clues are presented, authorizations and joint rights, Rights Holder Information, and signing keys.
- Not covered: verifying Users, collecting User information, and matching or judging on others' behalf (Design Plan D-5-1“The System does not confirm Users and collects no User information. No registration is required in order to use the System”, D-5-3“Acts of a person posing as the Rights Holder (signing first, replacing signatures, and reverse complaints) are treated as impossible to prevent; clues for matching are presented. No judgment is made”, D-5-5“The System provides the means of matching but does not perform matching on anyone’s behalf”). Checking rights holders of source works (“What the System protects and does not protect”).

What must be possible

- 2-1 Identity (who made the C2PA signature)
  - R-2-1-1 The signer (the information placed in the certificate) can be confirmed from an image's C2PA signature.
  - R-2-1-2 There are provisions for certificate expiry and renewal and for input errors.
- 2-2 Clues to Entitlement
  - R-2-2-1 There is a means by which anyone can confirm the link between the signer and the Rights Holder of the photograph by matching with the Notice account.
  - R-2-2-2 Possession of the Original can be used as a clue.
  - R-2-2-3 There is a provision for when a Notice account is taken over.
- 2-3 Persons posing as the Rights Holder
  - R-2-3-1 When prior signing, replacement of signatures, or counter-complaints occur, matching clues can be laid out side by side (without judgment).
  - R-2-3-2 For photographs published before adoption, the clues that can be shown and the range that cannot are stated (what cannot be shown is declared in Chapter 13).
  - R-2-3-3 From the clues, the true Rights Holder can see what actions are possible (Chapter 7).
- 2-4 Input and authorization
  - R-2-4-1 A non-technical person can complete the input of information for C2PA signatures (the criterion is set in the basic design).
  - R-2-4-2 Authorizations can be granted, scoped, time-limited, and revoked.
  - R-2-4-3 A photographer and a cosplayer can handle the same photograph.
- 2-5 Rights Holder Information
  - R-2-5-1 The range placed on images (made public) is determined.
  - R-2-5-2 Real names are not unintentionally made public in signatures or on screen.
  - R-2-5-3 The history of changes remains on the device.
- 2-6 Signing keys
  - R-2-6-1 They are stored safely and protected during processing.
  - R-2-6-2 On loss or leak, the key can be remade and the Notice re-posted, and the handling of past signatures is determined.
  - R-2-6-3 They can be used on multiple devices.

What to settle in the basic design

- How certificates are made and their contents, the matching procedure and how clues are presented, the key storage method, and the flow of the input screens.

Decisions in the basic design: per Basic Design Chapter 2, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies: per the chapter dependency table in Chapter 1 “Overall Architecture”.

To Legal: none (User information is not collected).

## 3. Signing

Decisions received: per the table at the head of Basic Design Chapter 3 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: which information is attached to images, in what order, and how.

Scope

- Covered: input, C2PA manifest, timestamps, invisible watermark, matching hashes, identification numbers and work data, processing order, batch processing, and output.
- Not covered: editing of Visible Signatures itself (Chapter 4 “Image Editing and Batch Application”).

What must be possible

- 3-1 Input
  - R-3-1-1 The supported formats are determined.
  - R-3-1-2 Broken images, huge images, and images that already have a signature can be handled safely.
- 3-2 Manifest
  - R-3-2-1 The items recorded and the items not recorded (personal information) are determined.
  - R-3-2-2 The version of the specification followed is determined.
  - R-3-2-3 Whether to record the Permitted Scope and Visible Signature operations is determined (decided together with Chapter 5).
- 3-3 Timestamps
  - R-3-3-1 The TSA (considering TSAs in China as well) is determined.
  - R-3-3-2 When unreachable, timestamps can be held and added later.
- 3-4 Invisible watermark
  - R-3-4-1 The information embedded and the strength are determined.
  - R-3-4-2 The relationship with “processing that does not change the appearance” in Figure 3 is sorted out.
  - R-3-4-3 It runs on typical machines without a GPU (the speed criterion is set in the basic design).
- 3-5 Matching hashes
  - R-3-5-1 What they are computed on and where they are recorded are determined.
- 3-6 Identification numbers and work data
  - R-3-6-1 Numbers are issued uniquely and can be written in file names and captions.
  - R-3-6-2 The items and recording location of work data are determined.
- 3-7 Processing order and streams
  - R-3-7-1 Processing runs in an order that does not break the signature.
  - R-3-7-2 The differences between delivery and social media are determined.
  - R-3-7-3 The Original is not changed.
  - R-3-7-4 Shooting information (location, etc.) is removed.
- 3-8 Batch processing
  - R-3-8-1 Interruption, resumption, and partial failure are handled.
  - R-3-8-2 Progress and results can be shown.
- 3-9 Output
  - R-3-9-1 The names and structure of output are determined.
  - R-3-9-2 Users can check their output themselves.
  - R-3-9-3 Whether C2PA signatures remain on posting sites and Cloud is known, and the handling when they do not is determined (by NRSD's decision Cloud is also investigated).

What to settle in the basic design

- The concrete items of the manifest, the TSA, watermark parameters, number formats, where work data is recorded, and the differences between the two streams.

Decisions in the basic design: per Basic Design Chapter 3, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies: per the chapter dependency table in Chapter 1 “Overall Architecture”.

To Legal: the effect of timestamps in each country.

## 4. Image Editing and Batch Application

Decisions received: per the table at the head of Basic Design Chapter 4 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: the functions that create Visible Signatures and apply them to images.

Scope

- Covered: Visible Signatures, templates, placement, editing operations, batch application, and output for social media.
- Not covered: image editing unrelated to showing rights.

What must be possible

- 4-1 Visible Signatures
  - R-4-1-1 Name, account name, date, overlay image, and license statement can be placed.
  - R-4-1-2 Orientation (vertical writing, rotation) and color can be changed.
  - R-4-1-3 The licenses of the fonts used have been checked.
  - R-4-1-4 Imported assets are handled safely.
  - R-4-1-5 Elements can be stacked as layers, with stacking order, visibility, locking, and opacity.
- 4-2 Templates
  - R-4-2-1 Designs can be saved and reused, and used on multiple devices.
  - R-4-2-2 Whether to provide default templates is determined.
- 4-3 Placement
  - R-4-3-1 Signatures can be placed avoiding the subject.
  - R-4-3-2 Nothing breaks even for images where the signature does not fit.
- 4-4 Editing operations
  - R-4-4-1 Adjustments can be made while viewing a preview, and undone.
  - R-4-4-2 The Original is not damaged.
  - R-4-4-3 The state of editing in progress is saved automatically and can be resumed later.
- 4-5 Batch application
  - R-4-5-1 Signatures can be applied to a folder in one batch and adjusted per photograph.
- 4-6 Output
  - R-4-6-1 Images can be output in the size, compression, and color space for social media.

What to settle in the basic design

- Whether there is auto-placement and its method, the template format, and the choice of fonts.

Decisions in the basic design: per Basic Design Chapter 4, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies: per the chapter dependency table in Chapter 1 “Overall Architecture”.

To Legal: rights in assets, font licenses.

## 5. Rights Documents

Decisions received: per the table at the head of Basic Design Chapter 5 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: how the Permitted Scope is chosen, and the content and management of Enclosed Documents and Notice texts.

Scope

- Covered: Permitted Scope, Enclosed Documents, management of wording, target countries, and support for the Notice.
- Not covered: guaranteeing the legal correctness of the wording (it goes through expert review; Chapter 12).

What must be possible

- 5-1 Permitted Scope
  - R-5-1-1 A non-technical person can understand the meaning and choose (the criterion is set in the basic design).
  - R-5-1-2 Which Permitted Scope was chosen for which image is recorded.
  - R-5-1-3 The handling of changes after sale is determined.
- 5-2 Enclosed Documents
  - R-5-2-1 They include the matters of Design Plan 9.2.
  - R-5-2-2 Rewriting can be detected.
- 5-3 Wording
  - R-5-3-1 Who wrote it, who checked it, and which version it is can be known.
  - R-5-3-2 Updates can be delivered to Users (distribution of reference information; 1-3).
- 5-4 Countries
  - R-5-4-1 There is a link covering all countries and a way to decide the nationalities enclosed.
- 5-5 Support for the Notice
  - R-5-5-1 Example Notice texts (three languages) can be shown.
  - R-5-5-2 The information needed for Entitlement matching can be included in the Notice.

What to settle in the basic design

- Options for the Permitted Scope, the format of the Enclosed Document, the format of wording data, and where the link target is placed.

Decisions in the basic design: per Basic Design Chapter 5, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies: per the chapter dependency table in Chapter 1 “Overall Architecture”.

To Legal: the legal position of the Enclosed Document, the terms of sales platforms, and the governing law.

## 6. Registration and Evidence Preservation

Decisions received: per the table at the head of Basic Design Chapter 6 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: from the moment a User finds a repost, through keeping evidence, to following what happens next.

Scope

- Covered: Registration, obtaining and preserving evidence, matching, status checking, correction, lists and possible actions (information is not shared among Users; Design Plan D-10-9“Information on registered reposts is listed on the User’s device, and the options for action are presented. No sharing among Users takes place (dealt with in Phase 2)”).
- Not covered: automated detection of reposts (Phase 2). The app does not crawl automatically for status checking.

What must be possible

- 6-1 Registration
  - R-6-1-1 Registration can be done with one button.
  - R-6-1-2 For pages that require login, what the User obtained themselves can be imported.
  - R-6-1-3 Registrations are recorded on the User's device and do not affect other Users' records.
  - R-6-1-4 There is room to accept Registrations from Phase 2.
- 6-2 Obtaining evidence
  - R-6-2-1 The matters to be shown later (Design Plan 10.3) can be kept.
  - R-6-2-2 There are provisions for failures to obtain and for malicious sites.
- 6-3 Preserving evidence
  - R-6-3-1 A timestamp is attached, and it can be shown that nothing was tampered with.
  - R-6-3-2 Evidence can be exported in a form that can be handed to experts.
- 6-4 Matching
  - R-6-4-1 The degree of match and what is violated can be shown.
- 6-5 Status checking
  - R-6-5-1 The User can see what happened to registered reposts afterward.
  - R-6-5-2 It is linked to the record of complaints.
- 6-6 Correction
  - R-6-6-1 Wrong Registrations can be withdrawn.
  - R-6-6-2 The handling of registering a repost that is not the User's own work is determined.
- 6-7 Lists and possible actions
  - R-6-7-1 Registered reposts can be gathered into a list and possible actions presented (not shared among Users).

What to settle in the basic design

- Items to obtain, storage format, and how status checks are updated.

Decisions in the basic design: per Basic Design Chapter 6, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies: per the chapter dependency table in Chapter 1 “Overall Architecture”.

To Legal: lawfulness of saving pages, evidentiary value in each country.

## 7. Legal Action Guidance

Decisions received: per the table at the head of Basic Design Chapter 7 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: the range that shows Users “what to look at to be able to respond”.

Scope

- Covered: contact points, example texts, references to each country's laws, the procedure for identifying operators, and the flow of complaints.
- Not covered: legal judgment, representation, advice on how to sue. Dealing with the legal problem itself.

What must be possible

- 7-1 Contact points
  - R-7-1-1 Contact points per posting site and provider can be known.
  - R-7-1-2 Changes in contact points can be followed (distribution of reference information; 1-3).
- 7-2 Example texts
  - R-7-2-1 They can be shown per country and language.
  - R-7-2-2 Users can know in advance that complaints may pass their personal information to the other party.
- 7-3 References
  - R-7-3-1 References to each country's laws are accumulated in an “all countries” frame and delivered to Users.
- 7-4 Identifying operators
  - R-7-4-1 The procedure can be shown.
  - R-7-4-2 It is shown that action is abandoned when the location cannot be determined even after investigation (declared as a gap in Chapter 13).
- 7-5 Flow of complaints
  - R-7-5-1 The flow of complaining with the evidence package, the Permitted Scope, and the Enclosed Document attached can be understood.
  - R-7-5-2 The approach of joint action and of prioritizing commercial use is shown.
- 7-6 Drawing the line
  - R-7-6-1 It is stated explicitly that this is not legal advice, and the sources of information are stated.

What to settle in the basic design

- Data formats for contact points, texts, and references, and the maintenance procedure.

Decisions in the basic design: per Basic Design Chapter 7, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies: per the chapter dependency table in Chapter 1 “Overall Architecture”.

To Legal: the range that does not constitute unauthorized practice of law.

## 8. Repository and Data Management

Decisions received: per the table at the head of Basic Design Chapter 8 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: what is placed in the Public Repository, and how data on the User's device is placed, protected, and exported.

Scope

- Covered: the structure of the Public Repository, the structure of data on the User's device, integrity, capacity, export and import, and dependence on GitHub.
- Not covered: a private repository (Design Plan D-12-1“The Public Repository is provided. No Private Repository is provided”; none is set up), collection of User information, sales history and sales contracts (out of scope).

What must be possible

- 8-1 Public Repository
  - R-8-1-1 It holds the software, the public page, and reference information, and no personal information.
  - R-8-1-2 Reference information and updates of modified versions can be distinguished from official ones.
- 8-2 Device data
  - R-8-2-1 The placement of Rights Holder Information, ledgers, evidence, and work data is determined.
  - R-8-2-2 It is guaranteed that they do not leave the device (except when the User exports them).
- 8-3 (Deleted: writing to a private repository was removed in Design Plan Edition 2)
- 8-4 Integrity
  - R-8-4-1 Tampering can be detected, history remains, and recovery from exports is possible.
- 8-5 Capacity and speed
  - R-8-5-1 A guide to device capacity is shown.
  - R-8-5-2 Distributions and reference information of the Public Repository can be obtained from mainland China as well.
- 8-6 (Deleted: Users are not added or removed)
- 8-7 Dependencies
  - R-8-7-1 There is a provision for when GitHub becomes unusable.

What to settle in the basic design

- The file layout of device data, the export format, and alternative routes.

Decisions in the basic design: per Basic Design Chapter 8, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies: per the chapter dependency table in Chapter 1 “Overall Architecture”.

To Legal: the terms of use of the Public Repository (GitHub).

## 9. Distribution and Updates

Decisions received: per the table at the head of Basic Design Chapter 9 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: the mechanisms by which Users obtain the app, can trust that it is official, and keep receiving updates.

Scope

- Covered: installation, proof of official builds, updates, download routes, and releases.
- Not covered: distribution through paid app stores (Microsoft Store, Mac App Store). Flathub for Linux and the browser extension stores are treated as routes alongside the distributables that NRSD distributes itself (Basic Design Chapter 9, 2.1 “Distributables per OS”; changed on 2026-10-01).

What must be possible

- 9-1 Installation
  - R-9-1-1 Non-technical people can install it on the three OSes.
  - R-9-1-2 The handling of device data on uninstall is determined.
- 9-2 Proof of official builds
  - R-9-2-1 It can be distinguished from fake apps.
  - R-9-2-2 The keys used for releases are protected.
- 9-3 Updates
  - R-9-3-1 It is updated without User operation, and what is received is verified.
  - R-9-3-2 Failure does not break it, and data is migrated.
  - R-9-3-3 There is a provision for when the update route is taken over.
- 9-4 Download routes
  - R-9-4-1 It can be obtained from the README without confusion, including from mainland China.
- 9-5 Releases
  - R-9-5-1 The required license notices are bundled.

What to settle in the basic design

- Installer formats, code signing, the update method, and the release procedure.

Decisions in the basic design: per Basic Design Chapter 9, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies: per the chapter dependency table in Chapter 1 “Overall Architecture”.

To Legal: export controls on software containing cryptography.

## 10. Screens and Design

Decisions received: per the table at the head of Basic Design Chapter 10 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: what must be possible on each screen, and the design policy.

Scope

- Covered: design policy, screen composition, languages, guidance, display considerations, OS integration, the feedback channel, and settings.
- Not covered: screens of the operator functions (handled in 1-10).

What must be possible

- 10-1 Design
  - R-10-1-1 The design policy (cute, a character) is set, and who judges it is determined.
  - R-10-1-2 Readability (text size, color vision) is considered.
- 10-2 Screen composition
  - R-10-2-1 The User operations of the requirements of all chapters (Chapters 2 to 9) exist on some screen.
- 10-3 Languages
  - R-10-3-1 It can be used in Japanese, Chinese, and English.
  - R-10-3-2 Kanji glyph forms and date and number notation suit the language and country.
- 10-4 Guidance
  - R-10-4-1 On first run, guidance is given on entering information for C2PA signatures, posting the Notice, agreeing to the Terms of Use, and not being involved in rights in source works (“What the System protects and does not protect”).
  - R-10-4-2 It is shown that this is not legal advice and that complaints may pass personal information to others (Chapter 7).
  - R-10-4-3 Errors and help are understandable to non-technical people.
- 10-5 Display considerations
  - R-10-5-1 Real names and key information are not shown on screen.
- 10-6 Integration with the OS
  - R-10-6-1 Whether to provide it is determined.
- 10-7 Feedback channel
  - R-10-7-1 Problems can be reported without looking at GitHub (the receiver is the operator functions; 1-10).
- 10-8 Settings
  - R-10-8-1 The setting items are determined.

What to settle in the basic design

- The list of screens and transitions, the design, and wording.

Decisions in the basic design: per Basic Design Chapter 10, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies: per the chapter dependency table in Chapter 1 “Overall Architecture”.

To Legal: none (receives what returns to the screens from Chapter 12 “Interface with Legal”).

## 11. Development Base

Decisions received: per the table at the head of Basic Design Chapter 11 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: what to build it with and how to maintain it.

Scope

- Covered: technology selection, libraries, licenses, development operations, and dependency safety.
- Not covered: none.

What must be possible

- 11-1 Selection
  - R-11-1-1 The conditions of Design Plan 6.2 are met.
- 11-2 Libraries
  - R-11-2-1 Quality and continued maintenance have been checked, and there is a provision for when maintenance stops.
- 11-3 Licenses
  - R-11-3-1 It complies with the licenses of third-party components and fonts.
  - R-11-3-2 NRSD's license is determined.
- 11-4 Development operations
  - R-11-4-1 It can be built on the three OSes, and wording is externalized.
- 11-5 Dependency safety
  - R-11-5-1 There are provisions for dependency updates and tampering.

What to settle in the basic design

- Concrete technologies and libraries, and the build environment.

Decisions in the basic design: per Basic Design Chapter 11, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies: per the chapter dependency table in Chapter 1 “Overall Architecture”.

To Legal: licenses, export controls.

## 12. Interface with Legal

Decisions received: per the table at the head of Basic Design Chapter 12 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: which laws, for which countries, are covered at which points of the System; what Users need to declare or register; what is taken as a source and what the System declares it follows.

Scope

- Covered: the list of laws covered, the points where they are covered, whether Users need to declare or register anything, the policy on sources, and disclaimers.
- Not covered: guaranteeing legal judgment. NRSD is not a lawyer. Rights in source works (“What the System protects and does not protect”).

Policy

- For each law, where the general interpretation was taken from (the source) is attached in the legal documents and linked from this document.
- The System does not deal with legal problems. It is enough that Users know “what to look at to be able to respond”.
- The country axis: Japan, China, and the US come first; others are accumulated in an “all countries” frame (Chapter 7 “Legal Action Guidance”).

What must be possible

- 12-1 Interface with Legal
  - R-12-1-1 The laws covered are listed on three axes—area of law, country, and point in the System—and each row links to the legal documents.
  - R-12-1-2 Whether Users need to declare or register anything is determined.
  - R-12-1-3 What returns to the screens (consent, disclaimers, rights in source works, passing on personal information) has been handed to Chapter 10.

Areas and points of law covered (countries: Japan, China, and the US first; others in the frame)

|Area of law|Countries|Points covered|
|---|---|---|
|Copyright (in the User's photographs)|Japan, China, US, others|Chapter 5 “Rights Documents”, Chapter 6 “Registration and Evidence Preservation”, Chapter 7 “Legal Action Guidance”, Chapter 12 “Interface with Legal”|
|Protection of rights management information|Japan, China, US, international, others|Chapter 3 “Signing”, Chapter 4 “Image Editing and Batch Application”, Chapter 5 “Rights Documents”, Chapter 6 “Registration and Evidence Preservation”, Chapter 7 “Legal Action Guidance”|
|Portrait rights and publicity rights|Japan, China, others|Chapter 5 “Rights Documents”, Chapter 7 “Legal Action Guidance”|
|Personal information protection (third-party information in evidence)|Japan (China, the EU, and other countries are handled when items are added to the Legal Research)|Chapter 6 “Registration and Evidence Preservation”, Chapter 8 “Repository and Data Management”|
|Personal information protection (feedback e-mails received by NRSD)|Japan, China, EU, others|Chapter 10 “Screens and Design”, Chapter 12 “Interface with Legal”|
|Treatment of electronic signatures and timestamps as evidence|Japan, China, US, others|Chapter 2 “Signing Information and Matching”, Chapter 3 “Signing”, Chapter 6 “Registration and Evidence Preservation”|
|Copying for evidence|Japan, China, US, others|Chapter 6 “Registration and Evidence Preservation”|
|Takedown request procedures|Japan, China, US, others|Chapter 7 “Legal Action Guidance”|
|Disclosure of sender information|Japan|Chapter 7 “Legal Action Guidance”|
|Handling of legal business (unauthorized practice of law)|Japan|Chapter 7 “Legal Action Guidance”|
|Export control (cryptography)|Japan, US, others|Chapter 9 “Distribution and Updates”, Chapter 11 “Development Base”|
|Labeling of AI generation and synthesis|China, others|Chapter 3 “Signing”|
|Software and font licenses|—|Chapter 4 “Image Editing and Batch Application”, Chapter 9 “Distribution and Updates”, Chapter 10 “Screens and Design”, Chapter 11 “Development Base”|
|Terms of sales platforms|—|Chapter 5 “Rights Documents”|
|GitHub terms of use|—|Chapter 8 “Repository and Data Management”|
|Terms of Use and Privacy Policy|Japan, abroad|Chapter 1 “Overall Architecture”, Chapter 10 “Screens and Design”, Chapter 12 “Interface with Legal”|
|Commissioning design and characters (if commissioned in the future; not commissioned as of 2026-09-30)|Japan|Chapter 10 “Screens and Design”|

Declarations and registrations by Users (whether required is checked by Legal)

- Consent to the Terms of Use and Privacy Policy (recorded on the device and not sent to NRSD)
- Posting the Notice (Chapter 2 “Signing Information and Matching”. Not a legal obligation but a clue for matching. The System does not collect User information and requires no registration)
- Work registration (Copyright Protection Center of China, etc.; optional)
- Copyright registration (in Japan, registration of the date of first publication and registration of the real name; in the US, registration; optional)

Decisions in the basic design: per Basic Design Chapter 12, 1 “List of Design Decisions” (not reproduced in this document).

Dependencies: per the chapter dependency table in Chapter 1 “Overall Architecture”.

## 13. Design Verification

Decisions received: per the table at the head of Basic Design Chapter 13 and Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” (not reproduced in this document).

What to define: the argument that this design achieves the purpose, and the declaration of gaps.

- 13-1 Coverage
  - By the mapping table of R-1-9-2“There is a mapping table from Design Plan decision numbers to this document's requirement numbers” (Basic Design Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design”), every decision number of the Design Plan corresponds to requirement numbers of this document.
  - Every row of the “Destination table” has been resolved or shows where it is resolved.
- 13-2 Reaching the purpose
  - For each usage scene (Design Plan 4.3 “Usage Scenes”), tracing requirement numbers runs uninterrupted from start to end.
  - How each chapter serves the purpose (protecting rights) and deterrence.
- 13-3 Consistency
  - For each threat (R-1-4-2“There is a list of threats (impersonation, fake apps, key theft, tampering)”), there is a corresponding requirement.
  - The dependency table in 1-9 has no one-way omissions, and mutually dependent chapters are designed together.
  - There are no overlaps or gaps in where data resides and in responsibilities. No scope line is crossed.
- 13-4 Declaration of gaps
  - Zero gaps is impossible. For each known gap, show why it cannot be closed, how far it was addressed, the remaining risk, and who bears it.
  - Gaps when a premise (“Premises”) breaks down.
  - The impact of unresolved legal points.
  - The list of gaps is per Basic Design Chapter 13, 4.1 “Gaps in the mechanism” (H-1 to H-48, with reasons, the extent addressed, remaining risks, and who bears them) (not reproduced in this document).
