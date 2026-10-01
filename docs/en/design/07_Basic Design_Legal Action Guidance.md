# Basic Design Document Chapter 7: Legal Action Guidance

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [07_Basic Design_Legal Action Guidance.docx](07_Basic%20Design_Legal%20Action%20Guidance.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

## Position of This Document

- Details Chapter 7 of the Outline Design Document.
- The decisions received are as in the following table (the decisions, items to be investigated, and open items of Design Plan Edition 2, and omissions found in the item breakdown). The texts of the Design Plan's decisions, items to be investigated, and open items are per Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” and the destination table of the Outline Design Document (not reproduced in this chapter; only the numbers and the omissions found in the item breakdown are listed).

|Number|Type|Content|
|---|---|---|
|A-12|Omission found in the item breakdown|Handling of joint action and priority targets in Section 8 of the Draft Project Proposal|
|A-16|Omission found in the item breakdown|Unauthorized practice of law (handled by Legal)|
|A-21|Omission found in the item breakdown|Complaints passing the User's personal information to the other party|

- The references are as in the following table.

|Reference|What is referred to|
|---|---|
|Legal Research L-7“Takedown request procedures (Japan)” to L-10“Handling of legal business (unauthorized practice of law) (Japan)”, L-20“Limitation periods for damages claims (Japan, China, US)”, L-21“Copying for evidence (Japan, China, US)”|Procedures for takedown requests, unauthorized practice of law, periods for claims, copying for evidence|
|Chapter 1, DD-1-6“NRSD signs reference information and places it in the Public Repository and the mirror”|NRSD signing and distributing the reference information|
|Chapter 6 “Registration and Evidence Preservation”|Case records, export, status|
|Section 1 of the study memo “Study of Chapter 7 Legal Action Guidance, Chapter 9 Distribution and Updates, Chapter 10 Screens and Design” (NRSD's internal record of study, not distributed with this document; the facts underlying the design are written with their sources in each section of this document)|Facts researched for Chapter 7 (including the second round)|

## 1. List of Design Decisions

|Number|Decision|Reason|Rejected alternatives|
|---|---|---|---|
|DD-7-1|For each case, the guidance shows “where (contact point)”, “what (required items and evidence to attach)”, and “how to write (model text)”. It does not give legal opinions on individual cases (whether one can win, how much can be claimed)|The Ministry of Justice's guidelines show cases where selecting and displaying registered model forms does not constitute legal services, while individual legal opinions may constitute “appraisal” (L-10“Handling of legal business (unauthorized practice of law) (Japan)”)|Automatically composing text to fit the case (approaches individual legal services)|
|DD-7-2|The guidance is free of charge, and there is no path leading to paid legal services of NRSD or third parties|A premise for not falling under “for the purpose of obtaining remuneration” (L-10“Handling of legal business (unauthorized practice of law) (Japan)”)|Introducing affiliated lawyers (referrals invite suspicion of a consideration relationship)|
|DD-7-3|Contact points, model texts, and references to each country's law are maintained by NRSD as reference information (with versions; protected by TUF metadata signed with NRSD's keys; Chapter 8, 3.4) and distributed to the app|Contact points change (R-7-1-2). They can be fixed without waiting for an app update|Embedding them fixed in the app|
|DD-7-4|The one who sends a complaint is the User. The app shows the complaint text in a copyable form and only opens the contact point's form or address; it does not send on the User's behalf|Acting as an agent may constitute legal services (L-10“Handling of legal business (unauthorized practice of law) (Japan)”). Complaints require the User's name and contact details, which may be passed to the other party (L-8“Takedown request procedures (China)”, L-9“Takedown request procedures (US)”, A-21“Complaints passing the User's personal information to the other party”), and the User sends only after understanding this|Sending automatically from the app|
|DD-7-5|The Japanese model text for requests follows the items of Form A of the “Information Distribution Platform Act Copyright Guidelines” (3rd edition, May 2025, Council for Reviewing Guidelines of the Information Distribution Platform Act). For providers designated as Large-Scale Specified Telecommunications Service Providers, each company's published request contact point is shown|Form A is the primary source that sets the items of the request form (the requester, identification of the infringing information, description of the work, the right claimed to be infringed, reasons, manner of infringement, method of confirmation) and the materials confirming that one is the copyright holder or the like. The second draft used items from a law firm's commentary|Using the items of a commentary article (second draft; not a primary source)|
|DD-7-6|Before showing the model text, the liability for mistaken or false complaints (United States: 17 U.S.C. Section 512(f); China: Regulations on the Protection of the Right of Communication through Information Network, Article 24) and the other party's counter procedures are shown, and the User is asked whether they have checked the matching clues (Chapter 2, 4.4 “Clues when a person posing as the Rights Holder appears”)|Persons posing as the Rights Holder cannot be prevented (Design Plan D-5-3“Acts of a person posing as the Rights Holder (signing first, replacing signatures, and reverse complaints) are treated as impossible to prevent; clues for matching are presented. No judgment is made”). Mistaken complaints cause damage to the other party, and the complaining User may be liable for compensation. Responsibility for the matching judgment lies with the User (D-10-6“The judgment on matching and the responsibility for Registration are borne by the User who registered”)|Showing the model text without showing liability|

## 2. Contact Points

### 2.1 List of contact points in the first edition (initial content of the reference information)

|Platform / provider|Contact point|Form|Notes|Source|
|---|---|---|---|---|
|X|Copyright report form (intellectual property via “Contact” at help.x.com). Requests under Japanese law at help.x.com/ja/forms/japan-report|Web form|Based on the DMCA. A Japanese Large-Scale Specified Telecommunications Service Provider (designated April 30, 2025)|help.x.com/en/rules-and-policies/copyright-policy, help.x.com/en/forms, the Ministry of Internal Affairs and Communications' “List of takedown request contact points and takedown criteria notified under Article 21”|
|Instagram|Copyright report form. Requests under Japanese law at help.meta.com/requests/1905988783530981/ (Instagram and Threads)|Web form|The same for Threads. A Japanese Large-Scale Specified Telecommunications Service Provider (designated April 30, 2025)|help.instagram.com/277982542336146, help.instagram.com/contact/552695131608132, the Ministry's list|
|TikTok|Copyright infringement report form. Requests under Japanese law at tiktok.com/legal/information-distribution-platform-act-jp|Web form (no account needed)|A Japanese Large-Scale Specified Telecommunications Service Provider (designated April 30, 2025)|tiktok.com/legal/report/Copyright, the Ministry's list|
|Patreon|DMCA notice form, or copyright@patreon.com|Web form, e-mail|States explicitly that the complainant's name and contact details may be given to the other user|support.patreon.com/hc/en-us/articles/208377833|
|Weibo|投诉页面, then 侵权保护专区, then 权益投诉 (知识产权纠纷)|Web, app|No official statement of the days until acceptance could be confirmed (commentary articles say about 15 business days)|kefu.weibo.com/faqdetail?id=21692 (official)|
|Xiaohongshu|权利保护中心 (ipp.xiaohongshu.com), “举报” on a note, then “侵权投诉”|Web, app|Bulk complaints via the Rights Protection Center|ipp.xiaohongshu.com (official contact point). The procedure follows commentary articles (whaleip.com, elawcn.com “小红书侵权投诉指引”). No official procedure document could be confirmed|
|Bilibili|侵权申诉表 to copyright@bilibili.com|E-mail|—|bilibili.com/html/copyright.html|
|pixiv|Guide and inquiry form for procedures related to the Information Distribution Platform Act|Web form|A Japanese Large-Scale Specified Telecommunications Service Provider (designated August 31, 2026). It notifies its request contact point within 3 months of designation (Article 21). [To be confirmed] Check the URL of the contact point after notification in the Ministry's list and revise the reference information|policies.pixiv.net/ja/provider.html, the Ministry's press release (August 31, 2026)|
|pixivFANBOX|“…” on the creator page, then Report, or the inquiry form|Web form|—|fanbox.pixiv.help (I want to report somebody)|
|Fantia|The operating company's inquiry contact (fantia.co.jp/contact/). The site also reportedly has a reporting function|Web form|No dedicated form for reporting rights infringement could be confirmed in public information; the inquiry contact is shown|fantia.co.jp/contact/, fantia.jp/help/terms (terms)|
|爱发电 (afdian.com; the “support service for China” of Design Plan 3.3.1)|E-mail to the contact in the site's footer, report@afdian.com (建议反馈)|E-mail|No dedicated form for reporting rights infringement could be confirmed in public information; the footer contact is shown (checked 2026-10-01; the same handling as Fantia)|The footer of afdian.com, guide.afdian.com|
|Cloudflare (CDN)|DMCA form at abuse.cloudflare.com|Web form|Complaints by e-mail are in principle not processed|cloudflare.com/trust-hub/reporting-abuse|
|Hosting providers (general)|The abuse contact obtained by RDAP (Chapter 6, 5 “Who and Where (④)”)|E-mail, form|—|Draft Project Proposal [23]|
|Sites in China (general)|The contact of the operator of the ICP filing. The filing lookup is in the filing management system of the Ministry of Industry and Information Technology|—|The lookup is done by the User|Draft Project Proposal [22]|

- For Fantia, no dedicated contact point for reporting rights infringement could be confirmed in public information, so the operating company's inquiry contact is shown.
- Japan's Large-Scale Specified Telecommunications Service Providers (designated under Article 20 of the Information Distribution Platform Act) publish their request methods (Article 22: requests can be made by electronic means, without excessive burden, and the requester can know when it was received), and only when a request is made following that published method do they investigate without delay (Article 23) and notify the result of their decision within 7 days of the request (Article 25 and Article 16 of the Enforcement Regulations). Even for the same provider, requests sent through contact points that are not Article 22 methods, such as DMCA report forms, do not carry this duty of investigation and notification. The list of designations follows the Ministry's press releases and the “List of takedown request contact points and takedown criteria”. The reference information holds, in addition to whether each provider is designated, whether each contact point “is an Article 22 method contact point”, used for the deadline display of 6.3.
- When a Japanese User makes a request under Japanese law, the Form A model text (3.3.3) is shown paired with the Article 22 method contact point (the “requests under Japanese law” column of the table above). The DMCA model text is shown paired with the report form under U.S. law.
- The reference information's `platforms.json` has one row per posting site or provider, with `id`, `name` (three languages), `hosts` (the host name patterns for detection) and `paths` (the path patterns of posts and profiles; the detection of Chapter 1, 10.10 uses these two, and Chapter 2, 2.3 and Chapter 6, 2.1 use the same table), `profile_limit` (Chapter 2, 2.2), `c2pa_survives` (Chapter 3, 10.3), `windows` (two: `copyright` and `portrait`, each with `url`, `method` (form or e-mail), `jp_art22` (whether it is an Article 22 method), and `notes`), `sources`, and `checked_at`. The contact points are two: “copyright” and “portrait/privacy” (the one shown for the standing “subject” of 3.4). The URLs of the first edition's portrait/privacy contact points (X's “private information” report, Meta's “privacy” report, and the corresponding reports of TikTok, Weibo, Xiaohongshu, and pixiv) are checked on each company's official pages and entered when the reference information sources are made.

### 2.2 Keeping up to date

- NRSD checks the URL and requirements of each contact point quarterly and raises the version of the reference information if they have changed (Chapter 1, 18.1 “NRSD's points of involvement and structure”).
- Reports from Users who notice changes to contact points are received through the feedback channel (Chapter 10, 9 “Feedback Channel”).
- Additions to the Ministry's designations (press releases) are included in the quarterly checks.
- The RDAP bootstrap list is also included in the reference information, and the operator tool M-01“Reference Information” shows the difference from IANA's list and raises the version (Chapter 8, 3.4).

## 3. Model Texts

### 3.1 Kinds of model text

|Kind|When used|Language|
|---|---|---|
|DMCA notice|Report forms under U.S. law of U.S. providers (X, Instagram, TikTok, Patreon, Cloudflare, etc.)|English|
|Chinese notice (Civil Code Article 1195, Regulations on the Protection of the Right of Communication through Information Network Article 14)|Chinese providers|Chinese|
|Japanese request for measures to prevent transmission of infringing information (Form A of the Copyright Guidelines)|Japanese providers, and the Article 22 method contact points of providers designated as Japanese Large-Scale Specified Telecommunications Service Providers (X, Instagram, TikTok, etc.)|Japanese|
|Abuse notice to hosting providers|General|English, Japanese, Chinese|

- Each model text follows the legal requirements (L-7“Takedown request procedures (Japan)”, L-8“Takedown request procedures (China)”, L-9“Takedown request procedures (US)”) and has the required items as fields (identification of the rights holder, identification of the infringed work, location of the infringing material, contact details, statement of good faith belief, statement of accuracy, and signature).
- The app fills in “location of the infringing material (URL)” and “identification of the infringed work (identification number, date/time of registration, matching result)” from the case record. Name, address, and phone number are entered by the User, and the app does not save them (Chapter 2, 6 “Rights Holder Information”).

### 3.2 Guidance that personal information is passed to the other party

- Before displaying the text, the following is shown, and the User proceeds after ticking a confirmation box.
  - Under the DMCA, a copy of the complaint may be passed to the other party in the counter procedure (L-9“Takedown request procedures (US)”). Patreon states explicitly that it may give the complainant's name and contact details to the other user (2.1).
  - A Chinese notice requires the rights holder's name, contact details, and address (L-8“Takedown request procedures (China)”).
  - In Japanese procedures too, the content of a request may be referred to the sender.
- It is also shown that by appointing an agent (a lawyer, etc.), one may be able to make the complaint with the agent's information instead of one's own (no referrals are made; DD-7-2“Guidance is free of charge and does not lead to paid services”).

### 3.2.1 Liability for mistaken or false complaints

- Before displaying the text, the following is shown on the same screen as the guidance on personal information (3.2), and the User proceeds after ticking a confirmation box.

|Country|What is shown|Basis|
|---|---|---|
|United States|A person who knowingly materially misrepresents that material is infringing is liable for damages (including costs and attorneys' fees) incurred by the other party, the copyright owner, or the service provider|17 U.S.C. Section 512(f)|
|China|A rights holder whose mistaken notice causes damage to the other party or the provider bears tort liability. For a malicious false notice on an e-commerce platform, double the loss is compensated|Civil Code Article 1195(3), E-Commerce Law Article 42, Regulations on the Protection of the Right of Communication through Information Network Article 24|
|Japan|Liability for damage caused to the other party by a mistaken request follows the general provision on torts (Civil Code Article 709) (requires confirmation by experts)|Civil Code Article 709 (added to Legal Research L-7“Takedown request procedures (Japan)”)|

- The confirmation text depends on the standing chosen in 3.4:
  - Copyright holder: “I have confirmed that I own the copyright of this photo. I have looked at the matching clues (identification number, signer of the C2PA signature, notice code, Original).”
  - Subject: “I have confirmed that the person in this photo is me. I have looked at the matching clues (identification number, signer of the C2PA signature, notice code, Original).”
  - Authorized agent: “I have confirmed that I have been entrusted by the copyright holder of this photo to make this complaint. I have looked at the matching clues (identification number, signer of the C2PA signature, notice code, Original).”
- For cases whose matching result is “A match cannot be confirmed”, “Similar (visual fingerprint): possibly the same photo edited”, or “Watermark with someone else's number” (Chapter 6, 4.1 “How the match level is shown”), a warning is shown at the top of this screen.
- Threat T-2 (spoofing): treating the true Rights Holder's post as a repost and complaining to the provider. Attacker: same as above. Target asset: the true Rights Holder's post. Remaining gap: complaints outside the System cannot be prevented

### 3.2.2 The other party's counter procedures

|Country|The other party's counter|What follows|
|---|---|---|
|United States|Counter-notification (signature, identification of the removed material, a statement under penalty of perjury that it was removed by mistake, contact details and consent to jurisdiction)|The provider sends a copy of the counter-notification to the complainant, tells them it will restore the material in 10 business days, and, unless notified that the complainant has filed an action, restores it no less than 10 and no more than 14 business days later (17 U.S.C. Section 512(g)(2)(B)(C), (g)(3))|
|China|A statement by the service recipient (network user) (true identity information and preliminary evidence of non-infringement; under the Regulations, name, contact details, address, and the name and location of the work to be restored)|The provider forwards the statement to the rights holder and informs them that they may complain or sue. If the rights holder does not notify within a reasonable period after the forwarded statement arrives (15 days for e-commerce platforms) that they have complained or sued, the measures end (Civil Code Article 1196, E-Commerce Law Article 43). The Regulations say that the provider must immediately restore the removed work (disconnected links may be restored) and that the rights holder may not again notify removal of the same work (Articles 16 and 17). The relation between the Regulations and the Civil Code requires confirmation by experts|
|Japan|Inquiry to the sender for their opinion|The provider asks the sender whether they consent to removal, and if no objection is made within 7 days, the provider that removed it is not liable to the sender (Information Distribution Platform Act Article 3(2)(ii))|

- Means after a counter (lawsuits, administrative procedures) are outside the scope of the System; export as preparation for consulting experts (Chapter 6, 8 “List and Means”) is shown.

### 3.3 Contents of the model texts

- The model texts are the reference information's `templates/<kind>.<language>.md` (the kinds are `dmca`, `cn_notice`, `jp_form_a`, `hosting`, and `portrait`), with `kind`, `standing` (the standings that can use it), `venue_kinds`, `version`, and `reviewed_by` and `reviewed_at` (expert review) in the front matter (YAML), and the fields in the body written as `{field}` (the placeholder rules are the same as Chapter 4, 3.1). The required items of each model text are set by law. The app fills in fields that can be filled from the case record and shows the fields the User enters as blanks. Draft texts are not distributed until reviewed by experts (the same as Chapter 5, 4 “Management of the Wording”).
- Language (R-7-2-1): the text to be sent is in the language of the contact point's country (English for the United States, Chinese for China, Japanese for Japan). Beside it, a reference translation in the screen language (Japanese, Chinese, English) is shown, with “What you send is the original. The translation is for checking the content”. Reference translations are also distributed as versioned reference information.

#### 3.3.1 United States (DMCA notice; 17 U.S.C. §512(c)(3)(A))

|Item required by law|Field of the model text|Filled by|
|---|---|---|
|(i) Signature of the owner or a person authorized (electronic is acceptable)|Signature (entering the name)|User|
|(ii) Identification of the copyrighted work claimed to have been infringed (a representative list is acceptable for multiple works at one site)|Description of the work, identification number, signer of the C2PA signature, URL of the original post|App (from work data; the URL of the original post is filled from the location of the original post the User recorded in G-12“Works” (`published` of Chapter 3, 7.2), and the User enters it if there is none. The System does not post, so it does not exist at export)|
|(iii) Identification of the infringing material and information sufficient for the provider to locate it|URL of the repost, URL of the image|App (from the case record)|
|(iv) The complainant's contact information (address, phone, e-mail)|Contact information|User|
|(v) A statement of good faith belief that the use is not authorized by the owner or the like|Standard text|App (standard)|
|(vi) A statement that the information in the notice is accurate and, under penalty of perjury, that the complainant is authorized|Standard text|App (standard)|

- Draft (English):
  - "I am the copyright owner (or authorized to act on behalf of the owner) of the photographs described below."
  - "Original work: {work description}. Identification number: {id}. Originally published at: {original URL}."
  - "Infringing material: {repost URL}, image(s): {image URLs}."
  - "Contact: {name}, {address}, {phone}, {email}."
  - "I have a good faith belief that use of the material in the manner complained of is not authorized by the copyright owner, its agent, or the law."
  - "I state that the information in this notification is accurate, and under penalty of perjury, that I am authorized to act on behalf of the owner of an exclusive right that is allegedly infringed."
  - "Signature: {name}  Date: {date}"
- The texts of (v) and (vi) follow the wording of the statute.

#### 3.3.2 China (Civil Code Article 1195, Regulations on the Protection of the Right of Communication through Information Network Article 14)

|Item required by law|Field of the model text|Filled by|
|---|---|---|
|（一）权利人的姓名（名称）、联系方式和地址|权利人信息|User|
|（二）要求删除或者断开链接的侵权作品的名称和网络地址|侵权作品的名称、网络地址|App (from the case record)|
|（三）构成侵权的初步证明材料|The list of the evidence package export (Chapter 6, 3.4 “Export”), the timestamp of the work data, the result of matching the C2PA signature and watermark|App (from the case record)|

- Civil Code Article 1195 requires the notice to include preliminary evidence constituting infringement and the true identity information of the rights holder (E-Commerce Law Article 42 requires preliminary evidence for e-commerce platforms). The “权利人信息” field corresponds to this.
- The Regulations provide that the rights holder is responsible for the truthfulness of the notice. A field for a statement to that effect is placed at the end of the model text.
- Draft (Chinese):
  - 「权利人：{姓名}，联系方式：{电话/邮箱}，地址：{地址}」
  - 「侵权作品名称：{作品说明}（识别编号：{识别编号}）」
  - 「侵权网络地址：{转载 URL}」
  - 「构成侵权的初步证明材料：见附件（作品的原始记录及可信时间戳、C2PA 签名及水印比对结果、侵权页面的保存记录）」
  - 「本人对本通知书的真实性负责。」

#### 3.3.3 Japan (request for measures to prevent transmission of infringing information; Form A of the Copyright Guidelines, 3rd edition)

|Item of Form A|Field of the model text|Filled by|
|---|---|---|
|Addressee (provider name)|Name of the contact point's provider|App (from reference information)|
|Requester's name (seal for paper; persons overseas may substitute a signature)|Name|User|
|1 Requester's address|Address|User|
|2 Requester's name|Name|User|
|3 Requester's contact information (phone number, e-mail)|Contact information|User|
|4 Information for identifying the infringing information (URL, file name, other features)|URL of the repost, URL of the image, size and format of the image, date/time of the post (Chapter 6, 2.3 “Input fields for evidence that should be kept”)|App (from the case record)|
|5 Description of the work (with a copy attached)|Description of the work, identification number, shooting date, URL of the original post|App (from work data; the shooting date is the shooting date/time of the work data, and the URL of the original post is `published` (Chapter 3, 7.2). The User enters them if there are none) and User|
|6 The right claimed to be infringed|Right of public transmission (including the right of making transmittable; Copyright Act Article 23), right of reproduction (Article 21)|Chosen by the User (the app shows the right of public transmission by default)|
|7 Reasons the copyright or the like is claimed to be infringed|That one holds the right, has not given permission to the sender, and has not assigned or entrusted the authority to permit to anyone (the statement of Guidelines IV 4(3))|App (standard) and User|
|8 Manner of copyright infringement|That it is “a file copying all or part of the work as is” (Guidelines II 4(1)b). If reduced or recompressed, as manner (2), write the method of comparison (PDQ distance, reading the invisible watermark)|App (from the matching result)|
|9 Method by which the infringement can be confirmed|That the original image and the reposted image can be compared side by side, and that the identification numbers of the invisible watermark match|App (from the matching result)|
|Attachment (materials confirming identity)|Copy of an official certificate, etc. (for paper requests)|User|
|Attachment (materials confirming that one is the copyright holder or the like)|The table below|App (export list) and User|

- Records of the System usable for confirming that one is the copyright holder or the like (Guidelines IV 2 a)):

|Example in the Guidelines|What the System can show|
|---|---|
|① Registration based on the Copyright Act (including registrations overseas)|The System does not register. If the User has made a registration voluntarily (Japan's registration of the date of first publication or of the real name, China's work registration, U.S. copyright registration; Chapter 12, 3.2 “Optional for Users (the System only guides)”, Legal Research L-22“Copyright registration (Japan)”, L-23“Work registration (China)”), the User is guided to attach a copy of the certificate of registration. It is the strongest material among the examples in the Guidelines|
|② Where the name of the copyright holder or the like is displayed in publishing or selling the work, a copy thereof (Copyright Act Article 14)|Social media images with the Visible Signature and screenshots of the original post, and the display of the signer of the C2PA signature (the verification result of Chapter 3, 10.2 “Checking by the User”)|
|③ Materials showing that the requester is the copyright holder in products or the like offered to the public before the request|Screenshots of the work's page on the sales venue, the Enclosed Document (Chapter 5)|
|④ An appropriately managed database through which the relation between the work and the copyright holder can be looked up|The System's work data is not published as a database, so this does not apply. An extract of the work data (with timestamp) is attached as material on the publication date and identification of the work|

- Caution (shown on screen): the presumption of authorship under Copyright Act Article 14 applies to pseudonyms (handle names, etc.) when “a well-known one” is displayed in the usual way. The User is guided to also attach the record of continuous activity on the Notice account (the posting of the Notice, the original posts). Whether the presumption applies is an individual judgment, and the System does not judge it.
- Source: Copyright Guidelines, 3rd edition (the public PDF on isplaw.jp; Form A, II 4, IV 1 to 4). The items of the “Telesa form” in the second draft were replaced with this form.
- Draft (Japanese; following the wording of Form A):
  - 「私は、貴社が管理する URL：{転載の URL} に掲載されている下記の情報の流通は、下記のとおり、私が有する著作権法第23条に規定する公衆送信権を侵害しているため、「情報流通プラットフォーム対処法著作権関係ガイドライン」に基づき、貴社に対して当該著作物等の送信を防止する措置を講じることを求めます。」
  - 「5 著作物等の説明：侵害情報により侵害された著作物は、私が撮影（または制作）した写真 {作品の説明}（識別番号 {識別番号}、撮影日 {撮影の日}）です。参考として当該著作物の写しを添付します。」
  - 「7 理由：私は、上記の写真に係る公衆送信権（送信可能化権を含む。）を有しています。私は、当該情報の発信者に対して、上記の写真を公衆送信（送信可能化を含む。）することに対し、いかなる許諾も与えておりません。私は、上記の写真を公衆送信することを許諾する権限をいかなる者にも譲渡又は委託しておりません。」
  - 「上記内容のうち、4・5・9 の項目については証拠書類を添付いたします。また、上記内容が、事実に相違ないことを証します。」
- The rights of the subject (cosplayer) concerning their portrait are outside the scope of the Guidelines (copyrights and the like), and are handled by requests of the “privacy / portrait” kind at each provider's contact point. The model text is shown separately from copyright requests (L-5“Portrait rights and publicity rights (Japan)”).

#### 3.3.4 Portrait and privacy requests (the common items of each provider's report form)

- To match the items each company's form asks for, the fields are: the target URL (from the case record), that the person depicted is the requester (the confirmation text of the standing “subject”), that the requester has not consented to publication, the measure sought (removal), and the requester's contact details (entered by the User; not saved). The legal basis of each country is attached per country (Japan: portrait rights (L-5); China: Civil Code Article 1019 (L-6); the United States: each state's right of publicity). Draft texts are not distributed until reviewed by experts.

#### 3.3.5 Abuse notices to hosting providers and CDNs

- The text sent to the abuse contact obtained by RDAP writes the six items of the DMCA notice (3.3.1) in English, with reference translations in Japanese and Chinese (most providers accept the DMCA format; Cloudflare's DMCA form has the same items).

### 3.4 Standing of the complainant

- Users include the copyright holder of the photo (the photographer, etc.), the subject (cosplayer), and persons authorized by either (Design Plan D-3-1“The Users shall be the Rights Holders of the images (cosplayers and photographers) and those authorized by either of them”). Copyright requests can be made by copyright holders and the like, and the Copyright Guidelines, 3rd edition, do not cover requests from third parties (note to II 1 and III 1 of the Guidelines). If a subject files with the copyright model text claiming “the copyright I hold”, it is a statement contrary to fact and leads directly to liability for mistaken complaints (3.2.1).
- Therefore, before showing the model text, the User is asked to choose their standing from the following three.

|Standing|Model text and contact point shown|What is shown as a caution|
|---|---|---|
|Copyright holder (photographer, etc.; including those who acquired the copyright)|The copyright model texts (3.3.1 to 3.3.3) and copyright contact points|—|
|Subject (holder of rights concerning the portrait)|Portrait/privacy requests (the “privacy / portrait” kind at each provider's contact point; 3.3.4)|The copyright model text cannot be used. If the photo's copyright holder is someone else, the copyright request is made by that person, or by someone entrusted by that person|
|Authorized agent (entrusted by the copyright holder to make the complaint)|The agent form of the copyright model text. For the DMCA, the sentence “authorized to act on behalf of the owner” ((vi) of 3.3.1); for Japan's Form A, the requester is the agent, and materials showing the relation to the rights holder are attached|Materials showing entrustment (a letter of authorization, etc.) may be required. Making complaints on behalf of others for remuneration may constitute handling legal services in some countries (L-10“Handling of legal business (unauthorized practice of law) (Japan)”)|

- For one photo, the photographer can make a copyright request and the subject a portrait request, each separately from their own app. The chosen standing is kept in the “complaint made” record (Chapter 6, 6.1 “Status records”).
- Whether materials must be attached to Japan's Form A in the agent case is not settled by the Guidelines and requires confirmation by experts.

## 4. References to Each Country's Law

- The reference information's `laws.json` contains a table of “countries and areas of law” (the same structure as the list in the Legal Research; a row has `country`, `area`, `L` (the Legal Research number), `summary` (three languages), `url`, `official` (whether an official source), and `checked_at`) with a summary and sources (URLs) per country.
- The first edition covers Japan, China, and the United States. Other countries are added to the reference information in the order NRSD adds them to the legal documents (accumulated under the “all” frame; D-11-3“The laws of each country are accumulated as references within an “all countries” framework and added with each implementation”).
- The same content is also placed at `/rights/` on the public page (the same as the link for complete coverage in Chapter 5).
- The first edition's reference items include Japan's requests and orders for disclosure of sender information (Information Distribution Platform Act Articles 5 and 8; the entry point to identifying the Reposter; Legal Research L-33“Disclosure of sender information (Japan)”) and remedies for copyright infringement (injunction, damages, criminal complaint; Legal Research L-32“Copyright infringement and remedies (Japan, China, US)”). All are procedures of courts and investigative authorities; the System does not perform them, and shows their names, links to official guides, and export as preparation for consulting experts (Chapter 6, 8 “List and Means”).
- Authorities' contact points: Design Plan 11.1 lists the authorities in charge of the information among contact points. For consultation on criminal complaints (in Japan, the National Police Agency's “Consultation contact for cyber incidents” https://www.npa.go.jp/bureau/cyber/soudan.html) and China's administrative contact points for copyright (the National Copyright Administration's “投诉指南” https://www.ncac.gov.cn/bsfw/tszn/ and “在线举报” https://www.ncac.gov.cn/bsfw/zxjb/; 在线举报 accepts reports of illegal acts of infringement and piracy), only names and links are placed in the per-country references (the pages were confirmed to open on 2026-09-30). The Illegal and Harmful Information Reporting Center of the Cyberspace Administration of China (12377.cn) is a contact point for reports of illegal and harmful information in general, and is not listed as a contact point for copyright infringement. For the United States, only the name and link of “Report IP Theft” of the National Intellectual Property Rights Coordination Center (IPR Center) (https://www.iprcenter.gov/referral/; the government reporting point where the FBI's intellectual property unit is placed) are placed (the page was confirmed to open on 2026-10-01).

## 5. Identifying the Operator

|Step|Content|
|---|---|
|1|Show the results of Chapter 6, 5 “Who and Where (④)” (IP, ASN, country, domain registration information, contact points)|
|2|For a CDN: go to the CDN provider's abuse contact (Cloudflare in 2.1). The CDN may forward to the provider of the real server|
|3|Chinese sites: find the ICP filing number at the bottom of the page and show the procedure for looking up the operator in the filing management system (done by the User)|
|4|If unknown: as in D-9-7“The location of the server is taken to be ascertainable by investigation; if it cannot be ascertained, action is abandoned”, show that action is abandoned for reposts whose location cannot be found even after investigation (declared in Chapter 13, 4.1 “Gaps in the mechanism”)|

## 6. Flow of Complaints

### 6.1 Flow

1. Open a case and choose “Take action”.
2. Candidate contact points (platform, CDN, hosting) are shown.
3. Choose the complainant's standing (3.4 “Standing of the complainant”). The model texts and kinds of contact points shown change by standing.
4. For the chosen contact point, the required items and the evidence to attach (from the export of Chapter 6) are shown.
5. Confirm the guidance on personal information (3.2).
6. The model text is displayed; the User enters their name and so on and copies it. The contact point's form or address is opened.
7. After sending, “complaint made” is recorded (Chapter 6, 6.1 “Status records”). The deadlines of procedures (6.3) are shown on the case screen.

### 6.2 Joint action and priority

- Joint action: each User can file separately to the same contact point against the same Reposter or provider. The System does not share repost information among Users (Design Plan D-10-9“Information on registered reposts is listed on the User’s device, and the options for action are presented. No sharing among Users takes place (dealt with in Phase 2)”), so it cannot show whether other Users have complained. The System does not organize mass complaints or joint lawsuits (Draft Project Proposal Section 8) (this approaches legal services and agency). Sharing and notifications to authors are handled in the design plan of Phase 2 (D-13-3“Registration of contact details for notifying authors is dealt with in the Phase 2 design plan”).
- Approach to priority: commercial use (paid distribution, advertising, merchandising, AI training data) is marked “priority” from the input of Chapter 6, 2.3 “Input fields for evidence that should be kept” and shown at the top of the list (Draft Project Proposal Section 8, Legal L-5“Portrait rights and publicity rights (Japan)”).

### 6.3 Display of procedural deadlines

- The reference information holds the following table, and guide dates counted from the date of the complaint are shown on the case screen (G-16“Case Details”). Deadlines are providers' duties or legal provisions, and what to do if they pass (inquiring with the provider, consulting experts) is shown. The System does not tell providers that a deadline has passed.

|Country / provider|Deadline|Basis|
|---|---|---|
|Japan's Large-Scale Specified Telecommunications Service Providers (only when the request was made through an Article 22 method contact point; not shown when sent through DMCA report forms or the like)|Notify the result of the decision (whether measures were taken, and if not, the reason) within 7 days of receiving the request. When hearing the sender's opinion or the like, notify that fact within 7 days and notify the result without delay after the decision|Information Distribution Platform Act Articles 23 and 25, Enforcement Regulations of the Act Article 16|
|Japan (inquiry to the sender)|7 days from receiving the inquiry|Article 3(2)(ii) of the same Act|
|United States (DMCA)|Restored no less than 10 and no more than 14 business days after a counter-notification. Not restored if the provider is notified beforehand that an action has been filed|17 U.S.C. Section 512(g)(2)(C)|
|China|The provider forwards the counter statement to the rights holder and informs them that they may complain or sue. If the rights holder does not notify the provider within a reasonable period after the forwarded statement reaches them (15 days for e-commerce platforms) that they have complained or sued, the provider ends the measures (the removed images return). Article 17 of the Regulations on the Protection of the Right of Communication through Information Network says that a provider receiving a written counter explanation must immediately restore the removed work (disconnected links may be restored). The relation between the Regulations and the Civil Code requires confirmation by experts|Civil Code Article 1196, E-Commerce Law Article 43, Regulations Articles 15 to 17 (Legal Research L-8“Takedown request procedures (China)”)|

- The display of procedural deadlines in Chapter 6, 6.1 “Status records” (the 7-day notification by Japan's Large-Scale Specified Telecommunications Service Providers, etc.) follows this table.
- The table of deadlines is distributed as the reference information's `deadlines.json` (a row has `id`, `country`, `venue_kind`, `trigger` (`filed`, `counter_notice`, or `response`), `days`, `unit` (`calendar` or `business`), `basis` (the legal basis), and `text` (three languages)). Business days (the 10 to 14 business days in the United States) are counted excluding weekends and holidays from the reference information's `holidays.json` (holidays per country: the list of federal holidays for the United States (published by OPM), “national holidays” for Japan, and the State Council's holiday notice for China; updated by NRSD every year).
- Deadlines are held as iCalendar (RFC 5545) VTODO (DUE = the deadline; the UID is `<case number>-<deadline number>@c2pa4cosplayer.nrsd.jp`; PRODID is `-//NRSD//c2pa4cosplayer <version>//JA`) with VALARM (TRIGGER relative to DUE, 3 days before and on the day; initial values), and can be exported per case as `.ics` (it goes into the User's calendar). The app itself also notifies at the same VALARM times through G-21“Notifications” and OS notifications (Chapter 10, 8), and shows at the next start those missed while it was not running. The case screen always shows “N days to the deadline” (NRSD's request, 2026-10-01).

## 7. Where the Line Is Drawn

- At the bottom of every guidance screen, the following is always shown: “This is not legal advice. It is general information and guidance on each provider's contact points. Consult an expert about individual cases.” (Japanese, Chinese, English)
- Each item shows its sources (the source URLs of the reference information).
- The range that does not constitute unauthorized practice of law (handled in legal; L-10“Handling of legal business (unauthorized practice of law) (Japan)”): free of charge, no leading to paid services, limited to displaying model texts and guiding to contact points, and the sender is the User (all the design decisions of Chapter 7 “Legal Action Guidance”).

## 8. Mapping to Requirements

|Requirement number (text in the Outline Design Document)|Sections in this chapter|
|---|---|
|R-7-1-1|2 “Contact Points”|
|R-7-1-2|2 “Contact Points”|
|R-7-2-1|3 “Model Texts”|
|R-7-2-2|3.2 “Guidance that personal information is passed to the other party”, 3.2.1 “Liability for mistaken or false complaints”|
|R-7-3-1|4 “References to Each Country's Law”|
|R-7-4-1|5 “Identifying the Operator”|
|R-7-4-2|5 “Identifying the Operator”|
|R-7-5-1|6 “Flow of Complaints”|
|R-7-5-2|6 “Flow of Complaints”|
|R-7-6-1|7 “Where the Line Is Drawn”|

## 9. Gaps Declared in This Chapter

- The gaps of this chapter follow the table in Chapter 13, 4.1 “Gaps in the mechanism” (the rows whose chapter column is this chapter; with why they cannot be closed, the extent addressed, the remaining risks, and who bears them) (not reproduced in this chapter).

## 10. Corrections to the Outline Design Document and Other Chapters

- The requirements of the Outline Design Document are not changed.
- The basis of the deadline display in Chapter 6, 6.1 “Status records” was changed to 6.3 of this chapter (reflected).
- Article 25 and the 7 days of the Enforcement Regulations, the inquiry of Article 3(2)(ii), Section 512(f) and (g), and Articles 15 to 17 and 24 of the Regulations were added to Legal Research L-7“Takedown request procedures (Japan)”, L-8“Takedown request procedures (China)”, and L-9“Takedown request procedures (US)” (reflected).
- Gap H-40“Even after checking matching clues, a User may still wrongly complain about someone else's photo” was added to Chapter 13, 4.1 “Gaps in the mechanism” (reflected).
