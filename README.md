# NanoSense Documentation

Public documentation for [NanoSense](https://nanosense.net) — the Medical RAG Intelligence Platform.

## Contents

### Highlights

- [What's New — October 2026](docs/whats-new.md) — patient-controlled consent, external provider sharing, care circle & Circle of Trust, operational Knowledge Graph insights, lab analytics, literature mining, improved data protection, and consent enforcement.

### API & Integration

- [API Reference (v3.2)](docs/api-reference.md) — Complete endpoint reference with authentication, query modes, billing, FHIR R4, imaging, analytics, admin, consent & data sharing, Knowledge Graph insights, lab analytics, and literature mining.
- [SDK Reference](docs/sdk-reference.md) — Python and TypeScript SDK documentation with installation, authentication, query examples, error handling, and async usage.
- [Partner Integration Guide](docs/partner-integration.md) — BFF templates (Express.js, FastAPI, React hook), webhook coupling, FHIR integration, and security checklist.
- [Platform Validation Report](docs/testing-validation.md) — Production test results for all subscriber tiers: mode access, rate limits, billing, FHIR ingest, clinical accuracy.
- [CSV Ingest Guide](docs/csv-ingest.md) — Import EMR CSV exports (Plato Medical + generic profiles): profile selection, export guidance, register → login → upload → verify walkthrough, error reports, known limits.

### Security & Compliance

- [Consent & Sharing Contract (v1)](docs/consent-sharing-contract.md) — Authoritative wire-level and semantic contract for patient consent and data-sharing grants across the platform (implementation target of MED-002, TAM-001, TAM-002).
- [Regional Privacy & Compliance Brief](docs/regional-compliance.md) — SG PDPA and AU Privacy Act obligations summary plus data residency (ap-southeast-1 live, ap-southeast-2 planned). Informational only, not legal advice.
- [Security Whitepaper](hipaa/security_whitepaper.md) — Architecture, encryption, access controls, tenant isolation, PHI protection, and HIPAA Security Rule coverage.
- [Data Retention Policy](hipaa/data_retention_policy.md) — Retention periods, disposal methods, automated enforcement, and tenant termination procedures.
- [Incident Response Plan](hipaa/incident_response_plan.md) — 7-phase incident response with severity classification and breach notification timelines.

### Legal

- [Business Associate Agreement](hipaa/baa_template.md) — Standard BAA template covering permitted uses, safeguards, and breach notification.

## Architecture conformance status

What these docs describe is tracked against what is actually implemented in the architecture gap review (review date 2026-09-30), tracked internally in ai-nanosense/med-intelligent `docs/architecture-gap-review.md`. Summary:

| Status | Capabilities |
|--------|--------------|
| **Supported** | HIPAA controls (audit logging, RLS tenant isolation, PHI redaction) · FHIR R4 (ingest, search, read) · Query modes (`fast`, `deep`, `rag_cag`, `mcp`, `radiology`, `auto`) · DICOM analysis · EHR integration (SMART on FHIR, HL7, CSV, CDS Hooks) · Consent propagation (patient-controlled grants enforced on `/query` + ingest; Tamar↔hub sync via signed `consent.*` webhooks per the [Consent & Sharing Contract](docs/consent-sharing-contract.md)) · Family/caregiver sharing (care-circle members, each with their own scopes) · Cross-institution sharing (scoped, expiring single-use invite links; FHIR R4 Consent + Provenance on redemption) · Care coordination ("Circle of Trust": shared care timeline with task and referral workflow) · Knowledge Graph insights (queryable API — patient entities, relations, trends, and a graph payload) · Lab analytics (trends, `low`/`normal`/`high` flags, per-analyte summaries, de-identified population aggregates) · Literature mining (batch PubMed search, saved corpora, evidence tables with strict citations) |
| **In Progress** | SOC 2 Type II attestation — **In Progress, Q4 2026** (not yet attained) |
| **Partial** | Portal workflows (clinician SPA is a Phase 1 read-only scaffold) |
| **Not Yet Implemented** | End-to-end encryption with client-held keys (zero-knowledge sharing) — future work (see the Encryption note below) |

**Encryption:** data is protected with TLS 1.2+ in transit, AWS KMS AES-256 at rest, Fernet field-level encryption for sensitive fields, and per-patient **envelope encryption** (each patient's data key wrapped by a dedicated AWS KMS key) for sensitive fields and shared documents. This is **not** end-to-end encryption — keys are held by the platform under KMS, not by clients; there is no client-held key material and no zero-knowledge sharing today. Client-held keys / zero-knowledge sharing are future work (see the [Security Whitepaper](hipaa/security_whitepaper.md#2-data-encryption)).

Per-claim evidence for every compliance/feature claim in this repo: [Claims Audit Worksheet](docs/claims-audit-worksheet.md). Status labels are governed by the architecture gap review, tracked internally in ai-nanosense/med-intelligent `docs/architecture-gap-review.md`; the SOC 2 label ("In Progress, Q4 2026") must stay consistent across the marketing site (WEB-001).

## Interactive API Docs

Explore the API interactively at [api.nanosense.net/docs](https://api.nanosense.net/docs) (Swagger UI).

## Contact

- Website: [nanosense.net](https://nanosense.net)
- Email: hello@nanosense.net
