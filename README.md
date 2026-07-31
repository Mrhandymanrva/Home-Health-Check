# Home Health Check

> "We Fixed Today's Problem. Let's Help You Avoid Tomorrow's."

The customer-expansion half of the **Small Jobs Growth Initiative**: after every eligible
service visit, the technician spends five minutes walking the property, grades what they
find, and the homeowner receives a photo report with budget ranges for anything worth
addressing.

This repository holds the **program specification, walkthrough checklist, and pilot
measurement definitions**.

## Where the software lives

The working implementation is a module inside the **CSR Tool v2** repository, on the
branch `feature/home-health-check`:

<https://github.com/Mrhandymanrva/CSR_Tool/tree/feature/home-health-check>

It was built there rather than standalone because it depends on infrastructure that
already exists in that app — ServiceTitan OAuth and job hydration, the technician field
PWA and its auth, the estimating engine, and the office approval queue. A standalone
franchise-deployable version is a separate decision, noted under
[Open questions](#open-questions).

| Concern | Location in CSR Tool v2 |
|---|---|
| Domain model, buckets, send gate | `shared/homeHealthCheck.ts` |
| Walkthrough checklist | `server/homeHealthCheck/checklistTemplate.ts` |
| Record storage + photos | `server/homeHealthCheck/store.ts` |
| Budget drafting | `server/homeHealthCheck/budgetDrafts.ts` |
| Report model / HTML / PDF | `server/homeHealthCheck/report*.ts` |
| Email + ServiceTitan write-back | `server/homeHealthCheck/delivery.ts` |
| Pilot scorecard | `server/homeHealthCheck/pilotScorecard.ts` |
| Technician walkthrough UI | `client/src/pages/FieldHealthCheck.tsx` |
| Customer report page | `client/src/pages/HealthCheckReport.tsx` |
| Office scorecard UI | `client/src/components/cockpit/HomeHealthCheckView.tsx` |

## The customer experience

1. **Complete the original repair.** The Home Health Check is step two of the visit, never
   a substitute for the work booked.
2. **Five-minute property walkthrough** — interior review, exterior review, photos.
   See [CHECKLIST.md](./CHECKLIST.md).
3. **Opportunity report.** Every finding lands in exactly one of four buckets.
4. **Follow-up.** Recommended work reaches the office queue for firm pricing.

## The four buckets

| | Bucket | Meaning | Carries a budget? |
|---|---|---|---|
| 🔴 | Urgent Safety Issue | Potential safety concern requiring prompt attention | Yes |
| 🟣 | Recommended Repair | Action recommended in the near future | Yes |
| 🟡 | Monitor | Monitor over next 6–12 months | No |
| 🟢 | Looks Great | No action required | No |

## On the numbers in the report

Budget figures are **planning ranges, not quotes**, and the report says so wherever a
number appears.

A walkthrough finding reaches the estimator as a photo and a sentence — without the
quantities, access, materials, and condition facts a normal scoped estimate collects. The
range therefore widens according to the estimator's own confidence, and firm pricing comes
from the standard estimate path after office review. Every Red and Purple finding is
written into the office approval queue for exactly that purpose.

A finding the estimator cannot price shows **"We'll follow up with pricing"**. It never
shows $0 — zero is treated as a failure to price, not as a price.

## Delivery

The report goes out three ways, and the three are deliberately independent:

| Channel | Purpose | Gates "sent"? |
|---|---|---|
| Email (PDF attachment) | The homeowner's copy | **Yes** |
| ServiceTitan job attachment | The PDF on the customer record | No |
| ServiceTitan customer note | Dated record of what was found and sent | No |

A report the customer received counts as sent even if ServiceTitan was briefly
unreachable. Failed write-backs stay visible and retryable from the office scorecard.

### Configuration

Outbound email is **off until configured**, and the technician's screen says so rather
than silently dropping reports.

```bash
# Option A — Resend (HTTP, no SMTP egress needed)
EMAIL_PROVIDER=resend
RESEND_API_KEY=re_...

# Option B — existing mail provider (Microsoft 365, Google Workspace, …)
EMAIL_PROVIDER=smtp
SMTP_HOST=smtp.office365.com
SMTP_PORT=587
SMTP_USER=reports@yourdomain.com
SMTP_PASSWORD=...

# Required either way
EMAIL_FROM_ADDRESS="Mr. Handyman of Richmond <reports@yourdomain.com>"
EMAIL_REPLY_TO=office@yourdomain.com
PUBLIC_APP_URL=https://your-app-host        # builds the "view online" link
```

ServiceTitan write-back uses credentials the app already holds, plus
`SERVICETITAN_NOTES_PATH` (defaulted to `/crm/v2/tenant/{tenantId}/customers/{customerId}/notes`).

## Pilot measurement

See [PILOT-METRICS.md](./PILOT-METRICS.md) for the exact definition and denominator of
each of the six criteria.

The brief's rule — **national rollout is recommended when at least five of six targets are
achieved** — is encoded in the scorecard rather than left to a spreadsheet, so the go/no-go
is reproducible. A criterion with no data reports *"no data yet"*; it is never counted as
a pass, and never reported as a miss.

## Open questions

- **Standalone deployment.** The pilot spans 8–12 franchise territories, none of which run
  CSR Tool v2. Territories outside Richmond need either access to this app or a
  standalone build — the latter means porting the ServiceTitan client, auth, and (if
  budget ranges are to survive) the estimating engine.
- **Lead growth input.** The sixth criterion measures the "We Do, You Do" acquisition
  campaign, which lives outside these records. It is entered manually on the scorecard
  until a small-job lead definition is agreed.
- **ServiceTitan note endpoint** has not yet been exercised against the live tenant for
  this payload shape; the attachment endpoint is already in production use for estimate
  scope documents.
