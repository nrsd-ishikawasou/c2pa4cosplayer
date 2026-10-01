# Sequences (for each entry point, every route of what is called where)

- Entry points = screen commands (app-client's B-280 to B-293), start-up, the background queue, receipts from the OS (deep-link, dropped files, line recovery), connections from a counterpart device.
- The names on the arrows are the “Bridges” operations of each block's page (`Block::op`, number B-). When an operation not on a page was needed, it was added to the page before being written here (that is a gap).
- Failure branches are written only for events that have a row in the specification's “Handling of failures” sections.
- Gaps (found later; present in neither the pages nor the specification) are written under “Gaps” at the end of each file, together with where they were fixed (page, specification section). Nothing is left unfixed.

|No.|Entry point|File|
|---|---|---|
|S-01|Start-up (normal, after an abnormal exit, second launch, deep-link, OS shutdown)|[01_Startup.md](01_Startup.md)|
|S-02|First run (G-01 to G-05)|[02_First Run.md](02_First Run.md)|
|S-03|Export (G-08 to G-11; the 14 steps of Chapter 3, 8.1, the queue, cancel, resume)|[03_Export.md](03_Export.md)|
|S-04|Verify and clues (G-13, G-24)|[04_Verify.md](04_Verify.md)|
|S-05|Registering a repost (G-14; the extension's `.wacz`, checking the current state, withdrawal, the evidence set)|[05_Registration.md](05_Registration.md)|
|S-06|Response guidance (G-17; complaint, deadlines, `.ics`)|[06_Response.md](06_Response.md)|
|S-07|Background queue (attaching timestamps later, ERS, reference information and updates, checking Originals, retention sweep, automatic backup, deadline notices)|[07_Background.md](07_Background.md)|
|S-08|Backup (create, restore, open as a successor, emergency kit, locations)|[08_Backup.md](08_Backup.md)|
|S-09|Devices and contacts (linking, adding a contact, sending documents, reconciliation, receiving)|[09_Sync.md](09_Sync.md)|
|S-10|Image editing and templates (G-25, touch-ups in G-09, handover)|[10_Editing.md](10_Editing.md)|
|S-11|Changing, remaking, and authorizing signing information (G-18, G-19)|[11_Keys and Authorizations.md](11_Keys and Authorizations.md)|
|S-12|Settings, erasing records, portable kit, reports (G-20 to G-23)|[12_Settings.md](12_Settings.md)|
