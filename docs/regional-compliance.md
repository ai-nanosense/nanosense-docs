# Regional Privacy & Compliance Brief

> **This page is informational only and does not constitute legal advice.** Consult qualified counsel in your jurisdiction for compliance obligations specific to your clinic.

NanoSense serves clinics across Southeast Asia and Oceania. This brief summarises how the platform's regional deployments map to the privacy regimes you most likely operate under.

---

## Singapore — Personal Data Protection Act (PDPA)

Clinics handling patient personal data in Singapore are subject to the PDPA, administered by the PDPC. Key obligations relevant to using NanoSense:

- **Consent & purpose limitation** — obtain patient consent for collection, use, and disclosure of personal data, and use it only for purposes a reasonable person would consider appropriate (care delivery, billing, quality improvement as notified).
- **Protection & accuracy** — make reasonable security arrangements for patient data. NanoSense supports this with encryption in transit and at rest, tenant isolation, and audit logging (see the [Security Whitepaper](hipaa/security_whitepaper.md)).
- **Mandatory breach notification** — notifiable data breaches must be reported to the PDPC **within 3 calendar days** of assessing that a breach is notifiable, and affected individuals must be informed. Our [Incident Response Plan](hipaa/incident_response_plan.md) defines severity classification and notification workflows aligned to this timeline.
- **Data Protection Officer (DPO)** — organisations must designate a DPO and make their contact available; the DPO oversees PDPA compliance and handles data-related enquiries.
- **Retention limitation** — cease retention when the purpose is served and retention is no longer necessary. See our [Data Retention Policy](hipaa/data_retention_policy.md).

## Australia — Privacy Act 1988 (APPs)

Australian clinics are covered by the *Privacy Act 1988* and the Australian Privacy Principles (APPs). Most relevant:

- **APP 1/5** — open and transparent management of personal data, plus a clearly expressed privacy policy.
- **APP 6** — use or disclosure only for the primary purpose of collection (secondary purposes permitted for health care under defined conditions).
- **APP 11** — security of personal information; take reasonable steps to protect it from misuse, interference, loss, and unauthorised access.
- **Notifiable Data Breaches scheme** — eligible data breaches must be reported to affected individuals and the OAIC as soon as practicable after assessment.

The OAIC publishes authoritative guidance on the APPs and health-information obligations — start there, alongside your own counsel.

## Data Residency

| Region | Stack | Status |
|--------|-------|--------|
| Singapore | `ap-southeast-1` (AWS Singapore) | Live |
| Australia | `ap-southeast-2` (AWS Sydney) | Planned |

Singapore-region tenants have their data processed and stored in AWS `ap-southeast-1`. An Australia-resident deployment (`ap-southeast-2`) is planned so AU clinics can keep patient data onshore; until it ships, confirm cross-border transfer arrangements suit your obligations.

Regardless of region: per-tenant isolation applies everywhere, and no patient data leaves the deployed region except at your direction (e.g., an explicit query routed to an external knowledge source such as PubMed, which receives the question text only — never your stored patient records).
