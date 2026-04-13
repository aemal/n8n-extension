# Research Report: AI Agent Prompt History Drawer

**Branch**: `001-ai-prompt-drawer` | **Date**: 2026-03-25
**Research Phase**: Phase 0 - Resolving NEEDS CLARIFICATION items

## Backend API Architecture

**Decision**: REST API with Bearer token authentication and cursor-based pagination
**Rationale**: Standard pattern for Chrome extensions accessing external services with proper security and performance characteristics
**Alternatives considered**: GraphQL (more complex for simple use case), WebSocket (overkill for request-response pattern)

### API Endpoint Design
- **Primary endpoint**: `GET /api/v1/nodes/{nodeId}/prompts/history`
- **Authentication**: Bearer token stored in Chrome extension storage
- **Pagination**: Cursor-based for performance with large datasets
- **Rate limiting**: 1000 requests/hour per user with proper error headers

### Data Format
```json
{
  "data": [{
    "id": "prompt-uuid-123",
    "nodeId": "node-456",
    "content": "You are a helpful assistant...",
    "createdAt": "2025-03-25T10:30:00.000Z",
    "metadata": {
      "model": "gpt-4",
      "executionTime": 1.2
    }
  }],
  "pagination": {
    "cursor": "base64-encoded-cursor",
    "hasMore": true
  }
}
```

## Chrome Extension Testing Framework

**Decision**: Jest with jest-chrome for unit tests, Puppeteer for E2E testing
**Rationale**: Most mature testing ecosystem for Chrome extensions with comprehensive Chrome API mocking
**Alternatives considered**: Playwright (less extension-specific tooling), Cypress (no Chrome extension support)

### Testing Stack
- **Unit Tests**: Jest + @testing-library/dom + jest-chrome
- **Integration Tests**: Puppeteer with extension loading
- **CI/CD**: GitHub Actions with automated extension building
- **Coverage**: Minimum 80% code coverage requirement

### Key Testing Patterns
```javascript
// Content script testing with DOM manipulation
describe('Content Script', () => {
  beforeEach(() => {
    document.body.innerHTML = '<div id="n8n-canvas"></div>';
  });

  it('should detect AI Agent nodes', () => {
    // Test node detection logic
  });
});
```

## n8n DOM Structure Analysis

**Decision**: Target Vue Flow node elements with data-id attributes and event delegation
**Rationale**: n8n uses Vue.js 3 with Vue Flow for canvas rendering, providing stable DOM structure
**Alternatives considered**: URL-based detection (unreliable), complex Vue component inspection (brittle)

### Node Detection Strategy
- **Framework**: Vue.js 3 + Vue Flow canvas system
- **Node rendering**: CanvasNodeDefault.vue component with CSS class variants
- **Identification**: Vue Flow adds `data-id` attributes to node elements
- **Event handling**: Use event delegation on canvas container

### DOM Selectors
```javascript
// Primary detection strategy
const nodeElements = document.querySelectorAll('[data-id]');

// Event delegation pattern
document.addEventListener('click', (event) => {
  const nodeElement = event.target.closest('[data-id]');
  if (nodeElement && isAIAgentNode(nodeElement)) {
    handleNodeClick(nodeElement);
  }
});
```

## Content Script Architecture

**Decision**: Shadow DOM with service worker communication and optimized DOM monitoring
**Rationale**: Complete CSS isolation prevents conflicts with n8n styling while maintaining security and performance
**Alternatives considered**: Direct DOM injection (style conflicts), iframe (security/communication complexity)

### Implementation Strategy
- **Isolation**: Closed Shadow DOM for complete style encapsulation
- **Communication**: Long-lived port connection to service worker
- **Performance**: MutationObserver with debounced processing
- **Memory**: Automatic cleanup on page unload and visibility changes

### Architecture Pattern
```javascript
class ExtensionContentScript {
  constructor() {
    this.shadowRoot = null;
    this.port = null;
    this.domMonitor = null;
  }

  async init() {
    this.createShadowDOM();
    this.setupCommunication();
    this.startDOMMonitoring();
  }
}
```

## Technical Architecture Decisions

### Storage Strategy
- **Token storage**: Chrome storage API for authentication tokens
- **Cache**: Limited prompt history caching in extension storage
- **Persistence**: Backend API as source of truth for all data

### Performance Optimizations
- **API timeout**: 8 seconds maximum request timeout
- **Debouncing**: 100ms debounce for DOM mutations
- **Lazy loading**: Only inject drawer UI when needed
- **Memory management**: Automatic cleanup on page visibility changes

### Security Implementation
- **CSP compliance**: No unsafe-inline, unsafe-eval, or external scripts
- **Input validation**: Sanitize all data from content scripts
- **Origin validation**: Whitelist allowed origins for API requests
- **Token management**: Secure storage with automatic refresh

## Resolved Clarifications

All "NEEDS CLARIFICATION" items from Technical Context have been resolved:

1. **Primary Dependencies**: Chrome Extension APIs (Manifest V3), fetch API for backend communication
2. **Testing**: Jest for unit tests with jest-chrome for Chrome API mocking, Puppeteer for E2E tests
3. **Backend API**: RESTful API with Bearer authentication and cursor-based pagination
4. **Content Scripts**: Shadow DOM isolation with MutationObserver for DOM monitoring

## Next Steps: Phase 1 Design

Research is complete. Ready to proceed to Phase 1:
- Generate data-model.md with prompt history entities
- Create contracts/ directory with API interface definitions
- Generate quickstart.md for development setup
- Update agent context with new technology stack