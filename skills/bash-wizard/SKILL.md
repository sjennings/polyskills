---
name: bash-wizard
description: Generate an interactive bash wizard that walks a human through a manual procedure step by step — system repair, third-party setup, a one-off migration, or any A→B state transition — opening URLs, running commands with confirmation gates, capturing values, and showing progress with time remaining.
polytoken:
  tags: [tooling]
---

# Bash Wizard

A **wizard** is a bash script that walks a human, step by step, through a manual
procedure that is tedious to do by hand and tedious to re-explain to an AI every
time. It announces each stage, shows progress and time remaining, runs commands
with confirmation gates (or tells the human exactly what to click and copy),
captures values, writes them where they belong, and shows a closing summary.

The delightful UX is already solved by [template.sh](template.sh) — progress
with time-remaining, confirmation gates, cross-platform URL opening (including
WSL), hidden secret entry, idempotent `.env` upserts, `gh secret`/`gh variable`
writes, a `run_cmd` helper for confirmed command execution, and a closing
summary. **Your job is only to scope the procedure and author its stages.** The
library above the `STAGES` marker is identical in every wizard; that consistency
is the point — never hand-edit it.

A wizard is ephemeral by default — built for one run, saved to a scratch or
`scripts/` path, deleted when the job is done. Commit it only when the user wants
a repeatable procedure that should live in the repo.

## Process

### 1. Scope the Procedure

Work out every step the human must take and every value that gets captured or
command that gets run along the way. Read the environment first — don't ask cold:

- For **system repair or infrastructure**: current state (logs, mounts,
  `systemctl` status, device listings), target state, and the irreversible
  actions between them. Understand what is broken and why before prescribing
  commands.
- For **third-party setup**: `.env`, `.env.example`, `docker-compose*`, framework
  config, and CI workflow files (every `secrets.*` / `vars.*` reference is a
  value the wizard must produce).
- For a **migration or transition**: the current state, the target state, and the
  irreversible actions between them.

Then show the user the ordered list of stages and the values each produces, and
confirm — they may add, drop, or reorder.

**Done when:** every stage is named in order, and for each stage you know
(a) what happens (a command runs, a human does something, or both),
(b) whether it is reversible or irreversible, and
(c) what value or state change it produces.

### 2. Map Each Stage's Journey

For each stage, write the precise path a human follows: which URL to open, what
to do there, where a value is shown — or which command to run and what to look
for in its output. Where you don't actually know the current UI or the exact
command, say so and ask the user or check the docs — never invent steps that may
not exist.

**Done when:** every stage traces to concrete instructions a stranger could
follow.

### 3. Author the wizard

Copy `template.sh` to the target path. Replace the example stage with one
`stage` per step, in dependency order. Use the library helpers:

| Helper | Purpose |
|--------|---------|
| `stage "Name" <min>` | Clear screen, announce stage, show progress + time left |
| `say "..."` | Plain instruction line |
| `step "..."` | A specific action the human takes |
| `note "..."` | Dimmed supplementary info |
| `warn "..."` | Yellow caution text |
| `pause "msg"` | Wait for Enter |
| `confirm "question"` | y/N gate; returns success on yes |
| `run_cmd "description" cmd...` | Show command, ask confirmation, run it, show output |
| `check "description" cmd...` | Run a check silently; report pass/fail |
| `open_url URL` | Open browser cross-platform (WSL aware) |
| `ask KEY "Prompt"` | Read a visible value (offers existing `.env` default) |
| `ask_secret KEY "Prompt"` | Read a hidden value |
| `write_env KEY VALUE` | Upsert into `.env` (idempotent) |
| `set_secret NAME VALUE` | Set a GitHub Actions secret via `gh` |
| `set_var NAME VALUE` | Set a GitHub Actions variable via `gh` |
| `fail "message"` | Print error and exit non-zero |
| `finish` | Closing summary of everything done |

Set `TOTAL_STAGES` and `TOTAL_MINUTES` to honest estimates (this drives the
time-remaining display). Each `stage` clears the screen so only the current step
is visible — keep a stage to one focused task so nothing the human needs scrolls
away. Don't touch the library above the `STAGES` marker.

**Irreversible actions** (formatting, deleting, `fsck -y`, dropping databases)
must always be gated behind `confirm`. **Destructive commands** should show
exactly what they will do before asking for confirmation. Use `check` for
pre-flight verification (is the device present? is the service running?) and
post-flight verification (did the mount succeed? is the share accessible?).

### 4. Verify and hand off

- `bash -n <script>`; run `shellcheck` if available.
- `chmod +x <script>`.
- Don't run it end-to-end yourself — it clears screens, runs privileged commands,
  and blocks on human input. Trace it statically instead: every value and command
  from step 1 is present, every `confirm` gates an irreversible action, and every
  `check` verifies something that matters.
- Tell the user how to run it. If it is a repeatable procedure, commit it and
  link it from the README.
