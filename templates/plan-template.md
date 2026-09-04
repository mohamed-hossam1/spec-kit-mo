# Implementation Plan: [FEATURE]
# Implementation Plan: [FEATURE_NAME]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**Feature Branch**: `[###-feature-name]`
**Spec Reference**: `specs/[FEATURE_DIR]/spec.md`
**Living State**: `specs/[FEATURE_DIR]/picture.md`
**Created**: [DATE]
**Status**: Ready for Tasks

**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`
---

**Note**: This template is filled in by the `/speckit-plan` command; its definition describes the execution workflow.
## 1. Executive Summary & Approach

## Summary
[Extract from feature spec: primary business requirement + technical approach agreed upon from research]

[Extract from feature spec: primary requirement + technical approach from research]
---

## Technical Context
## 2. Technical Context & Environment

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->
- **Language / Runtime**: [e.g. Node 20 / TypeScript 5.4, Python 3.12, Rust 1.78]
- **Primary Framework & Libraries**: [e.g. NestJS, FastAPI, Prisma, Tailwind]
- **Storage & State**: [e.g. PostgreSQL + Redis, SQLite, files or N/A]
- **Testing Runner**: [e.g. Jest, Vitest, pytest, cargo test]
- **Performance Targets**: [e.g. P95 latency < 150ms, throughput > 500 req/s]
- **Security & Concurrency Constraints**: [e.g. Idempotent webhook handling, row-level locks on balance mutation]

**Language/Version**: [e.g., Python 3.11, Swift 5.9, Rust 1.75 or NEEDS CLARIFICATION]
---

**Primary Dependencies**: [e.g., FastAPI, UIKit, LLVM or NEEDS CLARIFICATION]
## 3. Constitution Check & Quality Gates

**Storage**: [if applicable, e.g., PostgreSQL, CoreData, files or N/A]
*GATE: All items MUST pass or be explicitly justified in the Complexity Tracking section below before generating tasks.*

**Testing**: [e.g., pytest, XCTest, cargo test or NEEDS CLARIFICATION]
| Principle / Gate | Status | Compliance Verification & Evidence |
|---|---|---|
| **I. Test-First (TDD)** | PASS / FAIL | [How the red-green-refactor cycle will be enforced for this feature] |
| **II. High Testing Seam** | PASS / FAIL | [Verified through primary seam at: `[SEAM_PATH]` without internal mocks] |
| **III. Simplicity / YAGNI** | PASS / FAIL | [No unneeded abstractions, speculative generality, or premature microservices] |
| **IV. Explicit Contracts** | PASS / FAIL | [Standard error envelopes and validated schemas defined in `contracts/`] |
| **V. Performance & Safety** | PASS / FAIL | [Non-destructive operations, no table locks on large tables, safe rollbacks] |

**Target Platform**: [e.g., Linux server, iOS 15+, WASM or NEEDS CLARIFICATION]
---

**Project Type**: [e.g., library/cli/web-service/mobile-app/compiler/desktop-app or NEEDS CLARIFICATION]
## 4. Pre-factoring Assessment ("Make the change easy, then make the easy change")

**Performance Goals**: [domain-specific, e.g., 1000 req/s, 10k lines/sec, 60 fps or NEEDS CLARIFICATION]
<!--
  Assess whether existing codebase structures need refactoring BEFORE implementing the new feature.
  If existing code is tightly coupled or messy, prefactoring isolates the risk into its own safe commit.
-->

**Constraints**: [domain-specific, e.g., <200ms p95, <100MB memory, offline-capable or NEEDS CLARIFICATION]
- **Prefactoring Required?**: [YES / NO]
- **Existing Code Smells / Bottlenecks**: [e.g., God class in `order.service.ts` or scattered status enums]
- **Pre-factoring Plan**:
  1. [Step 1: Extract helper or split service before adding new logic]
  2. [Step 2: Ensure existing test suite remains green]

**Scale/Scope**: [domain-specific, e.g., 10k users, 1M LOC, 50 screens or NEEDS CLARIFICATION]
---

## Constitution Check
## 5. Architectural Deliverables

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

[Gates determined based on constitution file]

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
specs/[FEATURE_DIR]/
├── plan.md              # This master blueprint
├── research.md          # Technical choices, library evaluations, and benchmarks
├── data-model.md        # Entities, migrations, and database schema changes
├── quickstart.md        # Step-by-step verification and integration test walkthrough
├── contracts/           # API schemas, payload envelopes, and interface definitions
│   └── [contract-files]
└── decisions/           # Local ADR records for architectural trade-offs
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->
---

## 6. Target Codebase Structure & File Impact

### Source Code Changes
```text
# [REMOVE IF UNUSED] Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/
# Files to Create [NEW]
[NEW] src/modules/[feature]/[new-service].ts
[NEW] src/modules/[feature]/[new-controller].ts
[NEW] tests/integration/[feature]/[seam-test].spec.ts

tests/
├── contract/
├── integration/
└── unit/
# Files to Modify [MODIFY]
[MODIFY] src/database/schema.prisma (or migrations)
[MODIFY] src/modules/[existing-module]/[existing-file].ts

# [REMOVE IF UNUSED] Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/
# Files to Delete / Deprecate [DELETE]
[DELETE] (none or deprecated files)
```

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/
---

# [REMOVE IF UNUSED] Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]
## 7. Primary Testing Seam & Verification Plan

ios/ or android/
└── [platform-specific structure: feature modules, UI flows, platform tests]
```
- **Primary Seam**: `[HTTP Method / CLI command / Message Topic]`
- **End-to-End Verification Flow**:
  1. **Red Stage**: Run integration test against seam with missing implementation $\to$ must fail with clear assertion error.
  2. **Green Stage**: Implement prefactoring + minimal feature code $\to$ test turns green.
  3. **Refactor & Mutation Stage**: Revert implementation change $\to$ verify test fails immediately (mutation confidence).

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]
---

## Complexity Tracking
## 8. Complexity & Trade-Off Justification

> **Fill ONLY if Constitution Check has violations that must be justified**
> **Fill ONLY if Constitution Check has exceptions or violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
| Violation / Added Complexity | Why It Is Strictly Necessary | Simpler Alternative Rejected Because |
|---|---|---|
| [e.g. New Redis dependency] | [Async queue required for webhook retries] | [In-memory queue loses jobs on restart] |
