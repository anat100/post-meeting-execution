---
title: "Setup & Customization Checklist"
description: (reference)
---

# Setup & Customization Checklist

## 1. Trigger setup

This skill runs from a **meeting-completion trigger** — fired when
Fireflies.ai finishes processing a meeting involving
Anat Gilad. One skill instance per person. Reference trigger
instructions:

```
A meeting involving [Owner] has completed and its notes are ready.

Load and follow the <Post-meeting execution> skill in full.

Context for this run:
- Meeting owner: [Name] ([email])
- Meeting record: [meeting ID / link from the notes tool]
- Internal domain: [yourcompany.com]
```

- [ ] Fireflies.ai is connected and produces summary + full
      transcript + attendees. The transcript is load-bearing — the
      grounding rule, qualification scoring, and next-step distillation
      all read from it.
- [ ] Anat Gilad's LinkedIn account is connected (Step 12).
- [ ] #post-meeting-review exists and the owner actually reads it — this
      is the single surface for every approval.

## 2. Placeholders

- [x] All placeholders filled with test values (see "Configuration
      (test values)" at the top of SKILL.md).
- [x] HubSpot company properties created: `sales_active` (single
      checkbox) and `account_tier` (dropdown: A, B, C, D).
- [x] "GTM Meeting Log" Google Sheet created with columns: event_id,
      event_type, domain, company, meeting_date, external_attendees.
      Its ID is in the SKILL.md configuration.
- [x] Step 6 stage table mapped to the HubSpot "Sales Pipeline" stages.
- [x] First drafts of all three sub-skills written (anatco-followup-scenarios,
      anatco-voice-guide, anatco-lead-scoring). Adapt them before going live:
      anatco-followup-scenarios (your outcome taxonomy + per-category
      email frameworks) and anatco-voice-guide (how the owner
      actually writes — without it every draft sounds like a bot).
- [ ] If you don't track meeting metrics, delete Step 3 rather than
      leaving the "GTM Meeting Log" Google Sheet unfilled.

## 3. Policy decisions

- [ ] **The owner sends the email, always.** The skill drafts; the human
      copies it into their own inbox (preserves CC and threading). If
      you'd rather have the agent send on approval, change Step 10's
      closing rule deliberately — don't drift into it.
- [ ] **Deal creation is a recommendation, never an action.** Even a
      4/4 qualification score only produces a Yes/No question. Confirm
      the override criteria (authority in the room + exceptional fit +
      structural growth pressure) match your motion.
- [x] **Trial stage ownership:** the HubSpot "Sales Pipeline" has no
      trial stage set by a product signup event, so the Step 6 warning
      was replaced with a note. If you add such a stage later, restore
      the rule that meetings must never set it.
- [ ] LinkedIn connection requests (Step 12) default to queued for
      approval. If blank-note requests to actual attendees fit your
      norms, flip them to auto-send — that is how the workflow ran in
      production.
- [ ] Default follow-up reminder of 5 business days when the owner
      doesn't answer the timing question — adjust to your cycle length.

## 4. Behavioral invariants (do not remove)

- Non-blocking errors never skip downstream steps; only a missing
  meeting record aborts the run.
- Deal existence is never evidence of a prior meeting — the 6-month
  recency filter and same-day-deal rule stay, or first-touch metrics
  will be silently wrong.
- Read-after-write verification on every metrics-log write; a false
  success is surfaced, never swallowed.
- The grounding rule: nothing enters the email that wasn't in the call.
- Every contact enriched, new or existing — no title, not done.
- The full email body appears in the Slack post — no truncation, no
  "see task for details."
- The next-step line: hard blockers only, one line, under 150 chars.
- Step 12 runs in its own action, last, regardless of prior failures.
