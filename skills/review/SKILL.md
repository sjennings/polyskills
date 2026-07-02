---
name: review
description: Review the diff since a fixed point (commit, branch, tag, merge-base) or the staged/uncommitted changes, using parallel reviewers on multiple models across five lenses — Standards, Spec, blind defects, LLM pathologies, test quality. Use when the user wants a branch, PR, or work-in-progress reviewed or says "review since X", and when a facet or skill needs a Standards+Spec review layer over a diff.
---

# Review

Multi-model, multi-lens review of a diff. Independent reviewers run as parallel sub-agents — each gets precisely crafted context, never this session's history — and their findings are triaged against the actual code into one verdict. The review is **read-only end to end**: it produces a report; acting on findings belongs to the caller.

## Choose the branch

- **Prescribed** — a facet or skill invoked this skill and names the layers to run (typically Standards + Spec), the model policy, and the spec context (issue body, AC list, verification evidence). Run exactly those layers with exactly that policy: steps 1 and 4–8, using the caller's spec context instead of step 3, and skipping the step 2 gate when the caller already ran verification (pass its evidence through instead).
- **Full battery** — the user asked for a review directly. Run steps 1–8 with all five layers. Exception: if the diff is under ~25 changed lines (`git diff --stat`) and the user didn't ask for a full review, run a condensed battery — Standards, Spec, and LLM-pathology only.

## Process

### 1. Pin the diff

Whatever the user said is the fixed point — a SHA, branch, tag, `main`, `HEAD~5`. Don't be opinionated; pass it through. Watch for mode keywords that change the diff source:

- "staged" → `git diff --cached`
- "uncommitted" / "working tree" → `git diff HEAD`
- "last N commits" / `<a>..<b>` → that range
- a pasted diff → use the pasted text; reviewers get it inline instead of git commands

Default: `git diff <fixed-point>...HEAD` (three-dot, so the comparison is against the merge-base) plus `git log <fixed-point>..HEAD --oneline`. If no target was given: check the conversation for one; then look for a tracker issue in progress with a recorded baseline commit (per the project's issue-tracker docs, e.g. `docs/agents/issue-tracker.md`, if present); then offer the current branch against the default branch; ask only if all of those fail.

Validate before dispatching anything: `git rev-parse <fixed-point>` resolves and the diff is non-empty. A bad ref or empty diff fails here — not inside five parallel sub-agents. If the tree is dirty and the target is a committed range, list the excluded dirty files in the report header. If the diff exceeds ~3000 changed lines, chunk it by subsystem, run the battery per chunk, and finish with one whole-diff consistency pass; say so in the report.

### 2. Run the mechanical gate

Run the repo's documented build/lint/fast-test commands (from CLAUDE.md / AGENTS.md "Commands"). Record pass/fail output as context for every reviewer — never have a reviewer re-derive what tooling proves. A failing build is already a finding; still dispatch the battery, with the failure in every prompt. Skip anything slow and record it as skipped.

### 3. Identify the spec source

In order: contents the caller supplied → a path or issue id the user passed → issue references in the commit messages (per the project's issue-tracker docs, if present) → a PRD/spec under `docs/`, `specs/`, or `.scratch/` matching the branch or feature → the user's stated intent in this conversation, quoted verbatim → ask. If there is genuinely no spec, the Spec layer is skipped and the report records `spec:no spec available`.

### 4. Identify the standards sources

Collect paths to everything documenting how code should be written: `CLAUDE.md` / `AGENTS.md` (root and per-directory), `CONTRIBUTING.md`, `CONTEXT.md`, `docs/adr/`, any `STYLE.md` / `STANDARDS.md`. Note machine-enforced configs (`.editorconfig`, linter/formatter/type-checker configs) but don't re-review what tooling enforces.

### 5. Assemble the battery

| Layer | Brief | Model | Context it receives |
|---|---|---|---|
| Standards | `STANDARDS.md` | session default | diff + standards paths |
| Spec | `SPEC.md` | session default | diff + spec + AC list if any |
| Blind defect hunter | `DEFECTS.md` | cross-family | diff only — no spec, no standards |
| LLM-pathology hunter | `PATHOLOGY.md` | cross-family | diff + repo read access |
| Test auditor | `TEST-AUDIT.md` | cross-family | diff + repo read + gate results |

**Model policy.** "Cross-family" means a `model_override` naming a configured model whose family differs from this thread's active model, at comparable capability — never a mini/fast tier for a review layer. Resolve the override at runtime, not against fixed version strings. If no cross-family model is configured, run those layers on the default model and record in the report that cross-model independence was unavailable. A prescribed caller's model policy overrides this table.

### 6. Dispatch

Read `CONTRACT.md` and each active layer's brief from this skill's directory. Each reviewer prompt is: the full CONTRACT.md text, the layer's brief text, the exact diff/log commands (or the pasted diff), the layer's context column from the table, and the mechanical-gate results — and nothing else. No session history, no summary of the implementation, no hypotheses about what might be wrong: an anchored reviewer defeats the layer.

Dispatch all layers in one message as parallel `subagent` calls (`subagent_type: general-purpose`, `model_override` per the table); collect with `job_block` / `job_result`. If a layer fails or returns empty, retry it once, then record `layer:reason` and continue — never silently skip a layer, and never substitute your own inline pass for a failed one. (On a harness with no subagent tool: write each assembled prompt to a file and ask the user to run them in separate sessions — ideally on different models — and paste the results back.)

### 7. Triage

Print every reviewer's full output before acting on any of it. Then:

1. **Normalize** into one list: id, source, proposed severity, title, location, AC, evidence, suggested fix.
2. **Deduplicate.** The finding with the most specific evidence survives as the base; merge sources (`defects+pathology`).
3. **Re-rate from the code.** For each finding, read the cited location plus enough surrounding code — call sites, guards, validation outside the hunk — to judge reachability. Your rating replaces the reviewer's: reviewers propose under deliberate information asymmetry. Severity reflects the consequence at a real call site, not the worst theoretical reading. Scale: Critical / Important / Minor / Nit / Decision needed (defined in `CONTRACT.md`).
4. **Dismiss** only findings that are demonstrably false, duplicates, contradict a documented convention, or lie outside this change. A real issue that pre-exists the change is **Deferred**, not dismissed — it stays in the report.
5. **Adjudicate disagreements.** Where layers on different models overlap and disagree — one flags what the other could have seen and stayed silent on, or they contradict — resolve by reading the code. Evidence decides, never majority vote. Note each adjudication.

### 8. Report

The report has these slots, in order:

1. **Header** — range, commit list, mechanical-gate results, layers run, failed layers (`layer:reason` each), model per layer, and any dirty-tree, chunking, or cross-model-unavailable notes.
2. **Per-layer reports** — each layer's output under its own heading, verbatim or lightly cleaned. Never merge them: the layers are deliberately separate so one can't mask another.
3. **Consolidated findings** — the deduped, re-rated table: Severity | Source | Location | Finding | Suggested fix. Deferred (pre-existing) findings in their own sub-list.
4. **Disagreements** — each cross-model disagreement and its adjudication.
5. **Verdict** — Ready | Ready with fixes | Not ready, plus the worst finding *per layer* (no single cross-layer winner — that's the reranking the separation exists to prevent). A clean result with failed layers is an **incomplete review, not a clean one**; the verdict line must say so.
6. **Next actions** — Decision-needed questions for the operator; fix order for the caller (Critical, then Important, before proceeding; Minor and Nit fixed opportunistically or noted); an offer to file Deferred findings in the project's issue tracker, per its documented conventions.

Findings are the deliverable. Do not fix anything, and do not write to the tracker without direction.

## Why independent lenses

- **Axes don't share a verdict.** Code can follow every standard and implement the wrong thing, or match the spec and break every convention. Separate reports stop one axis from masking the other.
- **Fresh context beats anchoring.** A reviewer that sees the implementer's reasoning inherits the implementer's blind spots; each reviewer judges the artifact cold.
- **Different families, different blind spots.** A model reviewing its own family's output tends to re-accept the same fabricated APIs, idioms, and test shapes it would have generated. Cross-family review is what makes the pathology and test layers credible.
