---
name: hypothesis
description: Turn a framed problem into a falsifiable bet — "we believe X for Y will Z, we'll know within timeframe by signal." Use after a problem-statement doc exists and someone wants to propose what to build. Forces a measurable signal and a stated kill condition, refuses vague success language like "users like it."
---

# Hypothesis

A hypothesis is the one sentence that decides whether the bet worked. If it
can't be proven wrong, it isn't a hypothesis, it's a hope. This skill exists
to make every bet falsifiable before it reaches `/prioritize` or `/bet`.

## When to use

- A `/problem-statement` doc exists and it's time to propose what to build
  and how we'll know it worked.
- Someone hands you a solution idea directly ("let's add a tooltip") and a
  problem-statement already backs it, or can be created in the same pass.
- Preparing a candidate for `/prioritize` or the weekly `/bet` pack — every
  candidate in those needs a hypothesis, not just a title.

## When NOT to use

- No problem-statement exists yet — run `/problem-statement` first, this
  skill pulls its "why" straight from that doc rather than re-deriving it.
- Ready to score across multiple hypotheses — that's `/prioritize`.
- The bet is already decided and appetite is being set at jour fixe —
  that's `/bet`, which assembles hypotheses, doesn't write new ones.
- Writing acceptance criteria for an already-bet epic — that's
  `/acceptance-criteria`, a different altitude (checkable steps, not a
  falsifiable outcome statement).

## Instructions

1. **Locate the source problem-statement.** Read it in full — who's
   affected, what happens today, the cost, why now. The hypothesis's "who"
   and "outcome" must trace back to that doc's "who is affected" and "cost."
   If no problem-statement exists, stop and say so — offer to run
   `/problem-statement` first rather than writing a hypothesis on
   unexamined ground.

2. **Fill the exact template**, no paraphrasing it away:

   > We believe **[building X]** for **[who]** will **[outcome]**.
   > We'll know we're right when **[measurable signal]** within
   > **[timeframe]**.

3. **Force a real signal.** Reject anything that isn't observable and
   falsifiable:
   - Rejected: "users like it," "feels better," "improves clarity"
   - Accepted: "feature-related support questions drop below 1/week," "≥60% of
     the cohort views the explainer screen at least once," "next digest
     shows zero comprehension complaints in this area"
   If Priya or Sam can't name a way to observe the signal (a metric, a
   digest theme, a survey question), the hypothesis isn't ready — push back
   before writing it down.

4. **State the kill condition explicitly** — what result means the bet was
   wrong and should not be repeated or extended. This is not "we didn't
   finish in time" (that's an appetite question, logged separately per
   `company/ops/how-we-work.md`'s betting section) — it's "the signal came
   back negative, this is the wrong bet."

5. **Write the output** — append to the source problem-statement doc under
   a `## Hypothesis` heading, or as a standalone dated doc if no
   problem-statement doc exists to append to (rare — flag it, don't silently
   default to standalone).

## Output contract

Appended to the problem-statement doc (preferred), or a standalone file:

```
company/product/YYYY-MM-DD_problem-<slug>.md   (append ## Hypothesis)
```

```markdown
## Hypothesis

We believe **<building X>** for **<who>** will **<outcome>**.
We'll know we're right when **<measurable signal>** within **<timeframe>**.

**Kill condition:** <what result kills the bet — not "ran out of time">
```

## Quality checklist

- [ ] "Who" matches the problem-statement's "who is affected" — not a wider
      or different segment invented to sound more impressive
- [ ] Outcome is the problem's cost reversed, not a new goal
- [ ] Signal is observable through an existing or easily-added
      metric/digest/survey — named, not vague
- [ ] Timeframe is stated, not "eventually" or "soon"
- [ ] Kill condition is a result, not a deadline
- [ ] No "users like it" or unfalsifiable success language anywhere

## Example

**Source:** `company/product/2026-08-18_problem-health-score-comprehension.md`
(see `/problem-statement`'s example — 3 of 4 interviewees misread the health
score drop as a current failure rather than a forward-looking indicator).

**Output** (appended to that file):

```markdown
## Hypothesis

We believe **a tooltip plus a one-screen explainer for the health score**
for **beta cohort users viewing their dashboard** will **eliminate the
"something is already broken" misreading**.
We'll know we're right when **the next interview round (end of September)
shows zero comprehension complaints about the health score, and related
support messages in the beta Slack channel stay at zero** within
**4 weeks of shipping**.

**Kill condition:** if the September interview round still surfaces the
same misreading in ≥2 of the sampled users, the tooltip approach was wrong
— the fix needs a bigger structural change (e.g. splitting "current state"
vs. "forward-looking risk" into two separate numbers on the dashboard), not
more copy.
```

Ends with: "Ready for `/prioritize` alongside other candidates, or straight
to `/bet` if this is the only one on the table this week."
