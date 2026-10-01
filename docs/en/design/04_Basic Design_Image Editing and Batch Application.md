# Basic Design Document Chapter 4: Image Editing and Batch Application

Anti-Repost Tool for Cosplay Photos  Edition 1 (Draft)  2026-09-29 · Ishikawa Sou  
Nagareyama Software Development Co., Ltd. (NRSD)  
Word version: [04_Basic Design_Image Editing and Batch Application.docx](04_Basic%20Design_Image%20Editing%20and%20Batch%20Application.docx)

<sub>© 2026 Nagareyama Software Development Co., Ltd. (NRSD), Lead Developer: Ishikawa Sou (石川宗). Copyright in this document belongs to Nagareyama Software Development Co., Ltd. (NRSD). Rights in the third-party standards, software, commentaries on laws, and other materials referred to in this document belong to their respective holders. This is a translation of the Japanese original; where they differ, the Japanese original prevails.</sub>

---

## Position of This Document

- Details Chapter 4 of the Outline Design Document. This chapter sets the content of the function that draws names and credits onto photographs (the Visible Signature), templates, editing operations, undo and redo, saving of work sessions (the state in the middle of editing), batch application, and export. The appearance and layout of the screens are set in Chapter 10 “Screens and Design”.
- This chapter is based on the study memo “Study of Chapter 4 Image Editing and Batch Application” (NRSD's internal record of study, not distributed with this document; the facts underlying the design are written with their sources in each section of this document). The survey of similar software and the facts and sources of the components are kept in the study memo.
- The decisions received are as in the following table. The texts of the Design Plan's decisions, items to be investigated, and open items are per Chapter 13, 1.2 “Mapping from Design Plan decisions to the Outline Design Document and basic design” and the destination table of the Outline Design Document (not reproduced in this chapter; only the numbers and the omissions found in the item breakdown are listed).

|Number|Type|Content|
|---|---|---|
|A-11|Omission found in the item breakdown|Font licenses|

- Terms: in this chapter, a “work session” is the collected state in the middle of editing (what other software calls a project). A “group” is a set of layers that is the unit of placement. A “layer” is a single element of text or image.

## 1. List of Design Decisions

|Number|Decision|Reason|Rejected alternatives|
|---|---|---|---|
|DD-4-1|The Visible Signature is held in two parts: the template (the stacking of groups and layers, and formatting) and per-photo adjustments (the position, size, and rotation of groups; the visibility and color of layers; layers for that photo only)|A template is made once, and only what differs is adjusted per photo. Design Plan 8.3 “Reconciling Editing and Batch Application”|Holding a complete setting per photo (batch would be meaningless)|
|DD-4-2|There are two kinds of layers: text layers and image layers. The content of a text layer is a combination of free text and placeholders (values that change per photo)|Credits are written as combinations of text and values, like “Photo: @A / cos: @B”; making name, account, and so on separate kinds does not fit how they are written|Separate element kinds for name, account, date, license notice, and identification number (first draft)|
|DD-4-3|A “group” of layers is the unit of placement, and auto-placement and per-photo position adjustment are done per group|A credit is one block of several lines and a logo, and is moved as a block. Two separate blocks may be placed on one photo (a credit at bottom right and a notice at top left)|Holding a position per layer (auto-placement would scatter lines and logos)|
|DD-4-4|Position is held as one of 9 anchor points and a margin from the anchor (a ratio of the photo's short side); size is held as a ratio of the photo's short side|It looks the same on photos with different aspect ratios and pixel counts. Lightroom Classic and BatchMark Pro also use 9 anchor points and ratio-based positions|Holding pixels (would shift per photo and per export size)|
|DD-4-5|Text shaping (glyph selection and arrangement) and drawing are done by the same Rust mechanism for both preview and export. The components are HarfRust (shaping), skrifa (glyph outlines), and tiny-skia (fill, outline, transformation); vertical typesetting and shadow blur are self-made|The screen's canvas cannot draw vertical text, text looks different on each OS, and it does not match the output. No public Rust component does vertical text up to typesetting|Drawing the preview with the screen's canvas. Using cosmic-text or parley (no vertical text)|
|DD-4-6|Text is drawn only with bundled fonts and fonts brought in by the User; OS fonts are not used. Each layer holds a primary font and a list of fallback fonts|The output is the same on every device. Relying on OS fonts, the fallback font used when a glyph is missing differs by OS|Using OS fonts|
|DD-4-7|Bundled fonts are limited to those under SIL OFL 1.1 and are bundled unmodified. The defaults are Caveat, Klee One, Yomogi, LXGW WenKai Lite, and Noto Color Emoji|OFL allows bundling with software. LXGW WenKai has a Reserved Font Name, and modifications such as subsetting require changing the name|Subsetting to reduce size (Reserved Font Name restriction)|
|DD-4-8|The state in the middle of editing is automatically saved as a work session, written to a temporary file and then replaced so it is not corrupted mid-write. The immediately preceding version is kept|Adjusting hundreds of photos does not finish in one sitting. If lost on app exit, PC shutdown, or abnormal termination, it will not be used|Saving only when a save operation is done. Placing records in the photo folder (first draft; writes into the Original's location)|
|DD-4-9|Undo and redo are done with a single history per work session, and the history is saved with the work session. The default is 200 steps (initial value)|Going back in order across photos makes it clear how far one has gone back. One step is small, holding only the difference. It can be undone even after closing and reopening|Separate histories per photo. Not saving the history|
|DD-4-10|A work session holds a copy of the template and does not automatically take in changes to the original template. The image editing screen (Chapter 10, G-25“Image Editing”) has two states, “edit the template” and “edit the work session”; when opened from within a work session it edits the session's copy (13.1)|When a template is edited, the appearance of work sessions in progress should not change on its own. When the Visible Signature is edited in the middle of batch application, the change must take effect in that work session at once|The work session referring to the template. Editing the original template even in the middle of batch application (second draft; the session's copy does not change, so the edit appears to have no effect)|
|DD-4-11|Export is chosen from presets per posting site (size, format, file size limit), and one work session can have several presets|The same shoot is exported for X and for Instagram. Posting sites' limits (5 MB for X, etc.) are not exceeded|Entering the size every time|
|DD-4-12|Colors are converted from the original image's embedded ICC to sRGB, and an sRGB ICC is embedded in the output|Most social media display in sRGB. Photos in Adobe RGB or Display P3 output as is look dull|Not converting (not the default; selectable)|
|DD-4-13|Auto-placement finds the subject's area with a model that estimates salient regions (u2netp), and for each group chooses the candidate position that overlaps the subject least. The text color is white or black according to the brightness of the background where it is placed|Automates the manual work of Design Plan 3.3.1 “Example of a User” (avoiding the subject; white or black to suit the background)|Face detection (cannot avoid costumes and props other than faces)|
|DD-4-14|Editing is non-destructive. The Original is not touched; only templates and work sessions are saved, and compositing is done at export|The Original is not damaged (R-4-4-2)|—|

## 2. Scope of the Visible Signature

- The Visible Signature draws names and credits into the pixels of a photograph. It is always drawn on social media images; whether it is drawn on Delivery Images is decided per template (Chapter 3, DD-3-6“Visible Signatures on Delivery Images are decided per template”; O-11“Visible Signatures on Delivery Images” is resolved by this).
- It is drawn at export. The Original is not changed (DD-4-14“Editing is non-destructive”).
- The work Users have so far done with image editing tools (GIMP, etc.) of overlaying names and credits on photos can be done in this app. The boundary between what this app does and what is left to the User's development and retouching tools is as in the following table.

|What this app does|What is left to the User's development and retouching tools|Reason|
|---|---|---|
|Stack text layers and image layers, group them, and set position, size, rotation, stacking order, visibility, lock, and opacity|Freehand drawing with brushes and erasers|A handwritten signature is written on paper and scanned, or imported as a transparent image made with another tool (5)|
|Text formatting (font, size, color, alignment, line spacing, letter spacing, outline, shadow, background plate, vertical writing)|Selections, cropping, brightness and color correction, retouching|Processing photos is the role of development and retouching tools; the scope of the System is displaying licenses (Design Plan D-2-2“The scope of the System shall be the application of licenses”)|
|Size, rotation, opacity, and recoloring of image layers (logos, images of handwritten signatures)|Compositing photos together|Same|
|Per photo, adjust the position, size, and rotation of groups, hide layers, change colors, and add layers for that photo only|—|—|
|Export at the size, format, and color space for each posting site|Cropping that changes the aspect ratio|Cropping is a judgment of composition and is done with the User's tools. Photos outside Instagram's aspect ratio range are flagged before export (14.1)|

## 3. Text Layers

### 3.1 Content and placeholders

- The content of a text layer is a combination of free text and placeholders (values that change per photo). Several lines can be written.
- Placeholders are inserted by choosing from the “Placeholders” list on the editing screen. In the saved format they are held as `{name}` (15.1). To write braces themselves, the User doubles them as `{{` `}}`.

|Placeholder|Saved format|Content|When there is no value|
|---|---|---|---|
|Handle name|`{handle}`|The handle name in the signing information (Chapter 2, 2 “Input of Information for C2PA Signatures”)|— (always present)|
|Joint rights holder|`{coholder}`|The handle name of the counterparty of an imported joint-rights document (Chapter 2, 5 “Authorizations and Joint Rights”). One counterparty chosen per work session (chosen in Chapter 10, G-08“Export: Photos and Purpose”; `coholder` in 15.2)|That line is not drawn|
|Joint rights holder's role|`{coholder_role}`|The role of the counterparty of the joint-rights document (photographer or cosplayer), written in the screen language|That line is not drawn|
|Notice account|`{account}`|The Notice account chosen in the template from those in the signing information|That line is not drawn|
|Shooting date|`{shot_date}`|The date of the shooting date/time in the photo's shooting information. The format is chosen in the template (2026-09-29, 2026年9月29日, Sep 29, 2026)|That line is not drawn|
|Export date|`{export_date}`|The date of export. Same format as the shooting date|—|
|Identification number|`{work_id}`|The identification number of the work being exported (Chapter 3, 7.1 “Identification number”)|— (decided at export; the preview shows a sample number)|
|Shoot name|`{shoot}`|The shoot name entered in the work session (Chapter 10, G-08“Export: Photos and Purpose”)|That line is not drawn|
|Principal Rights Holder|`{grantor}`|The handle name of the issuer (principal Rights Holder) of the imported authorization (Chapter 2, 5.1 “Authorization”). Used when a User whose role is “authorized person” chooses an authorization and exports|That line is not drawn|
|Short notation of the Permitted Scope|`{license}`|The short notation of the Permitted Scope for the export (the `P1` / `P2+A1+A2` form of Chapter 5, 2.1 “Options (decision on O-04“Options for the Permitted Scope”)”)|—|

- The handling of “that line is not drawn” (the line is removed and the rest close up) follows 3.5 “Drawing”.
- The sample identification number in previews is the fixed value `00000-00000-000` (its 4 check bits do not match, so it does not exist).
- A template holds the Notice account for `{account}` as a URL. If the recipient does not have the same URL, that line is not drawn (the same handling as `{coholder}`).
- Rejected alternative: allowing all shooting information (model, lens, ISO, location, etc.) as placeholders. Not needed for the purpose of credits. Location is removed from output in Chapter 3, and drawing it would defeat that removal (Chapter 3, 8.4 “Removal of shooting information”).

### 3.2 Formatting

|Item|Content|Range|Default (initial value)|
|---|---|---|---|
|Font|Primary font and list of fallback fonts (6)|Bundled and imported fonts|The default list of 6.2|
|Weight|Chosen from the weights the font has|Depends on the font|The font's regular|
|Size|Text size (ratio of the photo's short side)|0.5% to 20%|3.0%|
|Color|Text color, or “auto (white or black)”|—|Auto|
|Alignment|Start, center, end (top, center, bottom in vertical writing)|—|Start|
|Line spacing|Multiple of the text size|0.8 to 3.0|1.3|
|Letter spacing|Increase or decrease as a ratio of the text size|−10% to +50%|0|
|Outline|Outside only. Width (ratio of the text size), color (any, or “opposite of the text color”)|0 to 30%|6%, opposite of the text color|
|Shadow|Angle, distance (ratio of the text size), blur (same), color, opacity|Distance 0 to 50%, blur 0 to 50%|None|
|Background plate|A rounded rectangle laid behind the text. Color, opacity, corner radius, inner padding|—|None|
|Writing direction|Horizontal, vertical|—|Horizontal|
|Rotation|Rotation of the layer (degrees)|−180 to 180|0|
|Opacity|Opacity of the whole layer|0 to 100%|100%|

- Reason for outside-only outlines: inside or centered outlines on thin handwriting-style fonts crush the letters. Photoshop's stroke has outside, inside, and center, but these are not adopted.
- “Opposite of the text color” is black if the text is white, white if black, and otherwise white or black, opposite in brightness.
- How colors are chosen (the same for outline, shadow, and background plate colors): a color field and hue strip, hexadecimal input (e.g., `#FFFFFF`), picking from the photo (eyedropper; takes the color of one point on the photo, averaging 3×3 pixels), recently used colors (up to 8; saved in the settings), and template colors (the list of colors used in the template). Colors are held as sRGB values (DD-4-12“Colors are converted to sRGB and the sRGB ICC profile is embedded”).
- “Auto” text color: compute the average color of the background where it is placed (the photo's pixels inside the group's frame) and its contrast ratio (WCAG formula) with white and with black, and choose the larger ratio. In terms of relative luminance, the boundary is about 0.179: black at or above, white below (the point where (1.05)/(L+0.05) and (L+0.05)/(0.05) are equal for WCAG relative luminance L). The second draft's “boundary at 0.5” chose white with the smaller ratio on backgrounds between 0.18 and 0.5, and was revised.
- Readability check: if the contrast ratio (WCAG formula) between the text color and the average background color is less than 3:1 (initial value), an outline is added automatically and the grid shows a “hard to read” mark.

### 3.3 Vertical writing

- Japanese and Chinese vertical typesetting is done. Lines are set from top to bottom, and lines are arranged from right to left.
- Glyphs are shaped by HarfRust in the top-to-bottom direction, using the font's vertical glyphs (`vert`: vertical forms of punctuation, brackets, the long vowel mark, and small kana) and vertical metrics (vhea, vmtx).
- Latin letters and digits within vertical text are rotated 90 degrees clockwise per word by default. Numbers of up to 2 digits can be set horizontally within vertical text (tate-chu-yoko: side by side in one character cell).
- Fonts without vertical glyphs (Caveat) cannot be chosen as the primary font of a vertical layer. They are used as a fallback only for Latin letters.
- There is no ready-made component for vertical typesetting, so it is self-made (DD-4-5“Text shaping and rendering use the same Rust mechanism for preview and export”).
- [To be measured] That in Klee One, Yomogi, and LXGW WenKai Lite (the three bundled fonts with vertical glyphs), vertical punctuation, brackets, the long vowel mark, and small kana take vertical forms. Criterion: draw a test sentence (including “「撮影：ー、。」ぁぃっ”) in each font and check by eye. LXGW WenKai has only `vert` and not `vrt2`, but `vert` is expected to suffice.
- Risk: implementing vertical typesetting in-house. Impact: more work and more room for errors. Preparation: decide test sentences and check them (Chapter 4, 3.3 “Vertical writing”)

### 3.4 Missing glyphs

- For each character, fallback fonts are searched in order starting from the primary font, and the character is drawn with the first font that has the glyph.
- Characters in no font are flagged on the editing screen with a mark under them. Before export, the list of photos with missing glyphs is shown and confirmation is requested. They are never exported as boxes (so-called tofu).

### 3.5 Drawing

- Drawing order (one text layer): background plate, shadow, outline, text fill. For each grapheme cluster (UAX #29), a fallback font is searched; the direction within a line is determined by UAX #9 (bidirectional). No automatic line wrapping (lines are divided by the User). When a placeholder in a line containing placeholders (3.1) is empty, that whole line is not drawn and the remaining lines close up toward the group's anchor. Outline joins and caps are round (round join and cap). The center of rotation is the group's center.
- Shadows are drawn by blurring an image filled with the shape of the text and outline (Gaussian blur; σ = half of the blur value, the same relation between the box-shadow blur radius and σ as in CSS) and shifting it by the angle and distance.
- Drawing is done with tiny-skia, and glyph outlines are extracted with skrifa. For emoji (Noto Color Emoji), the COLRv1 (vector color glyph) paint instructions are extracted with skrifa and drawn with tiny-skia (6.3).
- Drawing precision: positions are aligned in units of 1/4 pixel, and glyph edges are smoothed (tiny-skia's anti-aliasing).
- Color and bit depth of drawing: the Visible Signature is drawn with tiny-skia as an 8-bit RGBA (sRGB values) transparent image (tiny-skia handles only 8-bit RGBA; tiny-skia's README). If the photo is not sRGB or is 16-bit (original-size TIFF for delivery, etc.), the drawn transparent image is converted with moxcms to the photo's color space and widened to the photo's bit depth before compositing. Visible Signature colors are held as sRGB values (3.2), so they appear in the same visible color on photos in any color space. [To be measured] Overlay white, black, and primary-color text on a 16-bit Adobe RGB TIFF, and the color difference (ΔE2000) from the sRGB values is less than 1.
- [To be measured] Whether drawing results with the same input match on Windows, macOS, and Linux. Criterion: on 50 test images, every pixel value differs by 1 or less. If not met, identify the processing that causes the difference and fix it.

## 4. Groups and Layers

### 4.1 Structure

- A template consists of one or more groups, and a group consists of one or more layers.
- A layer's position is held as a relative position within the group (from the group's top-left; a ratio of the photo's short side).
- The group's size is the smallest rectangle enclosing its layers. Changing the group's size changes the layers' sizes and relative positions by the same factor.

### 4.2 Layer operations

|Operation|Content|
|---|---|
|Add|Add a text layer or image layer to the selected group|
|Duplicate|Copy a layer and place it at the top of the same group|
|Delete|Delete a layer|
|Name|The name in the layer list (by default the first 10 characters of the text, or the image's file name)|
|Stacking order|Drag within the layer list to reorder. Layers higher in the list are drawn in front|
|Show/hide|Hide a layer. Hidden layers are not drawn in export either|
|Lock|Make a layer unselectable and immovable on the photo (so a logo laid in the background is not moved by mistake). It can still be selected from the layer list|
|Opacity|0% to 100%|
|Move to another group|Move a layer to another group. After moving, its apparent position on the photo is kept|

### 4.3 Group operations

|Operation|Content|
|---|---|
|Add|Add an empty group|
|Delete|Delete a group and the layers in it|
|Name|The group's name (by default numbered in order, such as “Credit”, “Notice”)|
|Anchor and margin|Position (7.1)|
|Auto-placement target|Whether auto-placement moves it (groups placed at a fixed position are excluded)|

### 4.4 Limits on numbers

|Item|Limit (initial value)|Reason|
|---|---|---|
|Groups in one template|4|Credit, notice, logo, and one spare are expected to suffice. Keeps down the computation of combinations of auto-placement candidates|
|Total layers in one template|20|A number that can be seen at a glance in the layer list on the editing screen|
|Characters in one text layer|500|—|

## 5. Image Layers

- Importable formats are limited to PNG and JPEG. SVG is not imported (it may contain processing instructions).
- Import checks: up to 20 MB per file (initial value), up to 8,000 pixels in each dimension (initial value). Images are read with the same components as in Chapter 3, and broken files are not imported.
- Imported images are copied and saved as template assets and referenced by SHA-256 (they can be drawn even if the original file disappears).
- Recoloring: using the image's transparency as is, the color can be changed to white, black, any color, or auto (white or black to suit the background; the same rule as “auto” for text color (3.2), and because it is judged by the group's frame, text and logo in the same group get the same color). This is for using a single-color logo in white or black to suit the background (Design Plan 3.3.1 “Examples of Users”).
- Size, rotation, and opacity are handled as for text layers.
- Rights to images the User imports are the User's responsibility, and this is shown once at import (the same handling as Legal Research L-19“Font licenses”).

## 6. Fonts and Emoji

### 6.1 Bundled fonts

|Font|Use|License|Size|Kana|Japanese kanji (JIS Levels 1 and 2)|Simplified Chinese (Table of General Standard Chinese Characters, 8,105 characters)|Vertical writing|
|---|---|---|---|---|---|---|---|
|Caveat|Latin handwriting style|OFL 1.1|298 KB|None|None|None|No|
|Klee One|Japanese handwriting style (textbook style)|OFL 1.1|8.7 MB|All|All|4,004|Yes|
|Yomogi|Japanese handwriting style|OFL 1.1 (derivatives may not use the name “よもぎフォント”)|4.0 MB|2 missing (ゕ, ゖ)|All|3,507|Yes|
|LXGW WenKai Lite (v1.522, Regular)|Chinese (derived from Klee One)|OFL 1.1 (with Reserved Font Name)|13.9 MB|All|All|8,105|Yes (vert; no vrt2)|
|Noto Color Emoji (Google Fonts distribution; COLRv1)|Emoji|OFL 1.1|25.3 MB|—|—|—|—|

- Sources: each font's distributor (study memo “Study of Chapter 4 Image Editing and Batch Application”). Sizes and character coverage are values measured from the distributions on 2026-09-29.
- Not modified: with modifications such as subsetting, LXGW WenKai could not use its Reserved Font Name and would have to be renamed. It is bundled as is.
- For LXGW WenKai, the Lite version distributed by the author (the Regular of LXGW WenKai Lite v1.522, 13.9 MB) is adopted. It differs from the full version (25.6 MB) in the number of characters; the Lite version contains all 8,105 characters of the Table of General Standard Chinese Characters (3,500 Level 1, 3,000 Level 2, 1,605 Level 3) and has vertical glyph substitution (vert) (checked on 2026-09-30 by reading the distribution's character map and the list of GSUB features and comparing with the published character list). The Lite version is the author's own distribution, not a subsetting modification by the System, so it does not touch the Reserved Font Name restriction.
- License documents are bundled and listed in the list of third-party licenses (Chapter 11, DD-11-9“The license list is bundled automatically”).

### 6.2 Fallback font order

- Each text layer holds a primary font and a list of fallback fonts. The default order (initial value):

|Writing direction|Order|
|---|---|
|Horizontal|Caveat, Klee One, LXGW WenKai Lite, Noto Color Emoji|
|Vertical|Klee One, LXGW WenKai Lite, Noto Color Emoji (Latin letters drawn with Caveat rotated 90 degrees)|

- For layers mainly in Chinese, the primary font is LXGW WenKai Lite (Klee One's glyphs are Japanese forms and cover only about half of the simplified characters). When the screen language is Chinese, the default order puts LXGW WenKai first.

### 6.3 Emoji

- Noto Color Emoji distributed by Google Fonts (`NotoColorEmoji-Regular.ttf`, 25.3 MB, OFL 1.1) is bundled. This version has COLRv1 (vector color glyphs; `COLR` table version 1, `CPAL`) and an `SVG` table, and no bitmaps (`CBDT`) (checked on 2026-09-30 by obtaining the distribution and reading its table list). Being vector, it does not blur when drawn large.
- The second draft's “blurs when drawn large because it is bitmap” assumed the bitmap version (another version in the noto-emoji distribution), and was revised.
- [To be measured] That skrifa and tiny-skia can draw all COLRv1 paint types (solid, linear, radial, and sweep gradients, transforms, compositing). Criterion: draw all emoji and compare with the browser's (Chrome's) rendering for the same shapes and colors. Glyphs with paints that cannot be drawn are not drawn and are marked “cannot be drawn” (the same handling as missing glyphs in 3.4).

### 6.4 Imported fonts

- TTF and OTF can be imported. Up to 50 MB per file (initial value). Files that cannot be read are not imported.
- Imported fonts are copied and saved in the app's data location (Chapter 8, 2.1 “Arrangement”) and referenced from templates and work sessions by name (the font's family name) and SHA-256. They move to the User's other devices through synchronization (Chapter 8, 5.4; assets are carried with iroh-blobs) or the backup file. On a device that lacks a referenced font, drawing uses the default order (6.2) and the layer is marked “font missing” (“needs adjustment” in 12.2).
- The license is the User's responsibility, and this is shown once at import (the first draft's policy is continued).

## 7. Placement

### 7.1 Position and size

- A group's position is held as one of 9 anchor points (top left, top center, top right, middle left, center, middle right, bottom left, bottom center, bottom right) and a margin inward from the anchor (horizontal and vertical; a ratio of the photo's short side).
- The default margin is 3% of the photo's short side (initial value).
- A group's size is determined by the sizes of its layers (ratios of the photo's short side). Even if the photo's pixel count and aspect ratio differ, it is drawn at the same ratio.

### 7.2 Auto-placement

- Targets: groups set as “auto-placement target” in the template.
- Procedure:

  1. Shrink the photo to 320×320 pixels (aspect ratio not kept) and obtain a probability map (0 to 1) of salient regions with u2netp. Preprocessing follows u2netp's official preprocessing (shrink with LANCZOS, normalize with mean (0.485, 0.456, 0.406) and standard deviation (0.229, 0.224, 0.225); u2net_test.py of U-2-Net, rembg's u2netp processing).

  2. Restore the probability map to the photo's aspect ratio.

  3. For each group, compute the sum of probabilities inside the group's frame at each candidate position (the 9 anchor points plus the default margin; for vertical groups, also positions along the left and right edges at 1/4 and 3/4 of the height (4 positions on left and right)).

  4. If there are several groups, choose the combination of non-overlapping candidates with the smallest total of the sums (up to 4 groups and up to 13 candidates (the 9 points and the 4 vertical positions), so combinations are computed exhaustively).

  5. On ties, choose the candidate closer to the anchor set in the template.

- “Auto” text color and the readability check follow 3.2 “Formatting”.
- Components: u2netp (Apache-2.0, about 4.6 MB) is converted to ONNX and run with the ONNX runtime (ort) (ort in Chapter 11, 4.1 “Rust components”). There is no official ONNX distribution.
- [To be measured] Time for auto-placement of 500 photos. Criterion: within 1 minute on the PC of Chapter 1, 14.1 “Scale and performance targets”.
- [To be measured] Hit rate on cosplay photos (full-body shots with much background, close-ups with little background). Criterion: in User trials, no adjustment needed on 70% or more of photos.

### 7.3 When it does not fit

- If a group exceeds 40% of the photo's short side (initial value), the whole group is shrunk to fit.
- If the text size would become smaller than 1.5% of the photo's short side (initial value), it is not shrunk, and the photo is marked “needs adjustment”.
- The first draft's “reduce lines” is not adopted. Deleting lines the User wrote on its own would produce unintended credits.

### 7.4 Tiling

- As a way of placing a group, besides “single”, placed at one of the 9 anchor points, “tiling”, which repeats the group across the whole photo, can be chosen. Similar tools (watermarkly, etc.) have, alongside single placement, a method of repeating vertically and horizontally or diagonally. It is hard to remove by cropping and is used for sales sample images (watermarkly's “Tile mode” guide, etc.).

|Item|Range|Default (initial value)|
|---|---|---|
|Arrangement|Grid, diagonal (checkerboard)|Diagonal|
|Angle|−45° to 45°|−30°|
|Spacing (horizontal, vertical)|10% to 60% of the photo's short side|25%|
|Opacity|10% to 100%|20%|

- Tiled groups are not auto-placement targets (7.2). Repetitions cut off at the photo's edges are also drawn.
- Tiling greatly impairs how the photo looks, so the template's default placement is “single”, and tiling is used only when the User chooses it.
- When drawing, one unit of the repetition is drawn once and its pixels are laid across the photo (the group is not composed repeatedly; this keeps the time of 9.4 “Preview” unaffected).

## 8. Templates

### 8.1 Management

|Operation|Content|
|---|---|
|Create new|From empty, or from a default template|
|Create by copying|Copy an existing template|
|Rename|—|
|Delete|Deleting does not affect work sessions made from that template, because they hold their own copies (DD-4-10“A work session holds a copy of the template”). This is shown before deleting|
|Reorder|Change the order in the list|
|Export and import|8.3|

- There is no limit on the number. The list is ordered by most recently used.

### 8.2 Versions

- Each time a template is saved, its version number goes up by one.
- A work session holds which version of which template it copied (15.2). When the original template has a newer version, the work session screen shows “The template has a new version”, and it is taken in only when “Take in” is pressed. Taking it in can be undone (10).
- Changes while editing a template are automatically saved as a draft (the same method as 11.3), and “Save” confirms it and raises the version. Until confirmed, it does not appear as a “new version” to other work sessions.
- Reconciling with other devices and backups (called from Chapter 8, 5.3 and 5.4): for templates and work sessions with the same number, the newer one by device number and monotonic number (the version for templates, the save sequence for work sessions) is taken, and the older one is kept in the version backups (11.4). If both have changed, both are kept and shown.

### 8.3 Passing on (export and import)

- A template can be made into one file and given to someone else (so that photographer and cosplayer can use the same design).
- File content: the template JSON (15.1) and the images of image layers. The format is ZIP; the extension is `.nrsdtpl`, and the name is `<safe form of the template name>-<first 8 characters of the id>.nrsdtpl`.
- Bundled fonts are not included (the other party's app has them too). Fonts imported by the User are not included by default. If they are included, the User is told to confirm that the license allows redistribution.
- Import checks: in addition to the common ZIP checks (Chapter 1, 10.2 “Storage”), the number of files inside (up to 50), total size (up to 100 MB), file names (fixed names only), JSON format (anything not matching the form of 15.1 is not imported), and images get the checks of 5. All are initial values.
- Placeholders are drawn with the recipient's signing information (the handle name and so on become the recipient's).
- Importing a template with the same `id`: if the version is the same or older, it is not imported (already present). If the version is newer, it is imported as a new version (the same handling as “a new version is available” in 8.2). If the version is the same but the content differs, it is imported under a new `id` as “\<name> (imported)”, keeping both (the same as the template rule of Chapter 8, 5.4).
- A `.nrsdtpl` can be sent directly to a confirmed contact (Chapter 2, 5.5). The receiving side goes through the import checks above. The format is unchanged (the same as sharing Lightroom preset files) (NRSD's request, 2026-10-01).

### 8.4 Default templates

|Name|Groups and layers|
|---|---|
|Name only|Bottom right: `{handle}`|
|Photographer and cosplayer|Bottom right: a `{handle}` line and a `{coholder_role}: {coholder}` line|
|Name and no reposting|Bottom right: `{handle}`. Top left: “無断転載禁止 / Do not repost / 禁止转载”|

- Whether to draw the identification number is up to the User. It is not in the default templates, and `{work_id}` can be added from the placeholder list. The first draft's “include the identification number by default” is revised.
- The appearance (fonts, colors) follows the direction of Chapter 10, 2 “Design” and is finished by the Lead Developer (Chapter 10, DD-10-2“Design is done and judged by NRSD's Lead Developer”).

## 9. Editing Operations

### 9.1 Selecting and moving

|Operation|Method|
|---|---|
|Select|Click a group or layer. Shift-click to select several. Can also be selected from the layer list|
|Move|Drag. Arrow keys move by 0.1% of the photo's short side, Shift and arrow keys by 1%|
|Size|Drag a corner handle. The aspect ratio is kept|
|Rotate|Drag the top handle. In 15-degree steps while Shift is held|
|Snap|Snap to the photo's edges, center lines, and other groups' edges within 0.5% of the photo's short side (initial value), showing guide lines. No snapping while Alt (Option on macOS) is held|
|Align|Align several selected layers left, center, right, top, middle, or bottom. Distribute evenly|
|Copy and paste|Copy layers and groups and paste into the same or another template or photo|
|Copy formatting only|Copy a text layer's formatting and apply it to other text layers|
|Delete|Delete key (Delete on macOS)|
|Enter by numbers|Enter the position (anchor and margin), size, and rotation of the selected group or layer in numeric fields. The same as dragging, done with the keyboard alone (Chapter 10, 2.7) (NRSD's request, 2026-09-30)|

### 9.2 View

|Operation|Method|
|---|---|
|Zoom in/out|Ctrl and + / − (Command on macOS). Mouse wheel with Ctrl|
|Fit to screen|Ctrl and 0|
|Actual size|Ctrl and 1. Actual size shows one pixel of the full-size photo as one screen pixel; when zooming beyond the scale of the preview's reduced image, the visible area is redrawn from the full-size photo and sent (9.4)|
|Pan|Drag while holding the space bar|
|Before/after comparison|Switch between views with and without the Visible Signature only while a key is held (\; on a Mac with a Japanese keyboard layout, entered with Option and ¥) or while the on-screen “Compare before/after” button is held|
|Change the sample photo|When editing a template, switch to sample photos (bright, dark, portrait, landscape; the User's own photos chosen by the User) to check how it looks. The location of sample photos is kept in device settings separate from the template (`templates/<number>/local.json`) and is not included in the template file for passing on (`.nrsdtpl`; 8.3) (because paths to photos may contain the User's name)|

### 9.3 Key assignments

|Operation|Windows, Linux|macOS|
|---|---|---|
|Undo|Ctrl+Z|Command+Z|
|Redo|Ctrl+Shift+Z, Ctrl+Y|Command+Shift+Z|
|Save (confirm with a name)|Ctrl+S|Command+S|
|Copy, paste|Ctrl+C, Ctrl+V|Command+C, Command+V|
|Duplicate|Ctrl+D|Command+D|
|Select all|Ctrl+A|Command+A|
|Deselect|Esc|Esc|

- These follow each OS's conventions. While entering text on screen (editing the content of a text layer), text-entry keys take priority.

### 9.4 Preview

- The photo is sent to the screen once as a preview image reduced to fit the screen (14.3 “Resizing, color conversion, and JPEG export”) and displayed. Windows WebView2 and macOS WKWebView display sRGB images according to the display's color settings. Linux WebKitGTK does not reflect the display's color settings and outputs sRGB as is (the record of the fix for WebKit bug 177185), so on wide-gamut displays photo colors do not look correct (Chapter 13, H-48“On Linux, preview photo colors do not reflect the display's color settings”). For this reason, on Linux the display's ICC is obtained (14.3) and the preview image is converted to the display's color space instead of sRGB before sending (NRSD's request, 2026-10-01).
- When zoom exceeds the scale of the reduced image (including actual size), only the visible area is cut out from the full-size photo, redrawn with the Visible Signature, and sent. Thin outlines can be checked at actual size.
- For each group, the Visible Signature is drawn in Rust as a transparent image the size of the group's frame (DD-4-5“Text shaping and rendering use the same Rust mechanism for preview and export”) and sent to the screen. It is sent via Tauri's binary data response (`tauri::ipc::Response`). The screen overlays this image on the photo.
- While moving by dragging or arrow keys, the sent image is only moved on the screen and not redrawn. It is redrawn when size, rotation, formatting, or content changes.
- [To be measured] Time to draw one group and display it on screen. Criterion: within 100 ms (initial value).
- Match between preview and export: because they are drawn with the same drawing mechanism, what is seen at the preview's scale matches the export. Thin outlines are checked in the actual size view (Ctrl+1; redrawn from the full-size photo).

- The keys above do not collide with keys reserved by the OS (Alt+F4; Command+Q, W, H, M, Tab; Alt+Tab; Super). The WebView's built-in accelerator keys (F5 and Ctrl+R reload, F12 and Ctrl+Shift+I, Ctrl+F and F3, Ctrl+P, Ctrl+U, Ctrl+plus/minus) are stopped by setting WebView2's `AreBrowserAcceleratorKeysEnabled` to false on Windows and with tauri-plugin-prevent-default on all three OSes. Command+W on macOS is accepted as closing the window and goes through the “on close” saving and update (Chapter 9, 4.1).

## 10. Undo and Redo

### 10.1 Unit of history

- Each work session has a single history (DD-4-9“Undo and redo use a history per work session”). It goes back in order across photos.
- Template editing (the “edit the template” state of the image editing screen; 13.1) has a separate history per template. In the “edit the work session” state of the image editing screen, that work session's history is used (the same single history as the batch application screen).

### 10.2 What makes one step

|Operation|Unit of one step|
|---|---|
|Drag (move, size, rotate)|One step when the mouse is released|
|Move with arrow keys|Combined into one step 1 second (initial value) after the key is released|
|Entering text content|One step 1 second (initial value) after typing stops|
|Changing formatting (slider)|One step when the slider is released|
|Batch operations (re-place everything automatically, apply to selected photos)|The whole operation is one step|
|Taking in a new template version|One step|

### 10.3 Contents of the history

- Each step holds the difference before and after the change (JSON difference format; RFC 6902 JSON Patch), the time, and a description of “what, on which photo” (e.g., “Photo IMG_0123: Credit moved from bottom right to bottom left”).
- Undo applies the backward difference, and redo the forward difference. If a new operation is done after undoing, the redo portion is discarded.

### 10.4 Number of steps and memory

- The default is 200 steps (initial value). Photoshop's default is 50 steps with an upper limit of 1,000, and GIMP has a minimum number of steps and a memory limit. One step in this app is a difference document (containing no pixels) and small, and to allow going back across adjustments of dozens of photos, it is set higher than Photoshop's default.
- 50 to 1,000 steps can be chosen in the settings. When the number is exceeded, the oldest steps are removed.

### 10.5 History list

- The editing screen has a history list showing the description of each step, newest first. Pressing a step goes back to that point (the steps in between become undone).

### 10.6 Saving

- The history is saved with the work session (11), and it can be undone even after reopening the work session.

## 11. Work Sessions (the State in the Middle of Editing)

### 11.1 Unit of a work session

- One export (from choosing photos in Chapter 10, G-08“Export: Photos and Purpose”, to exporting in G-11“Export: Run and Results”) is one work session.
- Opening a single photo on the image editing screen (Chapter 10, G-25“Image Editing”) and drawing on it is also one work session.
- Any number of work sessions can be kept. They are listed in “Work in progress” on Home (Chapter 10, G-07“Home”) in order of recency.

### 11.2 Contents of a work session

- The work session's name, creation date/time, and last modified date/time.
- A copy of the template (which version of which template; DD-4-10“A work session holds a copy of the template”).
- The list of photos (15.2).
- Per-photo adjustments (the group's anchor, margin, size, and rotation; layer visibility and color; layers for that photo only) and status (12.2).
- Export presets (several; 14.1) and the record of whether export has been done per preset. Whether to add the identification number to file names (`id_in_filename` of 15.2).
- The choice of Permitted Scope (Chapter 5) and the choice of joint rights holder.
- The current stage (choosing photos, batch application, Permitted Scope, export).
- The undo history (10).
- The version of the reference information and copies of the wording files used (Chapter 5, 4) (NRSD's request, 2026-09-30).
- Photos are not duplicated. The photo's location, SHA-256, size, modification date/time, and pixel dimensions are recorded.

### 11.3 How saving works

- Automatic saving: on every change, saved 2 seconds (initial value) after operations stop. Also saved when moving between screens and when closing the app. Photoshop's automatic recovery defaults to 10 minutes, but work session files contain no pixels and are small, so saving every 2 seconds is not a burden.
- The top of the screen shows “Saved” and the time of saving. If saving fails, the top turns the warning color and the reason is shown.
- If the User presses “Save”, the work session is given a name and confirmed (the content is the same as automatic saving).
- How a work session is held: it is divided into `work.json` (the header: name, date/time, copy of the template, presets, stage, copy of the reference information, joint rights holder, Permitted Scope), `photos/<photo number>.json` (the per-photo record: location, SHA-256, size, status, adjustments, export done), `history.log` (an append-only history; one step per line), and `snapshot.json` (the index for the list; can be rebuilt if missing). Automatic saving writes only the files of photos that changed and appends to the history (the amount written is proportional to the amount of change, not to the number of photos). Loading the list reads only the index, and per-photo records are read when they come into view in the grid (NRSD's request, 2026-09-30).
- When the rows of `history.log` exceed twice the step limit (10.4), it is rewritten when the work session is opened, keeping only the limit's worth (temporary file and replacement).
- Writing follows the procedure of Chapter 1, 10.2 “Storage” (temporary file and replacement).

### 11.4 Version backups

- Each time it is saved, the immediately preceding version is kept as a backup. Up to 5 backups (initial value) are kept, oldest removed first.
- If the work session file is broken when opening (cannot be read as JSON, does not match the format), the User is told, and it is opened with the newest backup version that can be opened.

### 11.5 After abnormal termination

- Whether the app was closed properly is checked at start from the in-use marker (`state/running` in Chapter 8, 2.1 “Arrangement”) (Chapter 1, 7.8 “Starting after an abnormal exit”).
- If it was not closed properly last time, at start a notice “The app did not end properly last time” appears at the top of Home (Chapter 10, G-07“Home”), listing the last saved work sessions and unconfirmed template drafts (8.2) with “Continue” (Chapter 10, 3.7 “List of notices”). Because automatic saving is every 2 seconds, at most the last 2 seconds of operations are lost.

### 11.6 Replacing or moving photos

|Event|Determination|Handling|
|---|---|---|
|The photo is not at the recorded location|—|The User is told when opening the work session and asked to choose the location again. Only files with a matching SHA-256 are accepted. If a folder is chosen, the photos in it are searched by SHA-256 all at once|
|The photo's content changed (re-developed)|SHA-256 differs|If the pixel dimensions are the same, it is used with adjustments kept and the SHA-256 is updated. If they differ, adjustments can still be used because positions are held as ratios, but a “review position” mark is added|
|The photo was deleted|—|The User can remove that photo from the work session (its adjustment record is deleted)|

### 11.7 Opening at the same time

- Only one instance of the app runs per machine (Chapter 1, DD-1-11“Only one instance of the app runs per machine”). The in-use mark is a safeguard in case that does not work.
- The in-use mark per work session is the `lock` in the work session's folder (the folder in-use mark of Chapter 1, 10.2 “Storage”).
- If a second app opens the same work session, it opens read-only and shows so.
- If the lock can be taken (the holder has ended), the session is opened as it is.

### 11.8 Where kept

- `sessions/<work session number>/` in the app's data location (Chapter 8, 2.1 “Arrangement”).

|File|Content|
|---|---|
|`work.json`|The work session's header (15.2, 11.3)|
|`photos/<photo number>.json`|Per-photo records (11.3)|
|`history.log`|Append-only history (10, 11.3)|
|`snapshot.json`|The index for the list (can be rebuilt; 11.3)|
|`versions/`|Backups (11.4)|
|`lock`|In-use mark (11.7)|

- Reduced images of photos can be regenerated, so they are kept in the OS cache location (Chapter 1, 9.1 “Locations per OS”) and are not included in backup files. They are named by the photo's SHA-256 and shared across work sessions (NRSD's request, 2026-09-30).
- Nothing is written to the photo folder.
- Work sessions are included in backup files (Chapter 8, 5 “Backup Files”).

### 11.9 After export

- A work session whose export is finished remains as “exported” and can be reopened to export a revised version. Outputs of the revised version get new identification numbers (Chapter 3, 7.1 “Identification number”).
- Work sessions are not deleted until the User deletes them.

## 12. Batch Application

### 12.1 Flow

1. Choose photos (a folder, or several photos) and a template (Chapter 10, G-08“Export: Photos and Purpose”).
2. Create reduced images of the photos (14.3 “Resizing, color conversion, and JPEG export”), and apply auto-placement (7.2) to all photos.
3. In the photo list (grid), show reduced images with the Visible Signature overlaid, and their status (12.2) (Chapter 10, G-09“Export: Batch Application”).
4. Open photos to be fixed and adjust them (the operations of 9). Several photos can be selected and fixed together (12.3).
5. Choose the Permitted Scope (delivery only; Chapter 5) and export (14).
- Even if stopped midway, the work session is saved (11).

### 12.2 Photo status

|Status|Meaning|When it is set|
|---|---|---|
|Auto|As auto-placed|After auto-placement|
|Adjusted|The User has adjusted it|After adjustment|
|Needs adjustment|Does not fit, missing glyphs, hard to read|Checks of 7.3, 3.4, 7.2|
|Review position|The photo was replaced|11.6|
|Unreadable|The photo cannot be read|Loading failure|

- The display can be filtered by status. If photos marked “needs adjustment” remain, the User is told before export (export is not stopped; the User chooses).
- Transitions of status: “Auto” right after auto-placement. “Adjusted” after an adjustment; “Needs adjustment” when caught by the checks of 7.3, 3.4, or 7.2 (“Adjusted” once resolved by an adjustment). “Review position” when a replaced photo's aspect changes (“Adjusted” after an adjustment). “Unreadable” if it cannot be read (back to “Auto”, and re-placed, when a readable replacement is given). Photos that took in a new template version return to “Auto” (adjustments are kept). Export completion is held per preset, separately from the status (11.2).

### 12.3 Adjusting together

|Operation|Content|
|---|---|
|Change the anchor|Change the anchor of a group on the selected photos at once|
|Hide/show layers|Hide or show a layer on the selected photos|
|Change color|Change a layer's color on the selected photos (auto, white, black, any)|
|Re-place automatically|Re-run auto-placement on the selected photos (or all). Adjustments are lost, so it is done after confirmation|
|Clear adjustments|Return the selected photos to the template as is|

- Each becomes one step of history (10.2).

### 12.4 Large numbers of photos

- The location of reduced images follows 11.8 “Where kept”. They are not regenerated the next time they are opened (regenerated if missing).
- The grid draws only what is visible (Chapter 10, 3.4 “Displaying large lists”).
- No limit is set on the number of photos (Chapter 3, 9).
- [To be measured] Time to create reduced images and auto-place 500 photos (together with 7.2).

## 13. Relation to the Screens

|Function|Screen|
|---|---|
|Creating and editing templates, drawing on a single photo|Chapter 10, G-25“Image Editing”|
|Choosing photos, purpose, export presets|Chapter 10, G-08“Export: Photos and Purpose”|
|Batch application and per-photo adjustment|Chapter 10, G-09“Export: Batch Application”|
|Permitted Scope|Chapter 10, G-10“Export: Permitted Scope”|
|Export and results|Chapter 10, G-11“Export: Run and Results”|
|Work in progress|Chapter 10, G-07“Home”|
|Number of undo steps, default export presets, the default for adding the identification number to file names|Chapter 10, G-20“Settings”|

### 13.1 Two states of the image editing screen

|State|How opened|What is edited|Save (Ctrl+S)|Undo history|
|---|---|---|---|---|
|Edit the template|“Edit” in the navigation, “Create a Visible Signature” on Home, or choosing from the template list on the image editing screen|The template. How it looks is checked on sample photos|Confirms the template's version and raises it (8.2). Changes until confirmed are automatically saved as a draft|History per template (10.1)|
|Edit the work session|“Edit Visible Signature” on the batch application screen (Chapter 10, G-09“Export: Batch Application”), or “Open photo” on the image editing screen (creates a single-photo work session; 11.1)|The work session's copy of the template (affects all photos in that work session), and the opened photo's adjustments and layers for that photo only (affect only that photo). Which one is affected is shown with a switch “All photos in this work session” / “This photo only” at the top of the element settings|Gives the work session a name and confirms it (11.3; the work session is always saved automatically)|That work session's single history (10.1; shared with the batch application screen)|

- The top of the screen always shows the current state (“Template: \<name>” or “Work session: \<name> (Photo: \<file name>)”).
- To also keep the edited copy in the original template from the edit-the-work-session state, press “Also save to template”. It is confirmed as a new version of the original template (other work sessions are only told “There is a new version”, as in 8.2, and it is not taken in automatically).
- What was edited in the edit-the-work-session state takes effect in the grid display as soon as the User returns to the batch application screen.

## 14. Export

### 14.1 Export presets

|Preset|Size|Format|File size limit and handling|Basis|
|---|---|---|---|---|
|X|2048 pixels on the long side (initial value)|JPEG|If over 5 MB, lower quality in steps of 2 to fit (lower limit 80; initial value). If still over at the lower limit, the User is told|X's official guide (photos up to 5 MB)|
|Instagram|1080 pixels wide|JPEG|If the aspect ratio is outside 1.91:1 to 3:4, the User is told before export (not cropped)|Instagram's official guide (original size if 320 to 1080 pixels wide and aspect ratio 1.91:1 to 3:4)|
|Weibo|2048 pixels on the long side (initial value)|JPEG|Under 20 MB|Weibo's official guide (under 20 MB per image)|
|Xiaohongshu|1080 pixels wide (initial value)|JPEG|—|No official statement found. Initial value from a third-party guide (1080×1440 at 3:4)|
|pixiv|2048 pixels on the long side (initial value)|JPEG|32 MB|pixiv's official guide (32 MB per illustration)|
|Patreon|1920 pixels on the long side|JPEG|—|Patreon's official guide (recommended 1920×1080)|
|Delivery|Original size|Original format (JPEG, PNG, TIFF, WebP; Chapter 3, 8.5 “Quality of delivery export”)|—|Purchasers receive the original size (Design Plan Chapter 7 “Forms of Photographs and Processing Streams”)|
|Custom|Specify the long side, or original size|JPEG, PNG|—|—|

- One work session can have several presets (DD-4-11“Export uses presets per posting site”). Whether export has been done is recorded per preset.
- Reason for 2048 pixels on the long side as the social media default: posting the original size on social media would substitute for the sales images (original size) (Draft Project Proposal Section 1: photos are the Rights Holder's source of income). 2048 is an initial value judged sufficient for display, with no official basis. X's upper limit of 4096 pixels could not be confirmed in an official statement.
- [To be measured] Actually upload to each posting site and check the display size and whether it is recompressed.

### 14.2 Processing

- The processing order follows Chapter 3, 8 “Processing Order and Streams”. This chapter is responsible for drawing the Visible Signature, resizing, and color space conversion. Resizing, color space conversion, and conversion to 8 bits are done only for social media export; delivery keeps the original size, original color space, and original bit depth (Chapter 3, 8.2 “Differences between streams”, 8.5 “Quality of delivery export”).
- The methods of resizing and color conversion follow 14.3 “Resizing, color conversion, and JPEG export”.
- The Visible Signature is drawn on the resized image at that size (drawing before resizing and then shrinking would crush thin outlines). The handling of color space and bit depth when compositing the drawn transparent image onto the photo follows 3.5 “Drawing”.

### 14.3 Resizing, color conversion, and JPEG export

- Resizing: Lanczos3 of fast_image_resize. Images with transparency are premultiplied by alpha before resizing and then restored (fast_image_resize's default).
- Color (social media): convert from the original image's embedded ICC (ISO 15076-1; v2 and v4) to sRGB (IEC 61966-2-1) (DD-4-12“Colors are converted to sRGB and the sRGB ICC profile is embedded”). Conversion is done with moxcms. Images without ICC are treated as sRGB. [To be measured] The difference between moxcms and lcms2 conversions on Adobe RGB and Display P3 test images. Criterion: maximum color difference (ΔE2000) less than 1. If not met, lcms2 is used.
- 16-bit images (social media) are resized in 16 bits and converted to 8 bits after drawing.
- Preview image: JPEG, quality 85, up to 2560 pixels on the long side (initial value). Converted from the photo's ICC to sRGB (to avoid differences in how screen components handle color). When the display's ICC is passed (Linux; 9.4 “Preview”), converted to that color space (moxcms; the same path as darktable, GIMP, and Krita).
- Obtaining the display's ICC (Linux): on Wayland, the output's ICC through the color management protocol `wp_color_management_v1`; on X11, the `_ICC_PROFILE` property of the root window (ICC Profiles In X Specification); if neither, colord is queried. If none, sRGB as it is (NRSD's request, 2026-10-01).
- Reduced images: 512 pixels on the short side (initial value). Named by the photo's SHA-256. The location is passed by the caller (11.8 “Where kept”).
- Risk: color accuracy of moxcms. Impact: colors may shift. Preparation: compare with lcms2 and swap through the `ColorEngine` switch (14.3, Overall Implementation Design 2.1)
- Component: jpeg-encoder (has quality, chroma subsampling, progressive, ICC embedding). image's JPEG export cannot choose chroma subsampling or progressive, so it is not used.
- Quality 92 (initial value), no chroma subsampling (4:4:4), progressive. Social media recompress uploaded images, so colors are not subsampled at export to avoid stacking degradation.
- An sRGB ICC is embedded.

### 14.4 Large images and parallel processing

- Photos handled are up to 100 million pixels and 200 MB per image (Chapter 1, 14.1 “Scale and performance targets”). Photos larger than this are not taken in. The memory estimate per image and the loading limit follow Chapter 3, 2 “Input”.
- The number of images processed at the same time is the smaller of “the number of logical CPU cores” and “half of free memory divided by the estimate per image (height × width × bytes per pixel × 3; bytes per pixel are 4 for 8-bit RGBA and 8 for 16-bit RGBA; three images for loading, resizing, and drawing)”. At least 1.
- The watermark and auto-placement models are not held at the same time; only the one in use is loaded, and memory use is written to the operation log (Chapter 3, 5; Chapter 11, 7.1).

### 14.5 Location and names

- Location: under the folder the User chooses, the folders of Chapter 3, 10.1 “Names and structure” are created.
- Names: per Chapter 3, 10.1 “Names and structure”. If a file with the same name exists, it is not overwritten, and a number is added after the name.
- Files being written are written under a temporary name and renamed on completion. Even if stopped midway, no half-written files remain.
- The same folder as the Original cannot be chosen as the export destination (prevents accidentally overwriting the Original).
- Before export, the free space at the destination is estimated (estimated size per image × number of images), and export does not start if it is insufficient.

### 14.6 Interruption and resumption

- If the app ends in the middle of export, the exported photos are recorded in the work session. Open the work session and “Continue exporting” exports only the rest.
- Interruption and resumption work the same across restarts of the app or the OS (completion is kept in the per-photo records). Several exports can be queued, and one runs at a time (Chapter 3, 9) (NRSD's request, 2026-09-30).

## 15. Data Formats

### 15.1 Template (`schema: nrsd.template/2`)

|Field|Type|Content|
|---|---|---|
|`id`|String (UUID v7)|Template number|
|`name`|String|Name|
|`version`|Integer|Version number (8.2)|
|`created_at`, `updated_at`|Date/time (RFC 3339)|—|
|`delivery_too`|Boolean|Whether to draw on delivery as well (2)|
|`blocks`|List of groups|Groups (table below)|
|`assets`|List of assets|SHA-256 and kind of images of image layers and imported fonts|

Groups:

|Field|Type|Content|
|---|---|---|
|`id`, `name`|String|—|
|`anchor`|One of the 9 points|Anchor (7.1)|
|`inset`|Two numbers (ratios)|Margin from the anchor|
|`rotation`|Number (degrees)|—|
|`auto_place`|Boolean|Whether it is an auto-placement target|
|`placement`|`single`, `tile`|Placement (7.4 “Tiling”)|
|`tile`|Arrangement (`grid`, `diagonal`), angle, spacing (two numbers; ratios), opacity|Only when `placement` is `tile`|
|`layers`|List of layers|From front to back|

Layers (common): `id`, `name`, `kind` (`text`, `image`), `visible`, `locked`, `opacity`, `offset` (position within the group; ratio), `rotation`.

Text layers: `content` (text including placeholders), `style` (each formatting item of 3.2), `date_format`.

Image layers: `asset` (SHA-256), `width` (ratio), `recolor` (none, white, black, color, auto).

Forms of values: `anchor` is one of `top-left`, `top`, `top-right`, `left`, `center`, `right`, `bottom-left`, `bottom`, `bottom-right`. Ratios are numbers from 0 to 1. Colors are `#RRGGBB` (opacity is a separate field) or `auto`. `date_format` is `iso` (2026-09-29), `ja` (2026年9月29日), or `en` (Sep 29, 2026). A font reference is `{name, sha256}` (`sha256` is omitted for bundled fonts). `kind` is `text` or `image`. `placement` is `single` or `tile`, and `tile.pattern` is `grid` or `diagonal`. The JSON Schema (Chapter 1, 8.3) is written with these enumerations.

### 15.2 Work session (`schema: nrsd.session/1`)

|Field|Type|Content|
|---|---|---|
|`id`|String (UUID v7)|Work session number|
|`name`|String|Work session name|
|`created_at`, `updated_at`|Date/time|—|
|`stage`|Stage|Current stage (11.2)|
|`template`|Template|The copy (in the form of 15.1 as is), and the original's `id` and `version`|
|`photos`|List of photos|`path`, `sha256`, `bytes`, `mtime`, `width`, `height`, `status` (12.2), `overrides` (per group `anchor`, `inset`, `scale`, `rotation`; per layer `visible`, `color`; `extra_layers`), `exports` (export record per preset)|
|`export_presets`|List of presets|The presets of 14.1 and the User's changes|
|`id_in_filename`|Boolean|Whether to add the identification number to the file names of social media output (Chapter 3, 7.1 “Identification number”; the initial value is the setting `export.id_in_filename` (Chapter 10, 10), and afterwards the previous choice)|
|`rights`|Choice of Permitted Scope|Chapter 5|
|`coholder`|Choice of joint rights holder (one per work session, or none)|Chapter 2, 5 “Authorizations and Joint Rights”, Chapter 10, G-08“Export: Photos and Purpose”|
|`history`|History|List of steps (10.3) and the current position|

- If the format version (`schema`) goes up, work sessions in the old format are migrated to the new format when opened. The work session before migration is kept in the backups (11.4).
- The fields of 15.2 are held in the division of 11.3 (header, per-photo records, history, index). `photos` are the per-photo files, and `history` is the append-only record.

## 16. Handling Failures

|Event|How it is reported|What the User does|State of the work session|
|---|---|---|---|
|A photo cannot be read (broken, unsupported format)|“Unreadable” and the reason in the grid|Remove or replace it|Other photos can continue|
|A character is in no font|A mark on that character on the editing screen; a list before export|Change the font or the character|Confirmation is requested before export|
|The Visible Signature does not fit|“Needs adjustment” in the grid|Fix size, margin, or lines|—|
|Text is hard to distinguish from the background|“Hard to read” in the grid (an outline is added automatically)|Fix color, outline, or background plate|—|
|Not enough free space at the destination|The estimate is shown before export, and it does not start|Free up space, or choose another location|—|
|Failure in the middle of export|A list of failed photos and reasons (in the form of Chapter 10, DD-10-8“Error displays follow a uniform pattern”)|Fix and “Continue exporting”|Exported ones are recorded|
|Exceeds the file size limit (5 MB for X, etc.)|A list of photos that do not fit even with lowered quality|Reduce the size|—|
|The work session file is broken|Reported when opening|Open with a backup version|—|
|A photo cannot be found|Reported when opening the work session|Choose the location again|—|
|Saving the work session fails (no space, cannot write)|The top of the screen turns the warning color and the reason is shown|Free up space|Retried at the next save|
|Invalid file when importing a template, image, or font|Not imported and the reason is shown|—|—|
|The same work session opened in two apps (a safeguard in case single-instance (Chapter 1, DD-1-11“Only one instance of the app runs per machine”) does not work)|The second one shows that it opened read-only|—|—|

## 17. List of Measurements and Checks

|Item|Criterion|
|---|---|
|Vertical glyphs (three fonts)|Vertical forms in the test sentence (3.3)|
|Drawing match across three OSes|Every pixel value differs by 1 or less (3.5)|
|COLRv1 paints of emoji|All paints can be drawn, with the same shapes and colors as browser rendering (6.3)|
|Drawing and displaying one group|Within 100 ms (9.4)|
|Auto-placement time|Within 1 minute for 500 photos (7.2)|
|Auto-placement hit rate|No adjustment needed on 70% or more of photos (7.2)|
|Difference between moxcms and lcms2|Maximum ΔE2000 less than 1 (14.3)|
|Color difference of drawing onto a 16-bit Adobe RGB TIFF|ΔE2000 from the sRGB values less than 1 (3.5)|
|Display size and recompression on posting sites|Actually upload and check (14.1)|

## 18. Mapping to Requirements

|Requirement number (text in the Outline Design Document)|Sections in this chapter|
|---|---|
|R-4-1-1|3.1 “Content and placeholders”, 5 “Image Layers”|
|R-4-1-2|3.2 “Formatting”, 3.3 “Vertical writing”|
|R-4-1-3|6 “Fonts and Emoji”|
|R-4-1-4|5 “Image Layers”, 6.4 “Imported fonts”, 8.3 “Passing on (export and import)”|
|R-4-1-5|4 “Groups and Layers”|
|R-4-2-1|8 “Templates”|
|R-4-2-2|8.4 “Default templates”|
|R-4-3-1|7.2 “Auto-placement”|
|R-4-3-2|7.3 “When it does not fit”|
|R-4-4-1|9 “Editing Operations”, 10 “Undo and Redo”|
|R-4-4-2|2 “Scope of the Visible Signature”, 11.8 “Where kept”, 14.5 “Location and names”|
|R-4-4-3|11 “Work Sessions (the State in the Middle of Editing)”|
|R-4-5-1|12 “Batch Application”|
|R-4-6-1|14 “Export”|

## 19. Gaps Declared in This Chapter

- The gaps of this chapter follow the table in Chapter 13, 4.1 “Gaps in the mechanism” (the rows whose chapter column is this chapter; with why they cannot be closed, the extent addressed, the remaining risks, and who bears them) (not reproduced in this chapter).

## 20. Corrections to Other Chapters and the Outline Design Document

- Chapter 3: the input to the auto-placement model is 320×320 pixels (the first draft's 512 pixels on the short side was wrong). The Visible Signature is drawn after resizing (14.2). The export component is jpeg-encoder.
- Chapter 8, 2.1 “Arrangement”: add `sessions/` (work sessions) and a place for imported fonts to the app's data location.
- Chapter 10: add to G-25“Image Editing” the placeholder list, the history list, view operations (9.2), and switching sample photos. Add to G-09“Export: Batch Application” filtering by status and adjusting together (12.3). Add to G-08“Export: Photos and Purpose” the choice of export presets.
- Chapter 11: add HarfRust, skrifa, tiny-skia, fast_image_resize, moxcms (or lcms2), and jpeg-encoder to the components.
- Chapter 13: revise the number of decisions (DD-4-1“Visible Signatures are held as a template plus per-photo adjustments” to DD-4-14“Editing is non-destructive” of this chapter).
- Outline Design Document Chapter 4: no change (within the scope of the requirements).
