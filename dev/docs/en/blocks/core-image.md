# core-image (business)

## Sections taken
- [Chapter 1, 10.6 Checking what comes in from outside](../../../../docs/en/design/01_Basic Design_Overall Architecture.md#106-validation-of-inputs-from-outside)
- [Chapter 3, 2 Input](../../../../docs/en/design/03_Basic Design_Signing.md#2-input), [3.6.1 Writing per format](../../../../docs/en/design/03_Basic Design_Signing.md#361-writing-per-format), [8.4 Removing capture information](../../../../docs/en/design/03_Basic Design_Signing.md#84-removal-of-shooting-information), [8.5 Quality of delivery exports](../../../../docs/en/design/03_Basic Design_Signing.md#85-quality-of-delivery-export)
- [Chapter 4, 14.3 Resizing, color conversion, and JPEG export](../../../../docs/en/design/04_Basic Design_Image Editing and Batch Application.md#143-resizing-color-conversion-and-jpeg-export)
- Roles and external components: [Chapter 11, 7.1](../../../../docs/en/design/11_Basic Design_Development Base.md#71-how-the-rust-components-are-divided)

## Dependencies
- Below: [core-common](core-common.md), [core-hash](core-hash.md) (SHA-256 in thumbnail names).
- Handling of images that already carry a C2PA signature (the rows of the table in Chapter 3, 2) is core-sign (manifest verification is c2pa). This block handles only pixels, ICC, orientation, and segments.

## Class diagram
```mermaid
classDiagram
  class Format {
    <<enumeration>>
    Jpeg Png Tiff WebP
  }
  class Header {
    +InputKind kind  %% Decodable (JPEG, PNG, TIFF, WebP) | FingerprintOnly (RAW, HEIC, CMYK TIFF, others)
    +Format format
    +u32 width
    +u32 height
    +u64 bytes
    +u8 bit_depth
    +ColorModel color_model  %% Rgb | Gray | Cmyk
    +bool has_alpha
    +Option~Rfc3339~ taken_at  %% EXIF DateTimeOriginal
    +memory_estimate() u64  %% height x width x bytes per pixel (4 for RGBA8, 8 for RGBA16). Used as the load limit of decode
  }
  class Image {
    +Pixels pixels  %% RGB(A), 8 or 16 bits. Gray is expanded to RGB
    +Option~Icc~ icc
    +IccSource icc_source  %% Embedded | InferredAdobeRgb (EXIF Uncalibrated + InteropIndex R03) | AssumedSrgb
    +u8 bit_depth
    +u32 width
    +u32 height
    +CaptureFields capture
    +bool orientation_applied  %% pixels rotated; output Orientation is 1
    +Option~ColorSpace~ color_converted
    +to_pixels8() Pixels8  %% core-common B-014. The plane passed to the watermark, PDQ, and ISCC
  }
  class Decoder {
    +probe(path) Result~Header, Event~  %% header only. The limits of Chapter 3, 2 (100 million pixels, 200 MB) are here (IMG numbers)
    +decode(path) Result~Image, Event~  %% the load limit is memory_estimate. Broken means Event
    +validate_image(path, formats, max_bytes, max_px) Result  %% image layers, QR images, and other things whose limits are in the caller's section
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
    +encode(image, format, settings: EncodeSettings, tiff_tags: Option~RightsFields~) Bytes  %% for TIFF, XMP (700), Artist (315), and Copyright (33432) are written at encoding (Chapter 3, 3.6.1)
  }
  class EncodeSettings {
    +u8 jpeg_quality
    +bool chroma_subsampling  %% 4:4:4 for both delivery and social media
    +bool progressive
    +Option~Icc~ embed_icc  %% the original ICC for delivery, sRGB for social media
    +TiffCompression tiff  %% Deflate, bit depth kept
    +bool webp_lossless  %% VP8L
    +for_delivery(format) EncodeSettings  %% the table in Chapter 3, 8.5 (JPEG 95, PNG, TIFF Deflate, WebP lossless)
  }
  class StripReport {
    +[Removed] removed  %% Gps | PlaceNames | Serials | OwnerName | MakerNote ("Location information removed" in the processing list)
  }
  class Metadata {
    +strip_capture_metadata(bytes, keep: KeepFields) (Bytes, StripReport)  %% keep is the capture date/time and camera model (Chapter 3, 8.4). Orientation is set to 1
    +write_rights_metadata(bytes, fields: RightsFields) Bytes  %% JPEG APP1, PNG iTXt/eXIf, WebP RIFF (img-parts, little_exif). This block composes the XMP RDF/XML
    +read_rights_metadata(bytes) RightsFields
  }
  class RightsFields {
    +[String] dc_creator  %% both if there is a joint rights holder
    +String photoshop_credit  %% "{handle name} ({Notice account})"
    +String dc_rights
    +String xmpRights_WebStatement
    +[(Lang, String)] xmpRights_UsageTerms  %% per language (xml:lang). English plus the screen language (Chapter 3, 3.3)
    +[Url] plus_Licensor  %% the sequence of LicensorURL
    +String dc_title
    +String exif_artist  %% UTF-8 of Exif 3.0
    +String exif_copyright
  }
  class Thumbs {
    +thumbnail(path, short_side, cache_dir) Image
    +preview_jpeg(path, long_side, cache_dir, display_icc: Option~Icc~) Path
  }
  class DisplayIcc {
    <<module>>
    +display_icc() Option~Icc~  %% Linux: wp_color_management_v1, then X11 _ICC_PROFILE, then colord, otherwise None (sRGB)
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

## Bridges (operations owned by this block)
|No.|Operation (in the diagram above)|Used by|Origin|
|---|---|---|---|
|B-060|`Image`|sign, work, render, mark, case|Chapter 3, 2|
|B-061|`Decoder::probe`|work, sign, case|Chapter 1, 10.6; Chapter 3, 2|
|B-062|`Decoder::decode`|sign, work, case|Chapter 3, 2|
|B-063|`Resampler::resize`|work, sign|Chapter 4, 14.3|
|B-064|`ColorEngine`|sign, work|Chapter 4, 14.3 and DD-4-12|
|B-065|`Encoder::encode`|sign|Chapter 4, 14.3; Chapter 3, 8.5|
|B-066|`Metadata::strip_capture_metadata` (the caller passes the items to keep from the settings)|sign|Chapter 3, 8.4|
|B-067|`Metadata::write_rights_metadata` (the values are made by core-rights' B-124)|sign|Chapter 3, 3.6.1|
|B-068|`Metadata::read_rights_metadata` (read-back verification)|sign|Chapter 3, 3.6.1|
|B-069|`Thumbs` (the cache location is the caller's)|work|Chapter 4, 14.3|
|B-070|`Decoder::validate_image` (the limits are in the caller's section)|work, app-client (QR images)|Chapter 4, 5; Chapter 1, 10.6|
|B-071|`DisplayIcc::display_icc`|work|Chapter 4, 14.3|

## Algorithms (behind traits)
|Trait|Implementation|Switching|Origin|
|---|---|---|---|
|`Resampler`|Lanczos3 of fast_image_resize. Transparency is premultiplied, resized, then unpremultiplied|Fixed|Chapter 4, 14.3|
|`ColorEngine`|moxcms. lcms2 if it does not meet the criteria of Chapter 4, 14.3|Decided by measurement, a build feature|Chapter 4, 14.3|

## Data design
- Files this block writes: thumbnails (`thumbs/<SHA-256 of the photo>.jpg`) and previews (JPEG). Size and quality are Chapter 4, 14.3; the location is the caller's (Chapter 4, 11.8; the cache of Chapter 8, 2.1). They can be regenerated, so they are not records.
- The source of each value of `RightsFields` is Chapter 3, 3.6 (core-rights' page). This block only writes to and reads from the segments.
- Input limits and memory estimates are Chapter 3, 2.
