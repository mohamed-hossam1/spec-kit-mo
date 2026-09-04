---

description: "Task list template for feature implementation"
description: "Task list template for feature implementation with mandatory TDD and vertical slices"
---

# Tasks: [FEATURE NAME]
# Tasks: [FEATURE_NAME]

**Input**: Design documents from `/specs/[###-feature-name]/`
**Feature Branch**: `[###-feature-name]`
**Spec Reference**: `specs/[FEATURE_DIR]/spec.md`
**Plan Reference**: `specs/[FEATURE_DIR]/plan.md`
**Living State**: `specs/[FEATURE_DIR]/picture.md`
**Primary Testing Seam**: `[SEAM_PATH]`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: The examples below include test tasks. Tests are OPTIONAL - only include them if explicitly requested in the feature specification.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Single project**: `src/`, `tests/` at repository root
- **Web app**: `backend/src/`, `frontend/src/`
- **Mobile**: `api/src/`, `ios/src/` or `android/src/`
- Paths shown below assume single project - adjust based on plan.md structure

<!--
  ============================================================================
  IMPORTANT: The tasks below are SAMPLE TASKS for illustration purposes only.

  The /speckit-tasks command MUST replace these with actual tasks based on:
  - User stories from spec.md (with their priorities P1, P2, P3...)
  - Feature requirements from plan.md
  - Entities from data-model.md
  - Endpoints from contracts/

  Tasks MUST be organized by user story so each story can be:
  - Implemented independently
  - Tested independently
  - Delivered as an MVP increment

  DO NOT keep these sample tasks in the generated tasks.md file.
  ============================================================================
-->

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project structure per implementation plan
- [ ] T002 Initialize [language] project with [framework] dependencies
- [ ] T003 [P] Configure linting and formatting tools

---

## Phase 2: Foundational (Blocking Prerequisites)
## Task Standards & Format

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented
Every task follows the format:
```text
- [ ] [ID] [P?] [Story?] [Kind] Description with exact/relative/file/paths.ext
```
- **[ID]**: Sequential identifier (`T001`, `T002`...)
- **[P]**: Parallelizable (touches independent files, no unfinished dependencies)
- **[Story]**: User Story link (e.g. `[US1]`, `[US2]`)
- **[Kind]**: `[Pre-factor]`, `[Migration]`, `[TDD-Red]`, `[Impl]`, `[TDD-Green]`, `[Mutation]`, `[Polish]`

**⚠️ CRITICAL**: No user story work can begin until this phase is complete
> **THE IRON LAW OF TDD (NON-NEGOTIABLE)**:
> Tests are NEVER optional. Within each User Story, the failing seam test (`[TDD-Red]`) MUST be written and proven to fail BEFORE writing any production implementation code (`[Impl]`).

Examples of foundational tasks (adjust based on your project):

- [ ] T004 Setup database schema and migrations framework
- [ ] T005 [P] Implement authentication/authorization framework
- [ ] T006 [P] Setup API routing and middleware structure
- [ ] T007 Create base models/entities that all stories depend on
- [ ] T008 Configure error handling and logging infrastructure
- [ ] T009 Setup environment configuration management

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - [Title] (Priority: P1) 🎯 MVP
## Phase 1: Pre-factoring & Structural Preparation (If scheduled in plan.md)

**Goal**: [Brief description of what this story delivers]
**Goal**: Prepare and clean existing codebase structures so adding the new feature does not degrade architecture ("Make the change easy, then make the easy change").

**Independent Test**: [How to verify this story works on its own]
- [ ] T001 [Pre-factor] [Description of cleanup / helper extraction in existing files]
- [ ] T002 [Pre-factor] Run existing test suite to verify baseline regression safety (Must stay 100% green)

### Tests for User Story 1 (OPTIONAL - only if tests requested) ⚠️

> **NOTE: Write these tests FIRST, ensure they FAIL before implementation**

- [ ] T010 [P] [US1] Contract test for [endpoint] in tests/contract/test_[name].py
- [ ] T011 [P] [US1] Integration test for [user journey] in tests/integration/test_[name].py

### Implementation for User Story 1

- [ ] T012 [P] [US1] Create [Entity1] model in src/models/[entity1].py
- [ ] T013 [P] [US1] Create [Entity2] model in src/models/[entity2].py
- [ ] T014 [US1] Implement [Service] in src/services/[service].py (depends on T012, T013)
- [ ] T015 [US1] Implement [endpoint/feature] in src/[location]/[file].py
- [ ] T016 [US1] Add validation and error handling
- [ ] T017 [US1] Add logging for user story 1 operations

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - [Title] (Priority: P2)
## Phase 2: Foundational Data & Contracts Layer

**Goal**: [Brief description of what this story delivers]
**Goal**: Establish migrations, schemas, and shared types that block all subsequent user stories.

**Independent Test**: [How to verify this story works on its own]
- [ ] T003 [P] [Migration] Apply schema changes and migrations per `data-model.md` in [exact migration path]
- [ ] T004 [P] [Contracts] Define DTOs, interfaces, or payload schemas per `contracts/` in [exact types path]
- [ ] T005 [P] Setup or verify mock factories/fixtures needed for testing seam

### Tests for User Story 2 (OPTIONAL - only if tests requested) ⚠️

- [ ] T018 [P] [US2] Contract test for [endpoint] in tests/contract/test_[name].py
- [ ] T019 [P] [US2] Integration test for [user journey] in tests/integration/test_[name].py

### Implementation for User Story 2

- [ ] T020 [P] [US2] Create [Entity] model in src/models/[entity].py
- [ ] T021 [US2] Implement [Service] in src/services/[service].py
- [ ] T022 [US2] Implement [endpoint/feature] in src/[location]/[file].py
- [ ] T023 [US2] Integrate with User Story 1 components (if needed)

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - [Title] (Priority: P3)
## Phase 3: User Story 1 - [Title] (Priority: P1 - Core MVP) 🎯 Tracer Bullet

**Goal**: [Brief description of what this story delivers]
**Goal**: [Deliver end-to-end working MVP path via the primary testing seam]
**Testing Seam**: `[Exact route, CLI command, or queue topic]`

**Independent Test**: [How to verify this story works on its own]
### 1. The Red Stage (Failing Seam Test)
- [ ] T006 [TDD-Red] [US1] Write integration/contract test exercising primary seam for US1 in [tests/seam-test-path]
- [ ] T007 [US1] Run test and verify it FAILS cleanly with expected missing implementation / 404 / 501 assertion

### Tests for User Story 3 (OPTIONAL - only if tests requested) ⚠️
### 2. The Green Stage (Minimal Implementation)
- [ ] T008 [P] [US1] [Impl] Implement domain model / state transitions in [src/models/...]
- [ ] T009 [US1] [Impl] Implement service logic and validation rules in [src/services/...]
- [ ] T010 [US1] [Impl] Connect primary testing seam controller/handler in [src/controllers/...]
- [ ] T011 [TDD-Green] [US1] Run seam test from T006 and verify it turns 100% GREEN (Pass)

- [ ] T024 [P] [US3] Contract test for [endpoint] in tests/contract/test_[name].py
- [ ] T025 [P] [US3] Integration test for [user journey] in tests/integration/test_[name].py
### 3. Mutation & Refactor Stage
- [ ] T012 [Mutation] [US1] Revert minimal logic or assert boundary condition to prove test detects breakage (Mutation proof)
- [ ] T013 [Refactor] [US1] Clean up code, remove duplicate logic, format per project rules

### Implementation for User Story 3
**Checkpoint**: User Story 1 is now a fully functional, tested, and deployable standalone vertical slice (MVP).

- [ ] T026 [P] [US3] Create [Entity] model in src/models/[entity].py
- [ ] T027 [US3] Implement [Service] in src/services/[service].py
- [ ] T028 [US3] Implement [endpoint/feature] in src/[location]/[file].py

**Checkpoint**: All user stories should now be independently functional

---

[Add more user story phases as needed, following the same pattern]
## Phase 4: User Story 2 - [Title] (Priority: P2 - Edge Cases & Validations)

---
**Goal**: [Boundary conditions, error cases, and alternate flows]
**Testing Seam**: `[Exact route or seam handler]`

## Phase N: Polish & Cross-Cutting Concerns
### 1. The Red Stage
- [ ] T014 [TDD-Red] [US2] Add negative / boundary test cases to [tests/seam-test-path]
- [ ] T015 [US2] Verify new edge tests FAIL as expected

**Purpose**: Improvements that affect multiple user stories
### 2. The Green Stage
- [ ] T016 [US2] [Impl] Add validation, error handling, and state guards in [src/services/...]
- [ ] T017 [TDD-Green] [US2] Run tests and verify all edge scenarios turn 100% GREEN

- [ ] TXXX [P] Documentation updates in docs/
- [ ] TXXX Code cleanup and refactoring
- [ ] TXXX Performance optimization across all stories
- [ ] TXXX [P] Additional unit tests (if requested) in tests/unit/
- [ ] TXXX Security hardening
- [ ] TXXX Run quickstart.md validation

---

## Dependencies & Execution Order
## Phase 5: User Story 3 - [Title] (Priority: P3 - Polish / Optional)

### Phase Dependencies
**Goal**: [Secondary enhancements or non-blocking features]
- [ ] T018 [TDD-Red] [US3] Write seam test for secondary enhancement
- [ ] T019 [US3] [Impl] Implement enhancement
- [ ] T020 [TDD-Green] [US3] Verify tests pass

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Polish (Final Phase)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - May integrate with US1 but should be independently testable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - May integrate with US1/US2 but should be independently testable

### Within Each User Story

- Tests (if included) MUST be written and FAIL before implementation
- Models before services
- Services before endpoints
- Core implementation before integration
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- All tests for a user story marked [P] can run in parallel
- Models within a story marked [P] can run in parallel
- Different user stories can be worked on in parallel by different team members

---

## Parallel Example: User Story 1
## Phase N: Polish, Observability & Convergence Readiness

```bash
# Launch all tests for User Story 1 together (if tests requested):
Task: "Contract test for [endpoint] in tests/contract/test_[name].py"
Task: "Integration test for [user journey] in tests/integration/test_[name].py"
**Goal**: Production hardening, performance checks, and cleanup across the entire feature.

# Launch all models for User Story 1 together:
Task: "Create [Entity1] model in src/models/[entity1].py"
Task: "Create [Entity2] model in src/models/[entity2].py"
```
- [ ] T021 [P] [Polish] Add structured logging and error metrics per constitution
- [ ] T022 [Polish] Run complete regression test suite across the repository
- [ ] T023 [Polish] Run quickstart validation flow per `quickstart.md`
- [ ] T024 [Polish] Clean git status, verify no stray files or untracked artifacts

---

## Implementation Strategy
## Dependencies & Execution Order

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Test User Story 1 independently
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 → Test independently → Deploy/Demo
4. Add User Story 3 → Test independently → Deploy/Demo
5. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1
   - Developer B: User Story 2
   - Developer C: User Story 3
3. Stories complete and integrate independently

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Verify tests fail before implementing
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
1. **Phase 1 (Pre-factoring)** $\to$ Must complete before any new feature code is written.
2. **Phase 2 (Data Layer)** $\to$ Blocks all user story work.
3. **Phase 3 (User Story 1 - P1)** $\to$ Can start immediately after Phase 2. Delivers working MVP.
4. **Phase 4 (User Story 2 - P2)** $\to$ Builds on P1 foundation.
5. **Phase N (Polish)** $\to$ Final gate before review and merge.
