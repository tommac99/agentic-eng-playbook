# Agentic Eng Playbook

A working set of [Claude Code](https://docs.claude.com/en/docs/claude-code) skills, extracted from a real product-and-engineering operating loop and genericized for sharing. Not a tutorial — these are the actual skill files, in the actual format Claude Code routes to, ready to drop into your own `.claude/skills/`.

Built while running the full loop end to end on a real product: framing problems with evidence, turning them into falsifiable bets, breaking bets into agent-sized issues, and shipping the code with hard verification gates at every step a mistake would be expensive.

## How Claude Code skills work

A skill is a folder containing a `SKILL.md` with YAML frontmatter (`name`, `description`) and a body of instructions. Claude reads the `description` field to decide when a skill is relevant to the current conversation, then loads the full body on demand. Drop a skill folder into `.claude/skills/<skill-name>/` in any repo (or `~/.claude/skills/` globally) and it's available immediately — no build step, no registration.

Every skill here follows the same internal shape, because a skill without this shape is just a vague suggestion:

- **When to use / When NOT to use** — the boundary matters as much as the trigger. Half of what makes a skill reliable is knowing when *not* to reach for it.
- **Instructions** — the actual steps, in order.
- **Output contract** — exactly what the skill produces, so a human reviewing its output knows what "done" looks like.
- **Quality checklist** — a self-check the skill runs against its own output before handing back.
- **Example** — a full worked run, so the skill's judgment calls are visible, not just its mechanics.

## The two categories

**Product skills** turn a raw observation into a scoped, falsifiable, buildable piece of work:

```
problem-statement → hypothesis → prioritize → epic-spec → critic
   (evidence)      (falsifiable)   (ranked)    (issues)   (adversarial
                                                            review before
                                                            a human gate)
```

**Engineering skills** turn an approved issue into merged-ready code without the human losing control of the decisions that matter:

```
acceptance-criteria → build → ship → verify → retro
   (the contract)   (spec + plan,  (quality gate,  (walk the AC   (what
                     human approves  PR opened)      on staging)    pattern
                     twice)                                          changes)
```

The two chains connect: an `epic-spec` issue needs `acceptance-criteria` before it can move to Ready; `build` reads those criteria to write its spec; `ship` proves them with real command output, not an assertion; `verify` walks them again against the live deployment; `retro` closes the loop by asking whether the original hypothesis was actually right.

## Skills

|Skill|Category|What it does|
|-|-|-|
|`problem-statement`|Product|Frames a problem with an evidence link before any solution is proposed. Refuses solution language on sight.|
|`hypothesis`|Product|Turns a framed problem into a falsifiable bet with a measurable signal and a stated kill condition.|
|`prioritize`|Product|Scores competing bets with RICE, eliciting every number from a human rather than inventing them.|
|`epic-spec`|Product|Turns a decided bet into a PRD section plus agent-sized, file-disjoint sub-issues. Never creates anything without human confirmation of the decomposition.|
|`critic`|Product|Adversarial review of any PM artifact before it reaches a human gate — hunts for weaknesses, never rewrites.|
|`acceptance-criteria`|Engineering|Writes checkable criteria (Given/When/Then or plain checklist) that a non-author can verify without reading code.|
|`build`|Engineering|Takes an issue from the board to a dispatched build agent, running four intake gates and stopping twice for human approval — on the spec, then on the plan.|
|`ship`|Engineering|Runs the exact CI-parity check, routes reviewers to the diff, opens the PR with real test evidence. Never merges, under any circumstance.|
|`verify`|Engineering|Walks an issue's acceptance criteria against the real staging deployment, one at a time, with recorded evidence. Never moves the card to Done.|
|`retro`|Engineering|Runs a 15-minute epic-close retro: was the bet right, was the appetite right, what pattern should change.|

## Design principles

Patterns that repeat across all ten skills, because they're what actually make an agentic workflow trustworthy rather than merely fast:

- **Refuse to fabricate.** `prioritize` stops and asks rather than inventing a RICE score. `problem-statement` refuses to write a problem with no evidence link. If a number or fact has no visible source, the skill says so instead of filling the gap.
- **Hard human gates, not suggestions.** `build` stops twice — once on the spec, once on the plan — and `epic-spec` never calls `gh issue create` before a human confirms the decomposition. These are gates a skill cannot talk itself past, not steps it recommends.
- **Verify against reality, not metadata.** `build`'s blocked-by gate checks whether a blocker actually landed in the target branch, not whether its label says so. `ship` checks CI against the current commit SHA directly, because a PR page can show a stale result from an earlier commit and look green regardless.
- **Invoke skills, don't paraphrase them.** `build` dispatches its build agent with an instruction to *invoke* `ship`, never to reproduce its steps from memory — a paraphrased skill silently drops exactly the step the paraphraser already forgot.
- **Some actions are absolute.** `ship` never merges. Not for a one-line fix, not because a plan says to, not because the task said to. The one exception is an explicit human instruction in the current turn, and even then it checks CI first and states what it's about to do.

## Using these

Copy any `skills/product/<name>` or `skills/engineering/<name>` folder into your own `.claude/skills/`. They reference a generic fictional repo (`alexdev/northwind`) and generic roles (a product lead, a researcher, an ops teammate) — swap those for your own repo and team, and the gates, contracts, and checklists carry over as-is.

## License

MIT — see [LICENSE](LICENSE).
