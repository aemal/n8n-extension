# Feature Specification: AI Agent Prompt History Drawer

**Feature Branch**: `001-ai-prompt-drawer`
**Created**: 2026-03-25
**Status**: Draft
**Input**: User description: "Chrome Extension: AI Agent Node Prompts (N8n)"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - View Prompt History (Priority: P1)

As an AI workflow builder using n8n, I want to click on any AI Agent node and instantly see a history of all prompts that have been used with that node, so that I can understand what prompts were previously executed without navigating away from my workflow canvas.

**Why this priority**: This is the core functionality that provides immediate value. Users can debug and understand their AI nodes' behavior by seeing historical context, which directly addresses the main pain point of debugging LLM workflows.

**Independent Test**: Can be fully tested by clicking an AI Agent node in n8n and verifying that a drawer appears showing a chronological list of prompts, delivering immediate insight into node history.

**Acceptance Scenarios**:

1. **Given** I am viewing an n8n workflow with AI Agent nodes, **When** I click on an AI Agent node, **Then** a drawer slides in from the right showing that node's prompt history
2. **Given** the prompt history drawer is open, **When** I view the prompt list, **Then** I see prompts ordered by most recent first with timestamps
3. **Given** an AI Agent node has no prompt history, **When** I click on it, **Then** I see "No prompt history available" message in the drawer

---

### User Story 2 - Copy and Reuse Prompts (Priority: P2)

As a developer debugging LLM pipelines, I want to copy any previous prompt from the history drawer so that I can reuse, modify, or test prompts outside of n8n to accelerate my debugging process.

**Why this priority**: Essential for practical workflow improvement. Without copy functionality, users can only view prompts but can't act on them, severely limiting the feature's utility.

**Independent Test**: Can be tested by opening prompt history, clicking a copy button on any prompt, and verifying the prompt text is copied to clipboard for immediate reuse.

**Acceptance Scenarios**:

1. **Given** I am viewing prompt history in the drawer, **When** I click the copy button on any prompt, **Then** the full prompt text is copied to my clipboard
2. **Given** I have copied a prompt, **When** I paste it elsewhere, **Then** I see the complete prompt text exactly as it appeared in the node
3. **Given** a prompt text is very long, **When** I copy it, **Then** the entire prompt is copied regardless of display truncation

---

### User Story 3 - Navigate Prompt Details (Priority: P3)

As an automation engineer reviewing prompt evolution, I want to expand and collapse individual prompt entries in the history drawer so that I can efficiently scan through multiple prompts and focus on specific ones when needed.

**Why this priority**: Improves usability for users with many prompts, but the feature provides value even with basic list display. This enhances the experience but isn't required for core functionality.

**Independent Test**: Can be tested by opening a drawer with multiple prompts, toggling expand/collapse on entries, and verifying that long prompts can be viewed in full or summarized.

**Acceptance Scenarios**:

1. **Given** I am viewing prompts in the drawer, **When** I click expand on a collapsed prompt, **Then** I see the full prompt text displayed
2. **Given** I have an expanded prompt, **When** I click collapse, **Then** the prompt shows only a preview with "..." truncation
3. **Given** multiple prompts are displayed, **When** I expand one prompt, **Then** other prompts remain in their current state (expanded or collapsed)

---

### Edge Cases

- What happens when API request fails or times out?
- How does the system handle extremely long prompt text (>10,000 characters)?
- What occurs when user rapidly clicks between different AI Agent nodes?
- How does the drawer behave when n8n UI layout changes or resizes?
- What happens when the extension loses connection to the API server?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Extension MUST detect clicks on AI Agent nodes within n8n workflows
- **FR-002**: System MUST display a slide-in drawer (360-420px wide, full viewport height) from the right side when an AI Agent node is clicked
- **FR-003**: Drawer MUST fetch and display prompt history specific to the clicked node via API call
- **FR-004**: Prompt entries MUST display title (if available), timestamp (relative and absolute), and prompt text preview
- **FR-005**: System MUST provide copy functionality for each prompt entry's full text
- **FR-006**: Drawer MUST close when user clicks outside, presses ESC key, or clicks close button
- **FR-007**: Extension MUST handle API failures gracefully with appropriate error messages
- **FR-008**: System MUST display prompts in chronological order (most recent first)
- **FR-009**: Extension MUST provide expand/collapse functionality for prompt text display
- **FR-010**: System MUST show loading state while fetching prompt data
- **FR-011**: Extension MUST prevent conflicts with existing n8n UI through scoped CSS styling
- **FR-012**: System MUST timeout API requests after 8 seconds maximum

### Key Entities *(include if feature involves data)*

- **Prompt Entry**: Represents a single prompt execution with unique ID, node ID, prompt text, optional title, and creation timestamp
- **AI Agent Node**: N8n workflow component that executes AI prompts, identified by unique node ID for history association
- **Drawer UI**: Browser extension interface component containing prompt history list and interaction controls

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can view prompt history within 3 seconds of clicking an AI Agent node
- **SC-002**: Copy functionality works for prompts up to 50,000 characters without performance degradation
- **SC-003**: Extension operates without causing noticeable lag or interference with n8n workflow editing
- **SC-004**: 95% of prompt history requests complete successfully under normal network conditions
- **SC-005**: Drawer UI renders correctly across Chrome browser versions 100+ and common viewport sizes (1024px+ width)
- **SC-006**: Users can navigate between multiple AI Agent nodes and view their respective histories without browser tab refresh

## Assumptions

- Users are working with n8n workflows that contain AI Agent nodes with existing prompt execution history
- A backend API exists that can return prompt history data when queried by node ID
- Users have Chrome browser with extension permissions enabled for the n8n domain
- Network connectivity is available for API requests to retrieve prompt history
- N8n UI structure remains stable enough for reliable node detection via DOM selectors
- Prompt history data is stored and maintained by the backend system, not the browser extension
- Users primarily work on desktop/laptop devices with viewport width of at least 1024px