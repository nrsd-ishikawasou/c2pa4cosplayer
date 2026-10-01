# core-work (business; templates, work sessions, history, auto-placement, export presets)

## Sections taken
- [Chapter 3, 8.3 The Original is not changed](../../../../docs/en/design/03_Basic Design_Signing.md#83-the-original-is-not-changed)
- Chapter 4: [1 List of design decisions](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#1-list-of-design-decisions), [2 Scope of the Visible Signature](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#2-scope-of-the-visible-signature), [3.1 Content and insertions](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#31-content-and-placeholders), [4.1–4.4 Structure, layers, blocks, limits](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#41-structure), [5 Image layers](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#5-image-layers), [7.2 Auto-placement](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#72-auto-placement), [8.1–8.3 Managing, versioning, and handing over templates](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#81-management), [9.4 Preview](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#94-preview), [10.1–10.6 History](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#101-unit-of-history), [11.1–11.9 Work sessions](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#111-unit-of-a-work-session), [12.1–12.4 Batch application](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#121-flow), [14.1 Export presets](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#141-export-presets), [14.2 Processing](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#142-processing), [14.4–14.6 Concurrency, locations, resuming](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#144-large-images-and-parallel-processing), [15.1 Templates](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#151-template-schema-nrsdtemplate2), [15.2 Work sessions](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#152-work-session-schema-nrsdsession1), [16 Handling of failures](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#16-handling-failures), [17 List of measurements and confirmations](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#17-list-of-measurements-and-checks)
- Roles and external components: [Chapter 11, 7.1](../../../../docs/en/design/11_Basic Design_Development Base.md#71-how-the-rust-components-are-divided)

## Dependencies
- Below: [core-common](core-common.md), [core-store](core-store.md) (`AtomicFile`, `SessionLock`, `SafeArchive`, `Space`, `Schema`, `Settings`), [core-hash](core-hash.md), [core-image](core-image.md) (`probe`, `decode`, `Thumbs`, `validate_image`, `DisplayIcc`, `ColorEngine`), [core-render](core-render.md) (`Fonts`, `Layouter`, `Renderer`, `ContrastRule`), [core-mark](core-mark.md) (`Onnx::session`: u2netp inference), [core-rights](core-rights.md) (`license_short`, `PermittedScope`), [core-ref](core-ref.md) (`pinned`).
- Does not depend on core-identity: the insertion values `InsertContext` are gathered and passed by the caller (app-client, core-sign).
- Reconciliation with backups and other devices: this block's `merge_remote` holds the rules; core-backup and core-sync call it.

## Class diagram
```mermaid
classDiagram
  class Template {
    +Uuid id
    +String name
    +u32 version
    +Rfc3339 created_at
    +Rfc3339 updated_at
    +bool delivery_too
    +[Block] blocks
    +[AssetRef] assets
  }
  class Block {
    +Uuid id
    +String name
    +Anchor anchor
    +(f32, f32) inset
    +f32 rotation
    +bool auto_place
    +Placement placement
    +Option~Tile~ tile
    +[Layer] layers
  }
  class Layer {
    +Uuid id
    +String name
    +LayerKind kind
    +bool visible
    +bool locked
    +f32 opacity
    +(f32, f32) offset
    +f32 rotation
    +Option~TextLayer~ text
    +Option~ImageLayer~ image
  }
  class TextLayer {
    +String content  %% contains {insertions}. Literal braces are {{ }}. Up to 500 characters
    +TextStyle style
    +DateFormat date_format  %% iso | ja | en
    +Option~Url~ account_url  %% the Notice account used for {account} (the template holds it as a URL; if the recipient has no identical URL, that line is not drawn)
  }
  class ExportPreset {
    +PresetId id  %% X | Instagram | Weibo | Xiaohongshu | Pixiv | Patreon | Delivery | Custom
    +Option~u32~ long_edge  %% X, Weibo, pixiv 2048; Patreon 1920
    +Option~u32~ width  %% Instagram, Xiaohongshu 1080
    +Format format  %% JPEG. Delivery is the original format; Custom is JPEG or PNG
    +Option~u64~ max_bytes  %% X 5 MB, Weibo 20 MB, pixiv 32 MB
    +Option~(f32, f32)~ aspect_range  %% Instagram 1.91:1 to 3:4 (notified before export if outside)
    +u8 quality_floor  %% 80 (the floor of SizeFitter)
  }
  class Limits {
    <<module>>
    +MAX_BLOCKS 4
    +MAX_LAYERS 20
    +MAX_TEXT_CHARS 500  %% Chapter 4, 4.4. apply_edit refuses operations that exceed
  }
  class ImageLayer {
    +Hash asset
    +f32 width
    +Recolor recolor
  }
  class Templates {
    +list() [Template]
    +create(lang) Template
    +duplicate(id) Template
    +rename(id, name)
    +delete(id)
    +reorder(ids)
    +bundled_templates() [Template]
    +export_tpl(id) Path
    +import_tpl(path) ImportTplResult
    +draft_autosave(template)
    +set_sample_photo(id, kind, path)
    +save_from_session(session) Template
    +assets_gc_candidates() [Hash]  %% assets referenced by no template or work session (Chapter 1, 8.4)
  }
  class Session {
    +Uuid id
    +String name
    +Stage stage
    +TemplateCopy template
    +[Photo] photos
    +[ExportPreset] export_presets
    +bool id_in_filename
    +Option~PermittedScope~ rights
    +Option~Party~ coholder
    +RefPin ref_pin
    +InsertContext inserts
  }
  class Photo {
    +u32 index
    +Path path
    +Hash sha256
    +u64 bytes
    +Rfc3339 mtime
    +u32 width
    +u32 height
    +Option~Rfc3339~ taken_at  %% {shot_date} (core-image's Header)
    +PhotoStatus status  %% Auto | Edited | NeedsFix | Reposition | Unreadable (Chapter 4, 12.2)
    +Overrides overrides
    +[ExportRecord] exports
  }
  class Overrides {
    +Map~BlockId, BlockOverride~ blocks  %% anchor, inset, scale, rotation
    +Map~LayerId, LayerOverride~ layers  %% visible, color
    +[Layer] extra_layers
  }
  class InsertContext {
    +String handle
    +Role role
    +[Url] accounts
    +Option~Party~ grantor
    +Option~Uuid~ grant_id
    +Option~Party~ coholder
    +Option~Role~ coholder_role
    +Option~String~ shoot
    +Date export_date
    +Option~String~ license_short
    +WorkId work_id  %% the sample 00000-00000-000 or the real one
  }
  class Sessions {
    +create(photos: [Path], template, purpose, ctx: InsertContext) Session
    +open(id) Session
    +save(session)  %% keeps the previous version in versions/ and deletes old ones beyond five (Chapter 4, 11.4)
    +autosave(session)  %% 2 seconds after operations stop, when leaving the screen, when closing
    +versions(id) [Version]
    +recover() [Session]
    +status(id) Stage
    +adopt_template_version(session)
    +relink_photo(session, photo, path_or_folder)
    +remove_photo(session, photo)
    +set_insert_context(session, ctx)
    +pin_reference(session, bundle: RefBundle)
    +precheck(session) [Issue]  %% needs fixing, missing glyphs, hard to read, Instagram aspect ratio, free space estimate (one photo's estimate x count), same folder as the Original (Chapter 4, 14.5 and 16)
  }
  class HistoryTarget {
    <<enumeration>>
    Session  %% one per work session (shared by the batch application screen and "Edit work session")
    Template  %% the template editing state is separate per template
  }
  class History {
    +undo(target: HistoryTarget)
    +redo(target)
    +list(target) [Step]  %% newest first, with a description
    +goto(target, step)
    +coalesce(op) %% Chapter 4, 10.2: a drag at mouse release, arrow keys and typing at 1 second, a batch is one step for the whole operation
  }
  class Editor {
    +apply_edit(session, photo, op: EditOp)  %% afterwards updates PhotoStatus from the Layouter's issues (Chapter 4, 12.2)
    +apply_bulk(session, photos, op: BulkOp)
    +auto_place(session, photos)
    +preview(session, photo, viewport) PreviewFrame
  }
  class SaliencyModel {
    <<trait>>
    +saliency(image: Image) ProbMap  %% 320x320 (LANCZOS without keeping the aspect ratio), normalized with mean (0.485,0.456,0.406) and std (0.229,0.224,0.225), u2netp. The map is restored to the photo's aspect ratio
  }
  class Placer {
    <<trait>>
    +place(blocks, prob_map, photo: Size) [Anchor]  %% exhaustive over the candidates
  }
  class TemplateSnapshot {
    +[Block] blocks  %% the work session's copy with photo.overrides and extra_layers applied, in the form needed for composition (passed to core-render's Layouter)
    +[AssetRef] assets
  }
  class Inserts {
    +Map~String,Option~String~~ values  %% insertion values such as {handle}. Made from InsertContext and Photo (shot_date). A missing value is None (that line is not drawn)
  }
  class Exporting {
    +export_plan(session) [ExportItem]
    +presets() [ExportPreset]
    +mark_exported(session, photo, preset, work_id, output_hash)
    +exported(session) [ExportRecord]
    +parallelism(photo_sizes) u32
    +render_export(session, photo, target: Size, ctx: InsertContext) [(RgbaImage, Pos)]
  }
  class Merge {
    +merge_remote(templates, sessions) MergeReport
  }
  class PreviewFrame {
    +Option~Path~ photo_preview  %% the reduced preview (first time only; beyond the magnification, a full-size crop of the visible area)
    +[(BlockId, RgbaImage, Rect)] blocks  %% a transparent image and frame per block (binary as is via tauri::ipc::Response)
    +[Issue] issues
  }
  class ImportTplResult {
    <<enumeration>>
    Imported(Template)
    AlreadyHave  %% same id with the same or older version
    NewVersion(Template)  %% same id with a newer version
    ImportedAsCopy(Template)  %% same id and version with different content, then "(imported)"
    Rejected(Event)
  }
  Template --> Block
  Block --> Layer
  Session --> ExportPreset
  History ..> HistoryTarget
  Editor ..> Limits
  Layer --> TextLayer
  Layer --> ImageLayer
  Session --> Photo
  Session --> InsertContext
  Photo --> Overrides
  Templates --> Template
  Sessions --> Session
  Editor ..> SaliencyModel
  Editor ..> Placer
  Exporting ..> Session
  Exporting --> TemplateSnapshot
  Exporting --> Inserts
```

## Bridges (operations owned by this block)
|No.|Operation (in the diagram above)|Used by|Origin|
|---|---|---|---|
|B-140|`Templates` (`assets_gc_candidates` is used by backup)|app-client, backup|Chapter 4, 8 and 15.1; Chapter 1, 8.4|
|B-141|`Sessions` (create, open, save, recover, relink, insertions)|app-client, sign|Chapter 4, 11, 12.1, and 15.2|
|B-142|`History`|app-client|Chapter 4, 10|
|B-143|`Editor::auto_place`|app-client|Chapter 4, 7.2 and 12.3|
|B-144|`Editor::preview` (also returns the frames of the other blocks; ui computes the snap guides)|app-client|Chapter 4, 9.4|
|B-145|`Editor::apply_edit`, `apply_bulk`|app-client|Chapter 4, 4.2, 4.3, 3.2, and 10.2|
|B-146|`Exporting::export_plan` (assigns and holds an identification number per `ExportItem`; Chapter 3, 9), `presets`|sign, app-client|Chapter 4, 14.1 and 14.6; Chapter 3, 9|
|B-147|`Exporting::mark_exported`, `exported`|sign|Chapter 3, 9; Chapter 4, 14.6|
|B-148|`Sessions::precheck`|app-client|Chapter 4, 12.2, 14.5, and 16|
|B-149|`Exporting::parallelism`|sign|Chapter 4, 14.4|
|B-150|`Sessions::pin_reference`|app-client|Chapter 5, 4; Chapter 4, 11.2|
|B-151|`Merge::merge_remote`|sync, backup|Chapter 4, 8.2|
|B-152|`Exporting::render_export`|sign|Chapter 4, 14.2|
- The kinds of `EditOp` (select, move, size, rotate, align, copy, paste, layer operations, formatting, numeric entry) and the `coalesce` rules are Chapter 4, 10.2. `BulkOp` is the five of Chapter 4, 12.3.

## Algorithms (behind traits)
|Trait|Implementation|Switching|Origin|
|---|---|---|---|
|`SaliencyModel`|u2netp (ONNX; core-mark's `Onnx::session`)|Replacing the model file (the version is recorded)|Chapter 4, 7.2|
|`Placer`|Exhaustive over the candidates (9 points plus 4 vertical positions), minimum sum of probabilities|Fixed|Chapter 4, 7.2|
|`MergePolicy` (shared with core-store)|The same number takes the newer version or save sequence. If both changed, both are kept|Fixed|Chapter 4, 8.2|

## Data design (records whose form this block holds)
|Record|Form|Fields|Origin|
|---|---|---|---|
|`templates/<number>/template.json` (`nrsd.template/2`)|JSON|The table in Chapter 4, 15.1 (`Template`, `Block`, `Layer`). The enumerated values are also 15.1|Chapter 4, 15.1|
|`templates/<number>/local.json` (`nrsd.template_local/1`)|JSON|`sample_photos{kind → path}`. Not put into the handover file|Chapter 4, 8 and 9.2|
|`templates/<number>/draft.json`|JSON|The draft before confirmation (the same form)|Chapter 4, 8.2|
|`sessions/<number>/work.json` (the header of `nrsd.session/1`)|JSON|The fields of Chapter 4, 15.2 except `photos` and `history`|Chapter 4, 11.3 and 15.2|
|`sessions/<number>/photos/<number>.json` (`nrsd.session_photo/1`)|JSON|The fields of `Photo`|Chapter 4, 11.3|
|`sessions/<number>/history.log`|JSON Lines (one step per line; RFC 6902)|`patch_forward`, `patch_back`, `at`, `label`|Chapter 4, 10.3 and 11.3|
|`sessions/<number>/snapshot.json` (`nrsd.session_snapshot/1`)|JSON|Index for the list (can be regenerated)|Chapter 4, 11.3|
|`sessions/<number>/versions/`, `lock`|Copies, the OS lock|Chapter 4, 11.4 and 11.7|Chapter 4, 11.8|
|`assets/<SHA-256>`|Image and font files|Assets brought in|Chapter 4, 5 and 6.4|
|`.nrsdtpl` (handover)|ZIP|`template.json` and the images of image layers. Limits are Chapter 4, 8.3|Chapter 4, 8.3|
- Thumbnails are made by core-image in `thumbs/`; this block passes the location (Chapter 4, 11.8).
