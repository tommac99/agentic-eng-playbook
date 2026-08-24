---
name: retro
description: Runs the 15-minute epic-close micro-retro, pulls the epic's sub-issue and PR history, appetite versus actual duration, and verify results, asks three questions (was the bet right, was the appetite right, what pattern should change), runs the epic-close sweep checklist, and appends one section to company/ops/retros.md. Use when an epic's last sub-issue hits Done.
---

# Retro: epic close micro retro

Runs once, at the moment an epic's last sub-issue lands in Done. Fifteen
minutes, three questions, one checklist, one appended section. Never a
post-mortem essay, the product-operating-system doc is explicit that this is
a sweep, not a project.

## When to use

- An epic's last sub-issue moves to Done.
- User says "let's retro epic #N" or "close out this epic".

## When NOT to use

- Mid-epic, one specific friction point, not a full close, use `/lessons`
  directly.
- Deciding to abandon an epic before it's done, that's a pivot call
  (pm-skills:iterate-pivot-decision), not this.
- Daily orientation, use `scripts/board.sh standup`.
- Just recording what happened in an ordinary session with no epic closing, use
  `/save-session` instead.
- The pattern found here becomes a lesson, hand off to `/lessons`, don't
  write it into brain/improvements.md yourself.

## Instructions

1. Confirm the epic number. Verify every sub-issue is actually Done, don't
   assume: `gh issue view <epic#> --json subIssuesSummary,title,body,createdAt`.
   If any sub-issue isn't closed, stop and say so, this isn't ready to retro.
2. Pull history:
   - Appetite: grep `Appetite:` in the epic body.
   - Start: the epic issue's `createdAt`.
   - Close: the last sub-issue's `closedAt` (or its closing PR's merge date).
   - Elapsed vs appetite: a real date subtraction, not an estimate.
3. Pull the PRs that closed each sub-issue:
   `gh pr list --repo alexdev/northwind --search "linked:<subissue#>"`,
   or grep merged PR bodies for `Closes #<subissue#>`.
4. Pull verify evidence: if a `/verify` walkthrough or Verifying-column
   comment exists for each sub-issue, cite it. If none exists, say so, don't
   invent a verification that didn't happen.
5. Ask three questions in chat, one at a time, wait for the human's answer
   to each before moving on:
   a. **Was the bet right?** Hypothesis verdict, supported, refuted, or
      inconclusive, referencing the stage-2 hypothesis doc in
      `company/product/` if one exists for this epic.
   b. **Was the appetite right?** Actual duration versus stated appetite,
      over, under, or on target, and by how much.
   c. **What pattern should change?** Process, not blame, this is the input
      `/lessons` will use next.
6. Run the epic-close sweep checklist verbatim from
   `company/ops/product-operating-system.md`, checking each box only if
   actually verified this run:
   - [ ] Retro written, lessons appended
   - [ ] Stage-2 hypothesis doc updated with the verdict
   - [ ] Every doc the epic produced has its Status header set
   - [ ] Stale issues closed
   - [ ] Worktrees deleted, `git worktree list` names nothing unexplainable
   - [ ] New signals fed back to stage 1
7. Append `## Epic #<N>: <title> (closed YYYY-MM-DD)` to
   `company/ops/retros.md` (create the file with a one-line header if it
   doesn't exist yet) with the three answers and the sweep results.
8. Hand the "what pattern should change" answer to `/lessons`, don't
   duplicate the write into brain/improvements.md from here.

## Output contract

One new section in `company/ops/retros.md`: epic title, close date, appetite
vs actual, three answers, sweep checklist with real check states. No other
file is edited directly, hypothesis doc and Status-header updates are done
as part of running the sweep in step 6, not skipped and merely reported.

## Quality checklist

- [ ] All sub-issues confirmed Done via a live query before starting
- [ ] Appetite vs actual is a computed date difference, not a guess
- [ ] All three questions answered by the human this session, not invented
- [ ] Sweep checklist boxes reflect what was actually verified, not assumed complete
- [ ] Pattern finding handed to `/lessons`, not duplicated inline

## Example

**Epic #40, "Onboarding health-score clarity", appetite one week.**

- Created 2026-08-17, last sub-issue closed 2026-08-27, 10 days vs 7 day
  appetite, two days over.
- Sub-issues: #58 (tooltip copy), #61 (explainer screen), #62 (analytics
  event), all Done, closed via PR #63, #65, #66.

**Answers:**
a. Bet right: supported, follow-up interview notes
   (`company/research/2026-08-26_beta-check.md`) show 4 of 5 re-tested
   users correctly explained the health score afterward.
b. Appetite: slightly under, the explainer screen needed one extra design
   pass Alex hadn't scoped.
c. Pattern: appetite for anything touching copy should assume one design
   review round trip by default.

**Sweep:** all six boxes checked, worktrees for #58/#61/#62 confirmed
removed via `git worktree list`.
