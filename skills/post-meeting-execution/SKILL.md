---
name: "post-meeting-execution"
title: "Post-meeting execution"
description: "Use this skill when you want every sales meeting to produce its follow-through automatically: transcript pulled, outcome classified, account rescored, and a ready-to-send follow-up drafted in the meeting owner's voice within minutes of the call ending. It runs the full production loop — qualification scoring with a human-in-the-loop deal decision, contact and stakeholder loading, CRM lifecycle updates, and a single review post where the owner answers three Yes/No questions to trigger everything downstream. Built for teams where one person takes many external meetings and the follow-up quality (and CRM hygiene) shouldn't depend on their memory."
category: Deals
---

# Post-meeting execution

## Configuration (test values)

These values were filled in for a test setup. See the setup checklist reference for the full setup list.

- **Meeting owner:** Anat Gilad (one instance per person)
- **Meeting-notes tool:** Fireflies.ai
- **CRM:** HubSpot (portal "Anatco"), deal pipeline "Sales Pipeline"
- **Review channel:** #post-meeting-review (Slack) — all post-meeting outputs post here for review
- **Internal domain:** anatco.com — attendees on this domain are internal; everyone else is external
- **Follow-up scenarios sub-skill:** anatco-followup-scenarios — meeting-outcome categories, classification logic, per-category email frameworks
- **Voice guide sub-skill:** anatco-voice-guide — the owner's tone, formatting, greeting style, approved links
- **Lead scoring sub-skill:** anatco-lead-scoring — lead scoring & qualification methodology
- **Enterprise ACV threshold:** $20K ACV — above this, full sales treatment; below, self-serve
- **Sales-active tag:** the `sales_active` company property (checkbox) in HubSpot, marking accounts with a live sales conversation
- **Metrics log:** the "GTM Meeting Log" Google Sheet, where external meetings are recorded for GTM metrics
  - Spreadsheet ID: `YOUR_SPREADSHEET_ID` (replace with the ID from your sheet's URL: docs.google.com/spreadsheets/d/**ID**/edit)
  - Tab: `Sheet1`. Row 1 holds the headers; append one row per meeting below them, in column order: event_id (A), event_type (B), domain (C), company (D), meeting_date (E, YYYY-MM-DD), external_attendees (F, comma-separated names)
  - Read-after-write check: read column A and look for the event_id just written
- **Score tag:** the `account_tier` company property in HubSpot (values A / B / C / D, defined in anatco-lead-scoring)

---

### Overview

After every meeting involving Anat Gilad, run a structured workflow in two phases:

1. **Deal creation decision** — qualification scoring, human confirmation, contact/stakeholder loading
2. **Follow-up email** — classify the outcome, draft in the owner's voice, send for approval

**Non-blocking errors:** memory, enrichment, and HubSpot write failures are non-blocking — log the error in the review item and continue; a failed step is never a reason to skip subsequent steps. The only blocking failure is a missing meeting record (Step 1).

---

### Step 1 — Pull Meeting Context

Retrieve the full record from Fireflies.ai: summary, full transcript, action items, attendees (names, titles, companies), any logged decisions. If your notes tool auto-emails attendees the summary and recording, do NOT duplicate that in the follow-up email.

---

### Step 2 — Enrich the Account

Look up the company in your workspace and HubSpot: current funnel stage, size (headcount, revenue), industry, prior relationship history, and any existing deal.

Then **estimate ACV** from company size, use cases surfaced in the transcript, and any pricing signals. This gates the $20K ACV line between self-serve and enterprise treatment. If ACV cannot be estimated with reasonable confidence, flag it in the review item.

**Tagging:** if at least one attendee's email domain is not anatco.com, apply the `sales_active` company property (checkbox) in HubSpot to the company (idempotent). Internal-only meetings skip this and every external-meeting step below.

---

### Step 3 — Log the Meeting for Metrics (optional)

> Skip if internal-only, or if you don't maintain the "GTM Meeting Log" Google Sheet.

Classify the meeting type. **Never short-circuit on deal existence** — deals are often auto-created the moment a demo is booked, so an open deal is not evidence of a prior meeting.

1. **Closed-won check:** account is a current customer → `meeting_customer`. Stop.
2. **Check history sources in parallel:** (a) account memory — is a first-demo date recorded? If yes → `meeting_active_deal`, stop. (b) HubSpot company-level meeting engagements **within the last 6 months**. (c) HubSpot meeting engagements on any contact at the company, same 6-month filter. Older engagements are stale ghost history from a prior cycle — ignore them.
3. If any source confirms a qualifying prior meeting → `meeting_active_deal`. Otherwise → `meeting_first_touch`, even if an open deal exists.

> **Same-day deal rule:** If an open deal's create date falls on the same calendar day as this meeting, treat as `meeting_first_touch` — the deal was almost certainly auto-created by the booking, not evidence of a relationship. The recency filter and this rule together prevent the known false-`meeting_active_deal` failure modes.

Write one row to the "GTM Meeting Log" Google Sheet: unique event ID (`[type]-[domain]-[date]`, suffix `-2`, `-3` for multiple same-day meetings), event type, domain, company, meeting date, external attendee names.

> ⚠️ **Read-after-write verification required.** After every log write, read the row back by its event ID. Not found → retry once. Still not found → add a visible warning to the review item ("Log write failed — manual entry required"). Never silently accept a false success response.

**First-demo detection (only when `meeting_first_touch`):** If no first-demo date exists in account memory or the log, record one — using the date the meeting was **booked** (created in HubSpot), not the date it occurred; this metric tracks booking velocity. Never overwrite an existing first-demo date. Write the date to account memory.

---

### Step 4 — Update Account Memory

Runs for **every meeting**; depth varies by call type. **All calls:** key topics and tone, commitments from either side, stakeholder dynamics (champions, skeptics, blockers), anything that informs the next interaction.

**Scoping calls** additionally capture these structured fields — omit any field with no evidence from this meeting; never fabricate:

- **Pain & Friction** — the exact moment where their current workflow breaks down, not generic pain
- **Use Cases / Workflows to Test** — each workflow they agreed to evaluate, named with its expected output (e.g. "visitor identification → auto-enrich + HubSpot push", not "pipeline generation")
- **Trial Participants** — names, titles, roles in the evaluation
- **Success Criteria** — the specific outcomes that would make them say "this worked"

If your product's onboarding can consume this context (to pre-populate a trial account), push the structured block there whenever the account is still pre-trial. The more specific, the faster the prospect's time to value.

---

### Step 5 — Rescore the Account

Rescore against anatco-lead-scoring using signals from this specific meeting: new decision-makers joining, stated intent to trial or buy, a use case / budget / timeline named for the first time, enthusiasm shifting either way. If the tier changed, update the score tag (remove old, apply new) and log the reasoning. If unchanged, note that explicitly.

---

### Step 6 — Recommend a Funnel Stage (do not apply yet)

Determine the correct stage from the updated score plus meeting context — but **do not update it**; the move requires the owner's approval in Step 10. Use stage *semantics* as a guide; the transcript beats mechanical rules:

Stages are from the HubSpot deal pipeline "Sales Pipeline" (internal IDs in backticks — use the ID when writing `dealstage`).

| Signal from meeting | HubSpot deal stage |
| --- | --- |
| Demo booked, not yet held | Appointment Scheduled (`appointmentscheduled`) |
| Demo done, prospect wants to move forward | Qualified To Buy (`qualifiedtobuy`) |
| Scoping / evaluation call or solution presentation booked | Presentation Scheduled (`presentationscheduled`) |
| Trial or evaluation complete, decision-maker on board, discussing commercial terms | Decision Maker Bought-In (`decisionmakerboughtin`) |
| Terms agreed, contract out / in procurement or legal | Contract Sent (`contractsent`) |
| Contract signed | Closed Won (`closedwon`) |
| Said no or not a fit | Closed Lost (`closedlost`) |
| Strong interest, specific blocker (tech, timing, budget) | No deal-stage move — keep the current stage and set contact lifecycle stage to Nurturing (`other`) |
| No clear path forward | No deal-stage move; if no deal exists, do not create one. Set contacts to Lead (`lead`) and note the reason |

This pipeline has no trial stage and no stage set automatically by a product signup event, so every stage above is set from meeting evidence — but still only after the owner's Yes in Step 10.

---

### Step 7 — Qualification Scoring (BANT)

> **Skip entirely if an existing deal was found in Step 2** — already qualified. Go to Step 8.

Score each criterion **met / not met** on transcript evidence only:

- **Budget** — actual budget confirmed or referenced, not mere willingness to pay
- **Authority** — a purchasing decision-maker in the room (startups: VP/C-level required; larger orgs: Director may qualify, VP+ preferred)
- **Need** — BOTH clearly defined use cases driving fast time-to-value AND a stated desire to start a trial
- **Timeline** — a stated timeline or compelling event forcing speed

Count criteria met (0–4), record evidence for each.

**Override — exception criteria.** Even below 2/4, flag for deal creation when **all three** hold: (1) C-suite or VP-level authority confirmed in the room; (2) exceptional use-case alignment — their stated problems map directly onto your core value proposition, not generic interest; (3) structural growth pressure — aggressive targets, recent funding, team expansion, or a high-stakes operational gap (e.g. a one-person ops team scaling a motion that needs automation). Flag the override explicitly: name the unmet criteria and the compensating signals. The owner makes the exception call — never auto-create.

**Deal creation decision (human in loop):**

- **2+ criteria met (or override):** create a **review item** — not a plain message; the owner must be able to reply and act from Slack — with the scorecard and evidence, estimated ACV, and: "Based on [X/4], I recommend creating a deal. Agree? **Yes / No**". Post to #post-meeting-review. Mandatory — never continue past it silently.
- **Fewer than 2, no override:** do not suggest a deal. Note the score and reasoning inside the email review item (Step 10).

**If the owner says Yes:** re-check HubSpot for an existing open deal — if one exists, report it (ID + stage), do NOT create a duplicate. Otherwise ask the owner for the amount, then create the deal: name = company name only, stage matched to funnel position, close date estimated from timeline signals, owner = Anat Gilad. Associate it to the company, and mirror ownership and stage to your workspace automatically — the owner should not have to ask.

---

### Step 8 — Contacts & Stakeholders (always runs)

**A. Call participants from the target company:** find or create each in HubSpot; **enrich every contact, new and existing** (title, LinkedIn URL, email) — a contact without a job title is incomplete. If any deal exists (found or created), associate each contact to it.

**B. Stakeholders named but absent** ("you'll need to loop in our CISO"): find or create, enrich, associate to the deal as buying-committee members, and log a note: "Identified as key stakeholder during call on [date] — not yet engaged directly."

**Lifecycle stages:** with a deal, set participants and stakeholders to SQL/opportunity. Without one: clear interest but not ready → SQL; vague/exploratory → MQL; no pain or fit → lead; explicitly not a fit → clear the stage or mark unqualified. Log a note explaining the disposition.

---

### Step 9 — Classify & Draft the Follow-Up Email

Load two references in parallel:

1. anatco-followup-scenarios — outcome categories, classification logic, per-category email frameworks (what to say, structure)
2. anatco-voice-guide — how Anat Gilad says it: tone, formatting, greeting, when links belong

**Grounding rule (non-negotiable):** every element in the email — every bullet, link, and reference — must trace directly to something discussed, asked, or committed to in this call. Not in the transcript → not in the email. No default resources, no boilerplate bullets. It should read like a summary of this conversation, not a template with the company name swapped in.

---

### Step 10 — The Review Item

> ⚠️ **Mandatory on every execution.** Both parts are the primary outputs — never end the run without them.

**A. The canonical Slack post** to #post-meeting-review — full content every time, never a pointer elsewhere, exactly this structure:

```
*[Company] — Post-Meeting Follow-up Draft Ready*

*Meeting:* [name] — [date]
*Attendees:* [owner] + [external attendees with titles]
*Classification:* [outcome category]
*Stage:* [current] ([change or "no change"]) | *Score:* [tier] ([change or "no change"])
---
*Key Takeaways*
• [3–4 one-liners, specific to this call]
---
*Follow-up Email Draft*
*To:* / *CC:* / *Subject:*
[Full email body — no truncation]
---
*CRM Actions Taken*
• [bullet list]
---
*Funnel Stage:* [current] → [recommended] — [one-sentence rationale]. Move it? *Yes / No*
*Suggested Next Steps (HubSpot):* "[text]" — push to HubSpot? *Yes / No*
*Follow-up timing:* How many days until you want a reminder?
```

Thread replies: recording link, meeting-notes link, execution link.

**B. A review item (actionable task)** with: the chosen category and reasoning; qualification score summary (only if scoring ran); the full ready-to-send email (To/CC/Subject/Body); flags (ACV uncertainty, ambiguous classification, missing stakeholders); CRM actions taken; the stage question; and:

**Suggested next steps** — distill the single most important next action from the full meeting summary. Strict filter — include only hard blockers or direct paths to conversion: technical setup required to start a trial, an unbooked scoping/success-criteria call (for trial accounts, always a hard blocker), a dependency that gates the product working, or a step toward signature (pricing, procurement, legal). Exclude passing ideas, relationship-warmth actions, nice-to-haves, anything already underway. Multiple hard blockers → condense into one tight line. Written as a human action, **strictly under 150 characters** (e.g. "Place pixel, connect CRM, book scoping call before trial ends").

**Do not send the email yourself.** The owner copies the draft and sends from their own inbox — that preserves CC and proper threading. No signature in the body; their client adds it.

---

### Step 11 — Post-Send CRM Updates

After the owner approves and sends:

- Update contact lifecycle stages if not already set; log a note summarizing the email sent
- **Stage move (if approved Yes):** apply the recommended funnel stage and log the reasoning. If No, leave it and note that.
- **Next steps (if approved Yes):** write the approved text to the deal's next-step field. If No or unanswered, skip.
- **Stage sync (always):** if you mirror stages between your workspace and HubSpot, sync now using the *current* stage even if it didn't change in this step — this catches changes made earlier in the run.
- **Follow-up task:** create a HubSpot task "Follow up — [Company]", due today + the days the owner specified (default 5 business days, noted in the task), type email, assigned to Anat Gilad, associated to the deal (or the company if no deal).

---

### Step 12 — LinkedIn Connection Requests

> ⚠️ **Always executes**, even if earlier steps failed. Run it as its own dedicated action — never bundled with steps that could error and drop it.

For every external attendee, queue a LinkedIn connection request from Anat Gilad's connected account for approval alongside the review post (orgs that trust the filter can flip this to auto-send — see the setup checklist). Rules: blank connection note (no message text); only people who actually attended, never absent stakeholders; skip anyone already connected; skip entirely on a no-fit / no-clear-pain outcome — a connection request after that conversation is odd.

---

## What good looks like

A great run means the owner opens #post-meeting-review twenty minutes after the call and finds everything decided-but-not-done: the meeting correctly classified, the account rescored with reasoning, a follow-up draft where every line traces to the transcript, a qualification scorecard with evidence per criterion, one sub-150-character next step naming the real blocker, and three crisp Yes/No questions (deal? stage move? next-step push?). Human approval gates every consequential write: the email goes from the owner's inbox, the stage moves only on Yes, the trial stage is never set by a meeting outcome. Connection requests queued for attendees only. Mediocre looks like: templated follow-ups with swapped company names, deals created without asking, stale CRM ghost engagements inflating classifications, a truncated draft pointing "see task for details," or a failed memory write silently killing every step after it.
