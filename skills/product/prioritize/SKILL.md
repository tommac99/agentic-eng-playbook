---
name: prioritize
description: Score candidate bets with RICE (reach, impact, confidence, effort) using scripts/rice.py. Use when several hypotheses or Backlog issues are competing for the next bet and the team needs a ranked, defensible order before jour fixe. Elicits scores with the human rather than inventing them, and treats the ranking as an input to /bet, never the decision itself.
---

# Prioritize (RICE)

RICE gives the jour fixe conversation a starting order, not an answer. The
score is only as honest as the numbers behind it — this skill's whole job is
making sure those numbers come from a human, not a guess dressed up as math.

## When to use

- Two or more hypotheses (from `/hypothesis`) or tagged Backlog issues
  (`research-signal`, `build`) are competing for the next bet.
- Prepping the ranked input for `/bet` before the Monday jour fixe.
- Someone asks "what should we build next" with more than one real option
  on the table.

## When NOT to use

- Only one candidate exists — there's nothing to rank, go straight to
  `/bet` with it.
- No hypothesis exists yet for a candidate — send it through `/hypothesis`
  first; RICE on an un-falsifiable idea just produces a confident-looking
  wrong number.
- Scores would have to be invented because nobody's in the room to give
  them — stop and say so (see the refuse-to-fabricate clause below), don't
  run the script on guessed inputs.
- The ranking itself needs to become a decision — that's `/bet`'s job, this
  skill only orders the table.

## Instructions

1. **Gather candidates.** Pull from:
   - Recent `/hypothesis` outputs not yet bet on
   - Backlog issues tagged `research-signal` (Priya's digests) or carrying
     a clear build proposal
   Confirm the list with the human before scoring — don't silently drop or
   add candidates.

2. **Elicit scores with the user — never invent them.** This is the
   refuse-to-fabricate clause: ask for each candidate's reach, impact,
   confidence, and effort out loud, one at a time if needed. Anchor the
   scale so answers are comparable:
   - **Reach** (1-5): how much of the cohort this touches this
     quarter — 1 = a handful of users, 5 = the whole active cohort
   - **Impact** (0.25-3): 0.25 minimal, 0.5 low, 1 medium, 2 high,
     3 massive — per person reached
   - **Confidence** (0.5-1.0): 1.0 High (strong evidence, e.g. a digest
     theme with quotes), 0.8 Medium (some evidence, one source), 0.5 Low
     (a hunch, no evidence yet — and if it's this low, ask whether
     `/problem-statement` even has enough to stand on)
   - **Effort** (person-days, agent-adjusted): the real cost of the
     agent-buildable slice, not a pre-agent estimate — a three-day
     hand-built feature might be a half-day agent slice; use the number
     that reflects how this team actually builds
   If the human doesn't know a number, that's a stop, not a placeholder —
   say "I need your call on X before I can score this" and wait.

3. **Write the candidates to a temp file** (JSON or the flat-YAML list
   shape `scripts/rice.py` accepts — see the script's docstring) and run:
   ```bash
   python3 scripts/rice.py <candidates-file>
   ```

4. **Present the ranked table plus the sensitivity note as-is.** Don't
   editorialize the ranking into a recommendation — that framing belongs to
   `/bet`, where a human makes the call with release-goal fit and appetite
   also on the table.

5. **Hand off explicitly**: "This ranking is one input to `/bet`, not the
   decision — release-goal fit and appetite still need a human call."

## Output contract

No new file by default — the script's table + sensitivity note, presented
in the conversation, is the artifact. If the candidate list will be reused
(e.g. feeding straight into `/bet` the same week), save the input file as:

```
company/product/YYYY-MM-DD_rice-candidates.json
```

so `/bet` can re-run or re-check the script's output without re-eliciting
scores from scratch.

## Quality checklist

- [ ] Every candidate has a `/hypothesis` or a clear evidence-backed Backlog
      issue behind it — nothing scored from a title alone
- [ ] Every score came from the human in this conversation, not invented —
      if any score was assumed, say so explicitly and flag it as unverified
- [ ] Effort reflects agent-adjusted cost, not a pre-agent gut estimate
- [ ] `scripts/rice.py` actually ran — the table wasn't hand-computed or
      hallucinated
- [ ] Sensitivity note is included, not trimmed off
- [ ] Output framed as input to `/bet`, never as "here's what we should
      build"

## Example

Three candidates on the table for the Monday jour fixe: the health-score
tooltip (#58, from Priya's digest), bank-feed import parity (Sam's teardown
gap), and PDF export of the weekly report (Alex's idea, no evidence yet).

Eliciting scores with Alex:

```yaml
- name: Health score tooltip + explainer
  reach: 4
  impact: 2
  confidence: 0.8
  effort: 3
- name: Bank feed import parity
  reach: 3
  impact: 3
  confidence: 0.5
  effort: 8
- name: PDF export of weekly report
  reach: 2
  impact: 1
  confidence: 0.8
  effort: 2
```

Run:

```bash
python3 scripts/rice.py company/product/2026-08-18_rice-candidates.json
```

Output presented as-is:

```
#  name                             reach impact  conf effort   score
---------------------------------------------------------------------
1  Health score tooltip + explainer     4      2   0.8      3    2.13
2  PDF export of weekly report          2      1   0.8      2    0.80
3  Bank feed import parity              3      3   0.5      8    0.56

Sensitivity — confidence dropped one canonical band (High 1.0 -> Medium 0.8 -> Low 0.5):
  - Bank feed import parity: confidence already Low (0.5), cannot drop further.
  - PDF export of weekly report: confidence Medium -> Low (0.8 -> 0.5) drops score 0.80 -> 0.50, rank #2 -> #3.
```

Ends with: "Health score tooltip ranks first and its rank doesn't flip under
a confidence haircut — but reach, release-goal fit, and appetite are
`/bet`'s call, not this table's."
