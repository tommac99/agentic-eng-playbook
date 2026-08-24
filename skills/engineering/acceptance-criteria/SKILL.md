---
name: acceptance-criteria
description: Write checkable acceptance criteria for a user story or sub-issue — Given/When/Then where behavior fits that shape, plain checklist where it doesn't — plus the "How to test on staging" block the repo's issue template requires. Use when the user says "write AC for X", "what's the acceptance criteria", or during epic decomposition once stories exist. Every criterion must be verifiable by a non-author, since /verify tests exactly this later.
---

# Acceptance criteria

Writes the contract a sub-issue is built and reviewed against: acceptance
criteria an agent can build to without a clarifying question, a reviewer can
check a PR against, and `/verify` can walk on staging without the original
author in the room.

## When to use

- A story from `/stories`, or a sub-issue drafted by `/epic-spec`, has a
  What/Why but no checkable criteria yet.
- The user asks directly: "write AC for #161", "what's the acceptance
  criteria here."
- A sub-issue is about to move to Ready and its AC section is thin or missing
  — Ready requires AC per the collision rules in `company/ops/how-we-work.md`.

## When NOT to use

- No story or problem statement exists yet — AC without a "why" behind it
  turns into a spec-by-checklist exercise disconnected from the actual bet.
  Run `/stories` (or at minimum have a clear What/Why) first.
- The issue is already in Verifying with AC attached — that's `/verify`'s
  job, walking the existing criteria, not rewriting them.
- Writing criteria for something with no staging-testable surface (a pure
  internal refactor with no user-visible behavior) — say so, and note that
  this issue is better verified by tests + code review than by `/verify`.

## Instructions

1. **Start from the story or the sub-issue's What/Why.** Don't write criteria
   from the title alone — vague input produces vague, unverifiable criteria.

2. **Pick the shape per criterion, not per issue.** A single issue usually
   mixes both:
   - **Given/When/Then** for behavior with a clear trigger and observable
     result (state changes, user actions, conditional logic).
   - **Plain checklist item** for static facts that aren't really "behavior"
     (a copy string exists, a field is present, a link points somewhere) —
     forcing these into Given/When/Then produces stilted, harder-to-scan
     criteria for no benefit.

3. **Every criterion must be checkable by a non-author on staging.** This is
   the hard constraint the whole skill exists to enforce — it's the same
   contract `/verify` walks later, by a human who did not write the code. Test
   each criterion against: "could Priya or Sam, who didn't build this,
   confirm this is true or false just by using the app?" If the answer
   requires reading code, database state, or internal logs, rewrite the
   criterion around an observable, on-screen effect instead.

4. **Include negative cases**, not just the happy path. At minimum: what
   happens on invalid input, what happens when the precondition isn't met,
   what happens at a boundary (empty state, zero, max). A story with only
   positive criteria is half-specified — it tells an agent what to build but
   not what to guard against.

5. **Write the "How to test on staging" block** to match
   `.github/ISSUE_TEMPLATE/feature.yml`'s shape exactly — numbered, concrete
   steps, not a restatement of the criteria:
   ```
   1. Go to ...
   2. Do ...
   3. Expect ...
   ```
   Each numbered step should map to one or more criteria above it, so a
   reader can trace "criterion 3 ↔ step 2's expectation" without guessing.

6. **Output goes straight into the sub-issue body** (if editing an existing
   issue) or is handed back for `/epic-spec` to embed. Don't leave it
   floating in chat only — the issue body is the durable record.

## Output contract

- A Given/When/Then or checklist block per criterion, grouped under the
  issue/story it belongs to.
- At least one negative or boundary case per issue, not just happy path.
- A "How to test on staging" numbered block, matching the issue template's
  expected shape.
- Every criterion phrased so a non-author can verify it by using the app —
  no criterion that requires reading source or database state to check.

## Quality checklist

- [ ] Every criterion traces to the story's What/Why, nothing invented
- [ ] Given/When/Then used where behavior has a trigger; plain checklist
      elsewhere — not forced into one shape uniformly
- [ ] At least one negative/boundary case included
- [ ] Every criterion is checkable by someone who didn't write the code
- [ ] "How to test on staging" block present, numbered, matches issue
      template shape
- [ ] No criterion requires reading logs, database rows, or source to verify

## Example

Sub-issue #61, "Add health-score tooltip to the dashboard number" (from the
`/epic-spec` and `/stories` worked examples).

**Acceptance criteria:**

- Given a logged-in user on the dashboard, when they hover (desktop) or tap
  (mobile) the health score, then a tooltip appears within 300ms showing a
  one-sentence plain-language explanation of what the score means.
- Given the tooltip is open, when the user clicks/taps outside it, then it
  closes.
- [ ] Tooltip copy is reviewed and approved by Priya (research/persona
  accuracy) before merge — not just Alex.
- Given a screen reader user, when they focus the health score, then the
  tooltip content is announced (accessible via `aria-describedby`, not
  hover-only).
- **Negative case:** given the health score is still loading (undefined),
  when the user hovers, then no tooltip appears and no error is thrown —
  the number shows a loading skeleton instead.

**How to test on staging:**
```
1. Go to /dashboard, logged in as any test account.
2. Hover the health score (desktop) or tap it (mobile).
3. Expect a tooltip within ~300ms with a one-sentence explanation, in the
   active locale (EN default).
4. Click outside the tooltip.
5. Expect it to close.
```
