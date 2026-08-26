# CSV Ingest Guide (EMR Export)

**Base URL:** `https://api.nanosense.net` (production) | `http://localhost:8000` (local)

Import patient data from any EMR that can export CSV. The CSV ingest pipeline transforms your export into FHIR resources and upserts them into your tenant — the same store used by the FHIR push path and by RAG queries.

---

## CSV or FHIR Push?

Pick the data path by what your EMR supports:

| Your EMR | Use |
|----------|-----|
| Has a FHIR API (SMART-on-FHIR, FHIR bulk export) | [FHIR push](partner-integration.md#fhir-data-integration) (`POST /fhir/Bundle`) |
| CSV export only (e.g. Plato Medical and most clinic-management systems) | CSV ingest (this guide) |

CSV ingest is EMR-agnostic: each EMR has a *profile* that declares how its column names map to FHIR resources. You upload the same CSV your EMR already exports; the platform does the mapping.

---

## Step 1: Pick Your Profile

```bash
curl https://api.nanosense.net/ingest/profiles
```

```json
{
  "profiles": [
    {
      "name": "generic",
      "description": "Generic fallback — assumes FHIR-ish column names. Adjust per EMR.",
      "resources": ["AllergyIntolerance", "Condition", "Encounter", "Immunization", "MedicationRequest", "Observation", "Patient", "Procedure"]
    },
    {
      "name": "plato",
      "description": "Plato Medical CSV profile — STUB, awaiting Phase 0 sample",
      "resources": ["AllergyIntolerance", "Condition", "MedicationRequest", "Patient"]
    }
  ]
}
```

| Profile | For |
|---------|-----|
| `plato` | Plato Medical clinics |
| `generic` | Any other EMR (conventional `patient_id`, `code`, `display`, date columns) |

> **Note:** the `plato` profile is in preview while its column mappings are finalized against real pilot exports. Until then, Plato clinics should use `profile=generic` with conventional column names, or contact us to have your export format added.

This endpoint requires no authentication.

## Step 2: Export Your Data

Export one CSV per clinical domain. Recommended files:

| File | Required columns | Notes |
|------|------------------|-------|
| `patients.csv` | `patient_id` | Plus birth date, sex if available |
| `conditions.csv` | `patient_id`, `condition_id` | Diagnosis list with codes/displays |
| `medications.csv` | `patient_id`, `med_request_id` | Active + historical prescriptions |
| `observations.csv` | `patient_id`, `observation_id` | Vitals, labs (LOINC code preferred) |
| `allergies.csv` | `patient_id`, `allergy_id` | Critical for drug-interaction safety |
| `encounters.csv`, `procedures.csv`, `immunizations.csv` | `patient_id` + domain ID | Optional but recommended |

Rules:

- **One file per domain** — each upload handles one resource type.
- **≤ 10 MB per file** — larger exports must be split (413 otherwise).
- **UTF-8 preferred.** Latin-1 (common in older SEA exports) is handled automatically; a UTF-8 BOM is stripped automatically.
- Dates in `DD/MM/YYYY`, `MM/DD/YYYY`, or ISO `YYYY-MM-DD` are normalized automatically.
- Sex/gender accepts `M/F/X` (and common variants); numeric values are parsed automatically.
- Every row needs the `patient_id` column — rows missing it are reported as errors, not silently dropped.

## Step 3: Register and Log In

Skip this if you already have an API key or JWT.

```bash
curl -X POST https://api.nanosense.net/tenants/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Raffles Family Clinic",
    "admin_email": "admin@rafflesfc.sg",
    "admin_username": "raffles_admin",
    "admin_password": "<strong-password>",
    "plan": "professional"
  }'
```

Response `201`:

```json
{
  "tenant_id": "tnt_7f3a...",
  "api_key": { "api_key": "mrag_9NaZS71x..." }
}
```

> **The plaintext API key is shown once.** Store it securely.

For a JWT (expires in 30 minutes):

```bash
curl -X POST https://api.nanosense.net/tenants/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "raffles_admin", "password": "<strong-password>"}'
```

Both work for ingest: `Authorization: Bearer <jwt>` or `X-API-Key: mrag_...`.

## Step 4: Upload

```bash
curl -X POST https://api.nanosense.net/ingest/csv \
  -H "Authorization: Bearer <jwt>" \
  -F "file=@patients.csv" \
  -F "profile=plato"
```

- `file` — the CSV (multipart).
- `profile` — from Step 1. Defaults to `generic`.
- `resource_type` — optional. Omit it: the resource type is auto-detected from the profile's expected filename/columns. Set it only when auto-detect is ambiguous.

Repeat per domain file, e.g. `conditions.csv`, `medications.csv`, `allergies.csv`.

### Response

```json
{
  "profile": "plato",
  "resource_type": null,
  "rows_seen": 512,
  "resources_built": 510,
  "fhir_results": [
    { "status": "201 Created", "resourceType": "Patient" },
    { "status": "200 OK", "resourceType": "Patient" }
  ],
  "errors": [
    { "row": 47, "resource_type": "Patient", "field": "patient_id", "detail": "required field missing" },
    { "row": 213, "resource_type": "Patient", "field": "birth_date", "detail": "unparseable date: '3rd March'" }
  ],
  "ok": false
}
```

## Step 5: Verify

List the patients you just ingested:

```bash
curl "https://api.nanosense.net/patients?limit=5" \
  -H "Authorization: Bearer <jwt>"
```

Optional: search by ID substring with `?q=<patient_id>`; page with `limit`/`offset`.

FHIR search also works for core types:

```bash
curl "https://api.nanosense.net/fhir/Patient?_count=5" \
  -H "Authorization: Bearer <jwt>"
```

(FHIR read-back is subject to the stub-type limits below — some resource types return empty search results.)

And via RAG queries against the same data:

```bash
curl -X POST https://api.nanosense.net/query \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -d '{"question": "What are this patient'\''s active conditions?", "mode": "fast", "patient_id": "<patient_id>"}'
```

Mode availability depends on plan (`fast` on Starter; `deep` Professional+; `mcp` Enterprise). See the [API Reference](api-reference.md) for details.

---

## Reading the Ingest Report

The response is a per-upload report you can act on directly:

| Field | Meaning |
|-------|---------|
| `rows_seen` | Data rows read from the CSV (header excluded) |
| `resources_built` | Rows successfully transformed into FHIR resources |
| `fhir_results` | Per-resource DB upsert status (`201 Created` = new, `200 OK` = updated) |
| `errors` | Row-level failures: `{row, resource_type, field, detail}` |
| `ok` | `true` only when every row ingested cleanly |

**Re-upload semantics:** fix the failing rows in your source export and re-upload the whole file. Upserts are idempotent — rows that already succeeded are updated in place, not duplicated, as long as their IDs are unchanged.

If nothing could be transformed at all (wrong profile, missing required columns), you get `resources_built: 0`, `ok: false`, and the errors explain why. Re-check the profile choice before retrying.

## Known Limits

- **Stub read-back types:** FHIR search returns empty results for 14 stub resource types even after successful ingest. Patient, Condition, MedicationRequest, Observation, Procedure, Immunization, and Encounter read back normally. **Allergies are persisted and RAG-queryable, but FHIR search on AllergyIntolerance returns empty until the read route lands.**
- **RAG queries are unaffected** — they are SQL-backed and see all ingested data regardless of FHIR read-back support.
- One file per resource type; no cross-domain CSV in a single upload.
- Uploads require a tenant-scoped token — personal (non-tenant) tokens get 403.

## Common Issues

### 413 Request Entity Too Large
Your CSV exceeds 10 MB. Split the export (by date range or patient subset) and upload sequentially.

### 422 Unknown profile
Profile name typo or unsupported EMR. Run `GET /ingest/profiles` and use an exact name.

### Everything fails with "required field missing"
Wrong profile for your EMR — the column names don't match. Try `profile=generic` with conventional column names, or contact us to add a profile for your EMR.
