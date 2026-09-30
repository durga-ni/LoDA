# Minutes of Meeting — Data Migration Process

**Meeting:** Legacy Database to Local File Storage — Migration Planning Sync
**Date:** 2026-09-22
**Time:** 15:00–16:00 CET
**Location:** Google Meet
**Attendees:** Naga Durga (Data Engineering Lead), Priya Menon (Data Admin), Lars Gutmann (Infrastructure), Sahil Renapurkar (Product Owner), Meera Iyer (QA Lead)
**Absent:** Chris Bauer (Business Ops) — notes to be shared

---

## 1. Meeting Summary

The team met to align on the migration plan for moving customer records, transaction history, and activity logs from the legacy on-premise database system into a local file-based archive (structured flat files on the internal file server) ahead of the Q4 go-live. Naga walked through the current data model mapping and flagged three tables (`accounts`, `transaction_history`, `contact_notes`) with schema mismatches that need transformation logic before export. Priya confirmed that a full export of the legacy system is possible but will require a maintenance window since the export tool locks the production database. Lars raised concerns about the destination storage capacity given the size of the `contact_notes` table (approx. 40M rows with large text blobs). The group agreed on a phased migration: a dry run on a 5% sample first, followed by validation, then the full cutover. Meera flagged that no automated reconciliation script currently exists to compare record counts and field-level values between source and the exported files post-migration — this is now considered a blocker for go-live and needs an owner. The meeting closed with rough dates agreed for the dry run and a follow-up review.

---

## 2. Discussion Points

- **Schema mismatches:** `accounts.region_code` in the legacy system uses a 2-letter internal code with no documented reference list. Priya to supply the mapping table so the exported files use consistent, decoded region names.
- **`transaction_history` table:** stores status transitions as free-text notes rather than structured fields in ~30% of records (older data, pre-2023). Naga proposed a fallback rule: if no structured transition can be parsed, tag the record `needs_manual_review` rather than dropping it or guessing.
- **`contact_notes` size and format:** 40M rows, many containing HTML fragments from the old rich-text editor. Lars asked whether these should be sanitized (stripped to plain text) before export or preserved as-is. Decision deferred pending input from Business Ops on whether historical formatting matters to end users.
- **File format for the archive:** discussed CSV vs. newline-delimited JSON for the exported files. Leaning toward NDJSON for tables with nested/free-text fields (easier to parse later without escaping issues), CSV for simple flat tables. Not finalized.
- **Maintenance window:** legacy system export locks the DB. Priya proposed running the full export overnight Saturday (2026-10-03, 22:00–06:00 CET) to minimize business impact. Sahil to confirm this doesn't conflict with the weekend reporting jobs.
- **Destination storage:** current file server has 500GB allocated for this migration; Lars estimates the full `contact_notes` export alone could exceed 300GB once written to disk. Lars to size the actual requirement before the dry run and request more capacity if needed, rather than finding out mid-run.
- **Reconciliation / validation:** no existing tooling to verify exported files match the source database. Meera stated this is a hard blocker — "we can't sign off go-live on faith." Discussion on whether to build a lightweight row-count + checksum script in-house or evaluate a third-party data-diff tool; leaning toward in-house for the row-count pass, third-party only if free-tier options are unsuitable.
- **Rollback plan:** briefly discussed — if the full migration fails validation, is there a plan to revert or re-run cleanly without duplicating already-exported files? Not yet defined. Flagged as a gap, not resolved in this meeting.
- **Dry run scope:** agreed to migrate a random 5% sample of accounts (stratified by region, to catch the region_code mapping issue early) rather than the first N records, since early records may not be representative.

---

## 3. Action Items

| # | Action | Owner | Due Date |
|---|--------|-------|----------|
| 1 | Supply legacy region_code mapping/reference table | Priya Menon | 2026-09-25 |
| 2 | Confirm Saturday overnight export window doesn't conflict with weekend reporting jobs | Sahil Renapurkar | 2026-09-24 |
| 3 | Size actual storage requirement for `contact_notes` export; request file server capacity increase if needed | Lars Gutmann | 2026-09-26 |
| 4 | Get Business Ops input (via Chris Bauer) on whether historical HTML formatting in notes must be preserved | Naga Durga | 2026-09-26 |
| 5 | Draft fallback rule doc for unparseable `transaction_history` records (`needs_manual_review` tagging logic) | Naga Durga | 2026-09-29 |
| 6 | Decide file format (CSV vs. NDJSON) per table and document the decision | Lars Gutmann | 2026-09-29 |
| 7 | Scope and estimate build for row-count + checksum reconciliation script | Meera Iyer | 2026-09-30 |
| 8 | Draft rollback / clean re-run plan for a failed full migration (open gap, no owner yet — to be assigned next meeting) | *Unassigned* | Next review |
| 9 | Schedule dry-run review meeting after 5% sample migration completes | Naga Durga | Week of 2026-10-06 |

**Next meeting:** 2026-10-06, post dry-run review.
