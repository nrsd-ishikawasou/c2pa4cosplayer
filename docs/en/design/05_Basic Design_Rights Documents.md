# Basic Design Document Chapter 5: Rights Documents

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [05_Basic Design_Rights Documents.docx](05_Basic%20Design_Rights%20Documents.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

## Position of This Document

- Details Chapter 5 of the Outline Design Document. Decided together with Chapter 3 (recording in the manifest) (Chapters 3 and 5 depend on each other).
- The decisions received are as in the following table (the decisions, items to be investigated, and open items of Design Plan Edition 2, and omissions found in the item breakdown).

|Number|Type|Content|
|---|---|---|
|D-9-1|Decision|The Rights Holder chooses, at the time of sale, how far rights are granted|
|D-9-2|Decision|The Permitted Scope chosen at the time of sale is the criterion for judging what a repost infringes|
|D-9-3|Decision|Delivery Images are accompanied by an Enclosed Document stating the matters in Section 9.2|
|D-9-4|Decision|The Enclosed Document includes a sentence stating that the images must not be passed to others without permission. Its loss is not due to the fault of the Rights Holder|
|D-9-5|Decision|Information covering all countries is published as a link destination, and text versions are enclosed only for the nationalities common among purchasers|
|D-9-6|Decision|The party pursued is the Infringer, regardless of whether it is a purchaser|
|I-06|To Be Investigated|Differences in rights by country|
|I-07|To Be Investigated|Knowing the purchaser’s country|
|I-08|To Be Investigated|Wording of Rights Documents|
|O-03|Open (a proposal is placed in Basic Design Chapter 5, 3.5 “Legal position (handled in legal)”; adoption depends on confirmation by experts)|Approach to governing law|
|O-04|Open (decided in Basic Design Chapter 5, 2.1 “Options (decision on O-04“Options for the Permitted Scope”)”)|Options for the Permitted Scope|
|A-2|Omission found in the item breakdown|Functions supporting the Notice (the Notice is the premise of effectiveness, and Entitlement matching relies on it)|
|A-19|Omission found in the item breakdown|Consistency with the terms of sales platforms (handled by Legal)|

- The references are as in the following table.

|Reference|What is referred to|
|---|---|
|Legal Research L-1“Removal or alteration of rights management information (Japan)” to L-9“Takedown request procedures (US)”|Rights management information, portrait rights, procedures for takedown requests (Japan, China, the United States)|
|Legal Research L-17“Terms of sales platforms”|Terms of sales venues (Patreon)|
|Chapter 3, 3.1 “Fields recorded”|Recording in the manifest (rights information, whether AI training is allowed)|
|Chapter 1, DD-1-6“NRSD signs reference information and places it in the Public Repository and the mirror”|NRSD signing and distributing the reference information|

- For the wording (the provisions of Enclosed Documents and Notice texts), this document sets the structure and how they are made; the provisions themselves are written by NRSD as reference information and are reviewed by experts (I-08“Wording of Rights Documents”). The sample sentences in this document are outlines to explain the structure, not final provisions.

## 1. List of Design Decisions

|Number|Decision|Reason|Rejected alternatives|
|---|---|---|---|
|DD-5-1|The Permitted Scope is chosen as a “basic scope (one radio button)” and “additional permissions (checkboxes)”. Commercial use is not among the options and always “requires separate permission”|Non-technical people can understand it at a glance (Design Plan 8.2 “Screens”, 9.1 “easy-to-understand methods such as radio buttons”). The conditions of commercial permission (consideration, period, media, number of copies, etc.) are decided per transaction and cannot be expressed as options. Selecting and displaying model forms and reflecting input are ordinarily not legal services, ordinary contracts without disputes are in many cases considered not to involve a legal case (the Ministry of Justice's guidelines), and the System is free of charge, so the reason for not providing commercial options is not the unauthorized practice of law|Choosing everything with checkboxes (too many combinations; confusing)|
|DD-5-2|The Enclosed Document is divided into a “common part” (statements of fact independent of language) and a “country part” (summaries of the country's law and references to contact points); the common part is enclosed in all languages and the country part as TXT only for the chosen countries. Complete coverage is shown by a link to the public page|Design Plan 9.3 “Countries Covered” (countries, not languages), D-9-5“Information covering all countries is published as a link destination, and text versions are enclosed only for the nationalities common among purchasers” (complete coverage by link, TXT only for common nationalities)|Writing the full text per country (much duplication)|
|DD-5-3|The Enclosed Document places the sentence “do not give or receive the images to or from others without permission” near the top of the common part (D-9-4“The Enclosed Document includes a sentence stating that the images must not be passed to others without permission. Its loss is not due to the fault of the Rights Holder”)|Placed where purchasers read first|—|
|DD-5-4|A list of the names and SHA-256 of all files of the Enclosed Document (files beginning with `00_RIGHTS`) is recorded in the manifest (jp.nrsd.rights) of every image output in that sale|Rewriting of the Enclosed Document can be detected (R-5-2-2“Rewriting can be detected”). Links images and documents as a set|Attaching a record signature to the Enclosed Document (asking purchasers to verify it is difficult; recording hashes suffices)|
|DD-5-5|The wording is managed by NRSD with versions as reference information (Chapter 1, DD-1-6“NRSD signs reference information and places it in the Public Repository and the mirror”), and the app records the version when generating|Updates to the wording reach Users, and which version was used can be shown later (R-5-3-1“Who wrote it, who checked it, and which version it is can be known”, R-5-3-2“Updates can be delivered to Users (distribution of reference information; 1-3)”)|Embedding it fixed in the app|
|DD-5-6|Sample Notice texts, including the posting of the notice code, are prepared in three levels (short, medium, long) to fit each platform's character count|Fits profile limits (X 160, Instagram 150, Xiaohongshu about 100) and pinned posts (long) (Chapter 2, 2.2 “Limits of the places where notice codes are posted”)|—|
|DD-5-7|The machine-readable Permitted Scope (`00_RIGHTS.json`) is written in W3C ODRL 2.2 (a Recommendation for expressing usage conditions; JSON-LD). AI training and inference are not standard ODRL actions, so they are expressed as the System's extension actions|A standard format other software can read is better than a custom format. ODRL has permissions, prohibitions, and duties, and actions for display, print, reproduce, distribute, and modify (W3C Recommendation, 2018-02-15)|Custom JSON (first draft)|

## 2. Permitted Scope

### 2.1 Options (decision on O-04“Options for the Permitted Scope”)

Basic scope (choose one)

|ID|Name|Content|
|---|---|---|
|P1|Viewing only|The purchaser personally views the images as an individual. Copying is limited to saving on the purchaser's own devices|
|P2|Viewing and private printing|In addition to P1, the purchaser personally prints the images for personal viewing|

Additional permissions (optional)

|ID|Name|Content|
|---|---|---|
|A1|Introducing on social media|The purchaser introduces a reduced image (1080 pixels or less on the long side; initial value) on their own social media, showing the Rights Holder's name and the identification number. Paid distribution and sale are not included|
|A2|Personal editing|Editing (cropping, color adjustment, etc.) for the purchaser's personal enjoyment. Publishing edited images is not included (if A1 is chosen, they can be published within the scope of A1)|
|A3|Permission for AI training and inference|Allows use for AI training, inference, and data mining (not allowed by default; reflected in the four items of cawg.training-mining; Chapter 3, 3.1 “Fields recorded”)|

Conditions always included

- Commercial use (advertising, merchandising, paid distribution or sale) requires separate written permission from the Rights Holder.
- Transfer, sharing, and reposting to others are not allowed (except within the scope of A1).
- Rights (copyright and portrait rights) do not transfer by purchase.
- The Matching Data (C2PA signature, invisible watermark, identification number) must not be removed.
- The default Permitted Scope of social media images is “equivalent to P1: viewing only, no reposting” (Chapter 3, 8.2 “Differences between streams”).

### 2.2 Explanation for non-technical people

- Each option has a one-sentence explanation and illustrated examples of “what you can do / cannot do” (made into screens in Chapter 10 “Screens and Design”).
- Criterion: in User trials, 80% or more of Users who read the explanation can correctly answer whether purchasers may post on social media under the chosen scope (Chapter 10 “Screens and Design”).

### 2.3 Linking to images

- One Permitted Scope is chosen per sales output (delivery), and recorded in the work data (rights) and manifest (jp.nrsd.rights, the terms of use in cawg.metadata, cawg.training-mining) of all images in that folder (Chapter 3 “Signing”).
- At Registration (Chapter 6 “Registration and Evidence Preservation”), the work data is looked up by the identification number read from the invisible watermark of the reposted image and compared with the chosen Permitted Scope to show “what it violates” (D-9-2“The Permitted Scope chosen at the time of sale is the criterion for judging what a repost infringes”).

### 2.4 Changes after sale

- The Permitted Scope of images already distributed is not changed (the manifests and Enclosed Documents of the distributed images serve as evidence).
- To broaden the scope, the Rights Holder gives the purchaser separate permission (outside the System). The System does not show sample text for such permission, because the conditions (consideration, scope, period) are decided per transaction and cannot be expressed as a model form (DD-5-1“The Permitted Scope is chosen as a basic scope plus additional permissions”).
- The scope is not narrowed (that would unilaterally revoke permission the purchaser has already obtained).

## 3. Enclosed Document

### 3.1 Structure

|File|Content|Language|
|---|---|---|
|`00_RIGHTS_README.txt`|Common part (3.2)|Japanese, Chinese, and English side by side in one file|
|`00_RIGHTS_<country code>.txt`|Country part (3.3)|The country's main language and English|
|`00_RIGHTS.json`|Machine-readable Permitted Scope (ODRL 2.2; 3.6)|—|

- The format is TXT (UTF-8, no BOM, CRLF line endings). It opens on any OS and can be read in cloud previews (the “TXT” of D-9-5“Information covering all countries is published as a link destination, and text versions are enclosed only for the nationalities common among purchasers”).
- Placed at the top level of the folder, with names beginning `00_` so they are listed before the images.

### 3.2 Outline of the common part

- The five items of Design Plan 9.2 “Enclosed Documents” (① provisions on rights (including that rights do not transfer), ② the Permitted Scope, ③ the sentence about not giving or receiving without permission, ④ the means the Rights Holder takes if the Matching Data is removed and the images are used outside the scope, ⑤ which acts may give rise to which liabilities) correspond to the rows as follows.

|Order|Item|Outline|Item of Plan 9.2|
|---|---|---|---|
|1|The Rights Holder of these images|Handle name, Notice account|Added to ① (showing the Rights Holder)|
|2|Not giving or receiving to or from others without permission (DD-5-3“The sentence forbidding unauthorized passing-on is placed in the common part”)|“Do not give, share, or repost these images to others without the Rights Holder's permission”|③|
|3|Rights do not transfer|Copyright and portrait rights do not pass to the purchaser by purchase|①|
|4|Permitted Scope|Explanation of the chosen P and A|②|
|5|About the Matching Data|The images carry a C2PA signature, invisible watermark, and identification number (the list of numbers is in `00_RIGHTS.json`); intentionally removing them may violate the law in some countries (see the country part)|Premise of ④|
|6|Means the Rights Holder may take if used outside the scope|Takedown requests, claims for damages, etc. (see the country part)|④|
|7|What may give rise to what liability|See the country part. Complete coverage via the public page link|⑤|
|8|Contact|The Rights Holder's Notice account|Added|
|9|Nature of the permission|The permission is non-exclusive to the purchaser personally, cannot be transferred to others, and cannot be sublicensed. No period is set (as long as the images are held). No territory is limited|①|
|10|When used outside the scope|The permission is limited to use within the scope, and use outside the scope is not permitted in the first place. The Rights Holder may revoke the permission|①|

- The wording is a statement of facts, not strong contractual language (Design Plan 9.2 “Enclosed Documents”: consideration of the concern that images would stop selling).
- The nature of the permission in 9 is aligned so as not to conflict with the nature of the license that sales venues' terms give members (Patreon's terms: a limited, non-exclusive, non-transferable, non-sublicensable, revocable license for private, non-commercial viewing; Legal Research L-17“Terms of sales platforms”). Where a broader permission than the sales venue's terms is given (A1, A2), it is stated as chosen by the Rights Holder.

### 3.3 Outline of the country part

|Item|Example for Japan (legal sources)|
|---|---|
|Removal of rights management information|Copyright Act Article 113(8) (deemed infringement; removal due to technical constraints accompanying format conversion is excluded), Article 120-2(5) (penalty for commercial purposes; prosecution on complaint) (L-1“Removal or alteration of rights management information (Japan)”)|
|Publication without permission|Infringement of the right of public transmission (Copyright Act Article 23) and the right of reproduction (Article 21), and remedies (injunction, damages, criminal penalties) (L-32“Copyright infringement and remedies (Japan, China, US)”), the period for claiming damages (L-20“Limitation periods for damages claims (Japan, China, US)”)|
|Portraits|Portrait rights (the personal interest in not being photographed or published without good reason; the standard of the limit of tolerance in social life), and publicity rights (L-5“Portrait rights and publicity rights (Japan)”)|
|Takedown requests|Information Distribution Platform Act (L-7“Takedown request procedures (Japan)”)|

- China (L-2“Removal or alteration of rights management information (China)”, L-6“Portrait rights (China)”, L-8“Takedown request procedures (China)”) and the United States (L-3“Removal or alteration of rights management information (US)”, L-9“Takedown request procedures (US)”) are made with the same outline. Other countries are shown with the “link to complete coverage”.
- Each item of the country part carries the sources of the legal documents (the original source URLs of the L numbers).

### 3.4 Detecting rewriting

- DD-5-4“The SHA-256 of the Enclosed Documents is recorded in the manifest”. When images and the Enclosed Document folder are put into the app's “Verify” screen (Chapter 10, G-13“Verify”), the SHA-256 of each file is compared with the list in the manifest, and matches, mismatches, files not in the list, and missing files are shown. Checking the images themselves follows Chapter 3, 10.2 “Checking by the User”.

### 3.5 Legal position (handled in legal)

- The Enclosed Document shows the content of the “creator's permission” under the terms of sales venues such as Patreon (Legal L-17“Terms of sales platforms”). Whether it is formed as an independent contract differs by country and requires confirmation by experts (I-08“Wording of Rights Documents”). The System treats the Enclosed Document as “a statement of facts and of the scope of permission”, and does not guarantee the formation of a contract.

### 3.6 Machine-readable Permitted Scope (ODRL)

- Format: ODRL 2.2 JSON-LD (context `http://www.w3.org/ns/odrl.jsonld`). The policy type is Offer (conditions offered by the Rights Holder; the purchaser is not specified).
- Mapping:

|Permitted Scope|Expression in ODRL|
|---|---|
|P1 Viewing only|Permission: `display`, `reproduce` (constraint: limited to saving on the purchaser's own devices)|
|P2 Viewing and private printing|In addition to P1, permission: `print` (constraint: private use)|
|A1 Introducing on social media|Permission: `distribute` (constraints: 1080 pixels or less on the long side, not paid). Duty: `attribute` (show the Rights Holder's name and identification number)|
|A2 Personal editing|Permission: `modify` (constraint: private use)|
|A3 Permission for AI training and inference|Permission: the System's extension actions `nrsd:aiTraining`, `nrsd:aiInference`, `nrsd:dataMining`|
|Conditions always included|Prohibition: the System's extension action `nrsd:commercialUse` (commercial), `distribute` (except within A1), removal of Matching Data (`nrsd:removeProvenance`). Unless A3 is chosen, the three AI actions are prohibited|

- The namespace of the System's extension actions is a URI of NRSD's domain, and the definitions of the actions are placed on the public page.
- The target (Asset) is the list of identification numbers, and the assigner is the Rights Holder's handle name and notice code.
- Example (outline when P1 and A1 are chosen):

```
{
  "@context": "http://www.w3.org/ns/odrl.jsonld",
  "@type": "Offer",
  "uid": "urn:nrsd:policy:{batch number}",
  "assigner": {"uid": "urn:nrsd:notice:{notice code}", "name": "{handle name}"},
  "target": ["urn:nrsd:id:{identification number}", "..."],
  "permission": [
    {"action": "display"},
    {"action": "reproduce", "constraint": [{"leftOperand": "purpose", "operator": "eq", "rightOperand": "nrsd:personalDeviceStorage"}]},
    {"action": "distribute",
     "constraint": [{"leftOperand": "nrsd:longEdgePixels", "operator": "lteq", "rightOperand": 1080},
                    {"leftOperand": "payAmount", "operator": "eq", "rightOperand": 0}],
     "duty": [{"action": "attribute"}]}
  ],
  "prohibition": [
    {"action": "nrsd:commercialUse"}, {"action": "nrsd:removeProvenance"},
    {"action": "nrsd:aiTraining"}, {"action": "nrsd:aiInference"}, {"action": "nrsd:dataMining"}
  ]
}
```

- `purpose` and `payAmount` are left operands (subjects of constraints) in ODRL's common vocabulary. Names with `nrsd:` are the System's extensions and are defined on the public page. [To be confirmed] Before implementation, check the correctness of the form with an ODRL validation tool.
- Definitions of ODRL's standard actions: display is “temporary display”, print is “tangible and permanent display”, reproduce is “copying”, distribute is “supply to third parties”, modify is “changing content (without creating a new asset)”, attribute is “display at the time of use” (W3C's ODRL common vocabulary).

### 3.7 Draft sentences of the common part

- The sentences below are drafts. They are not distributed until reviewed by experts versed in each country's law (4, I-08“Wording of Rights Documents”). The app fills in what is in `{}`.
- Japanese:
  - 「この画像の権利者は {ハンドルネーム}（{告知先アカウント}）です。」
  - 「この画像を、権利者の許可なく他人に渡したり、共有したり、インターネットに載せたりしないでください。」
  - 「画像を購入しても、著作権と肖像権は購入者に移りません。」
  - 「あなたに許されている使い方：{許可範囲の説明}」
  - 「この画像には、C2PA署名、見えない透かし、識別番号（一覧は同封の 00_RIGHTS.json）が入っています。これらを故意に消すことは、国によっては法律に違反し得ます。」
  - 「許された範囲の外で使われた場合、権利者は削除の申立てや損害賠償の請求を行うことがあります。国ごとの内容は同封の国別の文書と、次のページを見てください：{公開ページの URL}」
  - 「問い合わせ：{告知先アカウント}」
- English:
  - "The rights holder of these images is {handle} ({account})."
  - "Do not give, share, or post these images to others without the rights holder's permission."
  - "Purchasing these images does not transfer copyright or portrait rights to you."
  - "You are permitted to: {scope}"
  - "These images contain a C2PA signature, an invisible watermark, and an identification number (listed in the enclosed 00_RIGHTS.json). In some countries, intentionally removing them may be unlawful."
  - "If these images are used beyond the permitted scope, the rights holder may request removal or claim damages. See the enclosed country document and: {URL}"
  - "Contact: {account}"
- Chinese (Simplified):
  - 「本图片的权利人为 {handle}（{account}）。」
  - 「未经权利人许可，请勿将本图片转交、分享或发布给他人。」
  - 「购买本图片并不会使著作权及肖像权转移给购买者。」
  - 「您被允许的使用方式：{scope}」
  - 「本图片含有 C2PA 签名、隐形水印及识别编号（列于随附的 00_RIGHTS.json）。在部分国家，故意删除这些信息可能违法。」
  - 「如在许可范围之外使用，权利人可能提出删除申请或请求损害赔偿。各国的具体内容请参阅随附的国别文件及以下页面：{URL}」
  - 「联系方式：{account}」
- The terms follow the English and Chinese versions of the Design Plan (権利者: rights holder / 权利人, 許可範囲: Permitted Scope / 许可范围, C2PA署名: C2PA signature / C2PA签名, 識別番号: identification number / 识别编号).

### 3.8 Country and language codes

- Countries are expressed with ISO 3166-1 two-letter codes (e.g., JP, CN, US). The file name of a country part is `00_RIGHTS_<country code>.txt`.
- When the same shoot is sold under several Permitted Scopes at different prices (e.g., per support tier), exports are divided per Permitted Scope (one Permitted Scope per export; 2.3). Each export has separate identification numbers.

## 4. Management of the Wording

|Item|Method|
|---|---|
|Drafting|NRSD drafts the provisions from the outline in Japanese and translates them into English and Chinese (the translations are checked by NRSD)|
|Review|Each country's wording is published only after review by experts versed in that country's law (I-08“Wording of Rights Documents”). Country parts are not made for countries not yet reviewed; they are shown only by the link to complete coverage|
|Versions|There is a version of the reference information package (an integer starting from 1; Chapter 8, 3.4 “Reference information package”) and a version per wording (a separate field)|
|Distribution|Placed in the Public Repository and mirror as the reference information package (Chapter 8, 3.4 “Reference information package”), and obtained and verified by the app (Chapter 1, DD-1-6“NRSD signs reference information and places it in the Public Repository and the mirror”)|
|Following law revisions|NRSD checks quarterly and raises the version if there is a revision (Chapter 1, 18.1 “NRSD's points of involvement and structure”)|

## 5. Countries

- Countries enclosed: the User chooses those of the purchasers' common nationalities in “target countries of Rights Documents” (Chapter 1, 10.8 “Languages and countries”) (several allowed; Design Plan 9.3 “Countries Covered”). The purchasers' common nationalities are known to the User (Design Plan 9.3 “Countries Covered”). The app sets no default country and holds no information about purchasers (sales management is out of scope). If no country is chosen, only the common part and the link to complete coverage are enclosed.
- Purchasers' countries cannot be narrowed down (the conclusion of I-07“Knowing the purchaser’s country”). Registration information of sales venues and the country of the connection source lose meaning when a VPN is used, and the purchaser's nationality cannot be determined. Images cross borders. Therefore the System is built not to depend on the purchaser's country: the common part is enclosed in all languages, complete coverage is shown by the public page link, and the country parts are chosen by the User at their own discretion (choosing none is allowed).
- Link for complete coverage: the public page `/rights/` of the Public Repository (a summary of each country's law with a list of sources; generated from the reference information).
- Approach to the governing law (handled in legal; O-03“Approach to governing law”): with reference to the two layers of Design Plan 9.5 “Approach to Governing Law (Reference)” (the Permitted Scope is a matter of contract, infringement a matter of the law of the country where protection is claimed), a proposal is placed to state in the Enclosed Document that “the interpretation of the Permitted Scope follows the law of the seller's (Rights Holder's) country”. Adoption depends on confirmation by experts (I-06“Differences in rights by country”, I-08“Wording of Rights Documents”). Until confirmed, it is not written in the Enclosed Document, and no input for the seller's country is provided (O-03“Approach to governing law” is at the stage of a proposal).

## 6. Support for Notices

### 6.1 Levels of sample text

|Level|Use|Outline|
|---|---|---|
|Short|Profiles on X, Instagram, Xiaohongshu (100 to 160 characters)|“No reposting. My photos carry a C2PA signature, watermark and identification number. NRSD-XXXXX-XXXXX-XXXXX-XXXXX”|
|Medium|When the profile has room|Short plus “How to check: \<public page URL>” and “The CR mark is a mark of a record of the photo's provenance (C2PA), not a label of AI generation”|
|Long|Pinned posts, Patreon's About|The following items: where the rights lie; that unauthorized reposting and removal of Matching Data may be unlawful; the intention to make complaints and claim damages; the meaning of the identification number (a number in the image's record that can also be added after the file name; Design Plan D-7-7“Identification numbers are issued by the Client App and recorded automatically in the C2PA manifest. Whether they are appended to file names is chosen by the User when saving”) and how to check it; an explanation that the C2PA check mark is not a label of AI generation but a record of the photo's provenance; the notice code; how to check (public page URL; for obtaining, points to the README). A work registration number (a number of public registration) is a string the User can optionally write if they have one|

- Each level is prepared in Japanese, Chinese, and English. The app fills in the User's handle name and notice code and shows them in an easy-to-copy form (a copy button).

### 6.2 Relation to matching Entitlement

- The notice code is that of Chapter 2, DD-2-3“The clue to Entitlement is the notice code posted on the Notice account” and Chapter 1, 8.2 “Identifier scheme” (the fingerprint of the personal root). It lets anyone match the link between the signer of an image's C2PA signature and the Notice account (Chapter 2, 4.1 “Matching with the Notice account”).
- If the posted Notice is deleted or rewritten, the clue for matching is lost. The app explains how to post the notice code and what it means to keep it posted (Chapter 10, G-04“Posting the Notice”). The System does not check the posting (D-5-5“The System provides the means of matching but does not perform matching on anyone’s behalf”).

### 6.3 Drafts of the short text (three languages)

- Japanese: 「無断転載禁止。写真には C2PA署名・透かし・識別番号があります。{告知コード}」
- English: "No reposting. My photos carry a C2PA signature, watermark and ID. {notice code}"
- Chinese (Simplified): 「禁止转载。照片含 C2PA 签名、水印及识别编号。{notice code}」
- Character count: the notice code is 28 characters, and the sentences above, including the notice code, are 62 characters in Japanese, 54 in Chinese, and 94 in English, all within 100 characters (within X's 160 characters, Instagram's 150 characters, and Xiaohongshu's roughly 100 characters; Chapter 2, 2.2 “Limits of the places where notice codes are posted”).
- These are drafts, and review is as in 3.7.

## 7. Mapping to Requirements

|Requirement number|Requirement|Sections in this chapter|
|---|---|---|
|R-5-1-1|Non-technical people can understand the meaning and choose (the criterion is set in the basic design)|2 “Permitted Scope”|
|R-5-1-2|Which Permitted Scope was chosen for which image is kept|2 “Permitted Scope”|
|R-5-1-3|The handling of changes after sale is determined|2 “Permitted Scope”|
|R-5-2-1|Contains the items of Design Plan 9.2|3 “Enclosed Document”|
|R-5-2-2|Rewriting can be detected|3 “Enclosed Document”|
|R-5-3-1|Who wrote it, who checked it, and which version it is can be known|4 “Management of the Wording”|
|R-5-3-2|Updates can be delivered to Users (distribution of reference information; 1-3)|4 “Management of the Wording”|
|R-5-4-1|There is a link for complete coverage and a way to decide which nationalities to enclose|5 “Countries”|
|R-5-5-1|Sample Notice texts (three languages) can be shown|6 “Support for Notices”|
|R-5-5-2|Information needed for matching Entitlement can be placed in the Notice|6 “Support for Notices”|

## 8. Gaps Declared in This Chapter

- The Enclosed Document may be removed in the course of giving and receiving (D-9-4“The Enclosed Document includes a sentence stating that the images must not be passed to others without permission. Its loss is not due to the fault of the Rights Holder”: in that case it is not the Rights Holder's fault). The terms of use in the image's manifest remain, but disappear once uploaded to social media.
- Whether the Enclosed Document is formed as a contract differs by country and is not guaranteed (3.5).
- Country parts cannot be made for countries not reviewed by experts (only the link to complete coverage).

## 9. Corrections to the Outline Design Document

- O-04“Options for the Permitted Scope” (the options of the Permitted Scope) is resolved in 2.1 of this document.
- State the format of 5-2 as “TXT (the common part with three languages in one file, the country part per country) and machine-readable JSON”.
