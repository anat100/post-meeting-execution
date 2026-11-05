---
name: anatco-followup-scenarios
description: Meeting-outcome categories, classification rules and per-category follow-up email frameworks for Anatco sales calls. Load when classifying a sales meeting's outcome or drafting its follow-up email, including from the post-meeting-execution skill.
---

# Anatco follow-up scenarios

Use this to (1) classify what happened in a sales meeting and (2) decide what the follow-up email should do and how it is structured. **How** it sounds comes from `anatco-voice-guide`; this file covers **what** to say.

The grounding rule from post-meeting-execution always applies: every bullet, link and reference in the email must trace to something said, asked or committed to in the call. The frameworks below are structures, not text to paste.

---

## Step 1 — Classify the outcome

Read the transcript, then walk this list top to bottom and stop at the first category whose test is clearly met. If two categories fit about equally, pick the earlier one and flag "ambiguous classification" in the review item with the runner-up.

| # | Category | Test (evidence must be in the transcript) |
| --- | --- | --- |
| 1 | **No-show / not held** | The meeting didn't happen, or was cut short and rescheduled before any discovery |
| 2 | **Not a fit** | Prospect said no, or there's no problem we solve, or a hard disqualifier (wrong market, no budget ever, using a competitor under multi-year contract) |
| 3 | **Contract / procurement** | Terms are agreed in principle; the next step is paperwork, security review, legal or vendor onboarding |
| 4 | **Commercial — pricing requested** | Prospect asked for pricing, a proposal or a quote, or is comparing us against alternatives on cost |
| 5 | **Moving to trial / evaluation** | Prospect explicitly agreed to try the product or run a structured evaluation |
| 6 | **Needs scoping** | Clear interest, but the next step is a deeper technical or use-case session, or more stakeholders must join before anything moves |
| 7 | **Interested but blocked** | Real interest plus a named blocker: timing, budget cycle, missing integration, internal priority |
| 8 | **Customer check-in / expansion** | Existing customer; the call was about adoption, renewal or adding seats / use cases |
| 9 | **Exploratory / early** | Intro or discovery call with no clear next step agreed either way |

---

## Step 2 — Email frameworks by category

Each framework lists the email's single goal, its parts in order, and what to leave out. Keep every email skimmable: the reader should know the ask within the first two lines.

### 1. No-show / not held
- **Goal:** rebook without guilt-tripping.
- **Parts:** one-line acknowledgement → offer two specific time options or ask for theirs → nothing else.
- **Leave out:** recap, product pitch, "just following up" filler.

### 2. Not a fit
- **Goal:** close the loop gracefully and leave the door open.
- **Parts:** thank them for their time → one line naming the reason it isn't a fit right now, in their words → if they mentioned a future trigger (e.g. "after we hire a RevOps lead"), name it as the moment to reconnect.
- **Leave out:** attempts to rescue the deal, resources they didn't ask for. No LinkedIn request either (Step 12 skips this outcome).

### 3. Contract / procurement
- **Goal:** keep paperwork moving and make ownership obvious.
- **Parts:** confirm what was agreed (scope, term, price only if stated in the call) → list each open item with an owner on each side (e.g. "Security questionnaire — your IT team / we return within 3 days") → the next date both sides committed to.
- **Leave out:** re-selling the product. They've decided.

### 4. Commercial — pricing requested
- **Goal:** deliver or schedule the pricing conversation, tied to their use case.
- **Parts:** restate the use cases and scale they described (seats, volume, teams) → either the pricing / proposal if it was promised in the call, or a proposed time to walk through it → one question to close any gap needed to price accurately.
- **Leave out:** list prices that weren't discussed. Above the $20K ACV threshold, prefer a walkthrough call over sending numbers cold.

### 5. Moving to trial / evaluation
- **Goal:** get the trial set up so it can succeed.
- **Parts:** the workflows they agreed to test, each with its expected output → success criteria in their words → setup steps and who owns each → the scoping / check-in call date (book one if it isn't booked; for trials it's a hard blocker).
- **Leave out:** generic onboarding docs not mentioned in the call.

### 6. Needs scoping
- **Goal:** lock in the deeper session with the right people.
- **Parts:** 2–3 bullets on the problems they described → what the next session will cover → who needs to be there (name the absent stakeholders they mentioned) → proposed times.
- **Leave out:** solutions to problems they didn't raise.

### 7. Interested but blocked
- **Goal:** stay useful until the blocker clears.
- **Parts:** acknowledge the blocker as they described it → anything we committed to do in the meantime → a concrete check-in point tied to the blocker (e.g. "when your Q1 budget opens", "once the Salesforce migration is done").
- **Leave out:** pressure or artificial urgency.

### 8. Customer check-in / expansion
- **Goal:** confirm what we'll do for them and any expansion next step.
- **Parts:** what's working (their words) → issues raised, each with owner and timing → expansion interest, if any, and the next step for it.
- **Leave out:** upsell that wasn't raised in the call.

### 9. Exploratory / early
- **Goal:** turn a loose conversation into one clear next step.
- **Parts:** 2–3 bullets on what we learned about their situation → the one thing they seemed most interested in → a single, low-friction ask (a short follow-up call, or a question to answer by email).
- **Leave out:** multiple asks, long recaps.

---

## Always

- **To:** external attendees who spoke or were addressed directly. **CC:** other external attendees and any internal Anatco attendee other than Anat.
- **Subject:** specific to the call (e.g. "Next steps — lead routing trial"), never "Following up" or "Great chatting".
- The meeting-notes tool (Fireflies.ai) may already send attendees the summary and recording; don't repeat that.
- One email per meeting, one primary ask per email.
