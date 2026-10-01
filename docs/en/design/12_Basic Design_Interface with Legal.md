# Basic Design Document Chapter 12: Interface with Legal

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [12_Basic Design_Interface with Legal.docx](12_Basic%20Design_Interface%20with%20Legal.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

## Position of This Document

- Details Chapter 12 of the Outline Design Document. It is designed together with Chapter 10 (screens) (Chapters 10 and 12 depend on each other).
- The decisions received are as in the following table (the decisions, items to be investigated, and open items of Design Plan Edition 2, and omissions found in the item breakdown). The texts of the Design Plan's decisions, items to be investigated, and open items are per Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” and the destination table of the Outline Design Document (not reproduced in this chapter; only the numbers and the omissions found in the item breakdown are listed).

|Number|Type|Content|
|---|---|---|
|A-14|Omission found in the item breakdown|NRSD's position and the Terms of Use|
|A-16|Omission found in the item breakdown|Unauthorized practice of law (handled by Legal)|
|A-17|Omission found in the item breakdown|Cross-border data transfer (the premise disappeared with Edition 2, in which NRSD receives no User information)|
|A-18|Omission found in the item breakdown|Export controls on software containing cryptography (handled by Legal)|
|A-19|Omission found in the item breakdown|Consistency with the terms of sales platforms (handled by Legal)|
|A-21|Omission found in the item breakdown|Complaints passing the User's personal information to the other party|

- The content of the law (provisions, interpretations, sources) is written only in the Legal Research. This chapter sets “which laws are covered, where, and how”, the structure of the Terms of Use and Privacy Policy, declarations and registrations by Users, and what is returned to the screens.
- The System does not deal with legal problems. It suffices that Users know “what to look at in order to respond” (the policy of Chapter 12 of the Outline Design Document). NRSD is not a legal expert, and the arrangement in this chapter does not guarantee legal judgments.
- The System does not collect Users' information and is completed within the Client App (Design Plan Edition 2, D-5-1“The System does not confirm Users and collects no User information. No registration is required in order to use the System”, D-12-3“Rights Holder Information, the signing key, work data, the ledger, and evidence are kept on the User’s device; NRSD does not collect them”). All NRSD may receive from Users is feedback e-mail the Users send themselves (Chapter 10, DD-10-6“Feedback is sent by the User from their own e-mail”). This chapter treats that e-mail as handling of personal information (5).
- Rights to the original works (characters, etc.) are not handled (Outline Design Document “What the System protects and does not protect”).
- The course of study and the facts researched are in the study memo “Study of Chapter 12 Interface with Legal” (NRSD's internal record of study, not distributed with this document; the facts underlying the design are written with their sources in each section of this document).

## 1. List of Design Decisions

|Number|Decision|Reason|Rejected alternatives|
|---|---|---|---|
|DD-12-1|Interpretations and sources of law are placed only in the items of the Legal Research, and the design documents refer to them by item number and name. When the Legal Research is revised, the sections of the referring chapters (the “point where covered” in the table of 2) are reviewed (8)|Writing the same interpretation in several documents leads to inconsistencies. Laws are amended (R-7-3-1)|Writing summaries of provisions in each chapter|
|DD-12-2|The Terms of Use and Privacy Policy, as standard terms, are shown in full on the first-run screen (G-02“Consent”), and the consent operation is recorded on the device. The full text can always be read in G-23“About This App” and on the public page. Changes set an effective date and are announced on the public page and in the app's notifications at least 30 days (initial value) before the effective date (4.3)|The requirements of display, consent, and announcement of changes for making the Terms of Use and Privacy Policy part of the contract follow the arrangement of Legal Research L-25“Terms of Use (standard terms) and disclaimers (Japan)”|Deeming consent by merely starting the app for the first time (whether the display requirement is met could be disputed)|
|DD-12-3|If the User does not agree to the Terms of Use, or does not re-agree to changes, what is stopped is only obtaining reference information, which is NRSD's service (the bundled package can be used). C2PA signing, editing, export, registration, verification, viewing and extracting records, creating backups, and updates are not stopped. Users who have not agreed are shown entrances where they can agree at any time (G-20“Settings” and G-21“Notifications”)|The System's software is published under Apache-2.0 and is completed within the device. No consent is needed to run software (GPLv3 Section 9, the idea of criterion 10 of the Open Source Definition), and stopping functions completed within the device by consent would have no effect because modified versions could remove it. Similar public desktop software (GIMP, Krita, darktable, Inkscape, Joplin, Audacity) does not stop functions by consent. Joplin requires no terms for the app and applies terms only to Joplin Cloud (the publisher's online service). Firefox's terms also cover the executable, but in a form that deems use as consent, and it does not stop functions for not agreeing. No precedent was found of stopping functions completed within the device by consent (research of 2026-09-30). Stopping updates would prevent Users who have not agreed from receiving vulnerability fixes. Making records unextractable would hold Users' evidence hostage|Making the app entirely unusable without consent (terms of VOICEVOX and the like impose obligations on Users (prohibiting redistribution, attribution), while this case imposes no obligations on Users). Also stopping new creation of C2PA signatures and registrations (second draft; a gate not in the Design Plan, creating an operator-side restriction from legal research)|
|DD-12-4|Disclaimer clauses do not exempt all liability. Clauses exempting part of it are written to make clear that they apply only to slight negligence, “except in cases of NRSD's intent or gross negligence” (4.2)|The limits of the effect of disclaimer clauses (restrictions that apply to consumer Users even when provided free of charge) follow the arrangement of Legal Research L-25“Terms of Use (standard terms) and disclaimers (Japan)”|“Bears no liability whatsoever” (void). Relying only on the no-warranty clause of the software license (may be void against consumers)|
|DD-12-5|The software license (Chapter 11, 9 “NRSD's License (Decision on O-09“NRSD’s license type”)”) is placed as setting the conditions for using the copyright in the source code and program (copying, modification, redistribution). The relationship in which NRSD distributes the official version and delivers reference information and updates is set by the Terms of Use. The Terms of Use state this division explicitly|How far the license's no-warranty clause works against consumer Users follows the arrangement of Legal Research L-25“Terms of Use (standard terms) and disclaimers (Japan)”. The relationship between those who distribute modified versions and NRSD is separate from the relationship between users of the official version and NRSD (Design Plan D-12-2“The software is published, and its being taken away is not prevented”)|Setting the usage relationship only by the license. Setting the use of the source code only by the Terms of Use (does not fit the practice of public software)|
|DD-12-6|NRSD treats itself as a business handling personal information insofar as it receives feedback e-mails, and publishes the matters of Article 32 of the Act on the Protection of Personal Information in the Privacy Policy (5)|A mailbox can be searched by computer and may constitute a personal information database (Legal Research L-11“Personal information protection (User information) (Japan)”). Even if not, publishing does no harm|Writing only “Information collected: none” (omits receiving e-mail)|
|DD-12-7|The governing law is Japanese law, with a statement that it does not prevent the application of mandatory rules of the country where the User resides. Jurisdiction is the courts of Japan, with a statement that it does not prevent consumer Users from suing in the courts of the country of their domicile|Consumer Users abroad retain the mandatory rules of their habitual residence and the courts of their domicile country (Legal Research L-31“Contracts with foreign consumers (governing law and jurisdiction) (Japan)”)|Exclusive jurisdiction of Japanese courts (not effective against consumers abroad)|
|DD-12-8|The System uses only public standard cryptography and does not implement non-standard cryptography. It publishes the source code and distributes distributables free of charge without restriction. These conditions are checked every time in the release procedure (Chapter 9, 6.3 “Export control of software containing cryptography”)|The treatment under Japanese and U.S. export control of publishing the source code of public standard cryptography and distributing it free of charge follows the arrangement of Legal Research L-14“Export control of software containing cryptography (Japan, US)”|Sending notification to the United States every time (unnecessary since the 2021 amendment). Making our own cryptography (requires notification and review)|
|DD-12-9|Generative AI values are not given to the digital source types of C2PA action records (Chapter 3, 3.4 “Recording of actions”)|The auto-placement model only chooses positions and does not generate pixels. Posting sites might read the metadata and label Users' photos as “made with AI” (Legal Research L-30“Labeling of AI-generated and synthesized content (China)”)|—|
|DD-12-10|The wording of Rights Documents (country parts of the Enclosed Document, sample Notice texts) and guidance model texts is distributed only after review by experts versed in that country's law|Design Plan 9.5 “Confirmation by experts is required before finalizing the wording”, I-08“Wording of Rights Documents”|Distributing without review|

## 2. List of Laws Covered

- The table “area of law, country, point in the System” in Chapter 12 of the Outline Design Document is made concrete with items of the Legal Research and sections of the basic design. In the country column, “JP” is Japan, “CN” China, “US” the United States, and “EU” the European Union.

|Area of law|Country|Item of the Legal Research|Point where covered (chapter and section of the basic design)|Design response|
|---|---|---|---|---|
|Copyright (Users' photos)|JP, CN, US|L-32“Copyright infringement and remedies (Japan, China, US)”, L-20“Limitation periods for damages claims (Japan, China, US)”, L-22“Copyright registration (Japan)”, L-23“Work registration (China)”|Chapter 5 (Rights Documents), Chapter 6 (keeping evidence), Chapter 7 (contact points), 3 of this chapter|Rights belong to Users. Evidence is kept unless the User deletes it, and the period for claims is shown before deletion. Registration is optional and guided|
|Protection of rights management information|JP, CN, US, international|L-1“Removal or alteration of rights management information (Japan)”, L-2“Removal or alteration of rights management information (China)”, L-3“Removal or alteration of rights management information (US)”, L-4“Protection of rights management information (treaty) (international)”|Chapter 3 (manifest, invisible watermark, identification number), Chapter 4 (Visible Signature), Chapter 5 (statements in the Enclosed Document), Chapter 6 (matching), Chapter 7 (model texts)|Information that is the basis for claims about removal and alteration is put in the C2PA signature, invisible watermark, and Enclosed Document. In Japan, China, and the United States alike, removal for technical reasons and unintentional removal are outside the scope, declared as a gap (Chapter 3 “Signing”, Chapter 13, H-5“Removal of rights management information by format conversion or recompression is excluded in Japan from deemed infringement, excluded in China as technically unavoidable, and in the US requires intent and knowledge of the link to infringement”)|
|Portrait rights, publicity rights|JP, CN|L-5“Portrait rights and publicity rights (Japan)”, L-6“Portrait rights (China)”|Chapter 5 (portrait caution in the Enclosed Document), Chapter 7 (priority for commercial use)|Portrait rights belong to the person depicted; the Enclosed Document only shows a caution|
|Protection of personal information (third parties' information captured in evidence)|JP (no items in the Legal Research for CN, the EU, or other countries; evidence exists only on the User's device and NRSD does not receive it)|L-34“Third-party personal information in evidence (Japan)”|Chapter 6 (evidence), Chapter 8 (not leaving the device)|Third parties' information captured in evidence is kept only on the User's device, and NRSD does not receive it. Whether the User is a business handling personal information depends on the User's circumstances, so no judgment is made|
|Protection of personal information (feedback e-mails NRSD receives)|JP, CN, EU|L-11“Personal information protection (User information) (Japan)”, L-12“Personal information protection and cross-border transfer (China)”, L-13“Personal information protection (extraterritorial application) (EU)”|Chapter 10, 9 “Feedback Channel”, 5 of this chapter|Publish the matters of Article 32. The laws of Users' countries abroad are operational issues (7)|
|Evidential treatment of electronic signatures and timestamps|JP, CN, US|L-15“Treatment of timestamps as evidence (China)”, L-16“The timestamp system (Japan)”, L-24“Treatment of electronic signatures and electronic data as evidence (Japan, China, US)”|Chapter 2 (keys kept in the OS keystore), Chapter 3 (TSA), Chapter 6 (list of SHA-256 of the evidence package)|Keys are made usable only by the User. The assessment of the evidential treatment of electronic signatures and electronic data follows the Legal Research (guidance does not say “a presumption applies”; 7)|
|Copying for evidence|JP, CN, US|L-21“Copying for evidence (Japan, China, US)”|Chapter 6 (fetching one page only)|No explicit exception could be confirmed for China; confirmation by experts is needed|
|Procedures for takedown requests|JP, CN, US|L-7“Takedown request procedures (Japan)”, L-8“Takedown request procedures (China)”, L-9“Takedown request procedures (US)”|Chapter 7 (contact points, model texts, guidance on personal information, the complainant's standing)|Complaints are sent by the User. The complainant's standing (copyright holder, subject, agent) is chosen first|
|Disclosure of sender information|JP|L-33“Disclosure of sender information (Japan)”|Chapter 7, 4 “References to Each Country's Law”|A court procedure; the System does not perform it. References and export are shown|
|Handling legal services (unauthorized practice of law)|JP|L-10“Handling of legal business (unauthorized practice of law) (Japan)”|Chapter 7, 7 “Where the Line Is Drawn”|Free of charge, no leading to paid legal services, only displaying model texts|
|Export control (cryptography)|JP, US, others|L-14“Export control of software containing cryptography (Japan, US)”|Chapter 9, 6.3 “Export control of software containing cryptography”, Chapter 11 (components)|DD-12-8“Only public standard cryptography is used, distributed openly and free of charge”|
|Labeling of AI generation and synthesis|CN|L-30“Labeling of AI-generated and synthesized content (China)”|Chapter 3, 3.4 “Recording of actions”|DD-12-9“No generative-AI values are attached”|
|Software and font licenses|—|L-19“Font licenses”|Chapter 4, 6 “Fonts and Emoji”, Chapter 10 (screen fonts), Chapter 11, 9 “NRSD's License (Decision on O-09“NRSD’s license type”)”, Chapter 9 (bundling)|OFL fonts are bundled unmodified. The list of licenses is made automatically|
|Terms of sales platforms|—|L-17“Terms of sales platforms”|Chapter 5 (Permitted Scope)|The Permitted Scope in the Enclosed Document works as the “creator's permission” under Patreon's terms|
|GitHub's terms of service|—|L-18“GitHub terms”|Chapter 8, 3 “Public Repository”|No personal information or repost information is placed in the Public Repository|
|Terms of Use, Privacy Policy|JP, abroad|L-25“Terms of Use (standard terms) and disclaimers (Japan)”, L-28“Telecommunications Business Act (notification, external transmission rules) (Japan)”, L-31“Contracts with foreign consumers (governing law and jurisdiction) (Japan)”|Chapter 1, 4.2 “Communication with the outside”, Chapter 10 (G-02“Consent”, G-21“Notifications”, G-23“About This App”), 4 and 5 of this chapter|Display, consent, and announcement as standard terms. Limiting the scope of disclaimers|
|Commissioning design and characters (if commissioned in the future)|JP|L-29“Commissioning design and characters (Japan)”|Chapter 10, DD-10-2“Design is done and judged by NRSD's Lead Developer”|Not commissioned now. The contractual issues for commissioning are placed in the Legal Research. Not imitating existing characters|

## 3. Declarations and Registrations by Users

### 3.1 Required for using NRSD's services

|Matter|Content|Screen|Basis|
|---|---|---|---|
|Consent to the Terms of Use and Privacy Policy (required for obtaining reference information; not needed for other functions; DD-12-3“Consent gates only the fetching of reference information”)|Display of the full text and a consent operation. The version agreed to and the date/time are recorded on the device (not sent to NRSD)|G-02“Consent”|L-25“Terms of Use (standard terms) and disclaimers (Japan)”|

- Posting the Notice (posting the notice code on the Notice account) is not a legal obligation but a premise of the System's effectiveness as a matching clue, and is recommended in G-04“Posting the Notice” (Chapter 2, DD-2-3“The clue to Entitlement is the notice code posted on the Notice account”). The System can be used without posting.
- The input for C2PA signatures (handle name, role, Notice accounts) is used to make certificates on the User's device, and NRSD does not receive it (Chapter 2, DD-2-1“The only inputs for C2PA signing are handle name, role, and Notice accounts”). E-mail, name, address, official ID, age, and country of residence are not asked for.

### 3.2 Optional for Users (the System only guides)

- The System does not make the following registrations on the User's behalf. Under “To prepare further” in the action guide (G-17“Action Guide”), summaries of the systems and the official contact points are shown.

|Matter|Country|Summary given|What is shown as a caution|Basis|
|---|---|---|---|---|
|Registration of the date of first publication, etc.|JP|The work is presumed to have been first published on the registered date. Registration tax JPY 3,000 per registration|—|L-22“Copyright registration (Japan)”|
|Registration of the real name|JP|The registrant is presumed to be the author. Registration tax JPY 9,000 per registration|The real name goes into the public register and can be searched. Users whose policy is not to reveal their real name should first consider registration of the date of first publication, etc.|L-22“Copyright registration (Japan)”, R-2-5-2|
|Work registration|CN|It serves as preliminary evidence of the ownership of rights. For foreign authors, it is under the jurisdiction of the National Copyright Administration|How the name is written on the registration certificate|L-23“Work registration (China)”|
|Copyright registration|US|Works from outside the United States do not require registration to sue. Statutory damages require timely registration. Published photos: up to 750 from the same year for USD 55 per registration|There are court decisions holding that claims for removal of rights management information (Section 1202) do not require registration|Legal Research, Section 3 “Declarations and Registrations by Users (Whether Required)”|

- Work registration numbers obtained can be written in the Notice (Chapter 5, 6.1 “Levels of sample text”) and the Visible Signature (Chapter 4, 3.1 “Content and placeholders”).
- The User is told that the System's work data (identification number, date/time of the C2PA signature, timestamp) can be used as clues to “the date of publication” and “identification of the work” when applying for registration. Preparing the application is not assisted (the same line as Chapter 7, DD-7-1“Guidance shows contact points, required items, and model texts”).

## 4. Terms of Use

- The text itself is made in the legal documents and finalized after review by experts. This chapter sets the clauses the design requires, the policy on their content, and the chapters that are their basis. The draft wording in 4.2 is a draft showing the design intent, not finalized text.

### 4.1 List of clauses

|Clause|Content|Basis chapter|
|---|---|---|
|Title and preamble|These Terms set out the use of the official version of the System distributed by NRSD (the provider's name and address), and the provision of reference information and updates|DD-12-5“The license and the Terms of Use are written separately”|
|Definitions|The System, official version, reference information, User, Rights Holder, C2PA signature, Visible Signature, notice code, identification number|Chapter 1, 16 “Glossary”|
|Relationship with the license|The conditions for using the copyright in the source code and program follow the bundled license. These Terms set out the relationship between NRSD and users of the official version. Which prevails if the license and these Terms conflict is set after confirmation by experts|DD-12-5“The license and the Terms of Use are written separately”|
|Purpose and scope of the System|It is a tool to protect the rights to Users' photos and does not handle the rights to the original works. It does not solve legal problems on Users' behalf|Outline Design Document “What the System protects and does not protect”, Chapter 7, 7 “Where the Line Is Drawn”|
|Consent to the Terms|Consent on the first-run screen. The treatment of minor Users is NRSD's operational decision (7), and whether to put it in a clause depends on that decision|DD-12-2“Terms of Use and Privacy Policy are agreed as standard terms”|
|Users' undertakings|C2PA-sign only photos to which one holds the rights (or has been authorized by the Rights Holder). Do not C2PA-sign by impersonating others. Do not enter others' Notice accounts as one's own|Chapter 2 “Signing Information and Matching”|
|Content of the services provided|It provides means of matching but does not match on others' behalf and does not judge. Agreement of matching clues is probabilistic and does not prove that one is the Rights Holder. Reference information (contact points, sample texts, references to each country's law) is general information and not legal advice on individual cases|Chapter 2, 4 “The Means of Matching and the Clues”, Chapter 6 “Registration and Evidence Preservation”, Chapter 7, 7 “Where the Line Is Drawn”, Design Plan D-5-3“Acts of a person posing as the Rights Holder (signing first, replacing signatures, and reverse complaints) are treated as impossible to prevent; clues for matching are presented. No judgment is made”, D-5-5“The System provides the means of matching but does not perform matching on anyone’s behalf”|
|Users' records|Records are on the User's device and managed by the User. NRSD does not receive them and cannot restore them. Creating backups is recommended|Chapter 8 “Repository and Data Management”|
|Disclaimer|Per the policy of the draft wording in 4.2|DD-12-4“The scope of disclaimers is stated explicitly”|
|Change and termination of services|If distribution of reference information and updates ends, it is announced at least 90 days (initial value) before the end date. Even after termination, the records on the device and the app already installed can be used|Chapter 1, 18 “Operations”, Chapter 8 “Repository and Data Management”|
|Changes to the Terms|Per the procedure of 4.3|DD-12-2“Terms of Use and Privacy Policy are agreed as standard terms”|
|How to contact|Feedback e-mail (G-22“Help and Feedback”). Announcements from NRSD are on the public page and in the app's notifications|Chapter 10, 9 “Feedback Channel”|
|Governing law and jurisdiction|Per the policy of the draft wording in 4.2|DD-12-7“The governing law is Japanese law without precluding mandatory rules of the User's country”|
|Language|The Japanese version is authoritative, and Chinese and English are translations. Differences in translation are not interpreted to the User's disadvantage|The clause on handling translations (translations of the Terms are also distributed only after review by experts, as in DD-12-10“The wording of Rights Documents is distributed only after expert review”)|
|Effective date and revision history|The version, effective date, and history of changes are placed at the end|DD-12-2“Terms of Use and Privacy Policy are agreed as standard terms”|

### 4.2 Draft wording of the key clauses (drafts)

- Disclaimer (DD-12-4“The scope of disclaimers is stated explicitly”)
  - “NRSD does not warrant that the official version of the System and the reference information are fit for the User's purpose, accurate, complete, or continuous.”
  - “Except in cases of NRSD's intent or gross negligence, NRSD is liable for damage arising to the User in connection with the use of the System only to the extent of direct and ordinary damage.”
  - Notes on writing: do not write “bears no liability whatsoever”. Do not place a clause that “NRSD decides whether it is liable” (Consumer Contract Act Article 8(1)(i) and (iii)).
- Relationship with the license (DD-12-5“The license and the Terms of Use are written separately”)
  - “The conditions for copying, modifying, and redistributing the source code and program of the System follow the license bundled with the System. These Terms set out the use of the official version distributed by NRSD, and the provision of reference information and updates by NRSD.”
- Governing law and jurisdiction (DD-12-7“The governing law is Japanese law without precluding mandatory rules of the User's country”)
  - “These Terms are governed by the laws of Japan. However, this does not prevent the application of provisions of the laws of the country where the User resides whose application cannot be excluded by agreement of the parties.”
  - “The courts of Japan shall have jurisdiction of first instance over disputes concerning these Terms. However, this does not prevent a User who is a consumer from bringing an action in the courts of the country where the User is domiciled.”
- If not agreed (DD-12-3“Consent gates only the fetching of reference information”)
  - “If the User does not agree to these Terms or their changes, NRSD does not provide reference information to that User. The other functions of the System can be used in accordance with the conditions of the bundled license.”

### 4.3 Procedure for changes

|Step|What is done|Deadline (initial value)|
|---|---|---|
|1|Decide the content of the change and record whether it falls under item 1 (a change conforming to the general interests of users) or item 2 (a change not contrary to the purpose and reasonable) of Article 548-4(1) of the Civil Code|—|
|2|Set the effective date|30 days or more after the date of announcement|
|3|Publish the full text of the new version and a summary of the changes on the public page and in the reference information (`reference/<version>/` of Chapter 8, 3.1 “Structure”)|30 days before the effective date|
|4|When the app takes in the reference information, it shows the change in G-21“Notifications”|—|
|5|At the first start after the effective date, show G-02“Consent” to ask for re-consent. If not re-agreed, as in DD-12-3“Consent gates only the fetching of reference information”|—|

- A change under item 2 does not take effect unless announced by the effective date (Legal Research L-25“Terms of Use (standard terms) and disclaimers (Japan)”). Users who do not obtain reference information (Users who keep using it offline) do not receive the announcement within the app. Announcement on the public page is treated as the announcement, and the app shows it at the next connection.
- Requests for display of the full text (Article 548-3 of the Civil Code) are met by always showing the full text in G-23“About This App” and on the public page. Past versions are also kept on the public page.

## 5. Privacy Policy

### 5.1 List of clauses

|Clause|Content|Basis|
|---|---|---|
|Business operator|NRSD's name, address, and the name of its representative|Act on the Protection of Personal Information Article 32(1)(i)|
|Information the System does not send to NRSD|Information for C2PA signatures, signing keys, photos, work data, case records, evidence, app usage records. These are kept only on the User's device|Chapter 1, DD-1-2“NRSD has no servers and receives no User information (except feedback e-mails Users send themselves)”, Chapter 8, 2 “Device Data”|
|Information NRSD receives|Feedback e-mails the User sends themselves (e-mail address, body, and the app version, OS, and error codes the User attached)|Chapter 10, 9 “Feedback Channel”|
|Purpose of use|Replying to feedback, improving the System. Not used for other purposes|Articles 17, 21, 32(1)(ii)|
|Provision to third parties|None (except as required by law)|Article 27|
|Retention period|Deleted one year (initial value) after the reply is completed. For improvement records, only summaries excluding e-mail addresses and names are kept|Article 22 (effort to erase personal data no longer needed)|
|Security measures taken|Two-factor authentication on the mailbox, limiting those who can view it to NRSD's persons in charge, device encryption|Enforcement Order Article 10(i)|
|External environment|The name of the e-mail receiving provider and the name of the country where data is kept (stated when NRSD decides the receiving means)|Personal Information Protection Commission guidelines (understanding the external environment), Legal Research L-11“Personal information protection (User information) (Japan)”|
|Procedure for requests for disclosure, etc.|Accepted by e-mail. Identity is confirmed by the request e-mail being sent from the same address|Article 32(1)(iii)|
|Where to make complaints|NRSD's e-mail address|Enforcement Order Article 10(ii)|
|Sending outside the device|The list of 5.2|L-28“Telecommunications Business Act (notification, external transmission rules) (Japan)”|
|Changes|The same procedure as 4.3|DD-12-2“Terms of Use and Privacy Policy are agreed as standard terms”|

### 5.2 List of what is sent outside the device

- The table of Chapter 1, 4.2 “Communication with the outside” is published in words for Users. No sending for analytics or advertising.

|Destination|Information sent|When|User information visible to the other party|
|---|---|---|---|
|Public Repository (GitHub, United States)|Requests to obtain updates and reference information|At start and every 24 hours while running (Chapter 9, 4 “Updates”)|IP address. The request header (User-Agent) is the System's name and version when obtaining reference information and TUF metadata (Chapter 1, 4.2 “Communication with the outside”), and the update component's name and version when obtaining update components (Chapter 9, 4.1 “Flow and states”)|
|Hong Kong mirror (Alibaba Cloud)|Same|When the Public Repository cannot be reached|Same|
|Timestamp providers|Hashes of photos or of evidence lists (photos themselves are not sent)|At export, at repost registration, when connecting after being offline|IP address (and, for paid TSAs set up by the User, the User's account)|
|RDAP, DNS|Domain names and IP addresses being looked up|At repost registration|IP address|
|Reposted pages|Requests to fetch the page|At registration (default; the User can turn it off for that registration only) and only when the User presses “Check current status”|IP address. Visible to the operator of the other site|
|E-mail software|Feedback text|Only when the User decides to send|The whole e-mail the User sends|
|The User's Cloud sync folder (when the User chose it as a backup location; carried by the OS's sync mechanism)|Backup files encrypted with the passphrase and the recovery key, the reference information package (Chapter 8, 5.5)|At every automatic backup|To the Cloud provider: the User's account and the encrypted files (the contents cannot be read)|
|The User's own other device (the same personal root; Chapter 1, 4.2)|Record rows, evidence and asset files, templates, work sessions, device-independent settings, the reference information package (encrypted end to end; Chapter 8, 5.4)|Automatically while running, after linking in “Devices”|IP address and device number (between the User's own devices)|
|The device of a “verified” counterparty (Chapter 2, 5.5)|Authorizations, revocations, joint-rights documents, renewal documents, the reference information package (encrypted end to end)|Automatically while running, after marking the counterparty “verified”|IP address and device number. The User's records are not sent|
|Relays for direct connections (public infrastructure; iroh's relays)|End-to-end encrypted contents (the relay cannot read them)|When another device or counterparty cannot be reached directly. Can be turned off in settings (G-20“Settings”)|IP address and a random device number (not linked to the notice code)|
|The public DHT (pkarr)|A record of the User's own device number, signed with a key derived from the passphrase (expiry 10 minutes)|Only when connecting to a counterparty or device by passphrase (Chapter 8, 5.4)|IP address and a random device number|

- The destination providers handle IP addresses under their own privacy policies. NRSD does not receive Users' information from the destination providers.

## 6. What Is Returned to the Screens

- Handed to Chapter 10 “Screens and Design”. Screen numbers follow the list of screens in Chapter 10.

|What is returned|Screen shown|How shown|Basis|
|---|---|---|---|
|Not being involved in the rights of the original works|G-01“Welcome” (first run), G-23“About This App”, public page|Shown at first run as one of the key points|Outline Design Document “What the System protects and does not protect”|
|Consent to the Terms of Use and Privacy Policy|G-02“Consent”|Key points, full text, version and effective date, consent check. The version agreed to is recorded on the device|DD-12-2“Terms of Use and Privacy Policy are agreed as standard terms”|
|What can be done without consent|G-02“Consent”|“If you do not agree, NRSD's reference information will not be obtained. Other functions can be used” shown above the consent buttons|DD-12-3“Consent gates only the fetching of reference information”|
|Announcement of changes to the terms|G-21“Notifications”, at start|The effective date and a summary of the changes. Re-consent is asked at the first start on or after the effective date|4.3|
|Full text of the terms and policy|G-23“About This App”|The full text of the current version, and guidance to past versions on the public page|Civil Code Article 548-3|
|Not being legal advice|G-17“Action Guide”, G-16“Case Details”|Always shown at the bottom of the screen|Chapter 7, 7 “Where the Line Is Drawn”|
|Personal information may be passed on in complaints|G-17“Action Guide”|A confirmation check before displaying model texts|Chapter 7, 3.2 “Guidance that personal information is passed to the other party”|
|Matching results are probabilistic|G-13“Verify”, G-14“Register a Repost”, G-24“Compare Clues”|Levels of match and “the final judgment is made by the User”|Chapter 6 “Registration and Evidence Preservation”|
|Cautions on saving reposted pages|G-14“Register a Repost”|The risk of saving pages containing illegal content, fetching only the one specified page, the IP address being visible to the other party|L-21“Copying for evidence (Japan, China, US)”, Chapter 1, 4.2 “Communication with the outside”|
|Real-name registration makes the real name public|G-17“Action Guide” (To prepare further)|A caution box|3.2 of this chapter|
|C2PA signatures disappear on social media|G-11“Export: Run and Results”|A word per destination|Chapter 3, 10.3 “Retention on posting sites and in the cloud”|
|Licenses of imported fonts|G-25“Image Editing” (importing fonts)|Once at import, that “the conditions for using fonts are for the User to check”|Chapter 4, 6.4 “Imported fonts”, L-19“Font licenses”|
|Handling of feedback e-mails|G-22“Help and Feedback”|Before sending, a sentence on the information NRSD receives and the purpose of use, and guidance to the Privacy Policy|5.1|
|Licenses|G-23“About This App”|The System's license and the list of third-party licenses|Chapter 9 “Distribution and Updates”, Chapter 11, 9 “NRSD's License (Decision on O-09“NRSD’s license type”)”|

## 7. Legal Issues (Placed in the Legal Research as Material for Operational Decisions)

- The following issues are not design decisions but are placed in the Legal Research as material for NRSD's operational decisions. The design does not restrict the range of the System's Users or the publication of functions on account of them.

|Issue|Item of the Legal Research|
|---|---|
|The arrangement that the guidance does not constitute unauthorized practice of law (NRSD's interpretation based on the Ministry of Justice's guidelines)|L-10“Handling of legal business (unauthorized practice of law) (Japan)”|
|Whether the presumption of Article 3 of the Electronic Signatures Act applies (guidance does not say “a presumption applies”)|L-24“Treatment of electronic signatures and electronic data as evidence (Japan, China, US)”|
|Copying for evidence in China|L-21“Copying for evidence (Japan, China, US)”|
|Whether the System constitutes a telecommunications business|L-28“Telecommunications Business Act (notification, external transmission rules) (Japan)”|
|Regarding feedback e-mails, whether a domestic representative in China (Personal Information Protection Law Article 53) or a representative in the EU and UK (GDPR Article 27) is needed|L-12“Personal information protection and cross-border transfer (China)”, L-13“Personal information protection (extraterritorial application) (EU)”, L-27“Representatives of foreign businesses and applicability thresholds (EU, UK, US)”|
|Treatment of minor Users|L-26“Minor Users (Japan, China, US)”|
|The precedence between the software license and the Terms of Use, and the wording of disclaimers|L-25“Terms of Use (standard terms) and disclaimers (Japan)”|
|Confirming that it does not fall under the scope of China's App filing|The addendum to L-30“Labeling of AI-generated and synthesized content (China)”|
|If donations or support are accepted, whether this touches the premise that the guidance is free (the arrangement that it does not constitute unauthorized practice of law) (not accepted as of 2026-09-30)|L-10“Handling of legal business (unauthorized practice of law) (Japan)”|

## 8. Reviewing the Legal Research

|Trigger|What is done|Parts of the design reviewed|
|---|---|---|
|Quarterly check of reference information (Chapter 7, 2.2 “Keeping up to date”)|Check amendments to the laws and terms that are sources of each item of the Legal Research|The “point where covered” in the table of 2|
|Notice of revisions to the terms of posting sites and sales venues|Revise L-17“Terms of sales platforms” and L-18“GitHub terms”|Chapter 5, Chapter 8|
|Before release|Check the conditions of L-14“Export control of software containing cryptography (Japan, US)” (standard cryptography only, published, free of charge)|Chapter 9, 6.3 “Export control of software containing cryptography”|
|Changes to the Terms of Use and Privacy Policy|The procedure of 4.3|Chapter 10, G-02“Consent”, G-21“Notifications”, G-23“About This App”|

- The version of the Legal Research and the date of revision are stated at the top of the Legal Research.

## 9. Mapping to Requirements

|Requirement number (text in the Outline Design Document)|Sections in this chapter|
|---|---|
|R-12-1-1|2 “List of Laws Covered”|
|R-12-1-2|3 “Declarations and Registrations by Users”|
|R-12-1-3|6 “What Is Returned to the Screens”|

## 10. Gaps Declared in This Chapter

- The gaps of this chapter follow the table in Chapter 13, 4.1 “Gaps in the mechanism” (the rows whose chapter column is this chapter; with why they cannot be closed, the extent addressed, the remaining risks, and who bears them) (not reproduced in this chapter).

## 11. Corrections to Other Chapters and the Outline Design Document

- In “Declarations and Registrations by Users” of Chapter 12 of the Outline Design Document, merge the duplicated “posting of the Notice” into one, and add “the System does not collect Users' information and does not ask for registration”.
- In the table of areas of law in Chapter 12 of the Outline Design Document, set the countries of “Export control (cryptography)” to “JP, US, others”, and add “Labeling of AI generation and synthesis (CN)”.
- Revise “no legal issues concerning Users' personal information arise” in “Decisions in the basic design (summary)” of Chapter 12 of the Outline Design Document to “All NRSD receives is feedback e-mails Users send themselves, and their handling is published in the Privacy Policy”.
- The [To be confirmed] on the digital source type in Chapter 3, 3.4 “Recording of actions” was decided as `humanEdits` (DD-12-9“No generative-AI values are attached”).
- Add to Chapter 10, G-02“Consent” what can be done without consent (DD-12-3“Consent gates only the fetching of reference information”). Add to G-22“Help and Feedback” a sentence on the purpose of use before sending.
- Descriptions in which the premises of Edition 1 of the Legal Research (accepting Users, e-mail registration, NRSD's private certificate authority, user repositories, shared ledgers) remained were revised to fit Edition 2 (L-11“Personal information protection (User information) (Japan)”, L-12“Personal information protection and cross-border transfer (China)”, L-14“Export control of software containing cryptography (Japan, US)”, L-18“GitHub terms”, L-24“Treatment of electronic signatures and electronic data as evidence (Japan, China, US)”, L-25“Terms of Use (standard terms) and disclaimers (Japan)”, L-28“Telecommunications Business Act (notification, external transmission rules) (Japan)”, Section 3 “Declarations and Registrations by Users (Whether Required)”).
