---
description: Decompose the architectural plan into an actionable, dependency-ordered tasks.md enforcing mandatory TDD (Red-Green-Refactor) and vertical tracer bullets per user story.
handoffs:
  - label: Implement Tasks
    agent: speckit.implement
    prompt: Implement the tasks from the tasks list
scripts:
  sh: scripts/bash/setup-tasks.sh --json
  ps: scripts/powershell/setup-tasks.ps1 -Json
  py: scripts/python/setup_tasks.py --json
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before tasks generation)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_tasks` key
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

Convert the architectural design (`plan.md`, `spec.md`, `data-model.md`, `contracts/`) into an executable, dependency-ordered `tasks.md`.

Every task is an independently grabbable unit of work sized for an AI agent's context window, governed by:
1. **The Iron Law of TDD (Non-Negotiable)**: Tests are NEVER optional. Every user story starts with an explicit `[TDD-Red]` seam test task that must fail before any implementation code is written.
2. **Vertical Slices (Tracer Bullets)**: Organize tasks by User Story (P1, P2, P3). User Story 1 (P1) is a complete, working, standalone MVP slice from the API seam to the database.
3. **Pre-factoring Isolation**: If `plan.md` identified pre-factoring needs, schedule them in Phase 1 before any feature code is added to ensure existing tests stay green.
4. **Exact File Paths & Parallel Tags (`[P]`)**: Every task must cite exact file paths and mark independent tasks with `[P]` for parallel subagent execution.

---

## Execution Workflow

### Phase 1: Context & Artifact Assembly
1. Resolve `FEATURE_DIR` and `TASKS_TEMPLATE`.
2. Load context sources:
   - `FEATURE_DIR/plan.md`: Technical context, pre-factoring assessment, file impact tree, testing seam.
   - `FEATURE_DIR/spec.md`: Prioritized user stories (P1/P2/P3), acceptance criteria.
   - `FEATURE_DIR/data-model.md`: Entities, relationships, schema migrations.
   - `FEATURE_DIR/contracts/`: API payload schemas, request/response formats.
   - `.specify/memory/constitution.md`: Core principles (TDD mandate, code standards).
   - `.specify/memory/CONTEXT.md`: Canonical domain vocabulary.

### Phase 2: Task Decomposition Rules
1. **Phase 1: Pre-factoring (Conditional)**:
   - If `plan.md` scheduled pre-factoring, create tasks to clean up existing structures and verify existing tests remain green.
2. **Phase 2: Foundational Data & Contracts**:
   - Tasks for database migrations and shared DTOs/interfaces.
3. **Phase 3+: User Stories as Tracer Bullets (P1, P2, P3...)**:
   - For each user story:
     1. `[TDD-Red]`: Write failing integration test exercising the primary testing seam.
     2. Verification: Run test, verify failure with expected assertion.
     3. `[Impl]`: Minimal implementation (model, service, controller/handler).
     4. `[TDD-Green]`: Run seam test, verify 100% green pass.
     5. `[Mutation]`: Revert minimal logic to prove test sensitivity.
4. **Phase N: Polish, Observability & Hardening**:
   - Structured logging, full regression suite, and `quickstart.md` validation.

### Phase 3: Synthesize `FEATURE_DIR/tasks.md`
Fill `.specify/templates/tasks-template.md` with:
- Strictly sequential task IDs (`T001`, `T002`...).
- Tag `[P]` for independent tasks.
- Concrete file paths in every single task.

### Phase 4: Living State Synchronization (`picture.md`)
Update `FEATURE_DIR/picture.md` to reflect `Tasks Ready` with total task counts and execution order.

### Phase 5: Post-Execution & Completion Report
Execute post-tasks hooks from `.specify/extensions.yml` under `hooks.after_tasks` if defined.

Report summary and suggest next command: `__SPECKIT_COMMAND_IMPLEMENT__`.
