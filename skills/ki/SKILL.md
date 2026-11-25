---
name: ki
description: Configure which model (opus, sonnet, haiku) shuhari uses per role and at what reasoning effort. Writes ~/.claude/shuhari/ki.md, which overrides the skill defaults. Use for /ki, "configure models", "reasoning budget", or changing which model a role uses.
---

# Ki

Write `~/.claude/shuhari/ki.md`, the file that sets shuhari's model per role. Use `$CLAUDE_CONFIG_DIR` in place of `~/.claude` when it is set. When the path is a symlink, write through it so its target (for example a dotfiles repo) holds the file.

## Steps

### 1. Detect available models

The models are fixed: `opus`, `sonnet`, and `haiku`, the values the Agent tool's `model` accepts. Never offer or write `fable`. Effort is separate and goes in the Agent tool's `effort`: `low`, `medium`, `high`, `xhigh`, or `max`. The aliases `inherit-parent` and `auto` are always valid and mean the parent chat model.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. If `~/.claude/shuhari/ki.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer AskUserQuestion over free text. Offer these four options with these exact labels, and name the current budget when the rule records one. With no rule, say that `large` matches the skill defaults.

- `unlimited — max reasoning`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role you changed by model, list, or alias (`inherit-parent`, `auto`). The budget does not change any model. It sets the starting effort for every role: `unlimited` is `max`, `large` is `xhigh`, `medium` is `high`, and `small` is `medium`, on the ladder `max` > `xhigh` > `high` > `medium` > `low`. dojo passes that effort with each spawn and adjusts it per job.

**(c) Show the roles and confirm.** Show every role with its model. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering `opus`, `sonnet`, `haiku`, `inherit-parent`, and `auto` (the last two mean: this role runs on the parent chat model) as the options. Prefer AskUserQuestion over free text. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena runs one judge. It tries only values that differ from the parent's model, in list order, so order it by preference. An `inherit-parent` or `auto` value never judges. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every value written must be `opus`, `sonnet`, `haiku`, `inherit-parent`, or `auto`. If a chosen value is anything else, including `fable`, stop and ask again.

### 5. Write the rule

Write `~/.claude/shuhari/ki.md` with a `# budget` line with the chosen label and its target effort, and one line per role, using the same labels dojo uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```
# shuhari model configuration (written by /ki). One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit the Agent `model`). Alias entries in a panel list still count toward its fan-out.
# budget: large (xhigh)
feature, refactoring: sonnet
bug-fix: sonnet
perf-issue: sonnet
hillclimb: sonnet
judgment and prose: opus
hardest tasks: opus
how explorer: sonnet
how explainer: opus
why investigators: sonnet
why synthesizer: opus
reflect tooling: sonnet
reflect judgment, divergent, synthesizer: opus
arena runners: opus, sonnet
arena cross-judge pool: opus, sonnet
swarm workers: sonnet
architect runners: opus, sonnet
interrogate reviewers: opus, sonnet
```

### 6. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever shuhari is installed: workspace, user, or plugin). On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, invoke `/create-formal-verification-skill` (resolves wherever shuhari is installed: workspace, user, or plugin). On no, move on without pushing.
