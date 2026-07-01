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

## Facets

No facets are included yet.

When facets are added, they should live under a top-level `facets/` directory and be documented here with:

- the facet name
- what behavior or tool profile it enables
- when to use it
- any setup requirements

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
