# S-10 Image editing (G-25) and templates

Origin: Chapter 4, 4, 8, 9, 10, 11, and 13; Chapter 10, G-25; Chapter 4, 6.4 (fonts), 5 (image layers).

## S-10a Editing a template
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant WK as core-work
  participant RD as core-render
  participant IM as core-image
  participant ST as core-store
  UI->>AC: templates_list / templates_create / duplicate / rename / delete / reorder (B-292)
  AC->>WK: Templates::* (B-140)
  UI->>AC: open_edit(template_id) (B-292)
  AC->>WK: Templates::list, then draft (draft_autosave)
  UI->>AC: set_sample_photo(kind, path) (B-292), then WK Templates::set_sample_photo (local.json)
  loop per operation
    UI->>AC: apply_edit(op) (B-292)
    AC->>WK: Editor::apply_edit (B-145; layers, blocks, formatting)
    AC->>WK: History (B-142; a history per template)
    AC->>WK: Templates::draft_autosave (2 seconds)
    UI->>AC: preview(viewport)
    AC->>WK: Editor::preview (B-144)
    WK->>RD: Layouter::layout / Renderer::render_block (B-111, B-112; needs fixing, hard to read)
    WK->>RD: ContrastRule::auto_color (B-114)
    AC-->>UI: PreviewFrame (the block frames; the snap guides are ui)
  end
  UI->>AC: import_font(path) (B-292)
  AC->>RD: Fonts::load (B-110; up to 50 MB; the one-line license notice shown once)
  UI->>AC: missing_glyphs(text) (B-292), then RD Fonts::missing_glyphs
  UI->>AC: import_layer_image(path) (B-292)
  AC->>IM: Decoder::validate_image (B-070; the limits of Chapter 4, 5)
  AC->>WK: into assets/ (referenced by Hash)
  UI->>AC: save_template (Ctrl+S) (B-292)
  AC->>WK: Templates::save (raises the version by one; Chapter 4, 8.2)
  AC->>ST: AtomicFile (B-021)
  UI->>AC: undo / redo / history / goto (B-292), then WK History
```

## S-10b Editing a work session (G-09's “Edit Visible Signature”, “Open photo”)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant WK as core-work
  UI->>AC: open_edit(session_id, photo) (B-292)
  AC->>WK: Sessions::open (B-141)
  UI->>AC: apply_edit(op, scope: AllPhotos | ThisPhoto) (B-292)
  AC->>WK: Editor::apply_edit (the copy of the template, or photo.overrides / extra_layers)
  AC->>WK: Sessions::autosave (photos/<number>.json, history.log)
  UI->>AC: save_template ("Also save to the template")
  AC->>WK: Templates::save_from_session (B-140; a new version of the original template)
  UI->>AC: route(G-09) (the edits take effect in the grid at once)
  UI->>AC: "Open photo", then create_session(one photo) (B-284; a work session of one photo)
```

## S-10c Handover (.nrsdtpl)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant WK as core-work
  participant ST as core-store
  participant IM as core-image
  UI->>AC: templates_export(id, dest) (B-292)
  AC->>WK: Templates::export_tpl (B-140; JSON and image layers; brought-in fonts are not included by default)
  WK->>ST: SafeArchive::zip_write (B-027)
  UI->>AC: templates_import(path) (B-292)
  AC->>WK: Templates::import_tpl (B-140)
  WK->>ST: SafeArchive::zip_read (B-027; 50 files, 100 MB, fixed names)
  WK->>ST: Schema::validate(nrsd.template) (B-024)
  WK->>IM: Decoder::validate_image (B-070)
  WK-->>AC: ImportTplResult (same id: not imported if older, a new version if newer, "(imported)" if the same version with different content)
```

## Gaps (found later and fixed)
- Saving image layer assets to `assets/` and counting references (Chapter 1, 8.4 “assets brought in that are no longer used”) were not on core-work's page, so `assets_gc_candidates() [Hash]` was added to `Templates` in core-work.md. The time of deletion is, as in Chapter 1, 8.4, at backup consolidation; core-backup's `consolidate` collects the candidates and passes them to core-store's `Retention::sweep_assets` (B-028 of core-store.md).
- Who changes the “needs fixing” status (the state transitions of Chapter 4, 12.2): after `Editor::apply_edit`, core-work updates `PhotoStatus` from the issues of `Layouter::layout` (the note of `Editor::apply_edit` in core-work.md).
