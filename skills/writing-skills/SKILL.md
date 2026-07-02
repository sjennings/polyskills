---
name: writing-skills
description: Use when writing, editing, reviewing, or testing a skill, or deciding whether something should be a skill at all — the vocabulary, principles, and testing loop that make a skill predictable.
polytoken:
  disable_model_invocation: false
---

A skill exists to wrangle determinism out of a stochastic system. **Predictability** — the agent taking the same _process_ every run, not producing the same output — is the root virtue; every lever below serves it.

**Bold terms** are defined in [`GLOSSARY.md`](GLOSSARY.md); look them up there for the full meaning.

## When to write a skill at all

Write a skill when the technique wasn't obvious to you, you'll reach for it again, and it applies beyond one project. Don't write one for a one-off solution, for standard practice already well documented elsewhere, or for a project convention — that belongs in the project's context file (`AGENTS.md`). And if a constraint is mechanically enforceable (a linter rule, a hook, a validator), automate it instead — spend skills on judgment calls.

## Invocation

Two choices, trading different costs:

- A **model-invoked** skill exposes its **description** to the agent, so the agent can fire it autonomously _and_ other skills can reach it (you can still invoke it by name too). It contributes to **context load** — the description sits in the window every turn. Mechanics: leave `polytoken.disable_model_invocation` unset, and write a model-facing description with rich trigger phrasing ("Use when the user wants…, mentions…").
- A **user-invoked** skill strips the description from the agent's reach: only you, referencing it by name, can invoke it — and no other skill can. Zero context load, but it spends **cognitive load**: _you_ are the index that must remember it exists. Mechanics: set `polytoken.disable_model_invocation: true`; the skill drops out of the agent's view and fires only through a typed `@skill:<name>` reference. The still-required `description` becomes human-facing — a one-line summary, trigger lists stripped.

Polytoken adds a scoping lever on the model-invoked side: tag the skill (`polytoken.tags`) and list `tag!<name>` in a facet's or subagent's `skills_allow`, and the skill is discoverable only inside those contexts. The description's context load is then paid only where the skill is relevant — a review skill need not ride along in every planning turn.

Pick model-invocation only when the agent must reach the skill on its own, or another skill must. If it only ever fires by hand, make it user-invoked and pay no context load. If it fires autonomously but only inside one kind of work, scope it to those facets.

When user-invoked skills multiply past what you can remember, that piled-up cognitive load is cured by a **router skill**: one user-invoked skill that names the others and when to reach for each.

## Writing the description

A model-invoked **description** does two jobs — state what the skill is, and list the **branches** that should trigger it. Every word increases **context load**, so a description earns even harder pruning than the body:

- **Front-load the skill's leading word** — the description is where it does its invocation work.
- **One trigger per branch.** Synonyms that rename a single branch are **duplication** — "build features using TDD … asks for test-first development" is one branch written twice. Collapse them; keep only genuinely distinct branches.
- **Cut identity that's already in the body.** Keep the description to triggers, plus any "when another skill needs…" reach clause.
- **Triggers, not workflow.** The description says _when_, never _how_. A description that summarizes the skill's process becomes a shortcut: the agent follows the one-line summary and never reads the body it abbreviates.

## Information hierarchy

A skill is built from two content types — **steps** and **reference** — that mix freely: a skill can be all steps, all reference, or both. The core decision is which to use and where each sits on the **information hierarchy**, a ladder ranked by how immediately the agent needs the material:

1. **In-skill step** — an ordered action in `SKILL.md`, the primary tier: what the agent does, in order. Each step ends on a **completion criterion**, the condition that tells the agent the work is done. Make it _checkable_ (can the agent tell done from not-done?) and, where it matters, _exhaustive_ ("every modified model accounted for", not "produce a change list") — a vague criterion invites **premature completion**.
2. **In-skill reference** — a definition, rule, or fact in `SKILL.md`, consulted on demand. Often a legitimately flat peer-set (every rule of a review on one rung) — a fine arrangement, not a smell. _This skill is all reference._
3. **External reference** — reference pushed out of `SKILL.md` into a separate file, reached by a **context pointer**, loaded only when the pointer fires. (Spans _disclosed_ reference — a sibling file like `GLOSSARY.md`, still part of the skill — through fully **external reference** that lives outside the skill system and any skill can point at.)

A demanding completion criterion drives thorough **legwork** — the digging the agent does within the work — whether the skill has steps or not, since "every rule applied" binds flat reference just as "every step done" binds a sequence.

Push too little down and the top bloats; push too much and you hide material the agent actually needs. That tension is the whole decision.

**Progressive disclosure** is the move down the ladder — out of `SKILL.md` into a linked file — so the top stays legible. Mechanics: a linked `.md` file in the skill folder, named for what it holds (this skill discloses its full definitions to `GLOSSARY.md`). Some skills are used in more than one way, and each distinct way is a **branch** — different runs taking different paths through the skill. Branching is the cleanest disclosure test: inline what every branch needs, and push behind a pointer what only some branches reach. A **context pointer**'s _wording_, not its target, decides when and how reliably the agent reaches the material.

Where the ladder decides _how far down_ a piece sits, **co-location** decides _what sits beside it_ once there: keep a concept's definition, rules, and caveats under one heading rather than scattered, so reading one part brings its neighbours with it.

## When to split

**Granularity** is how finely you divide skills, and each cut spends one of the two loads, so split only when the cut earns it. Two cuts:

- **By invocation** — split off a **model-invoked** skill when you have a distinct **leading word** that should trigger it on its own, or another skill must reach it. You pay **context load** for the new always-loaded **description**, so that independent reach has to be worth it.
- **By sequence** — split a run of **steps** when the steps still ahead (a step's **post-completion steps**) tempt the agent to rush the one in front of it (**premature completion**). Keeping them out of view encourages the agent to do more **legwork** on the current task.

## Pruning

Keep each meaning in a **single source of truth**: one authoritative place, so changing the behaviour is a one-place edit.

Check every line for **relevance**: does it still bear on what the skill does?

Then hunt **no-ops** sentence by sentence, not just line by line: run the no-op test on each sentence in isolation, and when one fails, delete the whole sentence rather than trim words from it. Be aggressive — most prose that fails should go, not be rewritten.

## Leading words

A **leading word** is a compact concept already living in the model's pretraining that the agent thinks with while running the skill (e.g. _lesson_, _fog of war_, _tracer bullets_). Repeated throughout the text (though not necessarily — a strong leading word might only be needed once), it accumulates a distributed definition and anchors a whole region of behaviour in the fewest tokens, by recruiting priors the model already holds.

It serves predictability twice. In the body it anchors _execution_: the agent reaches for the same behaviour every time the word appears. In the description it anchors _invocation_: when the same word lives in your prompts, docs, and code, the agent links that shared language to the skill and fires it more reliably.

Hunt for opportunities to refactor skills to use leading words. A triad spelled out at three sites (**duplication**), a description spending a sentence to gesture at one idea — each is a passage begging to **collapse** into a single token. Examples include:

- "fast, deterministic, low-overhead" -> _tight_ — one quality restated across a phase — into a single pretrained word (a _tight_ loop).
- "a loop you believe in" -> _red_ — converts a fuzzy gate into a binary observable state (the loop goes _red_ on the bug, or it doesn't).

You win twice over: fewer tokens, _and_ a sharper hook for the agent to hang its thinking on. Assume every skill is carrying restatements that leading words retire — go find them.

## Match the form to the failure

Before writing a behavioural line, classify the failure you observed in the **baseline** — the form that fixes one failure type backfires on another:

| Observed failure | Right form | Wrong form |
|---|---|---|
| Knows the rule, skips it under pressure | Prohibition + rationalization table + red flags (see [`TESTING.md`](TESTING.md)) | Soft guidance ("prefer…", "consider…") |
| Complies, but the output has the wrong shape | Positive recipe: state what the output IS — its parts, in order | Prohibition list ("don't restate", "never narrate") |
| Omits an element from something it already produces | A required slot in the template it fills in | Prose reminders near the template |
| Behaviour should depend on a condition | Conditional keyed to an observable predicate | Unconditional rule + exemption clauses |

Prohibitions invite negotiation under a competing incentive; a recipe leaves nothing to negotiate — the output matches the stated shape or it doesn't. Two rules hold for any form: **no nuance clauses** ("don't X unless it matters" reopens the negotiation — write a real exception as its own conditional on an observable predicate), and **exemption clauses don't scope** ("this limit doesn't apply to code blocks" still suppresses code blocks — restructure so the rule can't reach the exempt part).

## Failure modes

Use these to diagnose issues the user may be having with the skill.

- **Premature completion** — ending a step before it's genuinely done, attention slipping to _being done_. Defence, in order: sharpen the completion criterion first (cheap, local); only if it is irreducibly fuzzy _and_ you observe the rush, hide the post-completion steps by splitting (the sequence cut).
- **Duplication** — the same meaning in more than one place. Costs maintenance and tokens, and inflates a meaning's prominence on the ladder past its real rank.
- **Sediment** — stale layers that settle because adding feels safe and removing feels risky. The default fate of any skill without a pruning discipline.
- **Sprawl** — a skill simply too long, even when every line is live and unique. Hurts readability and maintainability and wastes tokens. The cure is the ladder: disclose **reference** behind pointers, and split by **branch** or sequence so each path carries only what it needs.
- **No-op** — a line the model already obeys by default, so you pay load to say nothing. The test: does it change behaviour versus the default? A weak leading word (_be thorough_ when the agent is already thorough-ish) is a no-op; the fix is a stronger word (_relentless_), not a different technique.

## Testing

A behavioural skill is tested like code, and the test comes first: run the scenario on a subagent _without_ the skill (the **baseline**) and watch it fail, or you're encoding what you _think_ needs preventing rather than what does. Then run it _with_ the skill under a **pressure scenario**, harvest the excuses verbatim into a **rationalization table**, and close loopholes until it holds. Disputes about whether a line is a **no-op** are settled the same way — by running it, not by debate. Pure reference skills skip all this; there is no rule to violate. Procedure — dispatch mechanics, scenario design, wording micro-tests, loophole-closing, **meta-testing** — in [`TESTING.md`](TESTING.md).

## Polytoken mechanics

- A skill is a directory holding `SKILL.md` plus any disclosed files; the **directory name is the skill's name**. Project skills live in `.polytoken/skills/`, global ones in `skills/` under the Polytoken config directory. Name by the activity, verb-first gerund (`writing-skills`, `investigating-a-codebase`), not the noun (`skill-creation`).
- With a `polytoken:` block in the frontmatter, the body is rendered as a MiniJinja template — literal curly-brace template markers get interpreted, so avoid them (or drop the block if you need them verbatim).
- Skills load at startup and on configuration reload; an edit takes effect on the next reload, not mid-session.
- Full authoring reference: https://docs.polytoken.dev/harness-engineering/skills/

## Before you ship

A skill — new or edited — is done when every line below checks out, not before:

- [ ] It should be a skill at all: reusable beyond one project, a judgment call rather than a mechanically enforceable constraint, not already documented elsewhere.
- [ ] Invocation chosen by the decision rule: model-invoked only if the agent (or another skill) must reach it alone; facet-scoped if it serves one kind of work; user-invoked otherwise.
- [ ] Description is triggers only — one per branch, no workflow summary (model-invoked skills).
- [ ] Every behavioural line traces to a **baseline** failure you observed, in the form that matches it (recipe, prohibition, slot, or conditional).
- [ ] The ladder is applied: material only some branches reach is disclosed behind a pointer; what every branch needs is inline.
- [ ] Pruned: each meaning has a single home, and every sentence survives the no-op test.
- [ ] Behavioural skills ran the [`TESTING.md`](TESTING.md) loop to bulletproof — including re-running covering scenarios after edits. Pure reference skills skip this item only.
- [ ] Mechanics: verb-first gerund name, no literal template markers under a `polytoken:` block, and a config reload before relying on the change.
