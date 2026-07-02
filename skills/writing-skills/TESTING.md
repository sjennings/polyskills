# Testing Skills on Subagents

Load this when a skill that shapes behaviour has been written or edited and needs verification before you trust it.

Testing a skill is TDD applied to documentation: watch the agent fail without the skill (the **baseline** — your failing test), write the skill against those observed failures, watch it comply, then close loopholes until it stays compliant. If you didn't watch an agent fail without the skill, you don't know whether the skill prevents the right failures.

## What to test — and what not to

Test skills that enforce discipline, carry compliance costs, could be rationalized away ("just this once"), or shape output form. Don't test pure reference skills or skills with no rule to violate — no incentive to bypass means nothing to bulletproof.

## The loop

1. **RED — baseline.** Run the pressure scenario on a fresh subagent *without* the skill. Document choices and rationalizations verbatim. Which pressures triggered which excuses?
2. **GREEN — write the minimal skill.** Address the specific failures you observed — nothing for hypothetical cases. Re-run the same scenarios *with* the skill; the agent should now comply. Still failing? The wording is unclear or incomplete — revise and re-run.
3. **REFACTOR — close loopholes.** Each new rationalization gets, in the skill: an explicit negation in the rule ("don't keep it as reference; delete means delete"), a row in the rationalization table, a red-flag entry the agent can self-check against, and — if it marks the *approach* to violation — a symptom in the description. Re-run until no new rationalizations appear.

Edits count too: an edit to a behavioural skill re-runs the scenarios that cover it.

## Dispatch mechanics (Polytoken)

- Run scenarios with the `subagent` tool (`general-purpose`); each dispatch is a fresh context, which is exactly what the test needs.
- Paste the draft skill body into the subagent prompt for with-skill arms and omit it for baselines. This gives per-arm control and sidesteps reload: registered skills only refresh on configuration reload, so iterating through the `skill` tool would test a stale copy.
- Frame it as real work, not a quiz: "This is a real scenario. Choose and act." An agent that knows it's being tested performs compliance.

## Writing pressure scenarios

An academic prompt ("what does the skill say about X?") tests recitation. A **pressure scenario** makes the agent *want* to violate:

- **Combine 3+ pressures:** time (deadline, deploy window), sunk cost (hours of work to delete), authority (senior says skip it), economic (job at stake), exhaustion (end of day), social (looking dogmatic), pragmatism ("adapting, not dogma").
- **Force a choice:** concrete A/B/C options, not open-ended questions. "What do you do?", never "what should you do?"
- **Real constraints:** specific times, file paths, consequences.
- **No easy outs:** "I'd ask my human partner" doesn't count as an answer; the scenario must require choosing.

## Micro-testing wording

Full scenarios are the final gate but slow per iteration. To compare candidate wordings first:

- One fresh-context subagent per sample, with the guidance embedded in its realistic surroundings (the whole skill), not in isolation.
- **Always include a no-guidance control.** If the control doesn't exhibit the failure, there is nothing to fix — stop; don't author the guidance.
- 5+ reps per variant; single samples lie.
- Read every flagged output yourself — template echoes and quoted counter-examples masquerade as hits in automated counts.
- Variance is a metric: when wording binds, reps converge on one shape. Five interpretations across five reps means tighten the form, not add words.

## Meta-testing

When an agent violates the rule *despite having the skill*, ask it: "You read the skill and still chose C — how should the skill have been written so A was the only acceptable answer?" Three diagnoses:

1. "The skill was clear; I chose to ignore it" → not a wording problem; add a foundational principle early ("violating the letter of the rules is violating the spirit of the rules").
2. "It should have said X" → wording problem; add X, usually verbatim.
3. "I didn't see section Y" → organization problem; move the point up the information hierarchy.

## Bulletproof

The skill holds when, under maximum combined pressure, the agent picks the right option, cites the skill's sections, and acknowledges the temptation. It is not bulletproof while runs still produce new rationalizations, "hybrid approaches", or strong arguments for exemption. Common ways to fool yourself: single-pressure scenarios (agents resist one pressure, fold under three), paraphrased instead of verbatim rationalization capture, stopping after the first green run, and generic counters ("don't cheat") where explicit negations are needed ("don't keep it as reference").
