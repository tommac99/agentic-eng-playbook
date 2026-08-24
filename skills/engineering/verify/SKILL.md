---
name: verify
description: Walk a Verifying-column issue's acceptance criteria on the real staging URL, recording evidence per criterion, and report pass or fail — never moves the card to Done itself. Use when the user says "verify issue 164", "walk the AC for X", or when /status flags cards sitting in Verifying. Exists because this column was historically skipped; makes verification cheap enough to actually happen.
---

# Verify

Walks a single issue's acceptance criteria against the real staging
deployment, one criterion at a time, and produces the evidence trail that
turns "I think it works" into "I checked it and here's what I saw."

## When to use

- An issue card is sitting in **Verifying** (merged, deployed, not yet
  confirmed against its AC). `scripts/board.sh standup` or `/status` surfaces
  these.
- The user says "verify #N" or "walk the AC for the health-score tooltip issue."
- A `promote-to-staging` PR has merged and its checklist items need real
  confirmation against the live staging URL (see
  `.claude/skills/promote-to-staging/SKILL.md` — that skill deploys the
  batch, this one confirms individual issues within it).

## When NOT to use

- The issue has no acceptance criteria yet — that's a specing gap, not a
  verify gap. Send it back through `/acceptance-criteria` first; don't
  improvise criteria mid-verify just to have something to check.
- The issue isn't actually deployed to staging yet (still In Review or the
  promotion PR hasn't merged) — verifying against `development` locally isn't
  what this skill is for; wait for the real staging URL.
- You're deciding whether to merge a promotion PR — that's
  `promote-to-staging`'s "Merging" section, a different gate than this one.
- Moving a card to Done — this skill never does that. It reports pass/fail;
  a human moves the card.

## Instructions

1. **Fetch the issue and confirm it's actually in Verifying**:
   ```bash
   gh issue view <N> --repo alexdev/northwind --json title,body,state,number
   ```
   If it's not in Verifying on the board, say so and ask whether to continue
   anyway (e.g. re-verifying a Done issue after a regression report is a
   legitimate reason).

2. **Extract the criteria.** Pull both the "Acceptance criteria" section and
   the "How to test on staging" section from the body — the AC is what's
   being checked, the staging steps are how to check it. If either is missing
   or clearly stale relative to what actually shipped (compare against the
   merged PR's diff via `gh pr view <PR#> --json files`), stop and flag it
   rather than inventing steps.

3. **Walk each criterion, one at a time.** For each:
   - State the check in plain language before performing it.
   - Perform it against the **real staging URL** — not localhost, not
     `development`'s preview. If browser automation/preview tooling is
     available, use it; otherwise, ask the human to perform the step and
     report what they saw.
   - Record the evidence: what was actually observed (exact text, actual
     behavior, screenshot path if one was captured), not a restatement of the
     criterion as if it passed. "Tooltip appeared with German copy matching
     the spec" is evidence; "works as expected" is not.
   - Mark it pass or fail immediately — don't batch judgment to the end,
     since later criteria can shed light on earlier ambiguous ones.

4. **On any failure:**
   - Comment on the issue with exactly which criteria failed and the evidence
     for each (what was expected, what was actually seen).
   - Confirm with the human before moving the card — then move it back to
     **In Progress** (not Backlog; the work exists, it just needs a fix):
     ```bash
     gh issue comment <N> --repo alexdev/northwind --body "<failing criteria + evidence>"
     ```
   - Do not close the issue, do not touch the PR, do not attempt the fix
     yourself unless separately asked — this skill's job stops at reporting.

5. **If everything passes:**
   - Comment on the issue with the full evidence summary, criterion by
     criterion.
   - Tell the human it's ready for them to move the card to **Done** — do not
     move it yourself:
     ```bash
     gh issue comment <N> --repo alexdev/northwind --body "<pass evidence summary>"
     ```

6. **If this issue closes out its epic** (check `Parent issue` field / whether
   it's the last open sub-issue), check whether the epic's originating
   hypothesis or bet doc (in `company/product/`) has a verdict section — if
   so, note the outcome there (or flag for `/retro`, which owns the fuller
   epic-close writeup). Don't silently leave a falsified-or-confirmed bet
   undocumented.

## Output contract

- One evidence entry per acceptance criterion: the check, what was observed,
  pass or fail.
- A posted GitHub comment on the issue with the full evidence trail —
  chat-only output doesn't count, the issue is the durable record.
- Explicit statement of what the human needs to do next (move to In Progress
  on failure, move to Done on pass) — this skill states the recommendation,
  never executes the board move.
- If this closes an epic, a note on whether the hypothesis doc needs a
  verdict update.

## Quality checklist

- [ ] Checked against the real staging URL, not localhost or a PR preview
- [ ] Every criterion in the issue body was walked, none skipped
- [ ] Evidence recorded is observation, not a restatement of the criterion
- [ ] At least the negative/boundary cases from AC were actually attempted
- [ ] Evidence posted as a GitHub comment on the issue, not left in chat only
- [ ] Card was NOT moved to Done by this skill under any circumstance
- [ ] Failures include specific evidence, not just "criterion 3 failed"

## Example

Issue #61, "Add health-score tooltip to the dashboard number," sitting in
Verifying after its promotion PR merged.

```bash
gh issue view 61 --repo alexdev/northwind --json title,body,state,number
```

Criteria walked on the real staging URL:

1. **Check:** hover the health score, tooltip appears within 300ms.
   **Evidence:** tooltip appeared in ~150ms with copy "Your health score is
   based on..." — matches spec. **Pass.**
2. **Check:** click outside closes the tooltip.
   **Evidence:** confirmed, tooltip closed on outside click. **Pass.**
3. **Check:** screen reader announces content via `aria-describedby`.
   **Evidence:** inspected DOM, tooltip has `role="tooltip"` but the trigger
   element has no `aria-describedby` attribute — VoiceOver does not announce
   it on focus. **Fail.**
4. **Check (negative):** health score still loading, hover shows no tooltip,
   no error. **Evidence:** loading skeleton shown correctly, no console
   error. **Pass.**

Result: 3 pass, 1 fail. Comment posted on #61 with all four entries.
Recommendation given to Alex: move #61 back to In Progress for the missing
`aria-describedby` wiring — his call to confirm, not auto-applied.
