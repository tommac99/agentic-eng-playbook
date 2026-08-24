---
name: epic-spec
description: Turn a made bet into an epic issue plus agent-sized sub-issues on GitHub Project 11. Use when a bet has just been decided (via /bet or a jour fixe decision), when the user says "spec this out", "turn this into an epic", "break this bet into issues", or "write the PRD and issues for X". Reads bet/problem/hypothesis docs, writes a lean PRD section, decomposes into tracer-bullet vertical slices, and always shows the decomposition to the human before creating anything.
---

# Epic spec

Turns a decided bet into the two artifacts the team actually works from: a lean
PRD section (the why + what, in `company/product/`) and an epic + sub-issues on
[GitHub Project 11](https://github.com/users/alexdev/projects/11) (the what's
being built, right now, by whom).

## When to use

- A bet just came out of `/bet` or a jour fixe decision and needs to become
  buildable work.
- The user says "spec this", "turn this into an epic", "break this into
  issues", or points at a research-signal / teardown-gap issue and says "let's
  build this."
- There's a clear release-goal fit and a stated appetite already — this skill
  does not re-litigate whether to bet, only how to build it.

## When NOT to use

- No appetite has been set yet → run `/bet` first (Decide stage). Specing
  without an appetite is how epics drift.
- The bet is a stray idea with no shape yet → `/problem-statement` or
  `/hypothesis` first.
- You need user stories or acceptance criteria for something that already has
  an epic → use `/stories` or `/acceptance-criteria` directly, don't re-run
  this.
- More than one epic is already in flight (sub-issues actively worked or
  dispatched) — `.claude/CLAUDE.md` rule 4 caps this at one. Check
  `gh project item-list 11 --owner alexdev --format json` first; if another
  epic has open, undispatched sub-issues, flag it and stop.

## Instructions

1. **Read the bet's evidence.** Follow the doc trail: the bet pack (jour fixe
   pack or `/bet` output), the research-signal or teardown issue it came from,
   and any linked `company/research/` digest. Do not spec from memory — cite
   what you read.

2. **Write the lean PRD section.** Append a dated section to a
   `company/product/YYYY-MM-DD_<slug>-spec.md` file (new file if none exists
   for this bet). Cover:
   - The problem, with the evidence link (issue #, digest date).
   - The release-goal fit — one line, why this over the other candidates.
   - The appetite, copied verbatim from the bet.
   - What's in scope, what's explicitly cut ("considered, out of appetite").
   - **Why, not how.** No component names, no schema, no implementation
     detail — that's the sub-issues' job.

3. **Decompose into sub-issues.** Each sub-issue must be:
   - **Agent-sized** — buildable inside one context window, by one agent, in
     one worktree.
   - **A tracer-bullet vertical slice** — a thin end-to-end path (UI through
     data through response), not a horizontal layer ("build the backend" /
     "build the frontend" is the anti-pattern).
   - **File-disjoint from every other sub-issue in this epic.** If two slices
     would both need to touch the same file, that's one issue, split
     differently — per the collision rules in `company/ops/how-we-work.md`.
   - Ordered only where a real dependency exists (declare with
     `Blocked by: #N`); otherwise mark `Parallel: yes`.

4. **Show the plan before creating anything.** Present the PRD section draft
   and the full sub-issue list (title + one-line scope each) to the human.
   **Do not call `gh issue create` or `gh sub-issue create` until they
   confirm.** This is a hard gate, not a formality — decomposition quality is
   the entire value of this skill, and it's cheap to fix on paper, expensive
   to fix after four agents have started building against a bad split.

5. **Check appetite fits.** If the decomposition's total scope clearly
   exceeds the stated appetite (too many slices, or slices that are each
   themselves multi-session), say so explicitly and propose a cut — don't
   silently create an epic that's going to blow its appetite in week one.

6. **Create the epic issue**, Release set, appetite stated in the
   body, assignee `alexdev`:
   ```bash
   gh issue create --repo acme/northwind --title "Epic: <epic title>" \
     --body "<problem, evidence link, appetite, cut list>" --label epic
   gh issue edit <epic#> --repo acme/northwind --add-assignee alexdev
   ```

7. **Create each sub-issue as a real GitHub sub-issue** of the epic (uses the
   `gh sub-issue` extension, already installed in this repo):
   ```bash
   gh sub-issue create --repo acme/northwind --parent <epic#> \
     --title "<slice title>" --body "<body from step 8>"
   gh issue edit <subissue#> --repo acme/northwind --add-assignee alexdev
   ```

8. **Each sub-issue body** follows the repo's feature issue template shape
   (`.github/ISSUE_TEMPLATE/feature.yml`) plus the metadata block:
   - **What / Why**, linking back to the epic and the evidence.
   - **Acceptance criteria** — if not yet written, delegate to
     `/acceptance-criteria` rather than freehanding it here.
   - **How to test on staging** — plain numbered steps.
   - **Out of scope** — what this slice explicitly does not cover.
   - Metadata block, verbatim format from `company/ops/how-we-work.md`:
     ```
     Schema change: none | <what changes>
     Blocked by: #N        (omit if unblocked)
     Parallel: yes | no
     ```

9. **Move sub-issues to Backlog or Ready** on the board depending on whether
   acceptance criteria are already attached — Ready requires AC, metadata, and
   the bet already made (per the collision rules).

10. **Report back**: epic URL, sub-issue URLs, and which ones landed in Ready
    vs Backlog and why.

## Output contract

- One dated PRD section in `company/product/`, why + what only.
- One epic issue, Release set, appetite stated, assignee `alexdev`.
- N sub-issues, each a real GitHub sub-issue of the epic, each file-disjoint,
  each carrying What/Why/AC/How-to-test/Out-of-scope/metadata block.
- Nothing created without the human confirming the decomposition first.
- A short summary: what was created, what's Ready vs Backlog, any scope cut
  to fit appetite.

## Quality checklist

- [ ] Evidence cited, not invented (issue #, digest date, or teardown link)
- [ ] PRD section is why + what, no implementation detail
- [ ] Every sub-issue is a vertical slice, not a horizontal layer
- [ ] No two sub-issues touch the same files
- [ ] Every sub-issue declares `Schema change:`, `Blocked by:`, `Parallel:`
- [ ] Total decomposition fits the stated appetite, or the cut was surfaced
- [ ] Human confirmed the plan before any `gh issue create` ran
- [ ] Assignee `alexdev` set on epic and every sub-issue

## Example

Bet: Priya's research signal, issue #58, "3 of 4 interviewees didn't
understand the health score." Jour fixe decided this wins over the bank-feed
import gap and the PDF export idea — release goal is "Priya's cohort gets
value weekly," and confused users churn before they get value. Appetite: one
week.

PRD section written to `company/product/2026-08-24_health-score-explainer-spec.md`:
problem (evidence: #58, digest `company/research/2026-08-20_cohort3-digest.md`),
release-goal fit, appetite: 1 week, in scope: tooltip + one explainer screen,
cut: "interactive health-score simulator — considered, out of appetite."

Decomposition shown to Alex before creation:

1. **#61** "Add health-score tooltip to the dashboard number" — `src/` only,
   no schema change, `Parallel: yes`.
2. **#62** "Build health-score explainer screen + route" — `src/` only, no
   schema change, `Blocked by: none`, `Parallel: yes`.
3. **#63** "Link tooltip → explainer screen, add analytics event" —
   `Blocked by: #61, #62`, `Parallel: no` (needs both built first).

Alex confirms. Epic #60 created (Release: R2 Self-Guided), three sub-issues
created under it via `gh sub-issue create --parent 60`, #61 and #62 moved to
Ready (AC attached, unblocked), #63 stays in Backlog until `Blocked by`
clears.
