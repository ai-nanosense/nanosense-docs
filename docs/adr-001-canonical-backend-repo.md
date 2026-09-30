# ADR-001: Canonical Backend Repository — `med-intelligent`

- **Status:** Accepted (decision); retirement half **blocked on owner sign-off**
- **Date:** 2026-09-24
- **Ticket:** [CORE-001](tickets/nanosense-core/CORE-001-source-of-truth.md) (closes architecture gap review G7)
- **Scope of this ADR:** decision + safe textual reference updates only. No repository is archived or deleted by this ticket; no remote git state is changed.

---

## 1. Context

The workspace contains two near-duplicate FastAPI backends: `nanosense-core` and `med-intelligent`. Both claim to hold the FastAPI-side marketplace routes, and cross-repo docs/CI disagree about which repo is "the core API".

### 1.1 Evidence — recency

| Repo | Last commit | Tip |
|---|---|---|
| `nanosense-core` | **2026-08-26** | `9ddf888` "fix(ci): drop gha cache options (docker driver has no cache export)" |
| `med-intelligent` | **2026-09-23** | `3f2644d` "fix(test): stage deploy_marketplace_stack.sh deletion; skip cross-repo checks in CI" |

`nanosense-core` is ~4 weeks stale relative to `med-intelligent`.

### 1.2 Evidence — feature diff (verified against local checkouts, 2026-09-24)

`nanosense-core` **lacks** artifacts that exist in `med-intelligent`:

| Artifact | nanosense-core | med-intelligent |
|---|:---:|:---:|
| `knowledge_graph/` (KG insights, gap G3) | ❌ | ✅ |
| `patient_routes.py` | ❌ | ✅ |
| `query_cache.py` | ❌ | ✅ |
| `integrated_medical_rag.py` | ❌ | ✅ |
| `consent_routes.py` / `consent_service.py` / `consent_db_migration.py` (gap G1 work) | ❌ | ✅ |
| `ingest_routes.py`, `lambda_handler.py`, `lambda_offload.py`, `mcp_integration.py`, `medical_rag_cag_system.py`, `aws_medical_rag_app.py`, `sdk/` | ❌ | ✅ |

> **Note:** the CORE-001 ticket text lists `imaging_db_migration.py` among the missing files; on inspection it exists in **both** repos. The four headline gaps (`knowledge_graph/`, `patient_routes.py`, `query_cache.py`, `integrated_medical_rag.py`) are confirmed; the table above is the verified version.

### 1.3 Evidence — conflicting "source of truth" claims

- Both repos carry a `marketplace/` package with the FastAPI-side routes (`marketplace_routes.py`, `aws_metering.py`, `middleware.py`, `tenant_isolation.py`). `med-intelligent/marketplace/RETIRED-lambdas.md` (2026-09-22) already establishes that the AWS-side Lambda copies were retired in favor of the canonical `nanosense-marketplace` repo — i.e. `med-intelligent` is being treated as the hub for the FastAPI side.
- `nanosense-marketplace/README.md` (pre-fix) said the FastAPI-side marketplace routes "remain in the `nanosense-core` repo".
- `nanosense-infra/docs/cost-review-2026-08-use1-marketplace.md:23-24` labels `nanosense-core` the "active codebase" and `med-intelligent` "legacy" — **inverted** relative to commit reality.
- Production CI already builds the live ECS image from **med-intelligent** source: `nanosense-core/.github/workflows/deploy.yml:15-16` — *"this repo's fork is stale/diverged (unmergeable), so the marketplace image builds from ai-nanosense/med-intelligent instead"* (`BACKEND_REPO: ai-nanosense/med-intelligent`, image `nanosense-prod/rag-backend`).
- `nanosense-infra` Terraform/OIDC config (`infra/terraform/envs/prod-use1-marketplace/main.tf:762,780`) still trusts `repo:ai-nanosense*/nanosense-core*` and attaches `policy/nanosense-core-deploy`.
- Corroborating assessments: `nanosense-docs/docs/architecture-gap-review.md` (G7) and `med-intelligent/docs/superpowers/plans/2026-08-26-plato-clinic-onboarding-gap-fill.md:329` ("diverged history — unmergeable. Fix = build med-intelligent instead").

## 2. Decision

**`med-intelligent` is the canonical backend repository.** All backend development, code review, and deployment source-of-truth happens there. `nanosense-core` becomes a read-only historical fork and is a **candidate for retirement** per the checklist in §4.

Rationale: newer commits (2026-09-23 vs 2026-08-26), strictly more features (§1.2), and the production image is already built from its source (§1.3) — so "canonical" ratifies reality rather than creating new work.

## 3. Consequences

**Positive**
- One backend source of truth ends the fork drift that already blessed stale contracts (see `med-intelligent/marketplace/RETIRED-lambdas.md`).
- Marketplace route claims resolve to a single location: `med-intelligent/marketplace/`.
- Docs, CI, and IAM naming can converge on one repo name over time.

**Negative / accepted costs**
- The fork is "diverged (unmergeable)" — we do **not** merge histories. Anything unique and still needed in `nanosense-core` must be re-implemented/ported as fresh PRs to `med-intelligent` (checklist step B1).
- The live deploy pipeline currently *lives in* the repo being retired (§5 F1) — this must be re-homed **before** archiving, or production deploys break.
- IAM/Terraform names (`nanosense-core-deploy`) outlive the repo until an infra change lands (§5 F2) — cosmetic debt, tracked not blocking.

## 4. Retirement checklist for `nanosense-core`

> ⚠️ **Phases B–D require owner sign-off.** CORE-001 produces the decision package and the safe reference updates only (Phase A, done here). **Do not archive or delete any repository, and do not run git commands that change remote state, without that sign-off.** The repo is never deleted — it is archived read-only.

### Phase A — safe reference updates (✅ executed with CORE-001)

| # | Step | Status |
|---|---|---|
| A1 | ADR-001 written and merged into `nanosense-docs` | ✅ this document |
| A2 | `nanosense-marketplace/README.md` marketplace-claim repointed to `med-intelligent/marketplace/` with migration-status note | ✅ done |
| A3 | `nanosense-infra/docs/cost-review-2026-08-use1-marketplace.md` annotated ("nanosense-core = active codebase" claim superseded) | ✅ done |
| A4 | Pointer banner added to top of `nanosense-core/README.md`; rest of README left intact | ✅ done |

### Phase B — porting & CI re-homing (⚠️ owner sign-off required)

| # | Step | Verification |
|---|---|---|
| B1 | **Drift review:** disposition every commit unique to `nanosense-core` (`git log med-intelligent..nanosense-core` in the local clones). Port anything still needed (candidate: `9ddf888` CI cache fix) to `med-intelligent` via normal PRs. | Each unique commit is either ported (PR link) or explicitly marked "won't fix / obsolete" in this ADR's appendix |
| B2 | **Re-home the production deploy workflow.** `nanosense-core/.github/workflows/deploy.yml` ("Deploy Core API") is the live pipeline that builds/pushes `nanosense-prod/rag-backend` (us-east-1) and rolls the `nanosense-api` ECS service — it already checks out source from `ai-nanosense/med-intelligent`. Move it to `med-intelligent` (or `nanosense-infra`), keeping `BACKEND_REPO` semantics (it becomes a plain in-repo checkout). | One `workflow_dispatch` deploy from the new home succeeds; `aws ecs describe-services` shows the new task-def revision serving |
| B3 | Only after B2 is green: remove/disable `deploy.yml` (and `ci-tests.yml`/`security-scan.yml` if superseded) in `nanosense-core`. | No workflow in `nanosense-core` triggers a deploy |
| B4 | **Terraform OIDC cleanup** via the normal TF pipeline (see §5 F2): drop `repo:ai-nanosense*/nanosense-core*` from the trust policy once B2/B3 land; decide keep-vs-rename of `policy/nanosense-core-deploy` (referenced by med-intelligent docs as the ap-southeast-1 deploy policy). | `terraform plan` reviewed; apply through the standard infra PR |

### Phase C — freeze & archive (⚠️ owner sign-off required)

| # | Step | Verification |
|---|---|---|
| C1 | Swap `nanosense-core/README.md` body for a read-only pointer: repo name, "superseded by `med-intelligent` — see ADR-001", link to this file, and the do-not-develop banner (banner already in place from A4). Keep the repo itself. | README renders the pointer |
| C2 | Disable branch protection / rulesets; rotate or remove repo deploy secrets (`CI_GH_TOKEN` usage etc.). | Settings reflect read-only intent |
| C3 | **Mark the repo archived on GitHub** (Settings → Danger Zone → *Archive this repository*). **Never delete.** | Repo page shows "This repository has been archived. It is now read-only." |
| C4 | Post-archive checks: (a) push attempt is rejected; (b) deploy pipeline green from its new home (rerun B2 verification); (c) `grep -rn "nanosense-core"` across `nanosense-workspace` returns only historical/decision docs (ADR-001, cost review, gap review, tickets) — no live config. | All three pass |

### Phase D — closeout

| # | Step |
|---|---|
| D1 | Tick CORE-001 acceptance criteria in `nanosense-docs/docs/tickets/README.md` and the ticket file; record the CI-deploy verification result (CORE-001 "Testing Requirements": deploy pipeline passes against the canonical repo). |
| D2 | File/refresh follow-ups F1–F4 (§5 and §6) so the naming debt (`nanosense-core-deploy` policy, `med-intelligent/docs/*` references) is tracked to zero. |

## 5. Follow-ups (documented here — deliberately NOT changed by this ticket)

- **F1 — CI home mismatch (blocking for Phase C):** the workflow that deploys the canonical API lives in the *non-canonical* repo (`nanosense-core/.github/workflows/deploy.yml`, which builds from `med-intelligent` source). Also `med-intelligent/.github/workflows/deploy-backend.yml` exists as a manual ECS deploy path whose prerequisites (OIDC trust, `AWS_ROLE_ARN`/`AWS_ECR_REPOSITORY`/… secrets) need reconciliation with the live pipeline during B2. Reconfiguring CI is out of CORE-001 scope → tracked as checklist step B2.
- **F2 — Terraform naming:** `nanosense-infra/infra/terraform/envs/prod-use1-marketplace/main.tf:762` trusts `repo:ai-nanosense*/nanosense-core*`; `:780` attaches `arn:aws:iam::…:policy/nanosense-core-deploy`. Per CORE-001 constraints no terraform logic was changed here — only this note. Action: checklist step B4.
- **F3 — stale docs in `med-intelligent`:** `med-intelligent/docs/activation-procedure.md:49-51,217` and `docs/marketplace-github-secrets.md:53` still describe `nanosense-core` CI jobs/policy names. Update during Phase B (out of the assigned edit scope for this ticket).
- **F4 — marketing-page duplication:** see §6.

## 6. Known duplication — marketing pages (`med-intelligent/website/` vs `nanosense-website/website/`)

**Recommendation: `nanosense-website/website/` is the canonical marketing site.** It is the dedicated marketing repo and already carries the WEB-001 claims fixes.

- The two trees are parallel copies: `index.html`, `docs.html`, `privacy.html`, `terms.html`, `marketplace-onboarding.html`, `css/`, `js/`, `Nanosense_logo/`, `marketplace-images/`.
- **Both** have live deploy workflows that `aws s3 sync … --delete` into the **same** production bucket/distribution (`nanosense.net-website` / CloudFront `E32NZ73IVW4RCS`): `nanosense-website/.github/workflows/deploy.yml` (paths `website/**`) and `med-intelligent/.github/workflows/deploy-website.yml` (stack `nanosense-website`, template `med-intelligent/infra/nanosense-website-stack.yml`). Last push wins and can delete the other copy's files — the same drift pattern as the backend fork.
- **`med-intelligent/website/` still contains the pre-WEB-001 overclaims**, verified 2026-09-24:
  - `med-intelligent/website/index.html:127` — "HIPAA Compliant · **SOC 2 Ready**" → canonical `nanosense-website/website/index.html:129` has "SOC 2 **In Progress**".
  - `med-intelligent/website/index.html:307-308` — "**End-to-end encryption**, HIPAA audit logging, field-level encryption, **SOC 2 controls**…" → canonical `nanosense-website/website/index.html:309-310` has "Encryption in transit and at rest… **SOC 2-aligned controls**" (E2E claim removed — matches gap review G4/G5).
- **Action (not executed here — code/claims edits are WEB-001/DOC-001 scope and ownership is undecided):** once ownership is decided, either (a) retire `med-intelligent/website/` + its `deploy-website.yml` and keep `nanosense-website` as the single deploy source, or (b) at minimum apply the same WEB-001 text fixes to `med-intelligent/website/` (`index.html:127,307-308`, plus a sweep of `docs.html`, `privacy.html`, `marketplace-images/*`, which also matched claims-wording greps). Until then, treat any push to `med-intelligent/website/**` as a risk of re-publishing overclaims to nanosense.net.

## 7. Acceptance criteria mapping (CORE-001)

| Criterion | State after this ticket |
|---|---|
| ADR merged | ✅ this document (`nanosense-docs/docs/adr-001-canonical-backend-repo.md`) |
| Single canonical backend repo | ✅ decided: `med-intelligent` (ratified by this ADR) |
| References consistent | ✅ marketplace README + infra cost-review doc updated; remaining naming debt tracked in F1–F3 |
| CI builds the canonical repo | ⚠️ already true in substance (`nanosense-core` deploy.yml builds `med-intelligent` source into `nanosense-prod/rag-backend`); formal re-homing is checklist B2, blocked on owner sign-off |
| Testing: CI deploy passes against canonical repo | ⚠️ to be re-verified after B2 (record result in D1) |

*Evidence commands used (read-only): `git log -1` per clone; `test -e` file matrix; `grep -rn "nanosense-core"` across `nanosense-marketplace`/`nanosense-infra`/`nanosense-docs`; claims-wording grep across both `website/` trees.*


