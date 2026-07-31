# Pilot Measurement

Six criteria. **National rollout is recommended when at least five are achieved.**

This document states the exact numerator and denominator behind each one, because a
scorecard nobody can audit is a scorecard nobody trusts. The scorecard UI shows each
metric's basis alongside its value.

Implementation: `server/homeHealthCheck/pilotScorecard.ts` in CSR Tool v2.

---

## 1. Home Health Check Acceptance — target 70%+

**accepted ÷ offered**, over checks *offered* in the window.

A record is written the moment the technician **offers** the check, before the customer
answers. That is deliberate: without it there is no denominator, and an acceptance *rate*
cannot be computed from accepted checks alone. A declined check is a stored record with a
decline reason, not an absence.

"Accepted" means any status past `offered` other than `declined` — a walkthrough abandoned
halfway still counts as accepted, because it was.

## 2. Opportunities Identified — target 3+ per check

**(Red + Purple findings) ÷ completed checks.**

- Yellow and Green are excluded. A walkthrough that finds twelve things in great shape has
  identified zero opportunities, and the scorecard must not be able to claim otherwise.
- The denominator is *completed* checks, not accepted ones. An accepted walkthrough still
  in progress has not finished finding things.

## 3. Proposal Conversion — target 25%+

**accepted proposals ÷ proposals generated from checks.**

Each Red/Purple finding creates a field estimate in the office approval queue; the
finding's `budget.localEstimateId` is the join key. Conversion reads the client decision on
those estimates.

A finding whose office-queue write failed is counted on **neither** side, which keeps the
ratio honest rather than inflating or deflating it.

## 4. Customer Satisfaction — target 90%+

**responses rating 4–5 ÷ total responses**, on a 1–5 scale collected on the technician's
phone with the customer present.

Checks with no response are excluded from both sides — never assumed satisfied.

## 5. Expansion Revenue per Check — target $500+

**sold value of accepted check-generated estimates ÷ completed checks.**

Sold value is the total of the option the customer actually selected — not the budget range
presented during the walkthrough. Presented budget and realised revenue are different
numbers and the scorecard reports the second.

## 6. Lead Growth — target 15%+

**(leads this window − baseline) ÷ baseline.**

This measures the **"We Do, You Do"** acquisition campaign, not the Home Health Check
program, so it is entered on the scorecard rather than derived from check records. Until a
small-job lead definition is agreed and both windows are entered, this criterion reports
*"no data yet"*.

---

## Reporting rules

- A criterion with no data is **`measurable: false`** → shown as *"no data yet"*.
  It is **not** counted as achieved, and it is **not** reported as a miss.
- The verdict distinguishes the three states honestly:
  - *"5 of 6 targets achieved — meets the threshold for a national rollout recommendation."*
  - *"3 of 6 achieved; 2 not yet measurable. Five of six are required."*
  - *"4 of 6 achieved — below the five-of-six threshold."*
- The window defaults to the trailing 16 weeks, matching the pilot period.

## Illustrative model, for reference

From the executive brief:

| | |
|---|---|
| Home Health Checks | 1,000 |
| Expansion revenue per check | $750 |
| **Potential expansion revenue** | **$750,000** |

At 25% of the modeled outcome the program still returns roughly **$187,500**. Actual pilot
results determine the rollout recommendation.
