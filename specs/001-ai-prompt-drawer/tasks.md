# Tasks: AI Agent Prompt History Drawer

**Input**: Design documents from `/specs/001-ai-prompt-drawer/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

Chrome Extension structure:
- **Extension files**: `src/` at repository root
- **Tests**: `tests/` at repository root
- **Assets**: `assets/` for icons and resources

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and Chrome extension structure

- [ ] T001 Create project structure following plan.md Chrome extension architecture
- [ ] T002 Initialize Node.js project with Webpack, Jest, and Chrome extension dependencies in package.json
- [ ] T003 [P] Configure ESLint and Prettier for vanilla JavaScript development in .eslintrc.js and .prettierrc
- [ ] T004 [P] Create Webpack configuration for Chrome extension build in webpack.config.js
- [ ] T005 [P] Setup Jest configuration with jest-chrome mocking in jest.config.js
- [ ] T006 [P] Create Chrome extension manifest.json with Manifest V3 structure

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core Chrome extension infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T007 Implement Chrome extension storage wrapper in src/api/storage.js
- [ ] T008 [P] Implement API client for backend communication in src/api/api-client.js
- [ ] T009 [P] Create service worker for background tasks in src/background/service-worker.js
- [ ] T010 [P] Implement DOM utilities for n8n node detection in src/utils/dom-utils.js
- [ ] T011 [P] Create configuration constants in src/utils/constants.js
- [ ] T012 [P] Setup Shadow DOM architecture for content script in src/content/content-script.js
- [ ] T013 [P] Create base drawer HTML structure in src/drawer/drawer.html
- [ ] T014 [P] Implement drawer CSS with scoped styling in src/drawer/drawer.css
- [ ] T015 Setup Chrome extension test fixtures and mocks in tests/fixtures/

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - View Prompt History (Priority: P1) 🎯 MVP

**Goal**: Click AI Agent node → Drawer slides in → Shows prompt history chronologically

**Independent Test**: Click an AI Agent node in n8n and verify that a drawer appears showing a chronological list of prompts with timestamps

### Implementation for User Story 1

- [ ] T016 [P] [US1] Implement AI Agent node detection logic in src/content/content-script.js
- [ ] T017 [P] [US1] Create PromptEntry data model class in src/models/prompt-entry.js
- [ ] T018 [P] [US1] Create DrawerState management in src/models/drawer-state.js
- [ ] T019 [US1] Implement drawer open/close functionality in src/drawer/drawer.js
- [ ] T020 [US1] Implement API call for prompt history retrieval in src/drawer/drawer.js
- [ ] T021 [US1] Create prompt list rendering logic in src/drawer/drawer.js
- [ ] T022 [US1] Add timestamp formatting and chronological sorting in src/drawer/drawer.js
- [ ] T023 [US1] Implement loading state display in src/drawer/drawer.js
- [ ] T024 [US1] Implement empty state ("No prompt history available") in src/drawer/drawer.js
- [ ] T025 [US1] Add keyboard shortcut (ESC) and outside click to close drawer in src/drawer/drawer.js

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - Copy and Reuse Prompts (Priority: P2)

**Goal**: Add copy button to each prompt → Copy full prompt text to clipboard

**Independent Test**: Open prompt history, click copy button on any prompt, and verify the prompt text is copied to clipboard for immediate reuse

### Implementation for User Story 2

- [ ] T026 [P] [US2] Add copy button to prompt entry UI in src/drawer/drawer.html
- [ ] T027 [P] [US2] Style copy button with hover states in src/drawer/drawer.css
- [ ] T028 [US2] Implement clipboard API integration in src/drawer/drawer.js
- [ ] T029 [US2] Add copy confirmation feedback (toast/visual indication) in src/drawer/drawer.js
- [ ] T030 [US2] Handle copy operation for large prompts (>10,000 characters) in src/drawer/drawer.js
- [ ] T031 [US2] Add error handling for clipboard access failures in src/drawer/drawer.js

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - Navigate Prompt Details (Priority: P3)

**Goal**: Expand/collapse individual prompts → Show full text or truncated preview

**Independent Test**: Open drawer with multiple prompts, toggle expand/collapse on entries, and verify that long prompts can be viewed in full or summarized

### Implementation for User Story 3

- [ ] T032 [P] [US3] Add expand/collapse button to prompt entries in src/drawer/drawer.html
- [ ] T033 [P] [US3] Implement prompt text truncation (200 characters) in src/drawer/drawer.js
- [ ] T034 [P] [US3] Style expanded and collapsed states in src/drawer/drawer.css
- [ ] T035 [US3] Implement expand/collapse toggle logic in src/drawer/drawer.js
- [ ] T036 [US3] Add smooth animation transitions for expand/collapse in src/drawer/drawer.css
- [ ] T037 [US3] Maintain expand/collapse state in DrawerState model in src/models/drawer-state.js
- [ ] T038 [US3] Handle multiple prompts with independent expand states in src/drawer/drawer.js

**Checkpoint**: All user stories should now be independently functional

---

## Phase 6: Error Handling & Edge Cases

**Purpose**: Robust error handling and edge case management

- [ ] T039 [P] Implement API timeout handling (8 seconds max) in src/api/api-client.js
- [ ] T040 [P] Add network error display and retry mechanism in src/drawer/drawer.js
- [ ] T041 [P] Handle rapid node clicking (debouncing) in src/content/content-script.js
- [ ] T042 [P] Add error boundaries for drawer rendering failures in src/drawer/drawer.js
- [ ] T043 [P] Implement graceful degradation when API is unavailable in src/drawer/drawer.js
- [ ] T044 Add performance optimization for large prompt datasets in src/drawer/drawer.js

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T045 [P] Add Chrome extension icons and assets in assets/icons/
- [ ] T046 [P] Implement caching strategy for prompt history in src/api/storage.js
- [ ] T047 [P] Add performance monitoring and logging in src/utils/performance-monitor.js
- [ ] T048 [P] Optimize Shadow DOM styles for n8n compatibility in src/content/content-script.css
- [ ] T049 [P] Add comprehensive JSDoc documentation across all modules
- [ ] T050 Create production build script with minification in package.json
- [ ] T051 Add Chrome Web Store assets (screenshots, descriptions) in docs/store/
- [ ] T052 Run quickstart.md validation for development setup

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Error Handling (Phase 6)**: Can run in parallel with user stories
- **Polish (Phase 7)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - Builds on US1 drawer UI but independently testable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - Builds on US1 drawer UI but independently testable

### Within Each User Story

- Data models before services
- Core drawer logic before UI enhancements
- Basic functionality before error handling
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- Models within a story marked [P] can run in parallel
- Different user stories can be worked on in parallel by different team members

---

## Parallel Example: User Story 1

```bash
# Launch all models for User Story 1 together:
Task: "Create PromptEntry data model class in src/models/prompt-entry.js"
Task: "Create DrawerState management in src/models/drawer-state.js"

# Launch all foundational components together:
Task: "Implement Chrome extension storage wrapper in src/api/storage.js"
Task: "Implement API client for backend communication in src/api/api-client.js"
Task: "Create service worker for background tasks in src/background/service-worker.js"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Test User Story 1 independently
5. Load in Chrome as unpacked extension and test with n8n

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Load extension (MVP!)
3. Add User Story 2 → Test independently → Update extension
4. Add User Story 3 → Test independently → Update extension
5. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1 (core drawer functionality)
   - Developer B: User Story 2 (copy functionality)
   - Developer C: User Story 3 (expand/collapse functionality)
3. Stories complete and integrate independently

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Chrome extension can be loaded as "unpacked extension" for testing
- Test with actual n8n instance for DOM detection validation
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Follow Manifest V3 security requirements throughout implementation