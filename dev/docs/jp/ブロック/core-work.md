# core-work（業務。テンプレート、作業、履歴、自動配置、書き出しの型）

## 受ける節
- [第3章 8.3 原本を変えない](../../../../docs/jp/design/03_基本設計書_署名処理.md#83-原本を変えない)
- 第4章：[1 設計上の決定の一覧](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#1-設計上の決定の一覧)、[2 可視署名の範囲](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#2-可視署名の範囲)、[3.1 中身と差し込み](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#31-中身と差し込み)、[4.1〜4.4 構造・層・まとまり・上限](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#41-構造)、[5 画像の層](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#5-画像の層)、[7.2 自動配置](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#72-自動配置)、[8.1〜8.3 テンプレートの管理・版・受け渡し](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#81-管理)、[9.4 プレビュー](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#94-プレビュー)、[10.1〜10.6 履歴](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#101-履歴の単位)、[11.1〜11.9 作業](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#111-作業の単位)、[12.1〜12.4 一括適用](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#121-流れ)、[14.1 書き出しの型](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#141-書き出しの型)、[14.2 処理](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#142-処理)、[14.4〜14.6 並行・置き場・再開](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#144-大きな画像と並行の処理)、[15.1 テンプレート](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#151-テンプレートschema-nrsdtemplate2)、[15.2 作業](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#152-作業schema-nrsdsession1)、[16 失敗の扱い](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#16-失敗の扱い)、[17 実測と確認の一覧](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#17-実測と確認の一覧)
- 役割と外部の部品：[第11章 7.1](../../../../docs/jp/design/11_基本設計書_開発基盤.md#71-rust-の部品の分け方)

## 依存
- 下：[core-common](core-common.md)、[core-store](core-store.md)（`AtomicFile`、`SessionLock`、`SafeArchive`、`Space`、`Schema`、`Settings`）、[core-hash](core-hash.md)、[core-image](core-image.md)（`probe`・`decode`・`Thumbs`・`validate_image`・`DisplayIcc`・`ColorEngine`）、[core-render](core-render.md)（`Fonts`・`Layouter`・`Renderer`・`ContrastRule`）、[core-mark](core-mark.md)（`Onnx::session`：u2netp の推論）、[core-rights](core-rights.md)（`license_short`、`PermittedScope`）、[core-ref](core-ref.md)（`pinned`）。
- core-identity に依存しない：差し込みの値 `InsertContext` は呼ぶ側（app-client・core-sign）が集めて渡す。
- 控え・別の端末との揃えは本ブロックの `merge_remote` が規則を持ち、core-backup・core-sync が呼ぶ。

## クラス図
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
    +String content  %% {差し込み} を含む。中括弧そのものは {{ }}。500文字まで
    +TextStyle style
    +DateFormat date_format  %% iso | ja | en
    +Option~Url~ account_url  %% {account} に使う告知先アカウント（テンプレートが URL で持つ。相手に同じ URL が無ければその行を描かない）
  }
  class ExportPreset {
    +PresetId id  %% X | Instagram | Weibo | Xiaohongshu | Pixiv | Patreon | Delivery | Custom
    +Option~u32~ long_edge  %% X・微博・pixiv 2048、Patreon 1920
    +Option~u32~ width  %% Instagram・小紅書 1080
    +Format format  %% JPEG。受け渡し用は元の形式、自分で決めるは JPEG・PNG
    +Option~u64~ max_bytes  %% X 5MB、微博 20MB、pixiv 32MB
    +Option~(f32, f32)~ aspect_range  %% Instagram 1.91:1〜3:4（外れれば書き出しの前に知らせる）
    +u8 quality_floor  %% 80（SizeFitter の下限）
  }
  class Limits {
    <<module>>
    +MAX_BLOCKS 4
    +MAX_LAYERS 20
    +MAX_TEXT_CHARS 500  %% 第4章 4.4。apply_edit が超える操作を拒む
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
    +assets_gc_candidates() [Hash]  %% どのテンプレート・作業からも参照されない素材（第1章 8.4）
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
    +Option~Rfc3339~ taken_at  %% {shot_date}（core-image の Header）
    +PhotoStatus status  %% Auto | Edited | NeedsFix | Reposition | Unreadable（第4章 12.2）
    +Overrides overrides
    +[ExportRecord] exports
  }
  class Overrides {
    +Map~BlockId, BlockOverride~ blocks  %% anchor・inset・scale・rotation
    +Map~LayerId, LayerOverride~ layers  %% visible・color
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
    +WorkId work_id  %% 見本 00000-00000-000 または実際
  }
  class Sessions {
    +create(photos: [Path], template, purpose, ctx: InsertContext) Session
    +open(id) Session
    +save(session)  %% 直前の版を versions/ に残し、5つを超えた古いものを消す（第4章 11.4）
    +autosave(session)  %% 操作が止まって2秒、画面を移る時、閉じる時
    +versions(id) [Version]
    +recover() [Session]
    +status(id) Stage
    +adopt_template_version(session)
    +relink_photo(session, photo, path_or_folder)
    +remove_photo(session, photo)
    +set_insert_context(session, ctx)
    +pin_reference(session, bundle: RefBundle)
    +precheck(session) [Issue]  %% 要手直し・字がない・読みにくい・Instagram の縦横比・空きの見積もり（1枚の見積もり×枚数）・原本と同じフォルダー（第4章 14.5、16）
  }
  class HistoryTarget {
    <<enumeration>>
    Session  %% 作業ごとに1本（一括適用の画面と「作業を直す」で共通）
    Template  %% テンプレートを直す状態はテンプレートごとに別
  }
  class History {
    +undo(target: HistoryTarget)
    +redo(target)
    +list(target) [Step]  %% 新しい順、説明の文
    +goto(target, step)
    +coalesce(op) %% 第4章 10.2：ドラッグはマウスを離した時、矢印キー・文字の入力は1秒、一括は操作全体で1段
  }
  class Editor {
    +apply_edit(session, photo, op: EditOp)  %% 後に Layouter の issues で PhotoStatus を更新（第4章 12.2）
    +apply_bulk(session, photos, op: BulkOp)
    +auto_place(session, photos)
    +preview(session, photo, viewport) PreviewFrame
  }
  class SaliencyModel {
    <<trait>>
    +saliency(image: Image) ProbMap  %% 320×320（縦横比を保たず LANCZOS）、平均 (0.485,0.456,0.406)・標準偏差 (0.229,0.224,0.225) で正規化、u2netp。地図を写真の縦横比に戻す
  }
  class Placer {
    <<trait>>
    +place(blocks, prob_map, photo: Size) [Anchor]  %% 候補の総当たり
  }
  class TemplateSnapshot {
    +[Block] blocks  %% 作業の写しに photo.overrides と extra_layers を当てた、組み立てに要る形（core-render の Layouter に渡す）
    +[AssetRef] assets
  }
  class Inserts {
    +Map~String,Option~String~~ values  %% {handle} 等の差し込みの値。InsertContext と Photo（shot_date）から作る。無い値は None（その行を描かない）
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
    +Option~Path~ photo_preview  %% 縮小のプレビュー（初回だけ。倍率を超えた時は見えている範囲の原寸の切り出し）
    +[(BlockId, RgbaImage, Rect)] blocks  %% まとまりごとの透過の画像と枠（tauri::ipc::Response で2進のまま）
    +[Issue] issues
  }
  class ImportTplResult {
    <<enumeration>>
    Imported(Template)
    AlreadyHave  %% 同じ id で版が同じか古い
    NewVersion(Template)  %% 同じ id で版が新しい
    ImportedAsCopy(Template)  %% 同じ id・同じ版で中身が違う →「（取り込み）」
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

## 橋（このブロックが所有する操作）
|番号|操作（上の図）|使う側|由来|
|---|---|---|---|
|B-140|`Templates`（`assets_gc_candidates` は backup が使う）|app-client・backup|第4章 8、15.1、第1章 8.4|
|B-141|`Sessions`（作成・開く・保存・復旧・差し替え・差し込み）|app-client・sign|第4章 11、12.1、15.2|
|B-142|`History`|app-client|第4章 10|
|B-143|`Editor::auto_place`|app-client|第4章 7.2、12.3|
|B-144|`Editor::preview`（他のまとまりの枠も返し、吸着の線は ui が計算）|app-client|第4章 9.4|
|B-145|`Editor::apply_edit`、`apply_bulk`|app-client|第4章 4.2、4.3、3.2、10.2|
|B-146|`Exporting::export_plan`（各 `ExportItem` に識別番号を振って持つ。第3章 9）、`presets`|sign・app-client|第4章 14.1、14.6、第3章 9|
|B-147|`Exporting::mark_exported`、`exported`|sign|第3章 9、第4章 14.6|
|B-148|`Sessions::precheck`|app-client|第4章 12.2、14.5、16|
|B-149|`Exporting::parallelism`|sign|第4章 14.4|
|B-150|`Sessions::pin_reference`|app-client|第5章 4、第4章 11.2|
|B-151|`Merge::merge_remote`|sync・backup|第4章 8.2|
|B-152|`Exporting::render_export`|sign|第4章 14.2|
- `EditOp` の種類（選ぶ・動かす・大きさ・回転・揃え・写す・貼る・層の操作・書式・数で入れる）と `coalesce` の規則は第4章 10.2。`BulkOp` は第4章 12.3 の5つ。

## 算法（trait の後ろ）
|trait|実装|切り替え|由来|
|---|---|---|---|
|`SaliencyModel`|u2netp（ONNX。core-mark の `Onnx::session`）|モデルのファイルの差し替え（版を記録）|第4章 7.2|
|`Placer`|候補（9点＋縦書き4か所）の総当たり、確率の合計の最小|固定|第4章 7.2|
|`MergePolicy`（core-store と共有）|同じ番号は版・保存の連番の新しい方。両方が変わっていれば両方|固定|第4章 8.2|

## データ設計（このブロックが形を持つ記録）
|記録|形|欄|由来|
|---|---|---|---|
|`templates/<番号>/template.json`（`nrsd.template/2`）|JSON|第4章 15.1 の表（`Template`・`Block`・`Layer`）。値の列挙も 15.1|第4章 15.1|
|`templates/<番号>/local.json`（`nrsd.template_local/1`）|JSON|`sample_photos{kind → path}`。受け渡しのファイルに入れない|第4章 8、9.2|
|`templates/<番号>/draft.json`|JSON|確定前の下書き（同じ形）|第4章 8.2|
|`sessions/<番号>/work.json`（`nrsd.session/1` のヘッダ）|JSON|第4章 15.2 のうち `photos`・`history` を除く欄|第4章 11.3、15.2|
|`sessions/<番号>/photos/<番号>.json`（`nrsd.session_photo/1`）|JSON|`Photo` の欄|第4章 11.3|
|`sessions/<番号>/history.log`|JSON Lines（1段1行。RFC 6902）|`patch_forward`、`patch_back`、`at`、`label`|第4章 10.3、11.3|
|`sessions/<番号>/snapshot.json`（`nrsd.session_snapshot/1`）|JSON|一覧の索引（作り直せる）|第4章 11.3|
|`sessions/<番号>/versions/`、`lock`|写し、OS の鍵|第4章 11.4、11.7|第4章 11.8|
|`assets/<SHA-256>`|画像・フォントのファイル|持ち込みの素材|第4章 5、6.4|
|`.nrsdtpl`（受け渡し）|ZIP|`template.json` と画像の層の画像。上限は第4章 8.3|第4章 8.3|
- 縮小の画像は core-image が `thumbs/` に作り、本ブロックは置き場を渡す（第4章 11.8）。
