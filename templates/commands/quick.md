---
description: Fast-track feature definition - synthesize specification and implementation plan in one shot for simple features, ready for tasks and implementation.
handoffs:
  - label: Generate Implementation Tasks
    agent: speckit.tasks
    prompt: Generate implementation tasks from the plan and spec
scripts:
  sh: scripts/bash/create-new-feature.sh --json
  ps: scripts/powershell/create-new-feature.ps1 -Json
  py: scripts/python/create_new_feature.py --json
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before quick feature)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_quick` or `hooks.before_specify` key
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

**The Fast-Track Feature Pipeline**: Synthesize the **Specification (`spec.md`)** and the **Implementation Plan (`plan.md`)** in a single, cohesive, frictionless step.

This command is designed for simple, focused, or medium features (e.g. adding a new button/component, new validation rule, simple endpoint, isolated refactor) where going through separate `specify` ➡️ `clarify` ➡️ `plan` steps introduces unnecessary delay.

After running `__SPECKIT_COMMAND_QUICK__`, the feature is immediately ready for `__SPECKIT_COMMAND_TASKS__` and `__SPECKIT_COMMAND_IMPLEMENT__`.

---

## Execution Workflow

### Phase 1: Context Resolution & Directory Setup

1. The text the user typed after `__SPECKIT_COMMAND_QUICK__` in the triggering message **is** the feature description. Assume you always have it available in this conversation even if `{ARGS}` appears literally below.
2. Generate a concise short name (2-4 words, kebab-case, e.g. `add-copy-button`, `user-avatar-upload`).
3. Run `{SCRIPT}` from repo root with arguments:
   `--json "$ARGUMENTS"`
   - Parse JSON for `FEATURE_DIR`, `SPEC_FILE`, and ensure `.specify/feature.json` is persisted.
   - All paths must be absolute.
4. Load architectural memory:
   - Read `.specify/memory/constitution.md`: Core governance principles and quality rules.
   - Read `.specify/memory/CONTEXT.md`: Canonical domain vocabulary.
   - Read `.specify/memory/rules/`: Any applicable situational rules.

---

### Phase 2: Scope Assessment

Analyze `$ARGUMENTS`:
- **Simple / Focused Scope (Ideal for Quick)**: Isolated feature or enhancement affecting 1-4 files with clear requirements.
- **Architectural Epic (Too Large for Quick)**: If the request involves multi-service redesigns, ambiguous business domains, or massive migrations, warn the user:
  > "⚠️ This feature appears to be a large architectural epic. We will proceed with a focused slice in Fast-Track mode, but for high ambiguity you may prefer the full `__SPECKIT_COMMAND_SPECIFY__` ➡️ `__SPECKIT_COMMAND_CLARIFY__` ➡️ `__SPECKIT_COMMAND_PLAN__` flow."

---

### Phase 3: Synthesize Specification (`FEATURE_DIR/spec.md`)

Write a clean, testable specification directly to `FEATURE_DIR/spec.md` incorporating:

1. **Feature Overview & Objective**: What capability is being added and why.
2. **Domain Vocabulary**: Strictly use terms from `CONTEXT.md`.
3. **User Story (P1 MVP)**:
   - E.g.: `As a <role>, I want <action> so that <benefit>.`
4. **Acceptance Criteria (Given-When-Then)**:
   - 2-4 concrete, testable scenarios including edge cases (e.g. empty input, failure handling).
5. **Primary Testing Seam**:
   - Explicitly identify the highest architectural boundary to test (e.g. API route, CLI handler, UI component).
6. **Out of Scope**:
   - Explicitly state what is NOT included in this change.

---

### Phase 4: Synthesize Implementation Plan (`FEATURE_DIR/plan.md`)

In the same feature directory, write `FEATURE_DIR/plan.md` to establish technical certainty:

1. **Technical Context**:
   - Stack, dependencies, and execution environment.
2. **File Impact & Touch-points**:
   - Exact files to create or modify (e.g. `src/components/CopyButton.tsx`, `tests/components/CopyButton.test.tsx`).
3. **Primary Testing Seam Strategy**:
   - Seam file path and terminal test command (e.g. `npm test -- path/to/seam.test.ts` or `pytest tests/seams/test_seam.py`).
4. **Pre-factoring Check**:
   - Verify that existing regression tests are 100% green before touching feature code.
5. **Constitution Gate Check**:
   - Confirm compliance with `.specify/memory/constitution.md` (no MUST violations).

---

### Phase 5: Initialize Living State (`FEATURE_DIR/picture.md`)

Write `FEATURE_DIR/picture.md` to track state:
- **Status**: `Specified & Planned (Fast-Track) — Ready for Tasks`
- **Testing Seam**: Identified seam path and verification command
- **Milestone**: `Phase 1: Ready for __SPECKIT_COMMAND_TASKS__`

---

### Phase 6: Completion Report & Handoff

Present a concise summary to the user:

```markdown
# ⚡ Fast-Track Feature Initialized

**Feature Directory**: `specs/<feature-dir>/`
**Testing Seam**: `<seam-path>`

### Generated Artifacts:
1. `spec.md` - Specification with Acceptance Scenarios (Given-When-Then) & Seam
2. `plan.md` - Implementation Plan with file impact, architecture, and testing strategy
3. `picture.md` - Living state tracking initialized

### Next Step:
Run `__SPECKIT_COMMAND_TASKS__` to generate actionable TDD tasks, then `__SPECKIT_COMMAND_IMPLEMENT__` to execute them!
```

---

## Done When

- [ ] Feature directory created under `specs/` and saved to `.specify/feature.json`
- [ ] `FEATURE_DIR/spec.md` written with User Stories, Acceptance Criteria, and Testing Seam
- [ ] `FEATURE_DIR/plan.md` written with file impact, technical approach, and testing strategy
- [ ] `FEATURE_DIR/picture.md` initialized with living state
- [ ] Next step prompt presented to run `__SPECKIT_COMMAND_TASKS__`

