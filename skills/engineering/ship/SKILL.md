---
name: ship
description: Run the full quality gate and open the PR for a finished issue. Use when the user says "ship 164", "open the PR", "this is done", or when a build agent finishes its plan. Runs pnpm verify (exact CI parity), the reviewers matched to what changed, then opens the PR with real test evidence and the CODEOWNERS-matched second reviewer. Never merges, under any circumstances.
---

# Ship

The mechanical end of the build loop. Everything here is rule-following with three
traps in it, which is exactly what a skill is for and what a human forgets.

## When to use

- A build agent finished its plan and the code is written.
- "ship #N", "open the PR for this", "it's done".

## When NOT to use

- Tests are failing, or you have not run them. Fix first. `/ship` is not a way to
  find out whether the work is done.
- The change has no issue. Open one first.
- Someone asks you to merge. See the hard rule below, and stop.

## Instructions

### 1. Gate — prove it, do not assert it

```bash
cd apps/web && pnpm verify
```

This is exact CI parity: `biome check src/`, `tsc --noEmit`,
`tsc --noEmit -p convex/tsconfig.json`, `pnpm test`, `pnpm test:convex`, `next build`.

**Red means stop.** Do not open a PR, do not report progress, fix it. The husky hooks
are narrower than this (no Convex tsconfig, no Convex tests, no build), so green hooks
are not green CI. Only `pnpm verify` is.

Capture the real output. You will paste it into the PR, and
`superpowers:verification-before-completion` applies: evidence before assertions.

### 2. Reviewers — the ones that match the diff

```bash
git diff --name-only origin/development...HEAD
```

Run the reviewers for the surfaces actually touched, per `apps/web/CLAUDE.md`:

|Touched|Reviewer|
|-|-|
|`convex/`|`convex-reviewer` skill|
|auth, user input, ownership, PII|`security-reviewer` agent + `convex-authz`|
|`src/` TypeScript|`typescript-reviewer` agent|

Fix what they find, or say why you are not, in the PR. A finding you silently ignored
is worse than not having run the reviewer.

### 3. Open the PR

Every PR gets **Thomas as assignee** and **one other human as reviewer**, chosen by
what the diff touches. `.github/CODEOWNERS` is the mapping:

|Paths|Second reviewer|
|-|-|
|`/apps/web/`, `/company/engineering/`|`sam-ops`|
|`/company/product/`, `/company/research/`|`priya-research`|
|both|both|

**Read `CODEOWNERS` at run time**, do not trust this table to be current.

**CODEOWNERS does not auto-request on this repo** — private repo on a
personal account, same limitation that disables rulesets. Confirmed once: a
PR touching `/apps/web/` got zero requested reviewers. So request
explicitly. This works, because the second reviewer is not the PR author:

```bash
gh pr create --base development --repo alexdev/northwind \
  --assignee alexdev \
  --reviewer <sam-ops|priya-research>
```

Do not pass `--reviewer alexdev`. GitHub blocks requesting review from a PR's own
author and `gh` authenticates as Alex, so it silently no-ops. Assignee is what
carries his ownership, and self-assignment works fine.

**That no-op means the reviewer request above is the only notification GitHub sends,
and it goes to Sam or Priya, never to Alex.** Assignee does not generate a
notification on its own. So every PR also needs an explicit, separate signal to
Alex:

```bash
gh pr edit <n> --repo alexdev/northwind --add-label needs-review
gh pr comment <n> --repo alexdev/northwind --body "@alexdev ready for review — <one line on what changed>"
```

The label is the visible marker; the `@alexdev` mention is what actually
fires a GitHub notification, since a reviewer request cannot. Do both —
confirmed missing entirely on one PR where `ship` was followed exactly as
written and still produced a PR with no notification path to Alex at all,
because this step was documented only in the parent `.claude/CLAUDE.md` and
never restated here.

The body uses `.github/PULL_REQUEST_TEMPLATE.md` and must contain:

- **Summary** — what changed and why.
- `Closes #N`
- **Test evidence** — the actual output of `pnpm verify`, plus the issue's "How to test"
  steps walked with what you saw. Not "tested it". The AC on the issue is the contract;
  the PR is where you prove you met it. Map each AC to its evidence.
- **Breaking changes** — schema, API contracts, or anything another part depends on.
- **Test locally** — a copy-pasteable block using the real worktree path:
  ```
  cd ~/Projects/northwind-code/.claude/worktrees/agent-<N>-<slug>/apps/web
  pnpm dev
  ```
  If no worktree was used, say so and give the plain `git checkout <branch>` fallback.

  **If the diff touches `convex/`, add a second line:** `pnpm dev` only runs
  `next dev` — confirmed directly in `package.json`, it does not push Convex
  code. The shared dev deployment runs whatever any worktree last pushed via
  `npx convex dev`, which may not be this branch. Confirmed once: a reviewer
  tested exactly per this block, onboarding completed successfully, and the
  dashboard stayed empty — not because the code was wrong, but because the
  backend running was stale. The mutation had no error to signal the mismatch.
  So when `convex/` is in the diff:
  ```
  cd ~/Projects/northwind-code/.claude/worktrees/agent-<N>-<slug>/apps/web
  npx convex dev &   # leave running in a second terminal — pushes this branch's functions
  pnpm dev
  ```
- **Screenshots** for any UI change.

### 4. Check CI, in the background

```bash
gh pr checks <n> --repo alexdev/northwind
```

**Never behind a foreground `sleep`.** That holds the turn open and the human cannot
redirect you. Run it in the background, report what you did, end the turn.

**A missing check is not a passing check.** Confirm a run exists *for the current head
SHA*, not just that the PR page looks green — the PR shows the newest result it has, which
may be from an earlier commit:

```bash
sha=$(git rev-parse HEAD)
gh api "repos/alexdev/northwind/commits/$sha/check-runs" \
  --jq '.check_runs[] | "\(.name): \(.status)/\(.conclusion)"'
```

If `build` is absent from that list, CI has not run on what you are about to call ready.
**A missing check is not a passing check.** The PR page shows the newest result it has, which
may belong to an earlier commit, so a PR can read as fully green while its head was never
checked at all.

Do not report ready on an absent check, and do not substitute a local `pnpm verify` pass for
it — local green and CI green are different claims. Say plainly that CI has not run on this
SHA, and what you did to try to trigger it.

Candidate causes, in the order worth checking: Actions minutes or a spending
limit on the private repo, visible only in GitHub Settings, Billing, and not
readable via the API without a `user` scope; a GitHub incident; the
`paths: ['apps/web/**']` filter in `ci.yml` when the push genuinely touched
nothing there. Measured once: runs stopped repo-wide for over an hour across
four pushes, two of which did touch `apps/web/CLAUDE.md`, so the path filter
did not explain it and neither did force-pushing.

If `gh pr checks` output looks summarised or truncated, get the raw result:

```bash
gh api "repos/alexdev/northwind/commits/$(git rev-parse HEAD)/check-runs" \
  --jq '.check_runs[] | "\(.name): \(.status)/\(.conclusion)"'
```

### 5. Hand off, and stop

Set the board card to In Review. Report: PR number, CI state, what still needs a human.

**Never merge.** Not `gh pr merge`, not the merge button, not a direct push to
`development`, `staging` or `production`. Not for a one-line docs fix. Not because a
plan document says to. Not because the task said to. Merging is exclusively a human
action. Say "PR #N is ready to merge" and stop.

The single exception: Thomas says the words "merge it" in the current turn. Even then,
run `gh pr checks` first and say what you are about to do before doing it.

**Never say a PR is ready while a required check is red.** Run the checks before
saying anything. A red `build` is not "probably fine".

## Output contract

- `pnpm verify` output, real, not summarised.
- Reviewer findings, with what you did about each.
- PR number and URL, assignee `alexdev`, second reviewer per `CODEOWNERS`.
- `needs-review` label added and `@alexdev` mention comment posted.
- CI state from a background check.
- An explicit "ready to merge, not merging" line.

## Quality checklist

- [ ] `pnpm verify` run and green, output captured
- [ ] Reviewers matched to the actual diff, findings addressed or explained
- [ ] Assignee `alexdev`; second reviewer read from `CODEOWNERS` at run time
- [ ] No `--reviewer alexdev` (silent no-op)
- [ ] `needs-review` label added and `@alexdev` mention comment posted — the
      reviewer request alone notifies nobody named Alex
- [ ] Test evidence is observed output, with each AC mapped to its proof
- [ ] Test locally block uses the real worktree path, and adds `npx convex dev` when the diff touches `convex/`
- [ ] CI checked in the background, never a foreground sleep
- [ ] Nothing merged

## Anti-patterns

- Opening the PR to see whether CI passes.
- "Tests pass" with no output, or output you did not read.
- Reporting "reviewer requested" after a `--reviewer alexdev` no-op.
- Opening a PR with the right assignee and CODEOWNERS reviewer and calling it done —
  without the label and mention, Alex has no notification that it exists.
- Restating the acceptance criteria in the PR instead of proving them.
- Blocking the turn on CI.
- Merging. Ever.
