# core-image（业务）

## 承担的节
- [第1章 10.6 从外部进入之物的确认](../../../../docs/cn/design/01_基本设计书_整体结构.md#106-来自外部输入的确认)
- [第3章 2 输入](../../../../docs/cn/design/03_基本设计书_签名处理.md#2-输入)、[3.6.1 按格式的写入](../../../../docs/cn/design/03_基本设计书_签名处理.md#361-各格式的写入)、[8.4 拍摄信息的去除](../../../../docs/cn/design/03_基本设计书_签名处理.md#84-删除拍摄信息)、[8.5 交付用导出的品质](../../../../docs/cn/design/03_基本设计书_签名处理.md#85-交付用导出的质量)
- [第4章 14.3 缩小・颜色的转换・JPEG的导出](../../../../docs/cn/design/04_基本设计书_图片编辑与批量套用.md#143-缩小颜色的转换jpeg的导出)
- 职责与外部的组件：[第11章 7.1](../../../../docs/cn/design/11_基本设计书_开发基础.md#71-rust组件的划分)

## 依赖
- 下层：[core-common](core-common.md)、[core-hash](core-hash.md)（缩小图像的名称用SHA-256）。
- 已有C2PA签名的图片的处理（第3章 2的表的行）为core-sign（清单的验证为c2pa）。本区块只处理像素・ICC・方向・区段。

## 类图
```mermaid
classDiagram
  class Format {
    <<enumeration>>
    Jpeg Png Tiff WebP
  }
  class Header {
    +InputKind kind  %% Decodable（JPEG・PNG・TIFF・WebP）| FingerprintOnly（RAW・HEIC・CMYK的TIFF・其他）
    +Format format
    +u32 width
    +u32 height
    +u64 bytes
    +u8 bit_depth
    +ColorModel color_model  %% Rgb | Gray | Cmyk
    +bool has_alpha
    +Option~Rfc3339~ taken_at  %% EXIF的DateTimeOriginal
    +memory_estimate() u64  %% 纵×横×1像素的字节数（RGBA8为4、RGBA16为8）。用于decode读取的上限
  }
  class Image {
    +Pixels pixels  %% RGB(A)、8或16位。Gray扩展为RGB
    +Option~Icc~ icc
    +IccSource icc_source  %% Embedded | InferredAdobeRgb（EXIF Uncalibrated + InteropIndex R03）| AssumedSrgb
    +u8 bit_depth
    +u32 width
    +u32 height
    +CaptureFields capture
    +bool orientation_applied  %% 旋转像素，输出的Orientation为1
    +Option~ColorSpace~ color_converted
    +to_pixels8() Pixels8  %% core-common B-014。传给水印・PDQ・ISCC的面
  }
  class Decoder {
    +probe(path) Result~Header, Event~  %% 仅头部。第3章 2的上限（1亿像素・200MB）在此（IMG的编号）
    +decode(path) Result~Image, Event~  %% 读取的上限为memory_estimate。损坏则为Event
    +validate_image(path, formats, max_bytes, max_px) Result  %% 图片的图层・QR的图片等，上限在调用方的节中的东西
  }
  class Resampler {
    <<trait>>
    +resize(image, target: Size) Image
  }
  class ColorEngine {
    <<trait>>
    +to_srgb(image) Image
    +convert_to(image, icc) Image
    +to_8bit(image) Image
  }
  class Encoder {
    +encode(image, format, settings: EncodeSettings, tiff_tags: Option~RightsFields~) Bytes  %% TIFF在编码时写入XMP（700）・Artist（315）・Copyright（33432）（第3章 3.6.1）
  }
  class EncodeSettings {
    +u8 jpeg_quality
    +bool chroma_subsampling  %% 交付用・社交平台用都为4:4:4
    +bool progressive
    +Option~Icc~ embed_icc  %% 交付用为原ICC，社交平台用为sRGB
    +TiffCompression tiff  %% Deflate，保持位数
    +bool webp_lossless  %% VP8L
    +for_delivery(format) EncodeSettings  %% 第3章 8.5的表（JPEG 95・PNG・TIFF Deflate・WebP无损）
  }
  class StripReport {
    +[Removed] removed  %% Gps | PlaceNames | Serials | OwnerName | MakerNote（处理一览中“已删除位置信息”）
  }
  class Metadata {
    +strip_capture_metadata(bytes, keep: KeepFields) (Bytes, StripReport)  %% keep为拍摄日期时间・机型（第3章 8.4）。Orientation设为1
    +write_rights_metadata(bytes, fields: RightsFields) Bytes  %% JPEG APP1・PNG iTXt/eXIf・WebP RIFF（img-parts、little_exif）。XMP的RDF/XML由本区块组成
    +read_rights_metadata(bytes) RightsFields
  }
  class RightsFields {
    +[String] dc_creator  %% 有共同权利人则两者都
    +String photoshop_credit  %% “{网名}（{声明所在账号}）”
    +String dc_rights
    +String xmpRights_WebStatement
    +[(Lang, String)] xmpRights_UsageTerms  %% 按语言（xml:lang）。英文并记界面的语言（第3章 3.3）
    +[Url] plus_Licensor  %% LicensorURL的序列
    +String dc_title
    +String exif_artist  %% Exif 3.0的UTF-8
    +String exif_copyright
  }
  class Thumbs {
    +thumbnail(path, short_side, cache_dir) Image
    +preview_jpeg(path, long_side, cache_dir, display_icc: Option~Icc~) Path
  }
  class DisplayIcc {
    <<module>>
    +display_icc() Option~Icc~  %% Linux：wp_color_management_v1 → X11的_ICC_PROFILE → colord → 没有则None（sRGB）
  }
  class InputKind {
    <<enumeration>>
    Decodable FingerprintOnly
  }
  Header --> InputKind
  Metadata --> StripReport
  Decoder --> Header
  Decoder --> Image
  Encoder ..> EncodeSettings
  Metadata --> RightsFields
  Thumbs ..> Resampler
  Thumbs ..> ColorEngine
  Thumbs ..> Encoder
```

## 桥（本区块所有的操作）
|编号|操作（上图）|使用方|由来|
|---|---|---|---|
|B-060|`Image`|sign・work・render・mark・case|第3章 2|
|B-061|`Decoder::probe`|work・sign・case|第1章 10.6、第3章 2|
|B-062|`Decoder::decode`|sign・work・case|第3章 2|
|B-063|`Resampler::resize`|work・sign|第4章 14.3|
|B-064|`ColorEngine`|sign・work|第4章 14.3、DD-4-12|
|B-065|`Encoder::encode`|sign|第4章 14.3、第3章 8.5|
|B-066|`Metadata::strip_capture_metadata`（保留的项目由调用方从设置传入）|sign|第3章 8.4|
|B-067|`Metadata::write_rights_metadata`（值由core-rights的B-124生成）|sign|第3章 3.6.1|
|B-068|`Metadata::read_rights_metadata`（回读的验证）|sign|第3章 3.6.1|
|B-069|`Thumbs`（缓存的位置由调用方指定）|work|第4章 14.3|
|B-070|`Decoder::validate_image`（上限在调用方的节）|work・app-client（QR的图片）|第4章 5、第1章 10.6|
|B-071|`DisplayIcc::display_icc`|work|第4章 14.3|

## 算法（trait之后）
|trait|实现|切换|由来|
|---|---|---|---|
|`Resampler`|fast_image_resize的Lanczos3。透明先预乘再缩小后还原|固定|第4章 14.3|
|`ColorEngine`|moxcms。不满足第4章 14.3的判定则用lcms2|以实测决定，构建的feature|第4章 14.3|

## 数据设计
- 本区块写入的文件：缩小图像（`thumbs/<照片的SHA-256>.jpg`）与预览（JPEG）。大小与品质依第4章 14.3，存放处由调用方指定（第4章 11.8、第8章 2.1的缓存）。可重新生成，故不是记录。
- `RightsFields`各栏的值的来源见第3章 3.6（core-rights的页面）。本区块只负责向区段的写入与读出。
- 输入的上限与内存的估算见第3章 2。
