---
description: Audit codebase completeness against spec, plan, and tasks; append remediation tasks for unbuilt work, then run two-axis review (comprehensiveness & standards smells) to seal merge confidence.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-spec --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireSpec -RequireTasks -IncludeTasks
  py: scripts/python/check_prerequisites.py --json --require-spec --require-tasks --include-tasks
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before convergence)**:

- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_converge` key
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue normally
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook and leave condition evaluation to the HookExecutor implementation
- When constructing slash commands from hook command names, replace dots (`.`) with hyphens (`-`). For example, `speckit.git.commit` → `/speckit-git-commit`.
- For each executable hook, output the following based on its `optional` flag:
  - **Optional hook** (`optional: true`):

    ```text
    ## Extension Hooks

    **Optional Pre-Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```

  - **Mandatory hook** (`optional: false`):

    ```text
    ## Extension Hooks

    **Automatic Pre-Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}

    Wait for the result of the hook command before proceeding to the Goal.
    ```
    After emitting the block above you MUST actually invoke the hook and wait for it to finish before continuing. Run it the same way you would run the command yourself in this agent/session (the invocation may differ from the literal `{command}` id shown above, e.g. a skills-mode agent runs it as `/skill:speckit-...` or `$speckit-...`). Emitting the block alone does not run the hook.

- If no hooks are registered or `.specify/extensions.yml` does not exist, skip silently

---

## Goal

Close the gap between what a feature's specification, architectural plan, and tasks promise, and what the codebase actually delivers.

This command acts as the **Final Quality & Completeness Auditor**:

1. **Functional Convergence**: Verifies that every requirement (FR-xxx), user story journey, acceptance scenario, API contract, and Constitution principle has working production code and passing tests. Any gap is **appended as a new remediation task** to `tasks.md` for `__SPECKIT_COMMAND_IMPLEMENT__` to complete.
2. **Two-Axis Quality Review (Spec vs. Standards)**:
   - **Axis A (Comprehensiveness)**: Audits 8 critical engineering dimensions (side effects, edge cases, accuracy, redundancy, failure modes, concurrency/race conditions, observability, reversibility).
   - **Axis B (Standards & Fowler Smells)**: Sweeps for code smells (Primitive Obsession, Feature Envy, Data Clumps, Shotgun Surgery, Speculative Generality).
3. **Merge Confidence & Living State Finalization**: Seals `picture.md` as `100% Converged & Verified`, clearing the feature branch for pull request.

---

## Operating Constraints

1. **APPEND-ONLY TO TASKS.MD**: The command's only permitted write to `tasks.md` is appending a new `## Phase N: Convergence Remediation` section. It MUST NOT:
   - Modify `spec.md` or `plan.md` in any way;
   - Rewrite, renumber, reorder, or delete any existing task;
   - Modify, create, or delete any application code — completing the appended tasks is the exclusive domain of `__SPECKIT_COMMAND_IMPLEMENT__`.
2. **Zero False Positives**: Do not invent requirements outside of `spec.md`, `plan.md`, `contracts/`, and `.specify/memory/constitution.md`.
3. **Evidence-Backed Verdicts**: Every finding reported MUST carry an exact `file:line` citation opened and verified directly in the codebase.
4. **Constitution Authority**: The project constitution (`.specify/memory/constitution.md`) is non-negotiable. Code violating a MUST principle is the highest-severity finding.

---

## Execution Workflow

### Phase 1: Context & Artifact Loading

1. Run `{SCRIPT}` once from repo root and parse JSON for `FEATURE_DIR` and `AVAILABLE_DOCS`. Derive absolute paths:
   - `SPEC` = `FEATURE_DIR/spec.md`
   - `PLAN` = `FEATURE_DIR/plan.md`
   - `TASKS` = `FEATURE_DIR/tasks.md`
   - `PICTURE` = `FEATURE_DIR/picture.md`
   - `CONSTITUTION` = `.specify/memory/constitution.md` (or `/memory/constitution.md`)
   - `CONTEXT` = `.specify/memory/CONTEXT.md`

   If `spec.md`, `plan.md`, or `tasks.md` is missing, STOP with a clear, actionable message naming the prerequisite command to run (`__SPECKIT_COMMAND_SPECIFY__` for a missing spec, `__SPECKIT_COMMAND_PLAN__` for a missing plan, `__SPECKIT_COMMAND_TASKS__` for missing tasks).

2. Load source artifacts:
   - `spec.md`: User Stories (P1/P2/P3), Acceptance Scenarios (Given-When-Then), Functional Requirements (FR-xxx), and Testing Seam.
   - `plan.md`: Technical context, file impact tree, testing strategy, and complexity tracking.
   - `data-model.md`: Entities, attributes, constraints, and migration status.
   - `contracts/`: API request/response schemas and error envelopes.
   - `constitution.md`: Non-negotiable principles.

---

### Phase 2: Functional Scope Audit

Audit the current codebase against each requirement:

1. **Acceptance Scenario Verification**:
   - For every Acceptance Scenario in `spec.md`: locate the test exercising the testing seam.
   - Does a test exist? Does the test actively assert this scenario? Is the production implementation in place?
2. **Contracts & Envelopes Verification**:
   - Verify every endpoint in `contracts/` exists in the routes/controllers.
   - Verify error codes match canonical constants.
3. **Data Integrity & Constraints**:
   - Verify database migrations have run and schema models match `data-model.md`.
4. **Constitution Compliance**:
   - Verify Iron Law of TDD: every new service/endpoint has automated test coverage.

#### Classify Findings:
- **`missing`**: The required work or test is absent entirely.
- **`partial`**: The work exists but does not yet fully satisfy the requirement / acceptance criterion / error envelope.
- **`contradicts`**: The code does something that conflicts with stated intent or a constitution MUST principle.
- **`unrequested`**: The code contains additions not called for by the spec, plan, or tasks.

#### Severity Grading:
- **CRITICAL**: Violates a constitution MUST principle, or missing/contradicting baseline functionality of a P1 user story.
- **HIGH**: Missing or partial gap on a core functional requirement or acceptance scenario.
- **MEDIUM**: Partial gap on a secondary requirement, or unrequested addition with unclear justification.
- **LOW**: Minor partial gaps, polish, or low-risk unrequested additions.

---

### Phase 3: Remediate or Advance

#### Path A: Actionable Functional Gaps Found (`tasks_appended`)

If any functional requirements are missing, partial, or contradictory:

1. Scan all existing task IDs in `tasks.md`; let `M` be the maximum ID. Determine the next phase number `N` (highest existing phase + 1).
2. Append a new section to the bottom of `tasks.md`:
   ```markdown
   ## Phase N: Convergence Remediation

   - [ ] T{M+1:03d} [TDD-Red] Write failing seam test for <requirement-ref> per <source-ref> (<gap-type>)
   - [ ] T{M+2:03d} [Impl] Implement missing behavior in <file-path>
   - [ ] T{M+3:03d} [TDD-Green] Verify seam test turns green
   ```
   *Constitution-violation tasks MUST be emitted first and marked CRITICAL.*
3. Update `FEATURE_DIR/picture.md`:
   - Under `## 2. Where We Stand`, record that Convergence identified `X` gaps awaiting `__SPECKIT_COMMAND_IMPLEMENT__`.
4. Output Convergence Report:
   - Display summary table of findings.
   - Instruct the user: **"Appended X remediation tasks under Phase N. Run `__SPECKIT_COMMAND_IMPLEMENT__` to complete them."**
5. **HALT here** until implementation closes the functional gaps.

#### Path B: Zero Functional Gaps Found (`converged`)

Proceed immediately to Phase 4 (Two-Axis Quality Review).

---

### Phase 4: Two-Axis Quality Review

#### Axis A: The 8 Dimensions of Comprehensiveness

Examine the implementation across 8 correctness dimensions:

1. **Side Effects**: Does this change inadvertently mutate unrelated state, cache, or external records?
2. **Edge Cases**: Empty lists, boundary IDs, null fields, expired tokens handled cleanly?
3. **Accuracy**: Does math, currency, or timestamp handling preserve exact precision?
4. **Redundancy**: Any dead code, commented-out blocks, or unused imports left behind?
5. **Failure Modes**: What happens when network drops, third-party API times out, or database connection is reset?
6. **Concurrency**: Idempotency keys present? Race condition locks on balance/state transitions?
7. **Observability**: Are structured logs emitted with correlation IDs and entity identifiers?
8. **Reversibility**: Can the operation be safely rolled back without data corruption?

#### Axis B: Standards & Fowler Smell Baseline

Scan modified files against canonical code smells:

- **Primitive Obsession**: Raw strings/numbers used where a branded type or enum belongs.
- **Feature Envy**: Service reaches into another entity's internal fields excessively.
- **Data Clumps**: Same 3-4 parameters constantly traveling together without a dedicated DTO.
- **Shotgun Surgery**: Unnecessary widespread edits across unrelated directories.
- **Speculative Generality**: Unused helper hooks or over-engineered abstractions not demanded by the spec.

---

### Phase 5: Merge Confidence & Living State Finalization

When functional scope is 100% converged and the Two-Axis review is clean:

1. **Update `FEATURE_DIR/picture.md`**:
   - Update `## 1. Objective & Definition of Done`: Mark as `COMPLETED`.
   - Update `## 2. Where We Stand`: Set status to:
     ```markdown
     Status: 100% Converged & Quality Audited. Ready for Pull Request.

     - Functional Scope: 100% Verified (0 gaps)
     - Quality Review: 8 Dimensions Clean, 0 Code Smells
     - Tests: All seam integration and unit tests passing green
     ```
   - Update `## 6. Evolution Trail`: Add final convergence and audit milestone.

2. **Produce Final Completion Report**:

   ```markdown
   # 🚀 Convergence & Review Seal: PASS (100% Converged)

   ## 1. Scope Convergence
   - Requirements Checked: [COUNT] (100% Implemented)
   - Testing Seams Verified: [SEAM_PATH] (Green)
   - Database Migrations: Verified in sync

   ## 2. Quality Audit Summary
   - Comprehensiveness: 8 Dimensions Checked (0 critical regressions)
   - Code Smells: 0 Fowler smells detected
   - Project Constitution: Fully Compliant

   ## 3. Next Action
   This feature branch is fully verified, complete, and ready for Pull Request!
   ```

---

### Phase 6: Post-Execution Hooks

After producing the result, check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.after_converge` key
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue normally
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook and leave condition evaluation to the HookExecutor implementation
- When constructing slash commands from hook command names, replace dots (`.`) with hyphens (`-`).
- Output and execute mandatory hooks or list optional hooks per standard conventions.

---

## Done When

- [ ] All functional requirements, acceptance scenarios, and contracts audited against codebase
- [ ] Any functional gaps appended as actionable TDD tasks in `tasks.md` (or confirmed 0 gaps)
- [ ] Two-Axis quality review completed (8 dimensions + Fowler smells)
- [ ] `picture.md` living state sealed and finalized
- [ ] Post-execution hooks processed
- [ ] Final convergence verdict presented to user
