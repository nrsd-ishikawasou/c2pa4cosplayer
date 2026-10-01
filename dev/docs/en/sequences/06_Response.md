# S-06 Response guidance (G-17)

Origin: Chapter 7, 3.1–3.4, 5, 6.1, 6.3, and 7; Chapter 6, 6.1; Chapter 12, 3.2; Chapter 10, G-17.

```mermaid
sequenceDiagram
  participant UI as ui
  participant AC as app-client
  participant GD as core-guidance
  participant CS as core-case
  participant RF as core-ref
  participant ST as core-store
  participant OS
  UI->>AC: guidance_options(case_id | None) (B-287; 1 to 2)
  AC->>RF: Store::current() (B-221; platforms, templates, laws, deadlines, holidays)
  AC->>CS: Cases::case(id) (B-184; whois, match, violations)
  AC->>GD: Venues::options(case, platforms, whois) (B-200; CDN and hosting from whois)
  AC->>GD: OperatorSteps::operator_steps(whois) (B-204; Chapter 7, 5)
  Note over UI: 3 choose the standing (copyright holder, depicted person, representative)
  UI->>AC: guidance_draft(case, venue, standing, user_fields) (B-287; 4 to 6)
  AC->>GD: Drafter::consent_texts(venue, standing) (the guidance of 3.2 and the per-standing confirmation of 3.2.1; no text is returned without the checks)
  AC->>GD: Drafter::draft(case, venue, standing, user_fields) (B-201)
  GD->>GD: templates(kind, lang), then fills the fields (URL, identification number, registration date/time, and match from the case record), then the reference translation, then warnings
  AC-->>UI: Draft (original, translation, warnings, blank fields)
  UI->>AC: open_url(the contact point's form) / open_mailto(recipient, text) (B-293, B-336)
  Note over UI: 7 after sending
  UI->>AC: record_filing(case, venue, method, sent_at, receipt_no, standing, filing_copy) (B-287)
  AC->>CS: Status::append_status(id, filed, filing_copy) (B-186; the copy, already [REDACTED], goes to filings/)
  CS->>ST: Chain::append(status, filed)
  AC->>GD: Deadlines::deadlines(case, filing, rules, holidays) (B-202)
  AC-->>UI: deadlines ("N days to the deadline" in G-16, "Add to calendar")
  UI->>AC: references(country) (B-287), then GD LawRefs::references / further_steps (B-203; the table in Chapter 12, 3.2)
  Note over UI: the permanent text "This is not legal advice" (Chapter 7, 7)
  AC->>OS: notify (3 days before and on the day; S-07f)
```

## Gaps (found later and fixed)
- Who does the `[REDACTED]` processing of the copy of the complaint (Chapter 6, 6.1): the one that knows the User's fields is core-guidance (`Draft.user_fields`), and saving is core-case, so `Drafter::draft` also returns a “text for the copy” (a version with the User's fields replaced by `[REDACTED]`), and app-client passes it to `append_status` as `filing_copy`. `redacted_copy: String` was added to `Draft` in core-guidance.md.
- Recording the provider's response (`response`), removal, and reposting calls `append_status` directly from G-16 (not through G-17). The commands of B-286 in app-client.md lacked `record_status(id, kind, fields)`, so it was added (`record_filing` is for G-17, `record_status` for G-16).
