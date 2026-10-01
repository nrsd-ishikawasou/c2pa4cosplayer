# core-work（业务。模板、作业、履历、自动布局、导出的预设）

## 承担的节
- [第3章 8.3 不改变原图](../../../../docs/cn/design/03_基本设计书_签名处理.md#83-不改变原图)
- 第4章：[1 设计上的决定的一览](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#1-设计决定的一览)、[2 可见签名的范围](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#2-可见签名的范围)、[3.1 内容与插入](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#31-内容与插入字段)、[4.1〜4.4 结构・图层・组・上限](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#41-结构)、[5 图片的图层](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#5-图像图层)、[7.2 自动布局](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#72-自动布局)、[8.1〜8.3 模板的管理・版本・交接](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#81-管理)、[9.4 预览](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#94-预览)、[10.1〜10.6 履历](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#101-历史的单位)、[11.1〜11.9 作业](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#111-作业的单位)、[12.1〜12.4 批量套用](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#121-流程)、[14.1 导出的预设](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#141-导出的预设)、[14.2 处理](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#142-处理)、[14.4〜14.6 并行・存放处・继续](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#144-大的图像与并行的处理)、[15.1 模板](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#151-模板schema-nrsdtemplate2)、[15.2 作业](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#152-作业schema-nrsdsession1)、[16 失败的处理](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#16-失败的处理)、[17 实测与确认的一览](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#17-实测与确认的一览)
- 职责与外部的组件：[第11章 7.1](../../../../docs/cn/design/11_基本设计书_开发基础.md#71-rust组件的划分)

## 依赖
- 下层：[core-common](core-common.md)、[core-store](core-store.md)（`AtomicFile`、`SessionLock`、`SafeArchive`、`Space`、`Schema`、`Settings`）、[core-hash](core-hash.md)、[core-image](core-image.md)（`probe`・`decode`・`Thumbs`・`validate_image`・`DisplayIcc`・`ColorEngine`）、[core-render](core-render.md)（`Fonts`・`Layouter`・`Renderer`・`ContrastRule`）、[core-mark](core-mark.md)（`Onnx::session`：u2netp的推理）、[core-rights](core-rights.md)（`license_short`、`PermittedScope`）、[core-ref](core-ref.md)（`pinned`）。
- 不依赖core-identity：插入的值`InsertContext`由调用方（app-client・core-sign）收集后传入。
- 与备份・其他终端的对齐由本区块的`merge_remote`持有规则，core-backup・core-sync调用。

## 类图
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
    +String content  %% 含{插入}。花括号本身为{{ }}。至多500字
    +TextStyle style
    +DateFormat date_format  %% iso | ja | en
    +Option~Url~ account_url  %% {account}所用的声明所在账号（模板以URL持有。对方没有同一URL则不绘制该行）
  }
  class ExportPreset {
    +PresetId id  %% X | Instagram | Weibo | Xiaohongshu | Pixiv | Patreon | Delivery | Custom
    +Option~u32~ long_edge  %% X・微博・pixiv 2048、Patreon 1920
    +Option~u32~ width  %% Instagram・小红书 1080
    +Format format  %% JPEG。交付用为原格式，自定义为JPEG・PNG
    +Option~u64~ max_bytes  %% X 5MB、微博20MB、pixiv 32MB
    +Option~(f32, f32)~ aspect_range  %% Instagram 1.91:1〜3:4（超出则在导出前通知）
    +u8 quality_floor  %% 80（SizeFitter的下限）
  }
  class Limits {
    <<module>>
    +MAX_BLOCKS 4
    +MAX_LAYERS 20
    +MAX_TEXT_CHARS 500  %% 第4章 4.4。apply_edit拒绝超过的操作
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
    +assets_gc_candidates() [Hash]  %% 任何模板・作业都不引用的素材（第1章 8.4）
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
    +Option~Rfc3339~ taken_at  %% {shot_date}（core-image的Header）
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
    +WorkId work_id  %% 样本00000-00000-000或实际值
  }
  class Sessions {
    +create(photos: [Path], template, purpose, ctx: InsertContext) Session
    +open(id) Session
    +save(session)  %% 把前一版本留在versions/，删除超过5个的旧版本（第4章 11.4）
    +autosave(session)  %% 操作停止2秒后、切换界面时、关闭时
    +versions(id) [Version]
    +recover() [Session]
    +status(id) Stage
    +adopt_template_version(session)
    +relink_photo(session, photo, path_or_folder)
    +remove_photo(session, photo)
    +set_insert_context(session, ctx)
    +pin_reference(session, bundle: RefBundle)
    +precheck(session) [Issue]  %% 需修改・缺字・难以阅读・Instagram的纵横比・空余的估算（1张的估算×张数）・与原图同一文件夹（第4章 14.5、16）
  }
  class HistoryTarget {
    <<enumeration>>
    Session  %% 每个作业1条（批量套用的界面与“修改作业”共用）
    Template  %% 修改模板的状态按模板分别
  }
  class History {
    +undo(target: HistoryTarget)
    +redo(target)
    +list(target) [Step]  %% 从新到旧，说明的文字
    +goto(target, step)
    +coalesce(op) %% 第4章 10.2：拖动在松开鼠标时，方向键・文字的输入为1秒，批量以整个操作为1步
  }
  class Editor {
    +apply_edit(session, photo, op: EditOp)  %% 之后以Layouter的issues更新PhotoStatus（第4章 12.2）
    +apply_bulk(session, photos, op: BulkOp)
    +auto_place(session, photos)
    +preview(session, photo, viewport) PreviewFrame
  }
  class SaliencyModel {
    <<trait>>
    +saliency(image: Image) ProbMap  %% 320×320（不保持纵横比的LANCZOS），以均值(0.485,0.456,0.406)・标准差(0.229,0.224,0.225)归一化，u2netp。把地图还原为照片的纵横比
  }
  class Placer {
    <<trait>>
    +place(blocks, prob_map, photo: Size) [Anchor]  %% 候选的穷举
  }
  class TemplateSnapshot {
    +[Block] blocks  %% 把photo.overrides与extra_layers套用到作业的副本上、组合所需的形式（传给core-render的Layouter）
    +[AssetRef] assets
  }
  class Inserts {
    +Map~String,Option~String~~ values  %% {handle}等插入的值。由InsertContext与Photo（shot_date）生成。没有的值为None（不绘制该行）
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
    +Option~Path~ photo_preview  %% 缩小的预览（仅首次。超过倍率时为可见范围的原尺寸裁切）
    +[(BlockId, RgbaImage, Rect)] blocks  %% 按组的透明图片与框（以tauri::ipc::Response保持二进制）
    +[Issue] issues
  }
  class ImportTplResult {
    <<enumeration>>
    Imported(Template)
    AlreadyHave  %% 同一id且版本相同或更旧
    NewVersion(Template)  %% 同一id且版本更新
    ImportedAsCopy(Template)  %% 同一id・同一版本而内容不同 →“（导入）”
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

## 桥（本区块所有的操作）
|编号|操作（上图）|使用方|由来|
|---|---|---|---|
|B-140|`Templates`（`assets_gc_candidates`由backup使用）|app-client・backup|第4章 8、15.1、第1章 8.4|
|B-141|`Sessions`（创建・打开・保存・恢复・替换・插入）|app-client・sign|第4章 11、12.1、15.2|
|B-142|`History`|app-client|第4章 10|
|B-143|`Editor::auto_place`|app-client|第4章 7.2、12.3|
|B-144|`Editor::preview`（也返回其他组的框，吸附的线由ui计算）|app-client|第4章 9.4|
|B-145|`Editor::apply_edit`、`apply_bulk`|app-client|第4章 4.2、4.3、3.2、10.2|
|B-146|`Exporting::export_plan`（为各`ExportItem`编识别编号并持有。第3章 9）、`presets`|sign・app-client|第4章 14.1、14.6、第3章 9|
|B-147|`Exporting::mark_exported`、`exported`|sign|第3章 9、第4章 14.6|
|B-148|`Sessions::precheck`|app-client|第4章 12.2、14.5、16|
|B-149|`Exporting::parallelism`|sign|第4章 14.4|
|B-150|`Sessions::pin_reference`|app-client|第5章 4、第4章 11.2|
|B-151|`Merge::merge_remote`|sync・backup|第4章 8.2|
|B-152|`Exporting::render_export`|sign|第4章 14.2|
- `EditOp`的种类（选择・移动・大小・旋转・对齐・复制・粘贴・图层的操作・格式・数值输入）与`coalesce`的规则依第4章 10.2。`BulkOp`为第4章 12.3的5种。

## 算法（trait之后）
|trait|实现|切换|由来|
|---|---|---|---|
|`SaliencyModel`|u2netp（ONNX。core-mark的`Onnx::session`）|模型文件的替换（记录版本）|第4章 7.2|
|`Placer`|候选（9点＋竖排4处）的穷举，概率合计的最小|固定|第4章 7.2|
|`MergePolicy`（与core-store共用）|同一编号取版本・保存连号较新者。两者都有变更则两者都保留|固定|第4章 8.2|

## 数据设计（本区块持有形式的记录）
|记录|形式|栏|由来|
|---|---|---|---|
|`templates/<编号>/template.json`（`nrsd.template/2`）|JSON|第4章 15.1的表（`Template`・`Block`・`Layer`）。值的枚举也在15.1|第4章 15.1|
|`templates/<编号>/local.json`（`nrsd.template_local/1`）|JSON|`sample_photos{kind → path}`。不放入交接的文件|第4章 8、9.2|
|`templates/<编号>/draft.json`|JSON|确定前的草稿（同一形式）|第4章 8.2|
|`sessions/<编号>/work.json`（`nrsd.session/1`的头部）|JSON|第4章 15.2中除`photos`・`history`以外的栏|第4章 11.3、15.2|
|`sessions/<编号>/photos/<编号>.json`（`nrsd.session_photo/1`）|JSON|`Photo`的栏|第4章 11.3|
|`sessions/<编号>/history.log`|JSON Lines（1步1行。RFC 6902）|`patch_forward`、`patch_back`、`at`、`label`|第4章 10.3、11.3|
|`sessions/<编号>/snapshot.json`（`nrsd.session_snapshot/1`）|JSON|一览的索引（可重新生成）|第4章 11.3|
|`sessions/<编号>/versions/`、`lock`|副本、OS的锁|第4章 11.4、11.7|第4章 11.8|
|`assets/<SHA-256>`|图片・字体的文件|自带的素材|第4章 5、6.4|
|`.nrsdtpl`（交接）|ZIP|`template.json`与图片图层的图片。上限依第4章 8.3|第4章 8.3|
- 缩小图像由core-image在`thumbs/`生成，本区块传入存放处（第4章 11.8）。
