# S-03 Export (G-08 → G-09 → G-10 → G-11)

Origin: Chapter 1, 7.2; Chapter 3, 2, 8.1 (14 steps), 9, 10, and 11; Chapter 4, 7.2, 11, 12, and 14; Chapter 5, 3 and 4; Chapter 2, 4.6 and 5.4.

## S-03a Photos and purpose (G-08), then a work session
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant IM as core-image
  participant HS as core-hash
  participant SG as core-sign
  participant ID as core-identity
  participant WK as core-work
  participant RF as core-ref
  UI->>AC: pick_folder / dropped_files
  UI->>AC: probe_photos(paths) (B-284)
  loop per photo
    AC->>IM: Decoder::probe(path) (B-061; format, size, limits)
    AC->>HS: Sha256::sha256_file (B-050; the same photo only once)
    AC->>SG: Inspector::inspect(path) (B-160; an existing C2PA signature: if it is someone else's identity record, export is not possible; Chapter 2, 4.6)
  end
  UI->>AC: list_presets (B-284), then WK Exporting::presets (B-146)
  UI->>AC: create_session(photos, template, purpose, shoot, id_in_filename, coholder, dest) (B-284)
  AC->>ID: IdentityStore::current (B-080)
  AC->>ID: Grants::active_for(root, shoot, date) (B-088; for an authorized person)
  AC->>RF: Store::current (B-221)
  AC->>WK: Sessions::create(photos, template, purpose, InsertContext) (B-141)
  AC->>WK: Sessions::pin_reference(session, bundle) (B-150; the pinning of Chapter 5, 4)
  WK->>IM: Thumbs::thumbnail / preview_jpeg (B-069; app-client passes the cache location)
```

## S-03b Batch application (G-09)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant WK as core-work
  participant MK as core-mark
  participant RD as core-render
  UI->>AC: session_grid (B-284)
  UI->>AC: auto_place_all (B-284)
  AC->>WK: Editor::auto_place(session, photos) (B-143)
  WK->>MK: Onnx::session(U2netp) (B-102)
  WK->>WK: SaliencyModel::saliency, then Placer::place (Chapter 4, 7.2)
  WK->>RD: Layouter::layout (B-111; judging "needs fixing")
  WK->>WK: Sessions::autosave (every 2 seconds)
  AC-->>UI: progress / save_state (B-294)
  loop touch-ups
    UI->>AC: apply_edit / bulk_edit (B-284)
    AC->>WK: Editor::apply_edit / apply_bulk (B-145)
    AC->>WK: History (B-142)
    UI->>AC: preview (B-292)
    AC->>WK: Editor::preview(session, photo, viewport) (B-144)
    WK->>RD: Renderer::render_block (B-112)
  end
  UI->>AC: adopt_template_version (B-284), then WK Sessions::adopt_template_version
```

## S-03c Permitted Scope (G-10; delivery only)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant RT as core-rights
  participant WK as core-work
  participant RF as core-ref
  UI->>AC: get_scope_options / available_countries (B-284)
  AC->>RT: PermittedScope (B-120), Deed::deed (B-125)
  AC->>RF: Store::pinned(session.ref_pin) (B-225; the list of countries is texts/)
  UI->>AC: set_scope(base, addons, countries)
  AC->>WK: Sessions::save (saved in rights)
  UI->>AC: preview_enclosure
  AC->>RT: EnclosureBuilder::build(scope, countries, holder, batch_id, work_ids=sample, ref_pinned) (B-121)
```

## S-03d Execution (G-11; the 14 steps of Chapter 3, 8.1)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant WK as core-work
  participant SG as core-sign
  participant ST as core-store
  participant ID as core-identity
  participant CM as core-common
  participant IM as core-image
  participant HS as core-hash
  participant RD as core-render
  participant MK as core-mark
  participant RT as core-rights
  participant NT as core-net
  UI->>AC: precheck (B-284)
  AC->>WK: Sessions::precheck (B-148; needs fixing, missing glyphs, free space, same folder as the Original)
  AC->>ST: Space::free_space (B-031)
  AC->>ID: IdentityStore::expiry_status (B-087; not started if Stop20)
  AC->>ID: Grants::is_revoked (B-088; checked at the start)
  UI->>AC: start_export (B-284)
  AC->>SG: Exporter::queue().push(session, preset) (export-queue.json)
  AC->>WK: Exporting::export_plan(session) (B-146)
  AC->>WK: Exporting::parallelism (B-149)
  AC->>ID: Keystore::begin_batch (B-086; OS user verification once)
  alt delivery
    AC->>RT: EnclosureBuilder::build (B-121; the Enclosed Document is made first; Chapter 3, 8.1 step 9)
  end
  loop per photo (parallelism photos in parallel)
    AC->>SG: Exporter::export_one(session, item, scope) (B-161)
    SG->>SG: step 1: the identification number is the item.work_id assigned by export_plan (Chapter 3, 9). The collision check at assignment is ST WorkIndex::lookup (B-029)
    SG->>IM: Decoder::decode (B-062; orientation normalized)
    SG->>HS: Sha256 / Pdq::hash (step 2; B-050, B-051)
    SG->>IM: Resampler::resize / ColorEngine::to_srgb (step 3; social media; B-063, B-064)
    SG->>WK: Exporting::render_export(session, photo, size, ctx) (step 4; B-152)
    WK->>RD: Layouter::layout / Renderer::render_block (B-111, B-112)
    SG->>RD: Renderer::composite (B-113)
    SG->>IM: Metadata::strip_capture_metadata (step 5; B-066; the items to keep from the settings)
    SG->>MK: Embedder::embed_image(image, id) (step 6; B-100)
    SG->>HS: Pdq::hash / Iscc::compute (step 7; B-051, B-052)
    SG->>SG: SizeFitter::fit (delivery 8.5, social media 14.1)
    SG->>IM: Encoder::encode (step 8; B-065)
    SG->>RT: RightsText::rights_fields (B-124)
    SG->>IM: Metadata::write_rights_metadata (step 8; B-067)
    SG->>SG: ManifestBuilder::build (step 9; the assertions of 3.1, the actions of 3.4, ingredients and redactions 3.8, the hashes of the Enclosed Document)
    SG->>ID: Keystore::with_signing_key (step 10; B-086)
    SG->>NT: Http::post_timestamp x3 (in parallel; B-042; app-client passes tsa.json)
    SG->>SG: Rfc3161::verify, then the first one into sigTst2, the rest into the tsa field
    SG->>ST: AtomicFile::atomic_write(temporary name ~$…tmp) (step 11; B-021)
    SG->>HS: Sha256::sha256_file (step 12)
    SG->>ST: Chain::append(works, WorkData) (step 12; B-022)
    SG->>SG: Inspector::verify_readback (step 13; C2PA verification, watermark B-101, rights items B-068, PDQ)
    alt fails
      SG->>ST: deletes the temporary file, Event (anomaly detection)
    else
      SG->>ST: AtomicFile::rename_final (step 14)
      SG->>WK: Exporting::mark_exported (B-147)
    end
    AC-->>UI: progress (B-294)
  end
  AC->>ID: Keystore::end_batch
  alt delivery
    AC->>SG: Naming (00_INDEX.txt, placing the Enclosed Document; Chapter 3, 10.1)
  end
  UI->>AC: export_result (B-284; the list of failures and reasons, Chapter 3, 11)
  UI->>AC: platform_residual(preset) (the one line of Chapter 3, 10.3)
  UI->>AC: open_output_folder
```

## S-03e Cancel, interruption, resume
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant SG as core-sign
  participant WK as core-work
  UI->>AC: cancel_export (B-284)
  AC->>SG: Cancel::request (core-common B-012; takes effect at a photo boundary. The one in progress either finishes through step 14 or is discarded with its temporary file)
  Note over AC: across an app exit, it is S-01b's cleanup_after_crash (B-168)
  UI->>AC: home_summary (B-283; "Resume export (N items)")
  UI->>AC: start_export (resume)
  AC->>WK: Exporting::exported(session) (B-147; skips the finished ones)
```

## Gaps (found later and fixed)
- The “check of revoked authorizations” at the start (Chapter 2, 5.4) was not in app-client's precheck flow, so this file places `Grants::is_revoked` after precheck, and one line was added to the dependencies of core-sign.md that app-client calls the `Grants::is_revoked` of core-sign.md's dependencies at the start, not per photo.
- Where `tsa.json` is passed: because core-sign does not depend on core-ref (core-sign.md), `export_one` needs a `tsa_set` argument, so `tsa_set` was added to the arguments of `Exporter::export_one` in core-sign.md.
- Making the Enclosed Document “first” (Chapter 3, 8.1 step 9) happens once at the beginning of the batch, but `00_RIGHTS.json` holds the list of all photos' identification numbers (Chapter 5, 3.1), and identification numbers are assigned per photo in step 1, so the list is not fixed at step 9 of the first photo. It was decided in Chapter 3, 9 that identification numbers for all photos are assigned at the start of the batch (`Exporting::export_plan` holds a `WorkId` per `ExportItem`; noted in B-146 of core-work.md).
- The position of `Keystore::begin_batch` (OS user verification once per batch) is in Chapter 2, 7.4 but was not in the flow diagram, so it was placed in this file (the page already has it).

## Added to the specification
- Chapter 3, 9: identification numbers for all photos are assigned at the start of the batch, and the Enclosed Document is made once at the beginning with those numbers.
