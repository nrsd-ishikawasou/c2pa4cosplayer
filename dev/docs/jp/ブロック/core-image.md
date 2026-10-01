# core-image（業務）

## 受ける節
- [第1章 10.6 外から入るものの確かめ](../../../../docs/jp/design/01_基本設計書_全体構成.md#106-外から入るものの確かめ)
- [第3章 2 入力](../../../../docs/jp/design/03_基本設計書_署名処理.md#2-入力)、[3.6.1 形式ごとの書き込み](../../../../docs/jp/design/03_基本設計書_署名処理.md#361-形式ごとの書き込み)、[8.4 撮影情報の除去](../../../../docs/jp/design/03_基本設計書_署名処理.md#84-撮影情報の除去)、[8.5 受け渡し用の書き出しの品質](../../../../docs/jp/design/03_基本設計書_署名処理.md#85-受け渡し用の書き出しの品質)
- [第4章 14.3 縮小・色の変換・JPEG の書き出し](../../../../docs/jp/design/04_基本設計書_画像編集と一括適用.md#143-縮小色の変換jpeg-の書き出し)
- 役割と外部の部品：[第11章 7.1](../../../../docs/jp/design/11_基本設計書_開発基盤.md#71-rust-の部品の分け方)

## 依存
- 下：[core-common](core-common.md)、[core-hash](core-hash.md)（縮小の画像の名前に SHA-256）。
- 既に C2PA署名がある画像の扱い（第3章 2 の表の行）は core-sign（マニフェストの検証は c2pa）。本ブロックは画素・ICC・向き・区画だけ。

## クラス図
```mermaid
classDiagram
  class Format {
    <<enumeration>>
    Jpeg Png Tiff WebP
  }
  class Header {
    +InputKind kind  %% Decodable（JPEG・PNG・TIFF・WebP）| FingerprintOnly（RAW・HEIC・CMYK の TIFF・その他）
    +Format format
    +u32 width
    +u32 height
    +u64 bytes
    +u8 bit_depth
    +ColorModel color_model  %% Rgb | Gray | Cmyk
    +bool has_alpha
    +Option~Rfc3339~ taken_at  %% EXIF の DateTimeOriginal
    +memory_estimate() u64  %% 縦×横×1画素のバイト数（RGBA8 は4、RGBA16 は8）。decode の読み込みの上限に使う
  }
  class Image {
    +Pixels pixels  %% RGB(A)、8 または 16 ビット。Gray は RGB に広げる
    +Option~Icc~ icc
    +IccSource icc_source  %% Embedded | InferredAdobeRgb（EXIF Uncalibrated + InteropIndex R03）| AssumedSrgb
    +u8 bit_depth
    +u32 width
    +u32 height
    +CaptureFields capture
    +bool orientation_applied  %% 画素を回し、出力の Orientation は 1
    +Option~ColorSpace~ color_converted
    +to_pixels8() Pixels8  %% core-common B-014。透かし・PDQ・ISCC に渡す面
  }
  class Decoder {
    +probe(path) Result~Header, Event~  %% 見出しだけ。第3章 2 の上限（1億画素・200MB）はここで（IMG の番号）
    +decode(path) Result~Image, Event~  %% 読み込みの上限は memory_estimate。壊れていれば Event
    +validate_image(path, formats, max_bytes, max_px) Result  %% 画像の層・QR の画像など、上限が呼ぶ側の節にあるもの
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
    +encode(image, format, settings: EncodeSettings, tiff_tags: Option~RightsFields~) Bytes  %% TIFF は XMP（700）・Artist（315）・Copyright（33432）を符号化の時に書く（第3章 3.6.1）
  }
  class EncodeSettings {
    +u8 jpeg_quality
    +bool chroma_subsampling  %% 受け渡し用・SNS 用とも 4:4:4
    +bool progressive
    +Option~Icc~ embed_icc  %% 受け渡し用は元の ICC、SNS 用は sRGB
    +TiffCompression tiff  %% Deflate、ビット数は保つ
    +bool webp_lossless  %% VP8L
    +for_delivery(format) EncodeSettings  %% 第3章 8.5 の表（JPEG 95・PNG・TIFF Deflate・WebP 劣化なし）
  }
  class StripReport {
    +[Removed] removed  %% Gps | PlaceNames | Serials | OwnerName | MakerNote（処理の一覧に「位置情報を消しました」）
  }
  class Metadata {
    +strip_capture_metadata(bytes, keep: KeepFields) (Bytes, StripReport)  %% keep は撮影日時・機種（第3章 8.4）。Orientation は 1 にする
    +write_rights_metadata(bytes, fields: RightsFields) Bytes  %% JPEG APP1・PNG iTXt/eXIf・WebP RIFF（img-parts、little_exif）。XMP の RDF/XML は本ブロックが組む
    +read_rights_metadata(bytes) RightsFields
  }
  class RightsFields {
    +[String] dc_creator  %% 共同の権利者がいれば両方
    +String photoshop_credit  %% 「{ハンドルネーム}（{告知先アカウント}）」
    +String dc_rights
    +String xmpRights_WebStatement
    +[(Lang, String)] xmpRights_UsageTerms  %% 言語ごと（xml:lang）。英語に画面の言語を併記（第3章 3.3）
    +[Url] plus_Licensor  %% LicensorURL の並び
    +String dc_title
    +String exif_artist  %% Exif 3.0 の UTF-8
    +String exif_copyright
  }
  class Thumbs {
    +thumbnail(path, short_side, cache_dir) Image
    +preview_jpeg(path, long_side, cache_dir, display_icc: Option~Icc~) Path
  }
  class DisplayIcc {
    <<module>>
    +display_icc() Option~Icc~  %% Linux：wp_color_management_v1 → X11 の _ICC_PROFILE → colord → 無ければ None（sRGB）
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

## 橋（このブロックが所有する操作）
|番号|操作（上の図）|使う側|由来|
|---|---|---|---|
|B-060|`Image`|sign・work・render・mark・case|第3章 2|
|B-061|`Decoder::probe`|work・sign・case|第1章 10.6、第3章 2|
|B-062|`Decoder::decode`|sign・work・case|第3章 2|
|B-063|`Resampler::resize`|work・sign|第4章 14.3|
|B-064|`ColorEngine`|sign・work|第4章 14.3、DD-4-12|
|B-065|`Encoder::encode`|sign|第4章 14.3、第3章 8.5|
|B-066|`Metadata::strip_capture_metadata`（残す項目は呼ぶ側が設定から渡す）|sign|第3章 8.4|
|B-067|`Metadata::write_rights_metadata`（値は core-rights の B-124 が作る）|sign|第3章 3.6.1|
|B-068|`Metadata::read_rights_metadata`（読み戻しの検証）|sign|第3章 3.6.1|
|B-069|`Thumbs`（キャッシュの場所は呼ぶ側）|work|第4章 14.3|
|B-070|`Decoder::validate_image`（上限は呼ぶ側の節）|work・app-client（QR の画像）|第4章 5、第1章 10.6|
|B-071|`DisplayIcc::display_icc`|work|第4章 14.3|

## 算法（trait の後ろ）
|trait|実装|切り替え|由来|
|---|---|---|---|
|`Resampler`|fast_image_resize の Lanczos3。透過は掛けてから縮めて戻す|固定|第4章 14.3|
|`ColorEngine`|moxcms。第4章 14.3 の判定を満たさなければ lcms2|実測で決め、ビルドの feature|第4章 14.3|

## データ設計
- 本ブロックが書くファイル：縮小の画像（`thumbs/<写真の SHA-256>.jpg`）とプレビュー（JPEG）。大きさと品質は第4章 14.3、置き場は呼ぶ側（第4章 11.8、第8章 2.1 のキャッシュ）。作り直せるため記録ではない。
- `RightsFields` の各欄の値の出どころは第3章 3.6（core-rights の頁）。本ブロックは区画への書き込みと読み出しだけ。
- 入力の上限と記憶の見積もりは第3章 2。
