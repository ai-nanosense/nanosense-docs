# Claims Audit Worksheet (DOC-001)

**Ticket:** DOC-001 — Claims Audit Across Public Docs · **Repo:** nanosense-docs · **Audit date:** 2026-09-30 · **Owner:** docs-lead

**Scope:** every compliance/feature claim in `docs/` (`api-reference.md`, `sdk-reference.md`, `partner-integration.md`, `regional-compliance.md`, `testing-validation.md`, `csv-ingest.md`) and `hipaa/` (`security_whitepaper.md`, `data_retention_policy.md`, `incident_response_plan.md`, `baa_template.md`) plus `README.md`. `docs/architecture-gap-review.md`, `docs/consent-sharing-contract.md`, and `docs/tickets/` are reference/spec material — audited for consistency but not swept as claim surfaces.

**Baseline:** [Architecture Gap Review](architecture-gap-review.md) (§3 capability verdicts, §4 gap register G1–G7).

**Evidence anchors:**
- **G4 (SOC 2):** `/Users/a1234/workspaces/nanosense-workspace/med-intelligent/docs/tenant-isolation-model.md:112` — "SOC2 Type II | In progress. Anticipated completion: Q4 2026." Approved status label: **"SOC 2 Type II — In Progress, Q4 2026"** (reuse verbatim in WEB-001).
- **G5 (encryption):** TLS 1.2+ in transit (`/Users/a1234/workspaces/nanosense-workspace/nanosense-infra/infra/terraform/modules/rds_postgres/main.tf:13` — `rds.force_ssl`), KMS AES-256 at rest, Fernet field-level (`/Users/a1234/workspaces/nanosense-workspace/med-intelligent/phi_utils.py:219` — `FieldEncryptor`). No client-held keys, no zero-knowledge sharing exists today.

## Verdict key

| Verdict | Meaning |
|---|---|
| **Accurate** | Claim traces to implementation evidence (repo + file path). |
| **Accurate (caveat)** | Traces to evidence, but a known gap (G1–G7) qualifies how far the claim reaches. |
| **Overclaim** | Claim outruns implementation — fixed in this ticket (Fix column). |
| **Non-claim** | Matched a search term but asserts no capability (process/protocol language). |

## Claims by file

### README.md

| # | Claim (location) | Evidence | Verdict | Fix |
|---|---|---|---|---|
| 1 | "billing, FHIR R4, imaging, analytics, and admin" (api-reference blurb) | `med-intelligent/fhir_routes.py`, `med-intelligent/imaging/`, `med-intelligent/analytics_routes.py`, `med-intelligent/admin_routes.py` | Accurate | — |
| 2 | "FHIR integration" (partner-integration blurb) | `med-intelligent/fhir_adapter/`, `med-intelligent/connectors/fhir_connector.py` | Accurate | — |
| 3 | "encryption, access controls, tenant isolation, PHI protection, and HIPAA Security Rule coverage" (whitepaper blurb) | Whitepaper §2–§6, §13.2 mapping | Accurate | — |
| 4 | "SG PDPA and AU Privacy Act … data residency (ap-southeast-1 live, ap-southeast-2 planned)" | `docs/regional-compliance.md` §Data Residency; `nanosense-infra` regional stacks | Accurate | — |
| 5 | *(gap)* No conformance status; `consent-sharing-contract.md` absent from contents | Gap review §3; `docs/consent-sharing-contract.md` exists | Overclaim (omission) | Added "Architecture conformance status" section + contents entry |

### docs/api-reference.md

| # | Claim (location) | Evidence | Verdict | Fix |
|---|---|---|---|---|
| 6 | Query modes `fast`, `deep`, `rag_cag`, `mcp`, `radiology`, `auto` (Plans & Mode Access; Query Modes) | `med-intelligent/fastapi_medical_rag_backend.py:773`, `:1469-1480` | Accurate | — |
| 7 | `include_knowledge_graph` — "Include knowledge graph data" (line 257) | Field exists on `/query`; KG extraction currently degraded — gap review G3 (`KNOWLEDGE_GRAPH_AVAILABLE=False`, mock in production path) | Accurate (caveat) | No doc change — field exists; G3 tracked by MED-001 |
| 8 | FHIR R4 API: CapabilityStatement, Patient search/read/`$everything`, 9 clinical resource searches, `POST /fhir/Bundle` ingest (lines 722–797) | `med-intelligent/fhir_adapter/` (`resources.py`, `connector.py`), `med-intelligent/fhir_routes.py` | Accurate | — |
| 9 | `AllergyIntolerance` listed as searchable (Clinical Resources table, line 781) | `med-intelligent/fhir_routes.py:616-654` — US Core **stub**: search returns empty Bundle; `docs/csv-ingest.md` Known Limits confirms | Overclaim | Footnote added: stub read-back; allergies are ingest + RAG-queryable only |
| 10 | "patient ID (hashed before storage for HIPAA)" (line 589) | `med-intelligent/phi_utils.py` — HMAC-SHA256 `patient_id_hash`; whitepaper §6.1 | Accurate | — |
| 11 | DICOM ingest — "Extracts metadata and indexes the report text … No pixel data is stored" (line 924) | `med-intelligent/imaging/dicom_ingest.py`, `radiology_rag.py`; gap review §3.2 DICOM ✅ | Accurate | — |
| 12 | CDS Hooks / EHR integration (lines 958–976) | `med-intelligent/connectors/smart_on_fhir.py`, `hl7_ingest.py`, `epic_app_orchard/` | Accurate | — |
| 13 | `/health` example shows `"knowledge_graph": "available"` (line 1000) | Component field exists; live KG is degraded per G3 | Accurate (caveat) | No doc change (illustrative example); G3 tracked by MED-001 |
| 14 | "HIPAA-compliant audit log (per 164.312(b)) … Patient IDs are hashed with HMAC-SHA256" (line 1126) | `med-intelligent/hipaa_db_migration.py:70` (`audit_log`), whitepaper §5.1; gap review HIPAA ✅ | Accurate | — |
| 15 | "Fully supported (read + search + ingest): … AllergyIntolerance …" (line 1249) | `fhir_routes.py:639` stub list includes AllergyIntolerance; `csv-ingest.md` Known Limits | Overclaim | AllergyIntolerance moved to a stub caveat (ingest + RAG only) |

### docs/sdk-reference.md

| # | Claim (location) | Evidence | Verdict | Fix |
|---|---|---|---|---|
| 16 | Query modes incl. `radiology`, `auto`; mode access by plan | `fastapi_medical_rag_backend.py:773`; `docs/testing-validation.md` mode matrix | Accurate | — |
| 17 | `include_knowledge_graph` client parameter (line 254) | Field exists on `/query`; G3 caveat as #7 | Accurate (caveat) | No doc change |
| 18 | `imaging_search` / `imaging_ingest` — DICOM upload, semantic search (lines 302–350) | `med-intelligent/imaging/` | Accurate | — |
| 19 | "Supported resource types: … AllergyIntolerance …" for **FHIR ingest** (line 473) | Ingest path builds + persists AllergyIntolerance (`fhir_adapter/resources.py:373-385`); `csv-ingest.md`: "Allergies are persisted and RAG-queryable" | Accurate (caveat) | None — claim is ingest-scoped; read-back limit documented in csv-ingest.md |

### docs/partner-integration.md

| # | Claim (location) | Evidence | Verdict | Fix |
|---|---|---|---|---|
| 20 | BFF template behaviors (mode blocking, rate-limit forwarding, error sanitization) | BFF templates in `nanosense-sdk`; gap review repo inventory | Accurate | — |
| 21 | FHIR push `POST /fhir/Bundle`, post-ingest search (lines 302–375) | `fhir_routes.py`, `fhir_adapter/` | Accurate | — |
| 22 | "Supported Resource Types" table — `AllergyIntolerance` **Read: Yes, Search: Yes** (line ~367) | `fhir_routes.py:616-654` stub: read/search return empty Bundle | Overclaim | Row corrected to stub + footnote (ingest supported; read/search empty until the read route lands) |
| 23 | "Supported resource types … AllergyIntolerance. Other types return 422" (ingest pitfalls, line 445) | Ingest scope — same as #19 | Accurate | — |
| 24 | "FHIR read-back … Patient, Condition, MedicationRequest, Observation search" (line 478) | Consistent with `fhir_routes.py` stub list and csv-ingest Known Limits | Accurate | — |
| 25 | "HTTPS is enforced end-to-end on the BFF" (security checklist, line 517) | Not an encryption claim, but "end-to-end" collides with the G5 reserved term | Non-claim (sweep hygiene) | Reworded: "HTTPS is enforced on all BFF routes (no plain-HTTP path)" |
| 26 | PrivateLink private path — "traffic never traverses the public internet" (lines 526–547) | `nanosense-infra/infra/terraform/modules/private_link/`, `snippets/tamar-privatelink-endpoint/` | Accurate | — |
| 27 | Multi-partner webhooks, per-partner HMAC secrets (`PARTNER_WEBHOOK_SECRETS`) | `med-intelligent/telemedicine_webhook.py`; `tests/test_partner_webhook.py` | Accurate | — |

### docs/regional-compliance.md

| # | Claim (location) | Evidence | Verdict | Fix |
|---|---|---|---|---|
| 28 | "encryption in transit and at rest, tenant isolation, and audit logging" (line 14) | Whitepaper §2 (TLS 1.2+ / KMS AES-256), §4 (RLS), §5 (audit log) — matches the accurate G5 control set | Accurate | — |
| 29 | IR plan notification workflows "aligned to this timeline" (3 calendar days, line 15) | `hipaa/incident_response_plan.md` Phase 5.1 (72-hour CE notification) | Accurate | — |
| 30 | Data residency: `ap-southeast-1` live, `us-east-1` live, `ap-southeast-2` Terraform-provisioned/staged (lines 32–38) | `nanosense-infra` regional stacks; consistent with partner-integration PrivateLink region table | Accurate (deployment claim) | — |

### docs/testing-validation.md

| # | Claim (location) | Evidence | Verdict | Fix |
|---|---|---|---|---|
| 31 | Mode access matrix, rate limits, billing metering "Verified" | Matches `api-reference.md` Plans/Rate Limits; `tests/test_prod_subscriber_tiers.py` (39 tests) referenced | Accurate | — |
| 32 | "All supported FHIR R4 resource types ingest correctly" (8 types tested) | Scoped to the tested Synthea bundle; consistent with `fhir_adapter/resources.py` | Accurate | — |
| 33 | Tenant isolation — "RLS enforced at the PostgreSQL layer" | Whitepaper §4; gap review HIPAA ✅ (RLS on 9 tables) | Accurate | — |

### docs/csv-ingest.md

| # | Claim (location) | Evidence | Verdict | Fix |
|---|---|---|---|---|
| 34 | CSV → FHIR mapping, per-EMR profiles (`plato` TEMPLATE, `generic`) | `med-intelligent/connectors/csv/`; `api-reference.md` `/ingest/profiles` | Accurate (discloses template status) | — |
| 35 | Known Limits — 14 stub read-back types; "FHIR search on AllergyIntolerance returns empty until the read route lands" | `fhir_routes.py:616-654` stub list | Accurate | — (this is the disclosure api-reference.md lacked; see #9/#15) |
| 36 | "upload via the Clinician Portal at nanosense.net/upload/" (line 50) | Portal workflows are Partial per gap review G6 (Phase 1 read-only scaffold) | Accurate (caveat) | No doc change — website upload page is WEB-001 territory; README conformance section labels portal workflows Partial |

### hipaa/security_whitepaper.md

| # | Claim (location) | Evidence | Verdict | Fix |
|---|---|---|---|---|
| 37 | "HIPAA-ready" CDS platform (Executive Summary) | Carefully scoped ("ready", not "certified"); safeguards mapped in §13.2 | Accurate | — |
| 38 | Encryption: KMS AES-256 at rest, TLS 1.2+ in transit, Fernet field-level (§2) | `nanosense-infra/infra/terraform/modules/rds_postgres/main.tf:13` (`rds.force_ssl`), `med-intelligent/phi_utils.py:219` (`FieldEncryptor`); gap review §3.4 G5 — **canonical accurate description** | Accurate | — (alignment target for G5) |
| 39 | RLS tenant isolation — "Tenant A can never see … Tenant B's data" (§4) | Whitepaper §4 mechanism; `hipaa_db_migration.py`; gap review HIPAA ✅ | Accurate | — |
| 40 | 3-layer audit logging; `audit_log` schema; 10-year retention (§5) | `hipaa_db_migration.py:70`; `data_retention_policy.md` §3.2 | Accurate | — |
| 41 | PHI identifier hashing + LLM redaction of 18 Safe Harbor identifiers (§6.1–6.3) | `med-intelligent/phi_utils.py`; gap review "PHI log redaction" | Accurate | — |
| 42 | `consent_records` table with `is_active` computed column (§6.4) | `med-intelligent/hipaa_db_migration.py:105-152` — table + generated column exist | Accurate (caveat) | — Platform-wide consent is still Partial (G1): hub mirror/enforcement added by MED-002 (`consent_service.py`), Tamar→hub propagation pending (TAM-001) |
| 43 | "Facility Access Controls … Inherited from AWS (SOC 2, ISO 27001)" (§13.2, line 434) | AWS holds SOC 2 / ISO 27001 — but the bare "SOC 2" is misreadable as a NanoSense attestation (G4) | Overclaim (ambiguous) | Clarified: AWS's own attestations + "MedIntelligent SOC 2 Type II — In Progress, anticipated completion Q4 2026 (not yet attained)" |
| 44 | "MedIntelligent includes built-in compliance validation tools" — `generate_hipaa_gap_report.py`, `check_rds_security.py` (§13.1) | **No such files anywhere in the workspace** (verified by `find` across all repos; `med-intelligent/scripts/`, `hipaa/`, `tests/` checked) | Overclaim | Relabeled **Planned — not yet shipped**; SLA Alert Evaluator (`observability/alerts.py`) marked Available |
| 45 | "Risk Analysis … Implemented (gap assessment tool)" (§13.2) | Depends on the missing tool (#44) | Overclaim | Reworded: manual gap assessment; automated tooling Planned |
| 46 | Subprocessors — Stripe "No PHI"; LLM providers "redacted clinical context" (§12) | Consistent with §6.2 redaction; `billing/` sends token counts only | Accurate | — |

### hipaa/data_retention_policy.md

| # | Claim (location) | Evidence | Verdict | Fix |
|---|---|---|---|---|
| 47 | Retention periods (6 yr clinical, 10 yr audit, `consent_records` 6 yr) | `hipaa_db_migration.py:160-183` seeds `data_retention_policies` incl. `consent_records` | Accurate | — |
| 48 | Automated enforcement (`retention_enforcer.py`, env config §6) | `med-intelligent/retention_enforcer.py` exists; gap review cites it for HIPAA ✅ | Accurate | — |
| 49 | Related Documents: `generate_hipaa_gap_report.py`, `check_rds_security.py` | Missing from workspace (same as #44) | Overclaim | Rows annotated "(Planned — not yet shipped)" |

### hipaa/incident_response_plan.md

| # | Claim (location) | Evidence | Verdict | Fix |
|---|---|---|---|---|
| 50 | PHI encryption assessment table: AES-256 via RDS KMS, TLS 1.2+, Fernet (AES-128-CBC + HMAC) (Phase 3) | `phi_utils.py:219`, `rds_postgres/main.tf:13`, whitepaper §2 — matches the G5 control set exactly | Accurate | — |
| 51 | 72-hour CE security-incident / 60-day breach notification commitments (§1) | `hipaa/baa_template.md` breach-notification timelines | Accurate | — |
| 52 | "Dry-run the CE notification process end-to-end" (notification drill, line 320) | Process language, not an encryption claim | Non-claim (sweep hygiene) | Reworded: "Dry-run the full CE notification process (detection through notification)" |
| 53 | PIR checklist "Run `generate_hipaa_gap_report.py`" (line 229) + Related Documents rows (lines 373–374) | Missing tool (same as #44) | Overclaim | Annotated "(Planned — not yet shipped)" |

### hipaa/baa_template.md

| # | Claim (location) | Evidence | Verdict | Fix |
|---|---|---|---|---|
| 54 | §2.2(a) Encryption at Rest — AES-256, AWS RDS storage encryption, KMS-managed keys | Whitepaper §2.1; `nanosense-infra` RDS module | Accurate | — |
| 55 | §2.2(b) Encryption in Transit — TLS 1.2 or higher | Whitepaper §2.2; `rds.force_ssl` | Accurate | — |
| 56 | §2.2(c)–(d) RBAC + RLS isolation; audit logs retained ≥ 6 years | Whitepaper §3.4/§4/§5; `hipaa_db_migration.py` | Accurate | — |
| 57 | §4 On-premises VPC deployment option (CE-controlled KMS/IAM) | `med-intelligent/deploy/` artifacts; template carries legal-review disclaimer | Accurate (template term) | — |

## Fixes applied (per file)

| File | Text changes | What changed |
|---|---:|---|
| `README.md` | 2 | Added `consent-sharing-contract.md` to contents (Security & Compliance); added "Architecture conformance status" section linking `docs/architecture-gap-review.md` |
| `docs/api-reference.md` | 2 | AllergyIntolerance stub footnote (Clinical Resources); corrected "Fully supported (read + search + ingest)" list (#15) |
| `docs/sdk-reference.md` | 0 | — (ingest-scoped claim at line 473 is accurate) |
| `docs/partner-integration.md` | 2 | AllergyIntolerance row corrected to stub + footnote (#22); "HTTPS is enforced end-to-end" → "HTTPS is enforced on all BFF routes" (#25) |
| `docs/regional-compliance.md` | 0 | — |
| `docs/testing-validation.md` | 0 | — |
| `docs/csv-ingest.md` | 0 | — |
| `hipaa/security_whitepaper.md` | 3 | §13.2 AWS SOC 2 attribution + "SOC 2 Type II — In Progress, Q4 2026" status (#43); §13.1 tools relabeled Planned/Available (#44); §13.2 Risk Analysis row (#45) |
| `hipaa/data_retention_policy.md` | 1 | Related Documents rows annotated "(Planned — not yet shipped)" (#49) |
| `hipaa/incident_response_plan.md` | 3 | Notification-drill wording (#52); PIR checklist + Related Documents annotations (#53) |
| `hipaa/baa_template.md` | 0 | — |
| `docs/claims-audit-worksheet.md` | new | This audit worksheet (ticket acceptance: "audit worksheet committed under `docs/`") |

## Reproducible grep sweep

Run from the repo root after fixes (acceptance: zero unqualified SOC 2 / E2E claims):

```bash
grep -rn -i -E 'SOC ?2|end-to-end|E2E' README.md docs/*.md hipaa/*.md
grep -rn -i -E 'encrypt' README.md docs/*.md hipaa/*.md
grep -rn -i -E 'consent|knowledge graph|care circle|Circle of Trust|family|caregiver' README.md docs/*.md hipaa/*.md
grep -rn -i -E 'compliant|certif|attest|HIPAA|FHIR' README.md docs/*.md hipaa/*.md
```

Post-fix expectation: every remaining `SOC 2` / `end-to-end` hit is either (a) inside `docs/architecture-gap-review.md` or `docs/adr-001-canonical-backend-repo.md`, where the overclaims are quoted as findings/decisions, or (b) explicitly qualified ("AWS's own SOC 2 …", "SOC 2 Type II — In Progress …", "not end-to-end encryption").

## Evidence spot-checks (acceptance)

- **HIPAA (3+):** `med-intelligent/hipaa_db_migration.py:70` (`audit_log` DDL) ↔ whitepaper §5.1 · `med-intelligent/retention_enforcer.py` ↔ `data_retention_policy.md` §6 · `med-intelligent/phi_utils.py` (log/identifier redaction) ↔ whitepaper §6.1–6.3 · gap review HIPAA ✅ (RLS on 9 tables).
- **FHIR R4 (3+):** `med-intelligent/fhir_adapter/resources.py` + `connector.py` ↔ `api-reference.md` §FHIR R4 API · `med-intelligent/fhir_routes.py` (`/fhir/Bundle`, `/fhir/metadata`, `/fhir/Patient/{id}/$everything`) ↔ `api-reference.md` lines 722–797 · `med-intelligent/connectors/smart_on_fhir.py` + `hl7_ingest.py` ↔ CDS Hooks / EHR integration claims.
- **Encryption (3+):** `nanosense-infra/infra/terraform/modules/rds_postgres/main.tf:13` (`rds.force_ssl = 1`) ↔ whitepaper §2.2 · KMS AES-256 RDS/S3/Secrets ↔ whitepaper §2.1 + `baa_template.md` §2.2(a) · `med-intelligent/phi_utils.py:219` `FieldEncryptor` (Fernet AES-128-CBC + HMAC-SHA256) ↔ whitepaper §2.3 + `incident_response_plan.md` Phase 3 table.

## Newly found mismatches — for docs-lead gap-review update

Per handoff notes these are reported, not written into `architecture-gap-review.md` directly:

1. **F2 — FHIR AllergyIntolerance read/search overclaim (docs, G-adjacent):** `api-reference.md`, `partner-integration.md` presented AllergyIntolerance as fully readable/searchable while `med-intelligent/fhir_routes.py:616-654` stubs it (empty Bundles). Fixed in this ticket; a durable "FHIR read-back stubs" row in the gap register would help future doc work. Candidate owner: docs-lead (gap register) + med-intelligent (read route).
2. **F3 — Unverifiable compliance tooling:** `generate_hipaa_gap_report.py` and `check_rds_security.py` are referenced across `hipaa/` docs but exist in no repo. Labeled "Planned — not yet shipped" here; if the scripts live outside version control, restore the claims with a path, or track a ticket to ship the tools.