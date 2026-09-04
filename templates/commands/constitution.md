---
description: Create or update the project constitution from interactive or provided principle inputs.
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

This command's own work is limited to updating the project constitution itself. Dependent templates
and commands read the constitution at runtime and are not modified here.
This command's own work is limited to establishing or updating the repo-native project memory in `.specify/memory/`.

- Classify every part of the user input as either constitution content or a separate,
  non-governance intent.
- If the input includes feature implementation, code generation, refactoring, building, or
  deployment requests, you **MUST NOT** execute them. Extract them as deferred intents instead.
- You **MUST NOT** create, modify, or delete application source files, feature routes,
  components, tests, deployment files, or other artifacts unrelated to the constitution
  workflow.
- If it is unclear whether an instruction is constitution content, ask for clarification before
  making changes.
- After completing the constitution update, include a `Next Actions` section for each deferred
  intent. List the original intent and suggest the appropriate follow-up Spec Kit command, such
  as `__SPECKIT_COMMAND_SPECIFY__`, without invoking it.
- If there are no non-governance intents, omit the `Next Actions` section.
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

## Outline
---

You are updating the project constitution at `.specify/memory/constitution.md`. The active
constitution scaffold is resolved at command time from `constitution-template` through the Spec Kit
preset/template resolution stack.
## Goal

Follow this execution flow:
You are establishing or amending the **Repo-Native Project Memory (Second Brain)** located in `.specify/memory/`.

1. Run `{SCRIPT}` from the repository root and parse `TEMPLATE_CONTENT` as the active template.
   - The shared resolver applies project overrides, composing preset layers, and extension layers
     before the core template fallback. It MUST succeed before continuing.
   - If it fails, stop and report the resolution error; do not continue with only one contributing
     template layer.
   - If `.specify/memory/constitution.md` exists, load it as the source of current project-specific
     values and amendments. Preserve information that is still applicable when applying the newly
     resolved scaffold.
   - If it does not exist, use the resolved template as the initial document.
   - Do not write back to any versioned template layer.
   - Identify every placeholder token of the form `[ALL_CAPS_IDENTIFIER]`.
   **IMPORTANT**: The user might require less or more principles than the ones used in the template. If a number is specified, respect that - follow the general template. You will update the doc accordingly.
Project Memory consists of three coordinated pillars:
1. **The Constitution (`.specify/memory/constitution.md`)**: The supreme law. 4 to 7 non-negotiable architectural principles, quality bars, and governance rules.
2. **The Domain Context (`.specify/memory/CONTEXT.md`)**: The canonical domain vocabulary (ubiquitous language), preventing entity naming drift and ambiguous terminology.
3. **The Situational Rule Map (`.specify/memory/rules/` and `_index.md`)**: Context-saving operational constraints (e.g. testing rules, git hygiene, safety guards) indexed by *situational triggers* (`Read this when...`) so agents only load relevant rules on-demand.

2. Collect/derive values for placeholders:
   - If user input (conversation) supplies a value, use it.
   - Otherwise infer from existing repo context (README, docs, prior constitution versions if embedded).
   - For governance dates: `RATIFICATION_DATE` is the original adoption date (if unknown ask or mark TODO), `LAST_AMENDED_DATE` is today if changes are made, otherwise keep previous.
   - `CONSTITUTION_VERSION` must increment according to semantic versioning rules:
     - MAJOR: Backward incompatible governance/principle removals or redefinitions.
     - MINOR: New principle/section added or materially expanded guidance.
     - PATCH: Clarifications, wording, typo fixes, non-semantic refinements.
   - If version bump type ambiguous, propose reasoning before finalizing.
---

3. Draft the updated constitution content using the resolved template as the required structure:
   - Replace every placeholder with concrete text (no bracketed tokens left except intentionally retained template slots that the project has chosen not to define yet—explicitly justify any left).
   - Preserve heading hierarchy and comments can be removed once replaced unless they still add clarifying guidance.
   - Ensure each Principle section: succinct name line, paragraph (or bullet list) capturing non‑negotiable rules, explicit rationale if not obvious.
   - Ensure Governance section lists amendment procedure, versioning policy, and compliance review expectations.
## Execution Workflow

4. Produce a Sync Impact Report (prepend as an HTML comment at top of the constitution file after update):
   - Version change: old → new
   - List of modified principles (old title → new title if renamed)
   - Added sections
   - Removed sections
   - Follow-up TODOs if any placeholders intentionally deferred.
### Phase 1: Repo Inspection & Discovery
Inspect the repository to infer technical context:
- Configs: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc.
- Testing frameworks and setups in `tests/`, `__tests__/`, or configuration.
- Existing documentation: `README.md`, `CONTRIBUTING.md`, existing `.specify/memory/` files.
- Inferred project name, tech stack, and testing runner.

5. Validation before final output:
   - No remaining unexplained bracket tokens.
   - Version line matches report.
   - Dates ISO format YYYY-MM-DD.
   - Principles are declarative, testable, and free of vague language ("should" → replace with MUST/SHOULD rationale where appropriate).
### Phase 2: Interactive Alignment / Input Resolution
If `$ARGUMENTS` provides specific instructions:
- Treat that input as the primary directive.
If running interactively (empty `$ARGUMENTS`):
- Propose 3-5 non-negotiable principles (Test-First TDD, High-seam testing, Simplicity/YAGNI, Explicit contracts).
- Propose core actors and domain vocabulary with forbidden terms to avoid.
- Propose operational rules (testing rules, git hygiene, safety guards).

6. Write the completed constitution back to `.specify/memory/constitution.md` (overwrite).
### Phase 3: Writing & Updating Project Memory
1. **The Constitution (`.specify/memory/constitution.md`)**:
   - Strictly declarative (MUST, MUST NOT, SHOULD).
   - SemVer increment (MAJOR, MINOR, PATCH).
   - Dates: `RATIFICATION_DATE` and `LAST_AMENDED_DATE` (YYYY-MM-DD).
2. **The Domain Context (`.specify/memory/CONTEXT.md`)**:
   - Canonical terms with 1-2 sentence definitions and explicit `_Avoid_` lists.
3. **The Situational Rule Map (`.specify/memory/rules/_index.md`)**:
   - Table of situational triggers mapping to rule files (e.g. `rules/testing.md`, `rules/git.md`, `rules/safety-guard.md`).

7. Output a final summary to the user with:
   - New version and bump rationale.
   - Any TODO placeholders or deferred items requiring manual follow-up.
   - Suggested commit message (e.g., `docs: amend constitution to vX.Y.Z (principle additions + governance update)`).
   - A `Next Actions` section for any deferred non-governance intents.
### Phase 4: Consistency & Template Propagation
- Update `.specify/templates/plan-template.md` (Constitution Check section).
- Update `.specify/templates/spec-template.md` and `tasks-template.md`.

Formatting & Style Requirements:
### Phase 5: Post-Execution & Completion Report
Execute post-constitution hooks from `.specify/extensions.yml` under `hooks.after_constitution` if defined.

- Use Markdown headings exactly as in the template (do not demote/promote levels).
- Wrap long rationale lines to keep readability (<100 chars ideally) but do not hard enforce with awkward breaks.
- Keep a single blank line between sections.
- Avoid trailing whitespace.

If the user supplies partial updates (e.g., only one principle revision), still perform validation and version decision steps.

If critical info missing (e.g., ratification date truly unknown), insert `TODO(<FIELD_NAME>): explanation` and include in the Sync Impact Report under deferred items.

Write only `.specify/memory/constitution.md`; do not create or modify template source files.

## Post-Execution Checks

**Check for extension hooks (after constitution update)**:
Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.after_constitution` key
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue normally
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook and leave condition evaluation to the HookExecutor implementation
- For each executable hook, output the following based on its `optional` flag:
  - **Optional hook** (`optional: true`):
    ```
    ## Extension Hooks

    **Optional Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```
  - **Mandatory hook** (`optional: false`):
    ```
    ## Extension Hooks

    **Automatic Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
    After emitting the block above you MUST actually invoke the hook and wait for it to finish before continuing. Run it the same way you would run the command yourself in this agent/session (the invocation may differ from the literal `{command}` id shown above, e.g. a skills-mode agent runs it as `/skill:speckit-...` or `$speckit-...`). Emitting the block alone does not run the hook.
- If no hooks are registered or `.specify/extensions.yml` does not exist, skip silently
Report summary to user:
- Constitution version and rationale.
- Domain terms registered in `CONTEXT.md`.
- Situational rules indexed.
- Suggested next command: `__SPECKIT_COMMAND_SPECIFY__`.
