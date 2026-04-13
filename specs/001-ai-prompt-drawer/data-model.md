# Data Model: AI Agent Prompt History Drawer

**Branch**: `001-ai-prompt-drawer` | **Date**: 2026-03-25
**Phase**: Phase 1 - Design and Contracts

## Core Entities

### PromptEntry
Represents a single prompt execution within an AI Agent node.

**Fields:**
- `id: string` - Unique identifier for the prompt execution (UUID)
- `nodeId: string` - N8n AI Agent node identifier that executed the prompt
- `workflowId: string` - N8n workflow identifier containing the node
- `title: string | null` - Optional human-readable title for the prompt
- `content: string` - Full prompt text sent to the AI model
- `status: PromptStatus` - Execution status (completed, failed, pending)
- `createdAt: Date` - ISO 8601 timestamp when prompt was created
- `updatedAt: Date` - ISO 8601 timestamp when prompt was last modified
- `metadata: PromptMetadata` - AI model configuration and execution details
- `executionContext: ExecutionContext` - Runtime context and variables

**Validation Rules:**
- `id`: Must be valid UUID format
- `nodeId`: Required, non-empty string, max 255 characters
- `workflowId`: Required, non-empty string, max 255 characters
- `content`: Required, max 50,000 characters as per success criteria
- `status`: Must be one of allowed enum values
- `createdAt`: Must be valid ISO 8601 timestamp
- `metadata.model`: Required when status is 'completed'

**Relationships:**
- Belongs to one AIAgentNode (via nodeId)
- Contains one PromptMetadata
- Contains one ExecutionContext

### AIAgentNode
Represents an n8n AI Agent node that executes prompts.

**Fields:**
- `nodeId: string` - Unique identifier for the node within n8n
- `workflowId: string` - Parent workflow identifier
- `nodeName: string` - Human-readable name of the node
- `nodeType: string` - Type identifier ('ai-agent')
- `lastModified: Date` - When the node configuration was last changed
- `promptCount: number` - Total number of prompts executed by this node

**Validation Rules:**
- `nodeId`: Required, unique within workflow, max 255 characters
- `workflowId`: Required, non-empty string
- `nodeName`: Required, max 255 characters
- `nodeType`: Must equal 'ai-agent'
- `promptCount`: Non-negative integer

**Relationships:**
- Has many PromptEntry records
- Belongs to one n8n Workflow (external system)

### DrawerState
Represents the UI state of the prompt history drawer.

**Fields:**
- `isOpen: boolean` - Whether the drawer is currently visible
- `selectedNodeId: string | null` - Currently selected AI Agent node
- `expandedPromptIds: Set<string>` - Set of prompt IDs that are expanded
- `loadingStates: Map<string, boolean>` - Loading state per node
- `errorStates: Map<string, string | null>` - Error messages per node
- `scrollPosition: number` - Current scroll position in drawer
- `lastUpdated: Date` - When drawer state was last modified

**Validation Rules:**
- `selectedNodeId`: Must be valid node ID or null
- `scrollPosition`: Non-negative number
- `expandedPromptIds`: Set containing only valid prompt IDs

**State Transitions:**
- `closed → opening → open` (when user clicks AI Agent node)
- `open → closing → closed` (when user clicks outside or ESC key)
- `open → loading` (when fetching prompt history)
- `loading → open | error` (when API request completes)

### PromptMetadata
Contains AI model configuration and performance metrics.

**Fields:**
- `model: string` - AI model name (e.g., 'gpt-4', 'claude-3')
- `temperature: number | null` - Model temperature setting
- `maxTokens: number | null` - Maximum tokens configuration
- `executionTime: number` - Time taken to execute prompt (seconds)
- `inputTokens: number | null` - Number of input tokens used
- `outputTokens: number | null` - Number of output tokens generated
- `cost: number | null` - API cost for the request

**Validation Rules:**
- `model`: Required, non-empty string, max 100 characters
- `temperature`: Between 0 and 2 if provided
- `maxTokens`: Positive integer if provided
- `executionTime`: Non-negative number
- `inputTokens`: Non-negative integer if provided
- `outputTokens`: Non-negative integer if provided
- `cost`: Non-negative number if provided

### ExecutionContext
Contains runtime context and input/output data.

**Fields:**
- `workflowExecutionId: string` - Unique ID for the workflow run
- `inputVariables: Record<string, any>` - Variables passed to the prompt
- `outputSummary: string | null` - Brief summary of the output
- `errorMessage: string | null` - Error message if execution failed
- `retryCount: number` - Number of retry attempts made
- `parentPromptId: string | null` - ID of parent prompt if this is a retry

**Validation Rules:**
- `workflowExecutionId`: Required, non-empty string
- `outputSummary`: Max 500 characters if provided
- `errorMessage`: Max 1000 characters if provided
- `retryCount`: Non-negative integer
- `parentPromptId`: Must be valid prompt ID if provided

## Enums

### PromptStatus
```typescript
enum PromptStatus {
  PENDING = 'pending',
  COMPLETED = 'completed',
  FAILED = 'failed',
  CANCELLED = 'cancelled'
}
```

### DrawerPosition
```typescript
enum DrawerPosition {
  RIGHT = 'right',
  LEFT = 'left',
  BOTTOM = 'bottom'
}
```

## Data Flow

### Prompt History Retrieval
1. User clicks AI Agent node → `DrawerState.selectedNodeId` updated
2. Extension queries API with `nodeId` → Returns `PromptEntry[]`
3. Drawer displays prompts ordered by `createdAt` descending
4. User interactions update `DrawerState` (expand/collapse, scroll)

### Caching Strategy
- **Local cache**: Store recent prompt entries in Chrome extension storage
- **Cache key**: `prompt-history:${nodeId}`
- **TTL**: 5 minutes for completed prompts, 30 seconds for pending
- **Size limit**: Maximum 100 entries per node to prevent storage bloat

### Error Handling
- **API failures**: Store error in `DrawerState.errorStates`
- **Network timeouts**: Retry up to 2 times with exponential backoff
- **Invalid data**: Log error and display generic message to user
- **Storage errors**: Fall back to memory-only operation

## Performance Considerations

### Large Prompt Handling
- **Display truncation**: Show first 200 characters with expand option
- **Copy operation**: Copy full content regardless of display truncation
- **Memory management**: Unload prompt content when drawer closes
- **Lazy loading**: Load prompt details only when expanded

### API Optimization
- **Cursor pagination**: Load 25 prompts initially, more on scroll
- **Field selection**: Request only needed fields for list view
- **Caching headers**: Respect API cache headers for efficient requests
- **Request debouncing**: Prevent rapid API calls during user interactions

This data model provides a comprehensive foundation for managing prompt history data while ensuring performance, validation, and user experience requirements are met.