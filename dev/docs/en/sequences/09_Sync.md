# S-09 Linking devices, adding contacts, sending documents, reconciliation, receiving

Origin: Chapter 8, 5.4; Chapter 2, 5.5 and 5.4; Chapter 4, 8.3; Chapter 10, G-18 and G-20.

## S-09a Linking a device (G-20 “Devices”)
```mermaid
sequenceDiagram
  participant UI2 as ui (new device)
  participant AC2 as app-client (new)
  participant SY2 as core-sync (new)
  participant SY1 as core-sync (existing)
  participant AC1 as app-client (existing)
  participant ID as core-identity
  participant ST as core-store
  UI2->>AC2: device_link_qr (B-290), then SY2 Link::link_qr (B-260; Link(device_pubkey, nonce))
  AC1->>SY1: Link::link_from(qr | code_phrase) (B-260; read on the existing device)
  alt passphrase
    SY2->>SY2: CodePhrase::publish (pkarr, 10 minutes)
    SY1->>SY1: CodePhrase::resolve, then Pake::agree (SPAKE2)
  end
  SY1->>SY2: Endpoint::connect (iroh; ALPN sync/1)
  SY1->>SY2: Hello (certificate chain, nonce) / Prove (signature)
  SY2->>ID: Keystore::with_signing_key (B-086; signs the nonce)
  SY1->>ID: IdentityStore::current (B-080; the same personal root?)
  SY1->>SY2: hands over the keys (the secret keys of the personal root and signing certificate) over the wire, then SY2 Keystore::import_keys (B-091)
  SY2->>ST: the device row in peers.json (at most 5 devices; Chapter 8, 2.1)
  SY1->>SY2: Sync::sync_now (S-09d)
```

## S-09b Adding a contact (G-18)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant SY as core-sync
  participant CM as core-common
  participant ID as core-identity
  UI->>AC: contact_qr / contact_passphrase (B-288)
  AC->>SY: Contacts::contact_qr (B-261), then CM Qr::encode(Contact(notice_code, root_sha256, device_pubkey)) (B-013)
  UI->>AC: add_contact(qr | phrase)
  AC->>SY: Contacts::add_contact, then Endpoint::connect, then Hello/Prove
  SY->>ID: IdentityStore::current (B-080; own personal root's SPKI and notice code)
  SY->>SY: SafetyNumber::compute(mine, theirs)
  AC-->>UI: safety number (60 digits in 12 groups)
  UI->>AC: confirm_contact(peer) (compared and the same)
  AC->>SY: Contacts::confirm, then state/peers.json
```

## S-09c Sending and receiving documents and templates
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant ID as core-identity
  participant SY as core-sync
  participant SYp as core-sync (counterpart)
  participant ACp as app-client (counterpart)
  participant IDp as core-identity (counterpart)
  participant WKp as core-work (counterpart)
  UI->>AC: create_grant(...) (B-288), then ID Grants::create_grant (B-088; .nrsdgrant)
  UI->>AC: send_document(peer, doc) (B-288)
  AC->>SY: Transfer::send_document (B-262; outbox; queued until it arrives)
  SY->>SYp: Doc (CBOR)
  SYp->>ACp: peer_event(inbound_document) (B-294)
  ACp->>IDp: Grants::import(bytes) (B-088; the checks of Chapter 2, 5.4; imported automatically from a confirmed contact)
  UI->>AC: send_template(peer, tpl) (B-288), then SY Transfer::send_template (B-263)
  SYp->>ACp: peer_event(inbound_template)
  ACp->>ACp: shown as a "received template" (only when the User presses "Import")
  ACp->>WKp: Templates::import_tpl(path) (B-140; the checks of Chapter 4, 8.3)
```

## S-09d Reconciliation (sync_now; at start, on connection, on a connection from a counterpart device)
```mermaid
sequenceDiagram
  participant AC as app-client
  participant SY as core-sync
  participant SYp as core-sync (another device)
  participant ST as core-store
  participant WK as core-work
  participant RF as core-ref
  AC->>SY: Sync::sync_now() (B-264)
  SY->>SYp: Endpoint::connect, then Hello/Prove (mutual challenge-response)
  SY->>ST: Chain::head x all streams (B-022)
  SY->>SYp: Heads (stream, device number, seq)
  SYp-->>SY: Rows (the missing rows; a root row of "suspected leak" is not sent)
  SY->>ST: Chain::merge_rows (B-022)
  SY->>SYp: Blob (BLAKE3; evidence, assets; iroh-blobs, resumable)
  SY->>WK: Merge::merge_remote(templates, sessions) (B-151)
  SY->>ST: Settings (device-independent items take the newer one; B-025)
  SY->>RF: BundleSource::latest (B-228; if the counterpart's package is newer, Fetcher::refresh imports it)
  SY->>SY: backup_folder (receives the counterpart's backups; B-264)
```

## Gaps (found later and fixed)
- “How the keys are handed to the new device” when linking was not in Chapter 8, 5.4 (Signal's linking hands over keys; “use the same personal root and signing certificate” is Chapter 1, 13 and Chapter 2, 7.3), so “after the link check, the existing device hands over the keys of the personal root and signing certificate over the wire (end-to-end encrypted), and the new device puts them in its keystore (the same result as restoring from a backup)” was added to the procedure between devices in Chapter 8, 5.4. A note was added to `Link::link_from` in core-sync.md, and sync was added to the users of core-identity's `Keystore::import_keys`.
- The trigger for a connection from a counterpart device (when one is the receiving side) was not on app-client's page, so `Endpoint::accept` is always waiting in core-sync while running, and received events are passed to app-client by `peer_event`. “inbound_document, inbound_template, sync_request” was added to the note of B-294 `peer_event` in app-client.md.
