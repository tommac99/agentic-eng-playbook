---
name: build
description: Take an epic or issue from the board to an approved spec and plan, then run the build agent to a PR. Use when the user says "build 164", "work on 139", "pick up the next one", "lets do epic 161", or "what is next in epic 161". Reads the issue, its epic, its Release and the PM artifacts behind it, runs the intake gates, routes to the right engineering skills, and stops twice for human approval, once on the spec and once on the plan, before any code is written. Never skips a gate to save time.
---

# Build

Turns a board item into merged-ready code without the human losing control of the
"how". The engineering conventions live in `apps/web/CLAUDE.md`; this skill is the
thing that actually walks them, in order, with the human in the loop at the two
points where a wrong call is expensive.

## When to use

- "build #164", "work on #139", "pick up the next one"
- "let's do epic #161", "what's next in the R1 epic"
- Any time work is about to start on a board item.

## When NOT to use

- The work has no issue. Create one first, that is rule 1 in `.claude/CLAUDE.md`.
- The issue is a research or product question, not code. Use `problem-statement`,
  `interview-plan` or `teardown` instead.
- Something is broken and you are diagnosing it. Use `superpowers:systematic-debugging`
  first, then come back here with an issue once you know what needs building.
- The user asked a question about the codebase. Just answer it.

## Instructions

### Phase 0 — Resolve the target

If given an **epic**, do not pick an issue silently:

```bash
gh api graphql -f query='
{ repository(owner:"alexdev", name:"northwind") {
    issue(number:<EPIC>) { title
      subIssues(first:50) { nodes { number title state } } } } }'
```

Read every open sub-issue's body for `Blocked by`, `Schema change:` and `Appetite`.
Derive the dependency order, then show the human:

- the chain, as a diagram, with what is done, what is unblocked, what is waiting
- which issue you recommend next and why
- anything whose `Blocked by` names an issue that has actually landed, that is
  stale metadata and worth fixing while you are here

Let the human pick. Do not start the recommended one on your own initiative.

If given an **issue**, go straight to Phase 1.

### Phase 1 — Intake

Pull the whole chain, never work from the title:

```bash
gh issue view <N> --repo alexdev/northwind --json title,body,labels,parent,projectItems
```

Extract and restate: **acceptance criteria**, **How to test on staging**,
**`Schema change:`**, **`Blocked by`**, **parent epic**, **Release**, **Appetite**.

Read up the chain: the epic's intent, the Release goal in
`company/product/roadmap.yaml`, and any `company/product/` doc the issue references.
A sub-issue that satisfies its own AC but does not move the epic is a miss.

**Run the four gates. Each one stops the skill.**

1. **AC gate.** Missing, vague, or not checkable → stop, route to `acceptance-criteria`.
   Do not invent AC. If you would need to ask a clarifying question mid-build, the
   issue is not Ready no matter what the board says.
2. **Schema conflict gate.** If the issue declares `Schema change:`:
   ```bash
   gh issue list --repo alexdev/northwind --state open \
     --search "Schema change: in:body" --json number,title,body
   ```
   Another in-flight issue also changing schema → blocked, say so, pick something else.
   One shared Convex dev deployment, they will corrupt each other.
3. **Blocked-by gate.** `Blocked by #N` and #N not merged → stop. **Verify it, do not
   trust it** — check whether the blocker's content actually landed in `development`
   (fields present in `convex/schema.ts`, files present, `git cherry`). Stale
   blocked-by lines are common and have hidden ready work before.
4. **Epic gate.** Belongs to a different epic than the one in flight → flag before starting.

### Phase 2 — Route

Classify by surface, load only those skills, per the routing table in
`apps/web/CLAUDE.md`. Read that table, do not reproduce it from memory — it changes.
Respect its blacklist. Loading every skill is as bad as loading none.

### Phase 3 — Explore, then GATE 1: the spec

Read the actual code first. A spec that cannot name real files is a guess.

Then present the spec and **stop for approval**. It must cover:

- **Behaviour** — what the user can do after this ships that they cannot now, one sentence.
- **Files** — the exact paths that will change. Named, not described.
- **UI composition** — walk the ladder in `apps/web/CLAUDE.md` **out loud**. Name the
  primitive you land on and why: "shadcn `Stepper` plus the existing `FormPanel`, not a
  new component, because the ladder stops at step 1." This is the decision the human
  most needs to see and most often wants to redirect.
- **Data** — tables read and written. Does it trigger a derived-cache recompute?
- **Invariants at risk** — which Critical Rules this could violate. Money stored in
  the smallest currency unit, a status field set only by mutations, no hard deletes.
- **AC mapping** — every acceptance criterion, mapped to how it will be proven. A test,
  a manual step, a screenshot. An AC with no proof route is a gap in the issue; raise it.
- **Best practice, stated** — where there was a choice, say which route you are taking,
  which you rejected, and why. "Rejected X because Y" is the point of this gate.
- **Out of scope** — what you are deliberately not doing.

If the spec reveals the issue is bigger than one agent-sized chunk, say so. Splitting
into sub-issues beats a 40-file PR nobody can review.

**Wait for approval. Do not continue on silence or on a vague "sounds good" if the
human has not addressed the UI composition call.**

### Phase 4 — GATE 2: the plan

Built from the approved spec, not from the issue. Ordered steps, each with its own
verification:

```
1. <step> → verify: <the command or check that proves it>
2. <step> → verify: <...>
```

Every step must be independently checkable. "Implement the component" is not a step,
it is the whole task. Invoke `superpowers:writing-plans` when it exceeds about five steps
or crosses more than one surface.

State what "done" looks like, and name the risk that most likely derails it.

**Wait for approval.**

### Phase 5 — Persist, then dispatch

On approval, post the approved spec and plan as a comment on the issue, so it outlives
this session and is visible on the board:

```bash
gh issue comment <N> --repo alexdev/northwind --body "..."
```

Set the board card to In Progress. Create the worktree:

```bash
git fetch origin
git worktree add .claude/worktrees/agent-<N>-<slug> -b feature/<N>-<slug> origin/development
ln -sf "$(git rev-parse --show-toplevel)/apps/web/.env.local" \
       .claude/worktrees/agent-<N>-<slug>/apps/web/.env.local
```

Verify the base before any commit — the left number must be `0`:

```bash
git rev-list --left-right --count origin/development...feature/<N>-<slug>
```

Then dispatch the build agent with the approved plan. It runs unattended to completion:
TDD per `superpowers:test-driven-development`, coverage targets from `apps/web/CLAUDE.md`
(critical business logic 100%), then `pnpm verify`, then it **invokes the `ship` skill**.

**The agent must invoke `ship` via the Skill tool. It must not reproduce ship's steps
from memory, and neither must you when writing its prompt.** Say "invoke the `ship`
skill" in the dispatch prompt and stop there — do not helpfully paste in the `gh pr
create` line, the reviewer table, or the PR body checklist.

The reason is specific, not stylistic. On one issue, this instruction read
as a description of a stage rather than a call, so the orchestrator restated
ship's steps inline. The PR mechanics it remembered came out correct, but
**ship step 2 — running the reviewers matched to the diff — was silently
dropped**, and nothing reported the omission because the orchestrator was
following its own reconstruction rather than the skill. That PR touched both
backend and frontend TypeScript and got neither reviewer. A paraphrased
skill loses exactly the steps the paraphraser already forgot, which is the
one thing skills exist to prevent.

The agent does not renegotiate the plan. If it hits something the plan did not
anticipate and the answer is not obvious, it stops and reports rather than improvising
— that is the whole reason the gates exist.

## Output contract

- Epic entry: dependency chain, recommended next issue, human picks.
- Gate 1: spec, with the UI composition call made explicit. Approval required.
- Gate 2: plan with per-step verification. Approval required.
- Then: issue comment with the approved spec and plan, worktree, dispatched agent, PR.
- Any gate failure: stop, say which gate and why, name the skill that fixes it.

## Quality checklist

- [ ] Read the epic and the Release goal, not just the issue
- [ ] All four intake gates run, blocked-by verified against `development` not trusted
- [ ] Routing table read from `apps/web/CLAUDE.md`, blacklist respected
- [ ] Real code read before the spec was written; spec names real files
- [ ] UI ladder walked out loud, rejected alternatives stated
- [ ] Every AC mapped to a proof route
- [ ] Stopped for approval twice, and actually waited
- [ ] Approved spec and plan posted to the issue before the agent started
- [ ] Dispatch prompt tells the agent to *invoke* `ship`, and does not restate its steps
- [ ] Worktree base verified `0`, `.env.local` symlinked not copied

## Anti-patterns

- Starting to code during Phase 3 because the change "looks small".
- Presenting a plan without the spec, so the UI decision never surfaces.
- Treating a `Blocked by` line as true without checking `development`.
- Inventing acceptance criteria for an issue that lacks them.
- Loading every skill in the routing table instead of the matched rows.
- Continuing past a gate because the human said something that was not approval.
- Inlining a skill's steps into a prompt instead of invoking the skill. Whatever you
  forget in the paraphrase is gone silently, and `ship`'s reviewer stage is the step
  that goes missing first.
