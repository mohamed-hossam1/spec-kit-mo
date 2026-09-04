---
description: Architect the implementation plan from the clarified specification, assess pre-factoring, generate data models and API contracts, enforce constitution quality gates, and synchronize the feature living state.
handoffs:
  - label: Generate Implementation Tasks
    agent: speckit.tasks
    prompt: Generate implementation tasks from the plan
scripts:
  sh: scripts/bash/setup-plan.sh --json
  ps: scripts/powershell/setup-plan.ps1 -Json
  py: scripts/python/setup_plan.py --json
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before planning)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_plan` key
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
    After emitting the block above you MUST actually invoke the hook and wait for it to finish before continuing. Run it the same way you would run the command yourself in this agent/session. Emitting the block alone does not run the hook.
- If no hooks are registered or `.specify/extensions.yml` does not exist, skip silently

---

## Goal

Produce a complete, robust architectural blueprint for the active feature.

The plan bridges functional requirements (`spec.md`) and executable tasks (`tasks.md`) by generating:
1. **Pre-factoring Assessment**: Checking if existing code needs structural cleanup first ("Make the change easy, then make the easy change").
2. **`research.md`**: Concrete technical evaluations, dependencies, and trade-offs.
3. **`data-model.md`**: Schemas, fields, relationships, state transitions, and migration paths.
4. **`contracts/`**: Formal API payload contracts, endpoints, and error envelopes.
5. **`quickstart.md`**: Step-by-step verification flows exercising the primary testing seam.
6. **`plan.md`**: Master blueprint incorporating the **Constitution Check Gate** and file impact tree.
7. **Living State Sync (`picture.md`)**: Updating where the feature stands and locking architectural calls.

---

## Execution Workflow

### Phase 1: Setup & Context Assembly
1. Run setup script or read `.specify/feature.json` to resolve `FEATURE_DIR`, `FEATURE_SPEC`, `IMPL_PLAN`, and `FEATURE_PICTURE`.
2. Load context sources:
   - `FEATURE_SPEC`: Clarified specification, user stories (P1/P2/P3), and designated testing seam.
   - `.specify/memory/constitution.md`: Supreme project principles.
   - `.specify/memory/CONTEXT.md`: Canonical domain vocabulary.
   - Target source tree: Inspect existing codebase files and database schemas.

### Phase 2: Pre-factoring & Research (`research.md`)
1. **Pre-factoring Assessment**: Inspect existing code where the feature lands. If high coupling or god classes exist, formulate a Pre-factoring Step to isolate cleanup into safe preparatory commits.
2. **Technical Research**: Evaluate libraries and tools, documenting decisions, rationales, and rejected alternatives in `research.md`.

### Phase 3: Architectural Deliverables (Data & Contracts)
1. **Data Modeling (`data-model.md`)**: Fields, types, relationships, state transitions, and database migration steps.
2. **Interface Contracts (`contracts/`)**: Formal request payload schemas, success envelopes, and structured error formats.
3. **Quickstart Verification Guide (`quickstart.md`)**: Step-by-step instructions to verify the primary testing seam end-to-end.

### Phase 4: Constitution Gate & Plan Synthesis
1. **Constitution Check Gate**: Evaluate the draft plan against `.specify/memory/constitution.md` (TDD compliance, high testing seam, YAGNI simplicity, explicit contracts). If violations exist, they MUST be documented and justified in the Complexity Tracking table of `plan.md`.
2. **Synthesize `plan.md`**: Fill `.specify/templates/plan-template.md` with technical context, constitution gates, pre-factoring findings, file impact tree (`[NEW]`, `[MODIFY]`, `[DELETE]`), and testing seam strategy.

### Phase 5: Living State Synchronization (`picture.md`)
Update `FEATURE_DIR/picture.md`:
- Set `Where We Stand` to `Planning Complete. Architecture locked.`
- Record locked architectural decisions.
- Add dated evolution trail entry.

### Phase 6: Post-Execution & Completion Report
Execute post-planning hooks from `.specify/extensions.yml` under `hooks.after_plan` if defined.

Report completion and prompt user for next step: `__SPECKIT_COMMAND_TASKS__`.
