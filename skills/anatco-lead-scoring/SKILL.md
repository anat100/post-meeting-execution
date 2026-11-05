---
name: anatco-lead-scoring
description: Anatco's account scoring and tiering method (A/B/C/D) combining company fit and buying intent, written to the HubSpot account_tier property. Load when scoring or rescoring an account or lead, including from the post-meeting-execution skill.
---

# Anatco lead scoring

> **Status: test draft.** The ideal customer profile (ICP) below is a placeholder for a B2B software company. Replace it with Anatco's real ICP and adjust point values against a sample of won and lost deals before relying on it.

Each account gets a **Fit score** (who they are, 0–50) and an **Intent score** (what they've shown, 0–50). The total (0–100) sets the tier. Score only on evidence — from HubSpot, enrichment, or the meeting transcript. Missing evidence scores 0 for that line; never assume.

This is separate from the BANT check in post-meeting-execution Step 7. BANT decides whether to suggest a deal; this tier decides how much attention the account gets.

---

## Fit score (0–50)

| Signal | Points |
| --- | --- |
| **Company size:** 50–1,000 employees | 15 |
| Company size: 1,001–5,000 | 10 |
| Company size: 10–49 | 5 |
| Company size: under 10 or over 5,000 | 0 |
| **Estimated ACV ≥ $20K** (enterprise threshold) | 15 |
| Estimated ACV $5K–$20K | 8 |
| Estimated ACV under $5K or unknown | 0 |
| **Industry:** B2B software / SaaS, or tech-enabled B2B services | 10 |
| Other B2B | 5 |
| B2C, public sector, or non-profit | 0 |
| **Has a revenue / GTM operations function** (RevOps, Sales Ops or Marketing Ops role exists) | 10 |

## Intent score (0–50)

Add every signal that applies, cap at 50.

| Signal | Points |
| --- | --- |
| Stated intent to trial or buy | 15 |
| Specific use case named, with expected outcome | 10 |
| Budget confirmed or referenced | 10 |
| Timeline or compelling event stated (e.g. "need this live by Q1") | 10 |
| VP / C-level decision-maker attended | 10 |
| Additional stakeholders joined or were named for next steps | 5 |
| Prospect booked or requested the next meeting themselves | 5 |
| Negative signals — subtract | |
| Said no / not a fit | −30 |
| Interest clearly cooled vs. the previous call | −10 |
| Locked into a competitor contract for 12+ months | −10 |

Intent decays: signals older than 90 days with no new activity count at half value.

---

## Tiers

| Tier | Total | Meaning | Treatment |
| --- | --- | --- | --- |
| **A** | 75–100 | Strong fit, active buying motion | Full sales treatment; Anat owns personally |
| **B** | 50–74 | Good fit, real interest, gaps remain | Active sales; focus the next step on the biggest gap |
| **C** | 25–49 | Partial fit or early interest | Self-serve path or light-touch nurture |
| **D** | 0–24 | Poor fit or no interest | No sales effort; marketing only |

**ACV override:** an account with estimated ACV under $20K can't be tier A, whatever its total — cap it at B.

---

## Writing the result

- Write the tier to the HubSpot company property `account_tier` (A / B / C / D). If the property doesn't exist yet, report that in the review item and continue — it's a non-blocking failure.
- Log a HubSpot note on the company: `Account tier: [old] → [new] (Fit [x]/50, Intent [y]/50). Reason: [the 1–3 signals that moved the score, with evidence].`
- If the tier didn't change, still log the note and say "no change".
