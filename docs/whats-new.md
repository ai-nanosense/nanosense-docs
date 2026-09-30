# What's New — October 2026

User-facing highlights for this release. Everything below describes shipped behavior in the Medical RAG API and the Tamartaw app. Compliance status labels are unchanged — see [Architecture conformance status](../README.md#architecture-conformance-status) in the README.

## Patient-controlled consent

Patients control exactly which data classes are shared and with whom. Grants are scoped to six data classes — `summary`, `treatment_history`, `lab_results`, `ai_insights`, `imaging`, and `research` — and are recorded as immutable consent receipts with a full audit trail. Revocation is **same-session**: once a grant is revoked, subsequent patient-scoped requests are refused immediately — even while the current login or access token is still valid — and cached answers for the patient are purged.

## External provider sharing

Share selected records with a provider outside your institution through a **scoped, expiring, single-use invite link**. Each link carries only the granted scopes, expires after at most 72 hours, and can be redeemed exactly once. Redemption creates a scoped grant for the receiving provider and records the disclosure as FHIR R4 `Consent` + `Provenance` resources.

## Care circle & Circle of Trust

Family members and caregivers can be added to a patient's care circle, each with their own scopes — every member sees only what the patient granted them. The circle shares a care timeline, and care tasks and referrals can be created, assigned, and tracked between members in the Tamartaw app.

## Knowledge Graph insights are genuinely operational

Knowledge-graph insights now reflect live extraction over ingested records. A wiring defect in September caused the insight path to fall back to mock data; that defect is fixed, and the mock fallback no longer shadows production results.

## Consent enforcement on queries and ingest

Every patient-scoped query and ingest is checked against the patient's active grants before anything runs. Without a valid covering grant the request is refused with `403` and an `X-Consent-Reason` header naming the failed check (for example `no_active_grant`, `revoked`, `expired`, or `scope_mismatch`). Denials never fall back to clinical data.

## Learn more

- [API Reference — Consent & Data Sharing](api-reference.md#consent--data-sharing) — endpoints, invite links, `X-Consent-Proof`, and denial semantics
- [Consent & Sharing Contract (v1)](consent-sharing-contract.md) — the authoritative wire-level and semantic contract

*These capabilities landed in the internal repositories `ai-nanosense/med-intelligent` and `ai-nanosense/Tamar-telemedicine`; this page is the user-facing summary.*
