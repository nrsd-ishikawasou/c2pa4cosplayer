# S-08 Backup (create, restore, open as a successor, monthly check)

Origin: Chapter 8, 5.1–5.3, 5.5, 5.6, and 8; Chapter 1, 7.7; Chapter 10, G-05 and G-06.

## S-08a Create (manual B-240 create, automatic run_due, on close)
```mermaid
sequenceDiagram
  participant AC as app-client
  participant BK as core-backup
  participant ID as core-identity
  participant ST as core-store
  participant SG as core-sign
  participant HS as core-hash
  participant UI as ui
  AC->>AC: ensures no export or registration is in progress (Chapter 8, 5.2 step 2)
  AC->>BK: Scheduler::run_due(now, changed) / Writer::create(dest, passphrase, kind, since) (B-240)
  BK->>BK: Destinations::list (only reachable locations. Device is what app-client wired from core-sync's backup_folder)
  BK->>ID: OsAuth::os_user_verify (B-092)
  BK->>ID: Keystore::export_keys (B-091; keys/)
  BK->>ST: Chain::read (B-022; rows since the previous manifest; all for a full backup)
  BK->>SG: Works (the Original diff; Cloud locations exclude Originals)
  BK->>ST: SafeArchive::zip_write (B-027; sequential), then age (the recovery key's public key; the secret key wrapped with the passphrase as an attachment)
  BK->>ST: AtomicFile (temporary name, then re-read and checked against the SHA-256 in manifest.json, then the official name)
  BK->>ST: Settings::set(backup.last_at) (B-025)
  AC-->>UI: progress / notice ("Last backup: N minutes ago")
  alt no location reachable
    BK->>BK: unreachable_days (UI notice at 7 days)
  end
```

## S-08b Restore (G-06)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant BK as core-backup
  participant CM as core-common
  participant ST as core-store
  participant ID as core-identity
  participant WK as core-work
  UI->>AC: restore_backup(path, secret, merge) (B-282)
  alt recovery key QR
    UI->>AC: decode_recovery_qr(image), then core-image Decoder::validate_image (20 MB, 8,000 pixels; Chapter 1, 10.10), then decode, then CM Qr::decode(pixels) (B-013)
  end
  AC->>BK: Reader::restore(path, secret, merge_policy) (B-241)
  BK->>BK: Reader::open (the age header; refuses above scrypt 2^22)
  BK->>ST: SafeArchive::zip_read (B-027; the extracted total matches the list)
  BK->>ST: Schema::validate(manifest.json) (B-024; a version newer than its own "prompts to update")
  BK->>BK: checks each file's SHA-256 against the list (diffs in seq order from the full one, the chain of parent_sha256)
  BK->>ID: IdentityStore::current (B-080; comparison of the personal root)
  alt no records
    BK->>ID: Keystore::import_keys (B-091)
    BK->>ST: extracted to a temporary place, then Chain::verify (B-022), then to the official place
  else same personal root
    BK->>ST: Chain::merge_rows (B-022; reconciliation per Chapter 1, 8.3)
    BK->>WK: Merge::merge_remote(templates, sessions) (B-151; Chapter 4, 8.2)
  else different personal root
    BK-->>AC: choice (create a backup and replace / cancel), then UI confirmation dialog
  end
  AC-->>UI: RestoreReport (counts, records that did not pass)
```

## S-08c Open as a successor, monthly check, consolidation
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant BK as core-backup
  participant ST as core-store
  UI->>AC: open_as_successor(path, recovery_key) (B-282)
  AC->>BK: Reader::open_as_successor (B-242; the keys are not put into the keystore)
  AC->>AC: state = read-only (viewing, evidence set, regenerating the Enclosed Document only)
  AC->>BK: Scheduler::verify_monthly (B-246; only with twice the free space; actually restores to a temporary place and Chain::verify)
  AC->>BK: Scheduler::consolidate (B-246; folds the diffs into a full one and deletes the folded diffs)
  BK->>ST: Settings::set(backup.last_verified[dest])
```

## Gaps (found later and fixed)
- The state “opened as a successor” was not among the seven states of Chapter 1, 7 (it is in Chapter 1, 13 and Chapter 8, 5.3), so “successor (read-only; Chapter 8, 5.3)” was added to the states of Chapter 1, 7, making eight. `Successor` was added to `AppState` in app-client.md.
- The judgment of “a backup made by a newer app than its own” when restoring uses the `record_versions` and `app_version` of manifest.json, so it is compared by core-backup, not by core-store's `Schema::validate` (as in the diagram). Added to the note of `Reader::open` in core-backup.md.
