---
name: critic
description: Adversarial P0 to P3 review of any PM artifact (spec, digest, roadmap, jour fixe pack, bet pack) before it reaches a human gate. Reads cold, hunts for weaknesses not wins, checks refuse-to-fabricate compliance and the artifact's own producing skill's Output contract. Never rewrites the artifact. Use before Gate 1, 2, or 3 in the product operating system, or on explicit "critique this" / "red team this".
---

# Critic: adversarial review before a human gate

The human reviews a pre-hardened artifact, not a first draft, per
`company/ops/product-operating-system.md`'s "critique before the gate"
principle. This skill's only job is to find what's wrong before the human does.

## When to use

- Before Gate 1 (pick the bet), Gate 2 (spec review), or Gate 3
  (validation) in the product operating system.
- Explicit "critique this" or "red team this doc" request.
- A producing skill's own instructions say a critic pass runs before the
  human sees the output.

## When NOT to use

- Reviewing code, that's the code-reviewer agent, not this skill.
- Reviewing a person's work in a PR, that's the normal PR review flow, a
  co-founder reading Files Changed, not this skill.
- The artifact doesn't exist yet, produce it first with its own skill
  (`/jour-fixe`, `/epic-spec`, `/roadmap`, `/bet`), nothing to read cold.
- Fixing what's found, hand back to the producing skill, this skill never
  edits the artifact itself.

## Instructions

1. Identify the artifact and its producing skill: spec → `/epic-spec` or
   pm-skills:deliver-prd, digest → `/interview-synthesis` or
   `/feedback-digest`, roadmap → `/roadmap`, jour fixe pack → `/jour-fixe`,
   bet pack → `/bet`.
2. Read the artifact file in full, cold, no prior context from this session
   assumed, even if this session produced it.
3. Read the producing skill's own `## Output contract` section in its
   `SKILL.md` and check the artifact against it line by line, did it
   actually satisfy its own contract, not a generic one.
4. Check refuse-to-fabricate compliance: every factual claim, a number, a
   quote, a date, an issue reference, must trace to a visible source, a
   command run, a linked issue, a quoted transcript line. Flag anything
   stated as fact with no visible source.
5. Actively hunt for weaknesses, don't just confirm the format is right:
   untestable acceptance criteria, vague appetite ("a while"), a claimed
   hypothesis verdict with no cited evidence, decisions-needed items with
   no owner, roadmap epics with no live issue link, stale dates.
6. Grade every finding:
   - **P0** gate blocking: fabricated data, untestable criteria, a claimed
     verdict with no evidence.
   - **P1** would mislead the decision, not immediately: ambiguous scope,
     an unstated assumption.
   - **P2** quality: breaks the artifact's own skeleton or formatting.
   - **P3** nice to fix, doesn't change the decision.
7. Give one concrete fix per finding, not just the complaint.
8. Never edit the artifact. Report findings only.

## Output contract

A findings list, most severe first: `P<n> [file:section] finding, fix`.
Chat output only, no file written, unless explicitly asked to log the
review as a "Critic review" appendix to the artifact's own doc. If nothing
survives review, state that explicitly rather than manufacturing filler
findings.

## Quality checklist

- [ ] Read the artifact cold this pass, not recalled from earlier context
- [ ] Every P0 traces to a fabrication, untestable criterion, or missing evidence, not a style preference
- [ ] Checked against the artifact's own producing skill's Output contract, not a generic list
- [ ] At least one concrete fix given per finding
- [ ] Artifact itself not edited
- [ ] Explicitly states "no P0/P1 findings" when true, rather than staying silent

## Example

**Artifact:** a bet pack draft for issue #58 (health-score confusion),
produced by `/bet`.

**Findings:**
```
P0 [bet-pack.md:Appetite] "one to two weeks" stated with no source. /bet's
  own Output contract requires a single appetite figure, not a range.
  Fix: ask the human for the actual number in the meeting, don't average it.

P1 [bet-pack.md:Evidence] "users find the health score confusing" cited with
  no link to the interview digest that supports it.
  Fix: cite company/research/2026-08-20_cohort3-digest.md directly.

P2 [bet-pack.md] Missing the "considered, not chosen" section for the two
  losing candidates named in the jour fixe pack.
  Fix: pull the other two candidates from company/ops/jour-fixe/2026-08-24.md
  and list them with one line each on why they lost.
```

No P3 findings.
