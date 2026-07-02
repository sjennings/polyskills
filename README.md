# Polyskills

<img src="logo.png" alt="Polyskills logo" width="400" height="400">

A curated collection of reusable [Polytoken](https://polytoken.dev/) skills, subagents and facets.

## Subagents

### `project-context-librarian`

Reviews what changed during a development phase or branch and keeps project context documentation accurate. It is a Polytoken subagent adapted to use `AGENTS.md` as the canonical default.

Use it near the end of implementation work when contracts, APIs, domain structure, dependencies, or architectural decisions may have changed. It analyzes diffs, maps contract changes to the relevant context files, and updates `AGENTS.md` files by default.

Required support skills included in this repo:

- `maintaining-project-context`
- `writing-agent-context-files`
- `writing-agent-directives`
- `prompt-security-hardening`

Originally from https://github.com/ed3dai/ed3d-plugins, adapted to use AGENTS.md for project context.

### `workflow-lead`

Executes a phased multi-agent workflow plan handed to it by a facet, fanning out child subagents per phase (parallel where independent) and returning only the synthesized result. Used by the `plan-more` facet when a planning run's fan-out is too large to hold in the facet's own context.

## Facets

### `plan-more`

<img src="more.jpg" alt="Kylo Ren screaming MORE!" width="400">

A read-only, workflow-native story-planning facet. It turns a vague feature request into an execution-ready story spec (summary, acceptance criteria, tasks, dev notes) through a phased multi-agent workflow: multi-surface investigation fan-out, operator design-grilling, adversarial multi-angle drafting, a grounding check against the real source tree, a three-layer spec review, and a `handoff_plan` into the shipped `execute` facet.

- **Tool profile:** read-only. It inspects the repo (`file_read`, `glob`, `grep`, read-only `shell_exec`), dispatches read-only subagents, and mutates only the Polytoken plan artifact (`write_plan` / `edit_plan` / `handoff_plan`). It never changes project files or issue-tracker state, so its tool calls are safe to auto-approve.
- **When to use:** any feature big enough to deserve a written spec before implementation — especially when you want broad grounding and adversarial review rather than a single-pass plan.
- **Setup requirements:** install the `grill-me` and `review-plan` skills and the `workflow-lead` subagent alongside it (all included in this repo). The built-in `researcher`, `plan-reviewer`, `general-purpose`, and `general-purpose-mini` subagents must be available (they ship with Polytoken).

Parts taken from https://github.com/ed3dai/ed3d-plugins, https://github.com/obra/superpowers,
https://github.com/bmad-code-org/BMAD-METHOD/ and my own work. 

## Skills

### `bash-wizard`

Generates an interactive Bash wizard for walking a human through a manual procedure step by step.

Use it when a task is too tedious, risky, or stateful to explain repeatedly in chat, such as:

- system repair
- third-party setup
- one-off migrations
- environment or repository setup
- guided procedures that need confirmation gates

The generated wizard UX supports progress display, time estimates, confirmation gates, cross-platform URL opening, hidden secret entry, idempotent `.env` updates, GitHub Actions secret/variable helpers, command execution prompts, checks, and a closing summary.

Originally from https://github.com/mattpocock/skills

### `grill-me`

Interviews the user relentlessly about a plan or design until reaching shared understanding, walking every branch of the decision tree and recommending an answer for each question. Used by the `plan-more` facet for design grilling, and useful standalone whenever you want a plan stress-tested.

Originally from https://github.com/mattpocock/skills

### `review-plan`

Runs a mandatory three-layer parallel spec review before an execution handoff: a story-spec checklist reviewer, an outside-model reviewer on a different model family, and the `plan-reviewer` subagent for plan shape. Findings are triaged by severity and gate `handoff_plan`. Support skill for the `plan-more` facet.

### `review`

<img src="fire_everything.jpg" alt="FIRE EVERYTHING! meme" width="400">

Multi-model, multi-lens review of a diff (a branch, PR, staged or uncommitted changes, or a pasted diff). Five independent read-only reviewers run as parallel subagents — repository standards, spec fidelity, blind defect hunting, LLM-pathology hunting, and test-quality audit — each given precisely scoped context, with cross-family model overrides so a model never reviews its own family's blind spots. Findings are triaged against the actual code, re-rated, and consolidated into one verdict report. Facets and skills can also invoke it as a prescribed Standards+Spec review layer. The reviewer contract and per-layer briefs ship as sibling files in the skill directory.

Parts taken from https://github.com/mattpocock/skills, https://github.com/obra/superpowers,
https://github.com/bmad-code-org/BMAD-METHOD/ and my own work.

### `writing-skills`

The vocabulary, principles, and testing loop for writing Polytoken skills that behave predictably. Covers when something should be a skill at all, model- versus user-invocation tradeoffs, writing trigger descriptions, information hierarchy and progressive disclosure, leading words, pruning, and failure modes. Ships with a glossary of the full domain model (`GLOSSARY.md`) and a subagent-based procedure for pressure-testing behavioural skills (`TESTING.md`).

Originally from https://github.com/mattpocock/skills and https://github.com/obra/superpowers, adapted to Polytoken invocation and scoping mechanics.

### `maintaining-project-context`

Coordinates end-of-branch or end-of-phase context maintenance. It identifies contract-relevant changes and routes AGENTS.md updates through the `writing-agent-context-files` skill.

Originally from https://github.com/ed3dai/ed3d-plugins -- support skill for project-context-librarian agent.

### `writing-agent-context-files`

Provides AGENTS.md structure, templates, freshness-date rules, and guidance for top-level versus domain-level context files.

Originally from https://github.com/ed3dai/ed3d-plugins -- support skill for project-context-librarian agent.

### `writing-agent-directives`

Provides guidance for writing concise, effective agent instructions, skills, subagent prompts, and AGENTS.md content. Includes supporting reference files for Graphviz conventions and long-running state patterns.

Originally from https://github.com/ed3dai/ed3d-plugins -- support skill for project-context-librarian agent.

### `prompt-security-hardening`

Documents safe patterns for directives and shell examples involving credentials, environment variables, file creation, git operations, and secret-bearing files.

Originally from https://github.com/ed3dai/ed3d-plugins -- support skill for project-context-librarian agent.

## Installation

Polytoken by default looks for user skills, subagents and facets in:

```text
~/.config/polytoken/skills/
~/.config/polytoken/subagents/
~/.config/polytoken/facets/
```

### Option 1: Manual install from a clone

Clone this repository, then copy the skill directories into your Polytoken skills folder:

```bash
git clone https://github.com/sjennings/polyskills.git polyskills
cp -R polyskills/* ~/.config/polytoken/
```

After copying, restart Polytoken until Ed supports live refresh of skills and facets.

Note that if you want these skills only in a specific project, Polytoken supports project-level configuration through a .polytoken directory in your project.

### Option 2: Ask Polytoken to do it

You can also ask Polytoken to do the installation for you. For example:

> Install the skills, subagents and facets from https://github.com/sjennings/polyskills into my local Polytoken config. Copy every directory under `skills/`, `subagents/` and `facets/` into `~/.config/polytoken/skills/`, `~/.config/polytoken/subagents/` and `~/.config/polytoken/facets/`, preserving each directory and file exactly. Do not overwrite an existing local skill, subagent or facet without asking first. Consult @skill:polytoken:modifying-polytoken for further guidance.

You may want to limit this to just install a specific skill or subagent:

> Install the `project-context-librarian` subagent and its required support skills from https://github.com/sjennings/polyskills into my local Polytoken config. Copy `subagents/project-context-librarian.md` into `~/.config/polytoken/subagents/` and copy `skills/maintaining-project-context`, `skills/writing-agent-context-files`, `skills/writing-agent-directives` and `skills/prompt-security-hardening` into `~/.config/polytoken/skills/`. Do not overwrite existing local files without asking first. Consult @skill:polytoken:modifying-polytoken for further guidance.

## Contributing

Not yet.

## License

Bomb. James Bomb. Agent 009. Licensed to ill.
