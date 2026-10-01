# S-11 Changing signing information, remaking, authorizations and joint rights, corrected versions

Origin: Chapter 2, 3.4, 3.6, 5.1–5.4, 6, and 7.2; Chapter 5, 2.4; Chapter 10, G-18 and G-19; the corrections of Chapter 3, 10.2.

## S-11a Changing signing information (G-19)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant ID as core-identity
  participant RF as core-ref
  participant ST as core-store
  UI->>AC: identity_status (B-289), then ID IdentityStore::expiry_status (B-087), history (B-083)
  UI->>AC: update_identity(input) (B-289)
  AC->>RF: Store::current().platforms (B-221)
  AC->>ID: Validator::validate_input (B-081)
  AC->>ID: OsAuth::os_user_verify (B-092)
  AC->>ID: IdentityStore::update_profile (B-083; remakes the signing certificate; the expiry is the same as the personal root)
  ID->>ST: Chain::append(identity/history, field=handle|role|accounts|cert) (B-022)
  AC-->>UI: that the notice code does not change (the example texts of G-04 shown again)
```

## S-11b Remaking the personal root (20 years, loss, suspected leak)
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant ID as core-identity
  participant ST as core-store
  participant SY as core-sync
  UI->>AC: rotate_root(reason) (B-289; confirmation dialog, Chapter 10, 3.6)
  AC->>ID: OsAuth::os_user_verify (B-092)
  AC->>ID: IdentityStore::rotate_root(reason) (B-084)
  ID->>ID: CertBuilder::root / signing (new)
  ID->>ID: PastRoots::add_past_root(old, period, reason) (public key only; the secret key is deleted)
  ID->>ST: Chain::append(identity/history, field=root; signed with the old key, the chain up to the old root) (B-022)
  ID->>ID: Keystore (replaced with the new keys)
  alt 20 years / loss
    AC->>SY: Sync::sync_now (B-264; the root row goes to the other devices)
  else suspected leak
    AC->>AC: not sent (conveyed by importing a backup; Chapter 2, 3.6)
  end
  AC-->>UI: the new notice code, guidance to keep the old code in the Notice as "old code (until date)" (G-04)
  UI->>AC: add_past_root_from_image(path) (B-289; from an old output)
  AC->>SG: Inspector::inspect (B-160; the root of x5chain), then ID PastRoots::add_past_root(provenance=work data present | backup | unverifiable) (B-085)
```

## S-11c Authorizations and joint rights (G-18), corrected Enclosed Documents
```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant ID as core-identity
  participant ST as core-store
  participant RT as core-rights
  participant SG as core-sign
  participant RF as core-ref
  UI->>AC: list_grants (B-288), then ID Grants::list (B-088; active, out of term, revoked)
  UI->>AC: create_grant(to, scope, term, text) / create_revoke(id) / co_rights_draft(...) (B-288)
  AC->>ID: Grants::* (B-088; JWS detached with Keystore::with_signing_key, x5c)
  ID->>ST: Chain::append(approvals) (B-022)
  UI->>AC: import_grant(path) (B-288)
  AC->>ID: Grants::import(bytes) (Chapter 2, 5.4: format, signature, notice code, counterpart (current or past root), term)
  ID->>ID: PastRoots (B-085; one addressed to an old code marked as suspected leak is confirmed with the User)
  UI->>AC: co_rights_countersign(doc) (B-288), then ID Grants::co_rights_countersign
  UI->>AC: renewals_due (B-288), then ID Grants::renewals_due (a renewal document is made automatically 30 days before expiry, with a proposal to send it to the confirmed contact)
  Note over UI: corrected versions (G-12, G-21)
  UI->>AC: corrections_pending (B-285)
  AC->>RF: Store::corrections_since(version) (B-227)
  AC->>SG: Works::works(query=the wording files and versions used) (B-167)
  UI->>AC: errata(work_ids) (B-285)
  AC->>RT: EnclosureBuilder::errata(work_ids, correction) (B-126; 00_RIGHTS_ERRATA.txt and .sig)
  AC->>SG: Works::correct(id, correction) (B-167; the appended correction row)
  UI->>AC: regenerate_enclosure(work_id) (B-285), then RT EnclosureBuilder::regenerate (B-126; the version at the time, Store::pinned B-225)
```

## Gaps (found later and fixed)
- The index for looking up `Works::works(query)` by “the SHA-256 of the wording files used” was not in core-sign's data design (it is in the `rights` field of the work data), so `lookup_by_text_hash(sha256)` was added to `WorkIndex` (core-store B-029) (core-store.md).
- The trigger for “automatically making” the renewal document of an authorization (`renews`) (Chapter 2, 5.1) was not in the background queue, so `Grants::renewals_due` is run on the same tick as S-07f and the documents made are shown in G-21 (“proposal to renew authorizations (Chapter 2, 5.1)” was added to “deadline notices” at the end of the background order in Chapter 1, 7).
