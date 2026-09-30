# NanoSense Documentation

Public documentation for [NanoSense](https://nanosense.net) — the Medical RAG Intelligence Platform.

## Contents

### API & Integration

- [API Reference (v3.0)](docs/api-reference.md) — Complete endpoint reference with authentication, query modes, billing, FHIR R4, imaging, analytics, and admin.
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

What these docs describe is tracked against what is actually implemented in the [Architecture Gap Review](docs/architecture-gap-review.md) (review date 2026-09-30). Summary:

| Status | Capabilities |
|--------|--------------|
| **Supported** | HIPAA controls (audit logging, RLS tenant isolation, PHI redaction) · FHIR R4 (ingest, search, read) · Query modes (`fast`, `deep`, `rag_cag`, `mcp`, `radiology`, `auto`) · DICOM analysis · EHR integration (SMART on FHIR, HL7, CSV, CDS Hooks) |
| **In Progress** | SOC 2 Type II attestation — **In Progress, Q4 2026** (not yet attained) |
| **Partial** | Consent propagation (hub consent service per the [Consent & Sharing Contract](docs/consent-sharing-contract.md); Tamar→hub propagation in progress via TAM-001) · Portal workflows (clinician SPA is a Phase 1 read-only scaffold) · Lab analytics (data flows in; no lab-facing analytics surface yet) · Literature mining (PubMed-grounded Q&A only; no corpus/batch mining) |
| **Not Yet Implemented** | Family/caregiver sharing (**now in progress via TAM-002**) · Cross-institution trusted-provider sharing (MED-003, TAM-003) · Care coordination "Circle of Trust" (TAM-004) |

**Encryption:** data is protected with TLS 1.2+ in transit, AWS KMS AES-256 at rest, and Fernet field-level encryption for sensitive fields. This is **not** end-to-end encryption — client-held keys / zero-knowledge sharing are future work (see the [Security Whitepaper](hipaa/security_whitepaper.md#2-data-encryption)).

Per-claim evidence for every compliance/feature claim in this repo: [Claims Audit Worksheet](docs/claims-audit-worksheet.md). Status labels are governed by the gap review; the SOC 2 label ("In Progress, Q4 2026") must stay consistent across the marketing site (WEB-001).

## Interactive API Docs

Explore the API interactively at [api.nanosense.net/docs](https://api.nanosense.net/docs) (Swagger UI).

## Contact

- Website: [nanosense.net](https://nanosense.net)
- Email: hello@nanosense.net
