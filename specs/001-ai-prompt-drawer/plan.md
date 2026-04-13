# Implementation Plan: AI Agent Prompt History Drawer

**Branch**: `001-ai-prompt-drawer` | **Date**: 2026-03-25 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-ai-prompt-drawer/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

Chrome extension that displays a slide-in drawer showing prompt history when users click on AI Agent nodes in n8n workflows. Built with vanilla JavaScript and CSS using Manifest V3 architecture for secure content script injection and API communication.

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: JavaScript ES2022, HTML5, CSS3
**Primary Dependencies**: Chrome Extension APIs (Manifest V3), NEEDS CLARIFICATION for backend API
**Storage**: Chrome extension storage API for local caching, backend API for prompt history data
**Testing**: NEEDS CLARIFICATION - likely Jest for unit tests, Chrome extension testing framework
**Target Platform**: Chrome browser (versions 100+), desktop/laptop devices (1024px+ viewport)
**Project Type**: Browser extension (Chrome Web Store distribution)
**Performance Goals**: <3 seconds prompt history load time, <8 seconds API timeout, smooth drawer animations
**Constraints**: Manifest V3 security requirements, no remote code execution, minimal memory footprint
**Scale/Scope**: Single-user extension, support for multiple n8n workflows, up to 50,000 character prompts

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

✅ **Manifest V3 Security-First**: Extension uses Manifest V3 architecture with service workers, Shadow DOM isolation, and bundled code only
✅ **Least Privilege Permissions**: Requesting only `activeTab`, `storage`, and host permissions for n8n domains and API endpoints
✅ **Secure Code Architecture**: All JavaScript/CSS bundled, no eval() or external code execution, API communication through service worker
✅ **Content Security Policy Compliance**: Strict CSP implemented, Shadow DOM prevents style conflicts, safe DOM manipulation patterns
✅ **Privacy-First Development**: Minimal data collection (only prompt history), explicit user consent, secure token storage in Chrome API

**POST-DESIGN RE-EVALUATION**: ✅ PASS - All constitutional principles maintained through detailed design phase. Shadow DOM architecture and service worker communication patterns enhance security beyond initial requirements.

**GATE RESULT**: ✅ PASS - All constitutional principles satisfied with enhanced security measures

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
# Chrome Extension Structure
src/
├── manifest.json           # Extension manifest (Manifest V3)
├── content/                 # Content scripts for n8n integration
│   ├── content-script.js   # Main content script for node detection
│   └── content-script.css  # Scoped styles for extension UI
├── popup/                  # Extension popup (if needed)
│   ├── popup.html
│   ├── popup.js
│   └── popup.css
├── background/             # Service worker
│   └── service-worker.js   # Background event handling
├── drawer/                 # Drawer UI components
│   ├── drawer.html         # Drawer template
│   ├── drawer.js           # Drawer logic and API calls
│   └── drawer.css          # Drawer styling
├── api/                    # API communication
│   ├── api-client.js       # HTTP client for prompt history API
│   └── storage.js          # Chrome storage wrapper
└── utils/                  # Shared utilities
    ├── dom-utils.js        # DOM manipulation helpers
    └── constants.js        # Configuration constants

tests/
├── unit/                   # Jest unit tests
├── integration/            # Chrome extension integration tests
└── fixtures/               # Test data and mocks
```

**Structure Decision**: Chrome extension architecture with content scripts for n8n DOM manipulation, service worker for background tasks, and modular components for drawer UI and API communication.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
