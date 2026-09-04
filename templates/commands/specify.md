---
description: Create or update the feature specification and feature memory (spec.md, picture.md, decisions/) from natural language, enforcing domain vocabulary and defining the primary testing seam.
handoffs:
  - label: Clarify Spec Requirements
    agent: speckit.clarify
    prompt: Clarify specification requirements
    send: true
  - label: Build Technical Plan
    agent: speckit.plan
    prompt: Create a plan for the spec. I am building with...
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before specification)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_specify` key
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

Transform a natural language feature description into a rigorous, testable specification and establish the **Feature-Level Second Brain** inside `specs/<feature-dir>/`.

Key Responsibilities:
1. **Grounding in Domain Context (`CONTEXT.md`)**: Enforcing canonical domain vocabulary and forbidding misleading synonyms.
2. **Scope & Feasibility Sizing (Wayfinding)**: Guarding against oversized, foggy requests by carving out viable slices and documenting deferred scope in "Fog of War".
3. **Defining the Primary Testing Seam**: Establishing the highest architectural boundary (API endpoint, CLI command, message interface) where the feature will be verified end-to-end.
4. **Dual-Document Initialization**:
   - `spec.md`: The prioritized user journeys (P1/P2/P3), acceptance scenarios, and functional requirements.
   - `picture.md`: The living state document tracking where the feature stands, decisions locked, and open forks across multi-turn sessions.
   - `decisions/`: Directory prepared for feature-level Architecture Decision Records (ADRs).

---

## Execution Workflow

### Phase 1: Load Project Memory & Context
1. **Domain Glossary (`.specify/memory/CONTEXT.md`)**:
   - Extract canonical entity names, actor roles, and forbidden terms (`_Avoid_`).
   - All user stories, acceptance criteria, and entity names in the specification **MUST** adhere to this vocabulary.
2. **The Constitution (`.specify/memory/constitution.md`)**:
   - Extract mandatory quality constraints, architectural principles (e.g. test-first, library-first), and performance thresholds.

### Phase 2: Scope & Fog Sizing Check (Wayfinding)
Analyze `$ARGUMENTS` to determine if the scope is appropriate for a single feature branch:
- **Appropriate Scope**: A cohesive capability deliverable in a single branch (e.g. "Add order cancellation", "Implement OAuth2 login").
- **Overly Broad / Foggy Scope**: Multi-system or company-wide initiatives (e.g. "Build an entire automated warehouse management and payment engine").
  - *Action if oversized*: Propose carving out **Slice 1 / Milestone 1**, and document the remaining unclear areas under `Fog of War / Out of Scope` to be ticketed in future features.

### Phase 3: Short Name & Feature Directory Resolution
1. Generate concise short name (2-4 words, kebab-case, e.g. `cancel-order`).
2. Resolve directory prefix: next sequential 3-digit number (`specs/001-cancel-order`).
3. Create directories:
   ```bash
   mkdir -p specs/<prefix>-<short-name>/decisions
   ```
4. Persist to `.specify/feature.json`:
   ```json
   {
     "feature_directory": "specs/<prefix>-<short-name>"
   }
   ```

### Phase 4: Primary Testing Seam Resolution
Identify the **highest architectural boundary** to verify this feature:
- Prefer an external or interface-level seam (e.g., `POST /api/orders/:id/cancel`, CLI command `store order cancel`, GraphQL mutation).
- The fewer seams, the better; the ideal target is **ONE** seam per feature slice.

### Phase 5: Synthesize `spec.md` and `picture.md`
1. **Generate `specs/<feature-dir>/spec.md`** using `.specify/templates/spec-template.md`:
   - Domain Vocabulary: Explicitly list Primary Actors, Core Entities, and forbidden synonyms from `CONTEXT.md`.
   - Testing Seam: Target boundary, contract type, expected status.
   - User Stories (Prioritized): P1 (Core MVP), P2 (Edge cases & validation), P3 (Polish), all with Given-When-Then.
   - Functional Requirements (FR-xxx): Max 3 `[NEEDS CLARIFICATION]` markers for downstream resolution.
   - Scope Boundaries & Fog of War: Explicitly document what is deferred.
2. **Generate `specs/<feature-dir>/picture.md`** using `.specify/templates/picture-template.md`:
   - Objective & Definition of Done.
   - Where We Stand: `Specification (Drafted)`.
   - Testing Seam.
   - Decisions Locked & Open Forks.
   - Evolution Trail with initial timestamp.

### Phase 6: Post-Execution & Handoff
Execute post-specify hooks from `.specify/extensions.yml` under `hooks.after_specify` if defined.

Completion Report:
1. Feature Directory: `specs/<prefix>-<short-name>/`
2. Primary Testing Seam: Designated boundary.
3. Next Command: Prompt user to run `__SPECKIT_COMMAND_CLARIFY__` or proceed to `__SPECKIT_COMMAND_PLAN__`.
