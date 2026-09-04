# Feature Specification: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`
**Created**: [DATE]
**Status**: Draft
**Input Description**: "$ARGUMENTS"

---

## 1. Domain Vocabulary & Alignment

<!--
  CRITICAL: All actors, entities, and actions in this specification MUST align with
  the project's canonical glossary in `.specify/memory/CONTEXT.md`.
-->

- **Primary Actor(s)**: [e.g. Customer, Admin, Service - avoid synonyms forbidden by CONTEXT.md]
- **Core Entities**: [e.g. Order, LineItem, Payment]
- **Forbidden / Avoided Synonyms**: [Explicitly note terms to avoid in this feature, e.g. avoid 'User' or 'Purchase']

---

## 2. Primary Testing Seam _(Highest Seam Preference)_

<!--
  Define the primary architectural boundary at which this feature will be tested end-to-end.
  The ideal number of seams is ONE. Prefer the highest interface (API route, CLI command, message consumer)
  rather than mocking internal service functions.
-->

- **Target Boundary**: [e.g. `POST /api/v1/orders/{id}/cancel` or CLI command `order cancel <id>`]
- **Contract Type**: [HTTP REST / GraphQL / CLI Text-JSON / Event Queue]
- **Expected Status / Output**: [e.g. HTTP 200 with `{ status: "CANCELLED" }` or explicit error contract]

---

## 3. Prioritized User Journeys & Acceptance Scenarios

<!--
  User stories MUST be prioritized (P1 = Essential MVP, P2 = Major flows & edge cases, P3 = Nice-to-have / polish).
  Each user story MUST be independently testable via the testing seam.
-->

### User Story 1 - [Brief Title] (Priority: P1 - Core MVP)

[Describe the core user journey in plain, unambiguous domain language]

**Why this priority**: [Explain business value and why this is the foundational slice]

**Independent Seam Test**: [Describe the exact call/action that verifies this story standalone]

**Acceptance Scenarios**:

1. **Given** [initial system state], **When** [action taken via seam], **Then** [expected outcome and state transition]
2. **Given** [initial system state], **When** [action taken via seam], **Then** [expected outcome and state transition]

---

### User Story 2 - [Brief Title] (Priority: P2 - Edge Cases & Validations)

[Describe this secondary journey or boundary condition]

**Why this priority**: [Explain the risk/importance of this flow]

**Independent Seam Test**: [Describe verification via seam]

**Acceptance Scenarios**:

1. **Given** [boundary condition], **When** [action taken via seam], **Then** [error response / graceful handling]

---

### User Story 3 - [Brief Title] (Priority: P3 - Polish & Extensions)

[Describe optional enhancements or secondary ergonomics]

**Why this priority**: [Why this can be deferred if time/scope is constrained]

**Acceptance Scenarios**:

1. **Given** [condition], **When** [action], **Then** [expected outcome]

---

## 4. Requirements & Invariants

### Functional Requirements

- **FR-001**: System MUST [specific capability in domain language]
- **FR-002**: System MUST [validation or state transition rule]
- **FR-003**: System MUST NOT [forbidden behavior or state mutation]
- **FR-004**: System MUST [data persistence requirement]

### Unresolved Items

<!-- Mark any ambiguous requirements with [NEEDS CLARIFICATION: reason] for speckit-clarify -->

- **FR-005**: [NEEDS CLARIFICATION: Unspecified edge case or third-party integration detail]

---

## 5. Scope Boundaries & Fog of War

### Explicitly In-Scope (This Feature)

- [Deliberate focus of this feature branch]

### Out of Scope / Deferred to Future Features

<!-- Work consciously excluded to prevent scope bloat and keep this feature manageable -->

- [Feature / capability explicitly postponed to a subsequent ticket or spec]
- [Edge case handled manually or deferred]

---

## 6. Measurable Success Criteria

- **SC-001**: [Measurable outcome, e.g. "Primary journey completes in < 200ms"]
- **SC-002**: [Integrity metric, e.g. "Zero orphaned records when cancellation occurs"]
- **SC-003**: [Test confidence metric, e.g. "100% of P1/P2 acceptance scenarios verified green via testing seam"]
