---
name: problem-statement
description: Frame a problem with evidence links before any solutioning starts. Use when someone proposes a feature, says "we should build X", opens a vague Backlog issue, or wants a problem written up from a research digest, teardown, board issue, or metric. Refuses to proceed without evidence and reframes solution language back to the underlying problem.
---

# Problem statement

A problem statement exists so the team bets on the right thing, not the first
thing someone said out loud. It is written *before* any solution is chosen —
if a solution is already in the sentence, this skill's job is to strip it
back out.

## When to use

- Alex, Priya, or Sam says "we should build X" and nobody has written down
  *why* yet.
- A Backlog issue is a feature request with no evidence attached.
- Kicking off the Decide stage of the loop for a candidate bet — this is
  always step one, before `/hypothesis`.
- Someone wants to turn a research digest finding, a teardown gap, or a
  metric into something the board can act on.

## When NOT to use

- Evidence already exists and is falsifiable — go straight to `/hypothesis`,
  which pulls from this doc.
- No evidence exists yet, only a hunch worth investigating — route to
  `/interview-plan` (Priya) to go get evidence first, don't fabricate a
  problem statement to fill the gap.
- Comparing competitors, not framing a problem — use `/teardown`.
- Already at the RICE/appetite stage — use `/prioritize` or `/bet`.

## Instructions

1. **Demand an evidence link before writing anything.** Acceptable sources:
   a research digest (`company/research/*.md`), a teardown parity matrix
   gap, an existing board issue number, or a named metric (WCU, churn,
   funnel drop-off). "I have a feeling" is not evidence. If none of these
   exist, stop and say so — offer `/interview-plan` instead of inventing a
   problem to justify a solution someone already wants.

2. **Refuse solution language on sight.** If the input is "we need a button
   that does X" or "add a Y feature," do not write that down as the problem.
   Ask "what happens today without it, and what does that cost?" and reframe
   in those terms. The output never contains a proposed solution — that is
   `/hypothesis`'s job, one step later.

3. **Structure the problem in four parts**, each grounded in the evidence
   link from step 1:
   - **Who is affected** — which segment, how many, how do we know
   - **What happens today** — the actual current behavior/experience, not
     the absence of the imagined feature
   - **Cost of the problem** — churn, lost time, support burden, trust —
     quantified if the evidence supports a number, qualitative with a
     verbatim quote otherwise
   - **Why now** — what changed, or what release goal this blocks, that
     makes this worth a bet this cycle rather than sitting in Backlog

4. **Write the output file** — see Output contract.

5. **Hand off explicitly.** End by naming the next step: "Ready for
   `/hypothesis` to turn this into a falsifiable bet," and note the file
   path so the next skill can find it.

## Output contract

A dated file in `company/product/`:

```
company/product/YYYY-MM-DD_problem-<slug>.md
```

Containing:

```markdown
# Problem: <one-line name>

**Date:** YYYY-MM-DD
**Evidence:** <link — digest path, issue #, teardown section, or metric name>

## Who is affected
...

## What happens today
...

## Cost of the problem
...

## Why now
...
```

No solution section. No proposed fix. If solutioning creeps in during
drafting, delete it before writing the file.

## Quality checklist

- [ ] Evidence link is real and checked, not asserted from memory
- [ ] Zero solution language anywhere in the doc (grep for "button", "add a",
      "we need a", "feature that")
- [ ] "Who is affected" names a segment or count, not "users"
- [ ] "Cost" is quantified or carries a verbatim quote — not "it's annoying"
- [ ] "Why now" ties to a release goal or a dated trigger, not vibes
- [ ] File is dated and lands in `company/product/`
- [ ] Next step named explicitly (`/hypothesis`)

## Example

**Input:** Alex says "we should add a tooltip explaining the health score."

**Refusal + evidence check:** that's a solution — pause. What's the
evidence? Priya's latest digest, `company/research/2026-08-15_interview-digest.md`,
has a "So what" item: "3 of 4 interviewees didn't understand the health
score," already filed as Backlog issue #58. That's usable.

**Output** (`company/product/2026-08-18_problem-health-score-comprehension.md`):

```markdown
# Problem: Users don't understand the health score

**Date:** 2026-08-18
**Evidence:** company/research/2026-08-15_interview-digest.md, issue #58

## Who is affected
3 of 4 August interviewees (all pre-launch beta candidates, Priya's cohort).
Small sample, but consistent — every session that surfaced the health score
also surfaced confusion about it.

## What happens today
The dashboard shows a "Health Score" figure with no explanation of what it
is, why it changes week to week, or what the user should do about it. Two
interviewees assumed a drop meant something was already broken; it is a
forward-looking risk indicator.

## Cost of the problem
Verbatim (interviewee 3): "I saw the score drop and just closed the app,
I'll deal with it later." Misreading a forward-looking indicator as a
current failure is the exact kind of anxiety-inducing confusion the product
exists to remove — this is the opposite of the core promise.

## Why now
Current release goal is "Priya's cohort gets value weekly." Confused users
disengage before they ever reach value — this blocks the release goal
directly, not a nice-to-have polish item.
```

Ends with: "Ready for `/hypothesis` — evidence file above."
