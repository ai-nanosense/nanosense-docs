# Tamar–NanoSense patient authorization v1

Implementation record: September 30, 2026. Local changes require coordinated deployment and synthetic staging verification. The canonical deployed RAG source is `ai-nanosense/med-intelligent`; `nanosense-core` builds that repository.

## Signed request contract

This supersedes the five-claim example in `consent-sharing-contract.md` §7.2. Every patient-scoped query, document ingestion, FHIR Bundle ingestion, and re-index request carries `X-Consent-Proof`. An API key authenticates the service; it cannot authorize patient records.

JWT uses HS256 with a dedicated `CONSENT_PROOF_SECRET` shared by Tamar and RAG. Never reuse the application JWT secret. Required claims:

| Claim | Binding |
| --- | --- |
| `v`, `iss`, `aud` | Integer `1`, `tamar`, `nanosense-rag` |
| `tenant_id` | Tamar tenant owning the patient |
| `rag_tenant_id` | Mapped RAG tenant; must match authenticated tenant |
| `patient_id` | FHIR patient reference, otherwise legacy HMAC reference |
| `recipient_id` | Opaque actor principal; distinct from the patient's cross-system reference |
| `recipient_role`, `recipient_tenant_id` | Actor role and tenant; tenant may be null only for explicitly granted recipients |
| `operation` | `query`, `ingest`, `fhir_ingest`, or `reindex`; must match the route |
| `purpose` | `healthcare_service_delivery` |
| `scopes` | Nonempty subset of `summary`, `treatment_history`, `lab_results`, `ai_insights` |
| `consent_version` | Active patient clinical receipt SHA-256 fingerprint |
| `grant_version` | Active recipient grant fingerprint, or null for self/authorized direct care |
| `iat`, `exp` | Integer epoch seconds; lifetime 1–300 seconds; no future issue time |
| `jti` | Request trace identifier |

RAG checks the current consent mirror on every patient request, before retrieving cached answers or writing records. It independently requires active patient clinical consent and, where applicable, an active recipient grant. Neither a sharing grant nor optional `ai_model_training` consent substitutes for clinical processing consent. Revoked, expired, stale-version, mismatched or over-scoped requests are denied.

Patient-self receipts identify the opaque patient principal in `granted_to.principal_id` with role `patient`; this principal need not equal the FHIR/HMAC reference. Same-tenant direct care additionally requires Tamar's analytics permission. Family/caregiver membership and external providers require versioned grants.

## Receipt fingerprint

Project exactly `receipt_id`, `patient_id`, `purposes`, `scopes`, `granted_to`, `granted_at`, `expires_at`, `channel`. Sort purpose/scope arrays; normalize timestamps to UTC ISO 8601 with `+00:00`; preserve null expiry. Hash UTF-8 JSON using sorted keys, compact separators and ASCII escaping. Withdrawal does not change that projection; the mirror separately checks terminal revocation state. New grants/updates use new immutable receipts.

Authorize every resource/document in a batch before any clinical write. Mixed patients or unsupported resource classes fail closed. Returned/indexed data must be limited to the approved classes, and every data query must bind both tenant and patient. Reserved `imaging` and `research` scopes release nothing. Query engines without record filtering must be denied until filtering is implemented and tested.

## Revocation, deletion and failure behavior

Tamar commits consent changes with outbox events, sends signed webhook envelopes, and retries failures to dead-letter delivery. Disabled or missing sync configuration must raise a retryable failure rather than acknowledge undelivered events. A successful withdrawal requires the mirror acknowledgement; failures remain visible while delivery retries. Purge both legacy HMAC and FHIR references. Service-authenticated deletion remains available after withdrawal.

Authorization denials must never trigger clinical fallback. Valid unavailable-service fallbacks identify `source`, `degraded=true`, and `degraded_reason=rag_unavailable`; unsafe fallbacks fail explicitly. Operational logs carry only operation, outcome class and trace ID. No patient identifiers/hashes, prompts, tokens or upstream bodies.

## Deployment evidence and gates

Read-only inspection on September 30, 2026, in account `232170401402` found Tamar production sync unset (default false), no dedicated proof/webhook secret bindings, and no corresponding RAG ECS proof/partner webhook secret bindings. Do not enable production enforcement or sync before provisioning those secrets and validating both deployed versions. Lambda now supports individual `CONSENT_PROOF_SECRET_ARN` and `CONSENT_WEBHOOK_SECRET_ARN` resolution; ECS requires explicit secret injection.

The same inspection found no custom RAG consumer endpoint in `ap-southeast-1` and no provider endpoint service in `us-east-1`. PrivateLink remains unverified. The owner must supply provider service name/ID, account, region and connection state, plus consumer endpoint ID, accepted connection, DNS resolution and TLS evidence. Historical or other-account deployments remain possible.

## Synthetic CI and staging handoff

Run Tamar's proof/client/consent/analytics tests and RAG's proof/policy/ingest/scope tests. `Tamar-telemedicine/.github/workflows/rag-wire-contract.yml` checks out both revisions and runs the non-PHI wire test. It requires a read-only `NANOSENSE_CONTRACT_READ_TOKEN` for the RAG repository; set the RAG revision explicitly. This workflow verifies wire compatibility, not deployed connectivity.

Before rollout, seed a dedicated synthetic tenant, patient, same-tenant authorized provider and external provider in staging. Grant patient clinical consent, sync it, and grant external access only to chosen scopes. Through Tamar's authenticated API, execute query, document/FHIR ingestion and re-index. Verify RAG decisions and Tamar audit events share trace IDs.

Exercise missing/malformed/expired/wrong-audience/tenant/patient/purpose/operation/stale-version/over-scoped proofs. Confirm API-key-only denial and denied ingestion writes nothing. Withdraw purpose consent with an active sharing grant; revoke/update/expire grants; verify stale proofs and cached responses cannot bypass denial, both patient references purge, and authenticated deletion still succeeds. Simulate sync delivery failures through retry and dead-letter recovery. Simulate timeout/503, inspect fallback labels or typed errors, and assert privacy-safe logs.

No production patient is a test fixture. Staging E2E has not been executed from this checkout; endpoint, credentials and seeded fixture details remain deployment prerequisites.
