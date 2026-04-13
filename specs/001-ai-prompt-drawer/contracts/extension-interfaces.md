# Extension Interface Contracts

**Branch**: `001-ai-prompt-drawer` | **Date**: 2026-03-25
**Contract Version**: 1.0.0

## Chrome Extension Architecture Interfaces

### Content Script → Service Worker Communication

#### Message Types
```typescript
// Message from content script to service worker
interface ContentToServiceMessage {
  action: 'GET_PROMPT_HISTORY' | 'CACHE_NODE_INFO' | 'LOG_EVENT';
  payload: GetPromptHistoryPayload | CacheNodeInfoPayload | LogEventPayload;
  requestId: string; // UUID for response correlation
}

interface ServiceToContentMessage {
  action: 'PROMPT_HISTORY_RESPONSE' | 'ERROR' | 'STATUS_UPDATE';
  payload: PromptHistoryResponse | ErrorPayload | StatusUpdatePayload;
  requestId: string; // Matches original request
}
```

#### Get Prompt History Interface
```typescript
interface GetPromptHistoryPayload {
  nodeId: string;
  workflowId?: string;
  options?: {
    limit?: number;     // 1-100, default 25
    cursor?: string;
    refresh?: boolean;  // Skip cache, force API call
  };
}

interface PromptHistoryResponse {
  success: true;
  data: {
    prompts: PromptEntry[];
    pagination: {
      cursor?: string;
      hasMore: boolean;
      total?: number;
    };
    cached: boolean; // Whether response came from cache
    cacheTimestamp?: string; // When data was cached
  };
}

interface ErrorPayload {
  success: false;
  error: {
    code: 'NETWORK_ERROR' | 'AUTH_ERROR' | 'VALIDATION_ERROR' | 'API_ERROR' | 'TIMEOUT_ERROR';
    message: string;
    details?: Record<string, any>;
    retryable: boolean;
  };
}
```

### DOM Event Interfaces

#### Node Click Detection
```typescript
interface NodeClickEvent {
  nodeId: string;
  nodeElement: HTMLElement;
  nodeType: string; // e.g., 'ai-agent'
  nodePosition: {
    x: number;
    y: number;
  };
  timestamp: number;
  workflowContext?: {
    workflowId: string;
    workflowName: string;
  };
}

// Event handler signature
type NodeClickHandler = (event: NodeClickEvent) => void;
```

#### Drawer State Events
```typescript
interface DrawerStateEvent {
  type: 'DRAWER_OPEN' | 'DRAWER_CLOSE' | 'DRAWER_SCROLL' | 'PROMPT_EXPAND' | 'PROMPT_COLLAPSE';
  payload: DrawerOpenPayload | DrawerScrollPayload | PromptTogglePayload;
  timestamp: number;
}

interface DrawerOpenPayload {
  nodeId: string;
  position: { x: number; y: number };
  drawerWidth: number;
}

interface DrawerScrollPayload {
  scrollTop: number;
  scrollHeight: number;
  clientHeight: number;
}

interface PromptTogglePayload {
  promptId: string;
  expanded: boolean;
}
```

### Storage Interfaces

#### Chrome Storage Schema
```typescript
interface ExtensionStorage {
  // Authentication
  'auth.token': string | null;
  'auth.refreshToken': string | null;
  'auth.tokenExpiry': number | null;

  // User preferences
  'preferences.drawerPosition': 'right' | 'left' | 'bottom';
  'preferences.drawerWidth': number;      // 360-600px range
  'preferences.autoExpand': boolean;       // Auto-expand first prompt
  'preferences.maxCacheSize': number;      // Max cached prompts per node

  // Cache data (dynamic keys)
  [`cache.promptHistory.${nodeId}`]: CachedPromptHistory;

  // Analytics (optional)
  'analytics.enabled': boolean;
  'analytics.lastSync': number;
}

interface CachedPromptHistory {
  nodeId: string;
  data: PromptEntry[];
  timestamp: number;           // When cached
  ttl: number;                // Time to live in ms
  pagination: {
    cursor?: string;
    hasMore: boolean;
    total?: number;
  };
}
```

### UI Component Interfaces

#### Drawer Component Props
```typescript
interface DrawerProps {
  isOpen: boolean;
  nodeId: string | null;
  position: 'right' | 'left' | 'bottom';
  width?: number;              // Pixel width for right/left positioning
  height?: number;             // Pixel height for bottom positioning
  onClose: () => void;
  onNodeChange: (nodeId: string) => void;
}

interface DrawerState {
  prompts: PromptEntry[];
  loading: boolean;
  error: string | null;
  expandedPrompts: Set<string>;
  scrollPosition: number;
  hasMore: boolean;
  cursor?: string;
}
```

#### Prompt List Component
```typescript
interface PromptListProps {
  prompts: PromptEntry[];
  expandedPrompts: Set<string>;
  onToggleExpand: (promptId: string) => void;
  onCopy: (content: string) => void;
  onLoadMore: () => void;
  hasMore: boolean;
  loading: boolean;
}

interface PromptItemProps {
  prompt: PromptEntry;
  expanded: boolean;
  onToggleExpand: () => void;
  onCopy: (content: string) => void;
}
```

### API Client Interface

#### HTTP Client Contract
```typescript
interface APIClient {
  getPromptHistory(
    nodeId: string,
    options?: GetPromptHistoryOptions
  ): Promise<APIResponse<PromptHistoryData>>;

  refreshToken(): Promise<APIResponse<AuthData>>;

  // Health check for API availability
  healthCheck(): Promise<APIResponse<HealthData>>;
}

interface GetPromptHistoryOptions {
  limit?: number;
  cursor?: string;
  sort?: 'timestamp:asc' | 'timestamp:desc';
  after?: string;   // ISO timestamp
  before?: string;  // ISO timestamp
  status?: 'pending' | 'completed' | 'failed';
}

interface APIResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: number;
    message: string;
    details?: Record<string, any>;
  };
  meta?: {
    requestId: string;
    timestamp: string;
    rateLimitRemaining?: number;
  };
}
```

### Error Handling Interfaces

#### Error Categories
```typescript
type ErrorCategory =
  | 'NETWORK_ERROR'      // Network connectivity issues
  | 'AUTH_ERROR'         // Authentication/authorization failures
  | 'VALIDATION_ERROR'   // Input validation failures
  | 'API_ERROR'          // API server errors
  | 'TIMEOUT_ERROR'      // Request timeout
  | 'STORAGE_ERROR'      // Chrome storage failures
  | 'DOM_ERROR'          // DOM manipulation errors
  | 'UNKNOWN_ERROR';     // Unexpected errors

interface ExtensionError {
  category: ErrorCategory;
  code: string;            // Specific error code
  message: string;         // User-friendly message
  details?: Record<string, any>;
  stack?: string;          // Stack trace for debugging
  timestamp: number;
  context?: {
    nodeId?: string;
    action?: string;
    url?: string;
  };
  retryable: boolean;      // Whether operation can be retried
}
```

#### Error Handler Interface
```typescript
interface ErrorHandler {
  handleError(error: ExtensionError): void;
  reportError(error: ExtensionError): Promise<void>;
  shouldRetry(error: ExtensionError): boolean;
  getRetryDelay(attemptCount: number): number;
}
```

### Performance Monitoring Interfaces

#### Performance Metrics
```typescript
interface PerformanceMetrics {
  // API performance
  apiResponseTime: number;      // milliseconds
  apiSuccessRate: number;       // 0-1 percentage
  cacheHitRate: number;        // 0-1 percentage

  // UI performance
  drawerOpenTime: number;       // Time to display drawer
  promptRenderTime: number;     // Time to render prompt list
  scrollPerformance: number;    // Scroll FPS

  // Memory usage
  memoryUsage: number;          // Bytes
  cacheSize: number;            // Number of cached entries

  // User interaction
  clickToDrawerTime: number;    // Node click to drawer display
  copyOperationTime: number;    // Time for copy operation
}

interface PerformanceMonitor {
  startTimer(operation: string): string; // Returns timer ID
  endTimer(timerId: string): number;    // Returns duration
  recordMetric(name: string, value: number): void;
  getMetrics(): PerformanceMetrics;
  reset(): void;
}
```

### Configuration Interface

#### Extension Configuration
```typescript
interface ExtensionConfig {
  // API configuration
  apiBaseUrl: string;
  apiTimeout: number;           // milliseconds
  retryAttempts: number;
  retryDelay: number;          // Base delay in milliseconds

  // UI configuration
  drawer: {
    defaultWidth: number;       // 420px
    minWidth: number;          // 360px
    maxWidth: number;          // 600px
    animationDuration: number; // milliseconds
  };

  // Cache configuration
  cache: {
    maxSize: number;           // Max entries per node
    defaultTTL: number;        // milliseconds
    cleanupInterval: number;   // milliseconds
  };

  // Performance configuration
  performance: {
    debounceDelay: number;     // DOM event debouncing
    throttleDelay: number;     // Scroll throttling
    maxConcurrentRequests: number;
  };

  // Debug configuration
  debug: {
    enabled: boolean;
    logLevel: 'error' | 'warn' | 'info' | 'debug';
    enablePerformanceLogging: boolean;
  };
}
```

These interface contracts provide a comprehensive specification for all communication patterns, data structures, and component interactions within the Chrome extension, ensuring consistent and type-safe implementation across all modules.