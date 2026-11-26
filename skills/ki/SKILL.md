---
name: ki
description: Configure the model and reasoning effort shuhari uses per role. Writes ~/.claude/shuhari/ki.md, which overrides the skill defaults. Use for /ki, "configure models", "reasoning effort", or changing which model or effort a role uses.
---

# Ki

Write `~/.claude/shuhari/ki.md`, the file that sets shuhari's model and effort per role. Use `$CLAUDE_CONFIG_DIR` in place of `~/.claude` when it is set. When the path is a symlink, write through it so its target (for example a dotfiles repo) holds the file.

## Steps

### 1. Models and efforts

The models are `fable`, `opus`, `sonnet`, and `haiku`, the values the Agent tool's `model` accepts. The efforts are `low`, `medium`, `high`, `xhigh`, and `max`, the values of its `effort`. The aliases `inherit-parent` and `auto` are always valid and mean the parent chat model.

### 2. Load current state

The defaults are the rule shape in step 5. If `~/.claude/shuhari/ki.md` already exists, read it and treat its `# panel effort` line and its role values as the current choices. Otherwise start from the defaults. Drop each line whose label is not in step 5 or whose value fails step 4, such as `# budget: large (xhigh)`, `judgment and prose: opus`, or `feature, refactoring: sonnet`.

### 3. Map and confirm

Build the working table from the defaults, and on a re-run keep every role you changed. Show every role with its value, the `# panel effort` line, and each line step 2 dropped. Say that the coding defaults are the pairs where more effort stopped paying for its cost on public coding benchmarks in November 2026 (FrontierCode, Artificial Analysis, Terminal-Bench), and that the other defaults are estimates. Ask whether to accept as-is or change specific roles. Prefer AskUserQuestion over free text.

- dojo uses a task role's pair as-is for every spawn. To spend more or less on a kind of work, change its line.
- `how explorer` also covers every reading and searching helper, and `swarm workers` covers every verification lane.
- A panel role (`arena runners`, `arena cross-judge pool`, `architect runners`, `interrogate reviewers`) runs each listed model at its `# panel effort`. For runners and reviewers, one subagent runs per entry, alias entries included, so the list length sets the count.
- Arena runs one cross-judge. It tries only the pool entries that differ from the parent's model, in list order, so order the pool by preference. An `inherit-parent` or `auto` entry never judges.
- `swarm workers` is the pair for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

A task role's value is a model and an effort from step 1, or an alias. A panel role's value is a comma-separated list of models or aliases. The `# panel effort` line names each of the four models once, each with an effort. If a chosen value is anything else, stop and ask again.

### 5. Write the rule

Write `~/.claude/shuhari/ki.md` with the `# panel effort` line and one line per role, using the same labels dojo uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```
# shuhari model and effort configuration (written by /ki). One line per role. Delete a line to fall back to the skill default.
# A task role takes a model and an effort. A panel role takes a list of models, each run at its effort on the panel effort line.
# `inherit-parent` or `auto` as a value runs the role on the parent chat model (omit the Agent `model` and `effort`). Alias entries in a panel list still count toward its fan-out.
# panel effort: fable high, opus xhigh, sonnet xhigh, haiku max
feature, refactoring: opus xhigh
bug-fix: opus xhigh
perf-issue: opus xhigh
hillclimb: opus xhigh
hardest tasks: fable high
prose: opus xhigh
judgment: opus high
mechanical edits: haiku max
how explorer: opus high
how explainer: opus xhigh
why investigators: opus high
why synthesizer: opus xhigh
reflect tooling: opus high
reflect judgment, divergent: opus high
reflect synthesizer: opus xhigh
swarm workers: opus high
arena runners: opus, fable
arena cross-judge pool: opus, fable
architect runners: opus, fable
interrogate reviewers: opus, fable, sonnet
```

### 6. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever shuhari is installed: workspace, user, or plugin). On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, invoke `/create-formal-verification-skill` (resolves wherever shuhari is installed: workspace, user, or plugin). On no, move on without pushing.
