# core-render（业务。字体、文字的组合、绘制、布局、合成、PDF）

## 承担的节
- [第1章 10.11 PDF的文档](../../../../docs/cn/design/01_基本设计书_整体结构.md#1011-pdf文件)
- [第4章 3.2 格式](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#32-格式)、[3.3 竖排](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#33-竖排)、[3.4 缺少的字](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#34-缺字)、[3.5 绘制](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#35-绘制)、[6.2 替代字体的顺序](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#62-备用字体的序列)、[6.4 自带的字体](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#64-带入的字体)、[7.1 位置与大小](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#71-位置与大小)、[7.3 放不下的情况](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#73-放不下时)、[7.4 平铺](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#74-平铺)
- 职责与外部的组件：[第11章 7.1](../../../../docs/cn/design/11_基本设计书_开发基础.md#71-rust组件的划分)

## 依赖
- 下层：[core-common](core-common.md)、[core-image](core-image.md)（`Image`、`ColorEngine`：把绘制的透明图片匹配到照片的色彩空间与位数）。
- 模板・作业的结构（图层、组、插入）由core-work持有，本区块只接收`TemplateSnapshot`（组合所需的副本）。

## 类图
```mermaid
classDiagram
  class Fonts {
    +load(path) Result~FontId, Event~  %% TTF・OTF，至多50MB。引用为家族名与SHA-256
    +bundled() [FontId]
    +fallback_chain(direction, lang) [FontId]  %% 6.2的默认（中文以LXGW WenKai为首）
    +missing_glyphs(text, chain: [FontId]) [char]
    +supports_vertical(font) bool  %% 不持有vert的字体（Caveat）不能选为竖排的主字体（3.3）
    +resolve(family, sha256) Option~FontId~  %% 没有则None → Layout的Issue::FontMissing
  }
  class Color {
    +opposite() Color  %% 白↔黑，其他按明度的相反取白或黑（3.2“文字颜色的相反”）
  }
  class Tiling {
    +Pattern pattern  %% Grid | Diagonal（默认）
    +f32 angle  %% −45〜45，默认−30
    +(f32, f32) spacing  %% 短边的10〜60%，默认25%
    +f32 opacity  %% 10〜100%，默认20%
  }
  class Issue {
    <<enumeration>>
    TooSmall  %% 字小于1.5%（7.3）
    LowContrast  %% 小于3:1。自动加描边之后再标记（3.2）
    MissingGlyph  %% 任何字体都没有的字（3.4）
    UndrawableGlyph  %% COLRv1无法绘制的填充（6.3）
    FontMissing  %% 没有自带字体的终端（6.4）
  }
  class TextStyle {
    +[FontId] fonts
    +Weight weight
    +f32 size_ratio  %% 相对于短边的比例
    +ColorSpec color  %% 固定 | Auto
    +Align align
    +f32 line_height
    +f32 letter_spacing
    +Option~Outline~ outline
    +Option~Shadow~ shadow
    +Option~Plate~ plate
    +Direction direction  %% 横排 | 竖排
    +bool tcy  %% 纵中横（至多2位的数字。竖排时）
    +f32 rotation
    +f32 opacity
  }
  class Shaper {
    <<trait>>
    +shape(text, style: TextStyle) ShapedLines  %% 按字素的替代、UAX #9，竖排为自制
  }
  class Layout {
    +[PlacedBlock] blocks
    +[Issue] issues  %% 需修改（小于1.5%）、难以阅读
  }
  class PlacedBlock {
    +BlockId block
    +Anchor anchor  %% 9点
    +Margin margin
    +Rect bounds_px
    +f32 scale  %% 超过40%时的缩小
    +Tiling tiling  %% 1处 | 平铺
  }
  class Layouter {
    +layout(snapshot: TemplateSnapshot, overrides, photo: Size, inserts: Inserts) Layout
  }
  class Renderer {
    +render_block(layout: Layout, block: BlockId) RgbaImage  %% 底板 → 阴影（高斯，σ＝模糊/2）→ 描边（round join・cap）→ 填充。表情符号为COLRv1。1/4像素。低对比度则自动加描边
    +render_tile(layout, block) RgbaImage  %% 平铺的1个单元（7.4）
    +composite(photo: Image, layers: [(RgbaImage, Pos)]) Image  %% 把sRGB 8位的透明转换为照片的色彩空间・位数后叠加（core-image的ColorEngine）。平铺则排列
  }
  class ContrastRule {
    <<trait>>
    +auto_color(bg: Pixels) Color  %% 相对亮度0.179的边界
    +contrast_ratio(a: Color, b: Color) f32
  }
  class PdfWriter {
    <<trait>>
    +pdf_a(doc: PdfDocument) Result~Bytes, Fault~  %% PDF/A-2u・3u、PDF/UA-1，未通过验证则为异常检测
  }
  class PdfDocument {
    +[Section] sections  %% 标题・段落・表・图（带替代文字）
    +[EmbeddedFile] embedded  %% AssociationKind
    +Archival level
  }
  Layouter --> Layout
  Layouter ..> Shaper
  Layouter ..> Fonts
  Layout --> PlacedBlock
  Layout --> Issue
  PlacedBlock --> Tiling
  TextStyle --> Color
  Renderer ..> Layout
  Renderer ..> ContrastRule
  Renderer ..> TextStyle
  PdfWriter ..> PdfDocument
```

## 桥（本区块所有的操作）
|编号|操作（上图）|使用方|由来|
|---|---|---|---|
|B-110|`Fonts`|work|第4章 6、3.4|
|B-111|`Layouter::layout`|work|第4章 7.1、7.3、7.4|
|B-112|`Renderer::render_block`|work|第4章 3.5、6.3|
|B-113|`Renderer::composite`|sign|第4章 3.5|
|B-114|`ContrastRule`|work|第4章 3.2|
|B-115|`PdfWriter::pdf_a`|backup・case・app-client（线索的导出）|第1章 10.11|
- 平铺只绘制重复的1个单元后排列（第4章 7.4）。旋转的中心为组的中心（第4章 3.5）。

## 算法（trait之后）
|trait|实现|切换|由来|
|---|---|---|---|
|`Shaper`|HarfRust。竖排（`vert`、vhea・vmtx、英数字的旋转、纵中横）为自制|固定|第4章 3.3|
|`ContrastRule`|WCAG的相对亮度与对比度|初始值的常量（3:1时自动描边）|第4章 3.2|
|`PdfWriter`|krilla（在CI中也用veraPDF验证）。替代为printpdf|固定|第1章 10.11、第11章 12|

## 数据设计
- 本区块不写入记录。自带字体的文件由core-work放在`assets/`（SHA-256的名称。第8章 2.1），本区块接收路径后读取。
- `TextStyle`的栏与范围・默认值依第4章 3.2的表，`PlacedBlock`的基准点・边距・缩小依7.1・7.3，平铺的栏依7.4的表。在模板的记录（`nrsd.template/2`）中持有这些的是core-work（第4章 15.1）。
