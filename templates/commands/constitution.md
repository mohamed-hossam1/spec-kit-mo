---
description: Establish or amend the repo-native project memory (constitution, domain context, and situational rule map).
handoffs:
  - label: Build Specification
    agent: speckit.specify
    prompt: Implement the feature specification based on the updated constitution. I want to build...
scripts:
  sh: scripts/bash/resolve-template.sh constitution-template --json
  ps: scripts/powershell/resolve-template.ps1 constitution-template -Json
  py: scripts/python/resolve_template.py constitution-template --json
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Scope Guard

This command's own work is limited to establishing or updating the repo-native project memory in `.specify/memory/`.

- Classify every part of the user input as either project memory content (principles, domain vocabulary, operational rules) or a separate, non-governance intent.
- If the input includes feature implementation or coding requests, you **MUST NOT** execute them. Extract them as deferred intents instead.
- You **MUST NOT** create or modify application source code files outside of `.specify/memory/` and `.specify/templates/`.
- After completing the update, suggest the appropriate follow-up Spec Kit command, such as `__SPECKIT_COMMAND_SPECIFY__`.

## Pre-Execution Checks

**Check for extension hooks (before constitution update)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_constitution` key
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue normally
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook and leave condition evaluation to the HookExecutor implementation
- When constructing slash commands from hook command names, replace dots (`.`) with hyphens (`-`). For example, `speckit.git.commit` → `/speckit-git-commit`.
- For each executable hook, output the following based on its `optional` flag:
  - **Optional hook** (`optional: true`):
    ```
    ## Extension Hooks

    **Optional Pre-Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```
  - **Mandatory hook** (`optional: false`):
    ```
    ## Extension Hooks

    **Automatic Pre-Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}

    Wait for the result of the hook command before proceeding to the Outline.
    ```
    After emitting the block above you MUST actually invoke the hook and wait for it to finish before continuing. Run it the same way you would run the command yourself in this agent/session (the invocation may differ from the literal `{command}` id shown above, e.g. a skills-mode agent runs it as `/skill:speckit-...` or `$speckit-...`). Emitting the block alone does not run the hook.
- If no hooks are registered or `.specify/extensions.yml` does not exist, skip silently

---

## Goal

You are establishing or amending the **Repo-Native Project Memory (Second Brain)** located in `.specify/memory/`.

Project Memory consists of three coordinated pillars:
1. **The Constitution (`.specify/memory/constitution.md`)**: The supreme law. 4 to 7 non-negotiable architectural principles, quality bars, and governance rules.
2. **The Domain Context (`.specify/memory/CONTEXT.md`)**: The canonical domain vocabulary (ubiquitous language), preventing entity naming drift and ambiguous terminology.
3. **The Situational Rule Map (`.specify/memory/rules/` and `_index.md`)**: Context-saving operational constraints (e.g. testing rules, git hygiene, safety guards) indexed by *situational triggers* (`Read this when...`) so agents only load relevant rules on-demand.

---

## Execution Workflow

### Phase 1: Repo Inspection & Discovery
Inspect the repository to infer technical context:
- Configs: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc.
- Testing frameworks and setups in `tests/`, `__tests__/`, or configuration.
- Existing documentation: `README.md`, `CONTRIBUTING.md`, existing `.specify/memory/` files.
- Inferred project name, tech stack, and testing runner.

### Phase 2: Interactive Alignment / Input Resolution
If `$ARGUMENTS` provides specific instructions:
- Treat that input as the primary directive.
If running interactively (empty `$ARGUMENTS`):
- Propose 3-5 non-negotiable principles (Test-First TDD, High-seam testing, Simplicity/YAGNI, Explicit contracts).
- Propose core actors and domain vocabulary with forbidden terms to avoid.
- Propose operational rules (testing rules, git hygiene, safety guards).

### Phase 3: Writing & Updating Project Memory
1. **The Constitution (`.specify/memory/constitution.md`)**:
   - Strictly declarative (MUST, MUST NOT, SHOULD).
   - SemVer increment (MAJOR, MINOR, PATCH).
   - Dates: `RATIFICATION_DATE` and `LAST_AMENDED_DATE` (YYYY-MM-DD).
2. **The Domain Context (`.specify/memory/CONTEXT.md`)**:
   - Canonical terms with 1-2 sentence definitions and explicit `_Avoid_` lists.
3. **The Situational Rule Map (`.specify/memory/rules/_index.md`)**:
   - Table of situational triggers mapping to rule files (e.g. `rules/testing.md`, `rules/git.md`, `rules/safety-guard.md`).

### Phase 4: Consistency & Template Propagation
- Update `.specify/templates/plan-template.md` (Constitution Check section).
- Update `.specify/templates/spec-template.md` and `tasks-template.md`.

### Phase 5: Post-Execution & Completion Report
Execute post-constitution hooks from `.specify/extensions.yml` under `hooks.after_constitution` if defined.

Report summary to user:
- Constitution version and rationale.
- Domain terms registered in `CONTEXT.md`.
- Situational rules indexed.
- Suggested next command: `__SPECKIT_COMMAND_SPECIFY__`.
