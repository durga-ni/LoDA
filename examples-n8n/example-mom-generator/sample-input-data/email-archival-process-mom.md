# Minutes of Meeting — Email Archival Process

**Meeting:** Corporate Mailbox Archival & Retention Policy — Kickoff
**Date:** 2026-09-24
**Time:** 11:00–11:45 CET
**Location:** Microsoft Teams
**Attendees:** Sahil Renapurkar (Product Owner), Naga Durga (Engineering), Priya Menon (Compliance), Lars Gutmann (Infrastructure), Meera Iyer (QA Lead)

---

## 1. Meeting Summary

The purpose of this kickoff was to define scope and ownership for archiving inactive employee mailboxes to comply with the updated 7-year data retention policy issued by Legal in August. Priya presented the compliance requirement: mailboxes belonging to employees who left the company must be archived (not deleted) and remain searchable for e-discovery requests for 7 years from the employee's last working day. Lars explained the current mail infrastructure (Google Workspace) supports export via the Vault API, but the team has never built automated tooling around it — archival has so far been done manually and inconsistently, which is itself a compliance risk. The group agreed to build an automated pipeline: trigger on offboarding (HR system event), export mailbox to cold storage, index metadata for searchability, and confirm deletion from the live tenant only after archival is verified. Sahil raised a scope question — whether shared/departmental mailboxes (not tied to a single departing employee) fall under the same policy; Priya to confirm with Legal. Meera flagged that "verified" archival needs a concrete, testable definition, not just "the export ran successfully," or QA can't sign off. The meeting ended with agreement to reconvene once the Legal scope question is answered and a rough technical design is drafted.

---

## 2. Discussion Points

- **Retention requirement:** 7 years from last working day, per Legal's August policy update. Applies to at minimum: sent/received mail, calendar invites, and attachments. Priya to confirm whether Drive files linked in emails are in scope or a separate policy.
- **Trigger point:** should archival kick off automatically when HR marks an employee as offboarded, or manually by IT after a review step? Team leaned toward automatic trigger with a 48-hour delay window, so IT can cancel/flag exceptions (e.g., active litigation hold) before the export runs.
- **Shared/departmental mailboxes:** e.g. `support@`, `sales-emea@` — these don't map to one departing employee and the current policy language doesn't address them. Open question, needs Legal clarification before design can finalize.
- **Storage target:** Lars proposed cold storage (object storage, infrequent-access tier) rather than keeping archived mail in the live Workspace tenant, both for cost and to reduce the live tenant's data footprint for e-discovery search performance.
- **Searchability requirement:** archived mail must remain searchable for legal e-discovery requests, not just "restorable." This means metadata (sender, recipient, date, subject, at minimum) must be indexed at archive time, not only after a restore. Raised as a hard requirement, not a nice-to-have.
- **Definition of "verified archived":** Meera pushed back on treating export success as sufficient proof. Proposed a verification pass: post-export, sample-check that N random messages are retrievable and match source checksums, before source mailbox is queued for deletion. Group agreed this is necessary but didn't finalize N or the checksum method in this meeting.
- **Deletion timing:** live mailbox should only be deleted from the tenant after archival is verified AND after a legal-hold check confirms no active hold applies to that employee. Priya to provide the legal-hold check process/API if one exists.
- **Audit trail:** every archival action (trigger, export, verify, delete) must be logged with timestamps and an actor (system or person), for later audit. Not just a "nice to have" per Priya — this is likely to be requested by Legal during an actual e-discovery event.
- **Rollout order:** given the shared-mailbox question is unresolved, team agreed to design and pilot the pipeline against individual employee mailboxes first, and treat shared mailboxes as a phase 2 once Legal responds.

---

## 3. Action Items

| # | Action | Owner | Due Date |
|---|--------|-------|----------|
| 1 | Confirm with Legal whether shared/departmental mailboxes fall under the same 7-year retention policy | Priya Menon | 2026-09-29 |
| 2 | Confirm whether Drive files linked in archived emails are in scope of this policy or a separate one | Priya Menon | 2026-09-29 |
| 3 | Provide the legal-hold check process/API so deletion can verify no active hold before proceeding | Priya Menon | 2026-10-01 |
| 4 | Draft technical design for the archival pipeline (HR trigger → 48h delay window → export → verify → delete) | Lars Gutmann | 2026-10-02 |
| 5 | Propose concrete "verified archived" definition (sample size N, checksum method) for QA sign-off | Meera Iyer | 2026-10-02 |
| 6 | Investigate Google Vault API export capabilities and rate limits for bulk offboarding scenarios | Lars Gutmann | 2026-10-02 |
| 7 | Define audit-log schema for archival actions (trigger, export, verify, delete — timestamp + actor) | Naga Durga | 2026-10-03 |
| 8 | Scope phase 1 (individual mailboxes only) vs. phase 2 (shared mailboxes) in the project plan | Sahil Renapurkar | 2026-10-03 |

**Next meeting:** 2026-10-06, technical design review (pending Legal's response on scope questions).
