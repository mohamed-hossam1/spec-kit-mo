---
description: Stress-test and clarify the active feature spec through relentless, context-rich grilling; resolve facts from code first, present visual trade-off options with recommendations, and lock decisions immediately into spec.md, picture.md, and local ADRs.
handoffs:
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

**Check for extension hooks (before clarification)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_clarify` key
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

Detect and eliminate ambiguity, unhandled edge cases, and missing architectural decision points in the active feature specification (`spec.md`) through a disciplined, interactive **Clarification & Grilling Loop**.

Every confirmed answer is **locked immediately**:
1. Written directly into `specs/<feature>/spec.md` (replacing `[NEEDS CLARIFICATION]` markers).
2. Documented in `specs/<feature>/picture.md` under `Decisions Locked`.
3. Captured as an **Architecture Decision Record (ADR)** under `specs/<feature>/decisions/` if it represents a significant, hard-to-reverse architectural trade-off.

---

## Prime Directives

1. **Facts vs. Decisions (Non-Negotiable)**:
   - **Finding facts is YOUR job, never the user's.** If an answer can be found by exploring the codebase, configuration, schema, or documentation, look it up immediately. Never ask the user a question you could answer yourself from the repo.
   - **Decisions belong to the user.** Put each fork, policy, or UX trade-off to the user with a recommended call, and wait for their decision.
2. **One Question at a Time**:
   - Never overwhelm the user with a wall of multiple questions. Ask exactly one question per turn.
3. **Always Provide "My Call"**:
   - Every question MUST include your recommended option, the engineering rationale, and the trade-off.
4. **Challenge against the Glossary (`CONTEXT.md`)**:
   - If the user uses vague or conflicting terminology, immediately guide them to the canonical domain term defined in `.specify/memory/CONTEXT.md`.
5. **Immediate In-Place Synchronization**:
   - Update `spec.md` and `picture.md` the turn the decision locks. Never batch updates to the end of the session.

---

## Execution Workflow

### Phase 1: Setup & Context Resolution
1. Identify active feature directory (`FEATURE_DIR`):
   - Check `.specify/feature.json` for `feature_directory` or run prerequisite checks.
   - If missing, halt and instruct the user to run `__SPECKIT_COMMAND_SPECIFY__` first.
2. Load context files:
   - `FEATURE_DIR/spec.md`: The active specification.
   - `FEATURE_DIR/picture.md`: The living state document.
   - `.specify/memory/constitution.md`: Project non-negotiable principles.
   - `.specify/memory/CONTEXT.md`: Canonical domain vocabulary.
   - Repository source code surrounding the testing seam (routes, models, services).

### Phase 2: Ambiguity & Completeness Scan (Design Tree)
Map the feature into a **Design Tree**: every high-level decision branches into the lower-level data model implications, boundary edge cases, and failure modes that hang off it.

Scan `spec.md` across:
1. Explicit `[NEEDS CLARIFICATION]` markers.
2. State Transitions & Invariants (triggers, terminals, concurrency).
3. Edge Cases & Boundary Failures (timeouts, downtimes, idempotency).
4. Data Model impact (cascades, uniqueness, audit).
5. Testing Seam Alignment (clean verification without brittle mocks).

**No Artificial Question Cap**:
Do **NOT** cap the interrogation at an arbitrary number. Continue grilling relentlessly down every branch of the design tree until:
1. **The Frontier is Empty**: Every branch visited, all edge cases resolved, zero hidden assumptions.
2. **Explicit User Early Termination**: User explicitly says "proceed", "done", or "let's plan".

### Phase 3: The Relentless Grilling Loop (Frontier Walk)
For each unresolved question on the frontier:

#### 1. Codebase Pre-Check (Fact Sweep)
Inspect the codebase first. Never ask what the code already tells you.

#### 2. Present Question in Context-Rich Visual Format:
````markdown
**Context Picture**
- 2-4 bullets explaining current flow, actors, and downstream effects.

```text
Current path:
Actor -> Action -> Current state effect

Proposed path:
Actor -> Changed action -> New state / API effect
```

**What We're Deciding**
[One clear sentence naming the architectural or business fork.]

**Options**
| Option | System Behavior | Low-Level Consequence | Trade-off |
|---|---|---|---|
| A | [Behavior description] | [DB / API / Queue impact] | [Pros / Cons] |
| B (Recommended) | [Behavior description] | [DB / API / Queue impact] | [Pros / Cons] |

**My Call**
[Clear recommendation stating why this option best satisfies the Constitution, performance, and UX.]

**Question**
[One standalone question for the user, easily answerable by option letter A, B or brief confirmation.]
````

Wait for the user's response before proceeding.

#### 3. Resolving Deadlocks: The Arena (Optional)
When a critical decision genuinely deadlocks between two strong architectures:
- Offer to run **The Arena**:
  1. Define 3-5 concrete gradeable criteria.
  2. Spin up candidate subagents to produce competing code/schema sketches in `/tmp/arena-<feature>-<slug>/candidate-<n>/`.
  3. Evaluate candidates against criteria and present the winning architecture back to the user.

### Phase 4: Lock Decision & Synchronize (In-Place)
Immediately upon receiving the user's choice:
1. **Update `spec.md`**: Replace `[NEEDS CLARIFICATION]` or append requirement using canonical `CONTEXT.md` terms.
2. **Update `picture.md`**: Move question from `Open Forks` to `Decisions Locked` with timestamp and rationale.
3. **ADR Evaluation**: If the decision is hard to reverse, surprising without context, and the result of a real trade-off, write an ADR in `specs/<feature>/decisions/NNNN-<slug>.md` using `.specify/templates/adr-template.md`.

### Phase 5: Re-Validate Spec Quality Checklist (If Exists)
If `FEATURE_DIR/checklists/requirements.md` exists, re-evaluate checkbox items against updated `spec.md`.

### Phase 6: Post-Execution & Completion Report
Execute post-clarification hooks from `.specify/extensions.yml` under `hooks.after_clarify` if defined.

Report summary to user and suggest next command: `__SPECKIT_COMMAND_PLAN__`.
