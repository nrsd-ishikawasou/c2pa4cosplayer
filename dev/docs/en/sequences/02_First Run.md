# S-02 First run (G-01 → G-02 → G-03 → G-04 → G-05 → G-07)

Origin: Chapter 1, 7.1; Chapter 2, 2.1, 2.3, 2.4, 3, and 7.1; Chapter 8, 5.1 and 5.5; Chapter 10, 3.2 and 6.1; Chapter 12, 4.3.

```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant RF as core-ref
  participant CM as core-common
  participant ID as core-identity
  participant RT as core-rights
  participant ST as core-store
  participant BK as core-backup
  participant RD as core-render
  UI->>AC: get_startup_state (B-280), then before first run
  UI->>AC: set_language (the OS language is the default)
  Note over UI: G-01 Welcome (the one screen on the rights of the original work, the recommendation of OS encryption)
  UI->>AC: get_terms (B-280)
  AC->>RF: Consent::terms_current (B-223; the bundled package)
  UI->>AC: record_consent(version, accepted)
  AC->>RF: Consent::consent (consent.json)
  Note over UI: G-03 Signing information
  UI->>AC: identity_draft_load (B-281)
  AC->>ID: Draft::draft_load
  loop on every input
    UI->>AC: validate_identity_input(input)
    AC->>RF: Store::current().platforms (B-221)
    AC->>ID: Validator::validate_input(input, platforms) (B-081)
    ID->>CM: UrlNormalizer::normalize(url, Account, rules), then Platforms::detect(url, platforms) (B-007)
    AC->>ID: Draft::draft_save
  end
  UI->>AC: keystore_status (B-281)
  alt Linux without Secret Service
    UI->>AC: set_keystore_passphrase (PassphrasePolicy is the same rule as core-backup B-245)
    AC->>BK: PassphrasePolicy::check
    AC->>ID: Keystore::unlock
  end
  UI->>AC: create_identity(input)
  AC->>ID: OsAuth::os_user_verify (B-092)
  AC->>ID: IdentityStore::create(normalized) (B-082)
  ID->>ID: CertBuilder::root / signing (P-256, 20 years; Chapter 2, 3)
  ID->>ID: Keystore::set (root-key, signing-key, device-key)
  ID->>CM: NoticeCode::from_spki_der (B-005; Chapter 2, 2.4)
  ID->>ST: Chain::append(identity, the first row of the change history) (B-022; RecordSigner B-023)
  AC->>ID: IdentityStore::current (B-080)
  Note over UI: G-04 Posting the Notice
  UI->>AC: notice_code / notice_texts(level, lang, platform) (B-281)
  AC->>RT: RightsText::notice_texts (B-123; the example texts of the reference information and the posting site limit)
  UI->>AC: set_setting(identity.notice_posted) (an optional mark)
  Note over UI: G-05 Create a backup (can be skipped)
  UI->>AC: backup_destinations (B-282)
  AC->>BK: Destinations::detect (B-244; at the first time only external media and Cloud)
  UI->>AC: add_destination / set_backup_options(include_originals)
  AC->>BK: Destinations::add
  AC->>BK: RecoveryKey::generate (the public key goes to Settings; Chapter 8, 5.1)
  UI->>AC: check_passphrase(p) (B-282)
  AC->>BK: PassphrasePolicy::check (B-245)
  UI->>AC: emergency_kit (B-282)
  AC->>BK: EmergencyKit::emergency_kit (B-243)
  BK->>CM: Qr::encode(Recovery) (B-013)
  BK->>RD: PdfWriter::pdf_a (B-115; PDF/A-2u + PDF/UA-1)
  UI->>AC: create_backup(dest, passphrase)
  AC->>BK: Writer::create (B-240; S-08a)
  UI->>AC: route(G-07)
  AC->>AC: state = normal (starts the background queue S-01b)
```

## Gaps (found later and fixed)
- When the recovery key is made is in Chapter 8, 5.1 (when the first backup location is decided), but core-backup's page had no “make” operation among its bridges, so, since `RecoveryKey::generate` is in the diagram of core-backup.md, a note was added to the bridge table B-244 of core-backup.md that the first `Destinations::add` of B-244 calls `RecoveryKey::generate`.
- The rule for the strength of the Linux keystore passphrase in the first-run G-03 is Chapter 2, 7.5 “the same as the backup passphrase”, so app-client uses core-backup's `PassphrasePolicy` (B-245). Because core-identity does not depend on core-backup, app-client checks the passphrase first and passes only one that passed to `Keystore::unlock` (one line in the dependencies of app-client.md).
- When the device key (`device-key`) is made was not stated in `IdentityStore::create` on core-identity's page, so “also makes the device key (Chapter 2, 7.1)” was added to the note of `create` in core-identity.md.
