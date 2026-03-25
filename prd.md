# 🚀 Refined PRD — Chrome Extension: AI Agent Node Prompts (N8n)

## 1. 🎯 Product Goal

Enable users of n8n to **instantly view prompt history for AI Agent nodes** without leaving the canvas—reducing friction in debugging, auditing, and iteration.

---

## 2. 👤 Target Users

* AI workflow builders using N8n
* Developers debugging LLM pipelines
* Automation engineers reviewing prompt evolution

---

## 3. 🧩 Core Use Cases

1. Inspect previous prompts used in an AI Agent node
2. Debug incorrect outputs by reviewing past prompts
3. Track prompt evolution over time
4. Quickly copy/reuse past prompts

---

## 4. 🧠 Feature Breakdown

### 4.1 Prompt History Panel (Drawer)

**Trigger:** Click on an AI Agent node

**Behavior:**

* Slide-in drawer from right side
* Overlay on top of N8n UI (non-destructive)

**Content:**
Each prompt entry includes:

* Title (editable in future versions)
* Full prompt text (expandable/collapsible)
* Timestamp (relative + absolute)

**Enhancements (recommended):**

* Copy button 📋
* Expand/collapse toggle
* Scrollable list (not fixed single view)

---

### 4.2 Prompt Ordering

* Default: **Descending by timestamp (latest first)**
* Optional future:

  * Filter by date
  * Search prompts

---

### 4.3 Empty / Edge States

* No prompts → “No prompt history available”
* API error → “Failed to load prompts”
* Loading → skeleton UI

---

## 5. 🎨 UI/UX Specifications

### Drawer Design

* Width: **360–420px** (380px is good baseline)
* Height: **100% viewport height (recommended change)**

  * Current spec (100px) is too small → not usable
* Position: right side
* Z-index above N8n canvas

### Interaction

* Smooth slide animation (~200–300ms)
* Close:

  * Click outside
  * ESC key
  * Close button (top-right)

---

### Visual Example (Structure)

```
---------------------------------
| Prompt History              X |
---------------------------------
| Prompt Title                |
| Timestamp                   |
| -------------------------- |
| Prompt text preview...      |
| [Expand] [Copy]             |
---------------------------------
| Prompt Title                |
...
```

---

## 6. 🔌 Technical Architecture

### 6.1 Chrome Extension Components

#### 1. Manifest (v3)

Permissions:

* activeTab
* scripting
* storage
* host permissions (API domain + n8n domain)

---

#### 2. Content Script

Injected into N8n UI:

* Detect node clicks
* Extract node ID
* Render drawer UI
* Manage DOM injection

---

#### 3. Background Script (optional but recommended)

* Handle API requests
* Manage caching
* Retry logic

---

#### 4. UI Layer

* Vanilla JS OR lightweight framework (recommended: Preact)
* CSS scoped to avoid conflicts with N8n

---

## 7. 🔍 Node Detection Logic

### Key Challenge ⚠️

N8n doesn’t provide a public API for node click events.

### Strategy:

* Observe DOM mutations
* Listen for click events on node elements
* Extract node ID from:

  * URL params OR
  * DOM attributes (preferred)

---

## 8. 🌐 API Integration

### Request

```
GET /prompts?nodeId=<NODE_ID>
```

### Response Example

```json
[
  {
    "id": "123",
    "title": "Initial prompt",
    "text": "Summarize the following...",
    "createdAt": "2026-03-20T10:30:00Z"
  }
]
```

---

### Handling

* Timeout: 5–8 seconds
* Retry: 1 retry max
* Cache: optional (per node ID)

---

## 9. ⚡ Performance Considerations

* Lazy load drawer (only inject when needed)
* Avoid heavy frameworks
* Debounce click handling
* Cache recent node queries

---

## 10. 🔐 Security & Privacy

* HTTPS required
* No local storage of prompt text unless explicitly needed
* Sanitize rendered content (prevent XSS)

---

## 11. 🧪 Testing Strategy

### Functional

* Node click detection works reliably
* Correct prompts fetched per node
* Drawer opens/closes correctly

### Edge Cases

* Rapid node switching
* API downtime
* Large prompt text

### Performance

* No noticeable lag in N8n UI
* Memory usage stable

---

## 12. 🚀 MVP vs Future Roadmap

### MVP (Must Have)

* Node click detection
* Drawer UI
* Prompt list
* API fetch
* Copy prompt

---

### V2 (Nice to Have)

* Search prompts 🔍
* Tagging
* Edit prompt titles
* Export prompts
* Diff view between prompts

---

## 13. ⚠️ Key Risks & Mitigations

| Risk            | Mitigation                               |
| --------------- | ---------------------------------------- |
| N8n DOM changes | Use resilient selectors + fallback logic |
| API latency     | Show loading state + caching             |
| UI conflicts    | Scoped CSS + shadow DOM (optional)       |
| Large prompts   | Collapse + virtualization                |

---

## 14. 💡 Suggested Improvements to Your Original PRD

Here are the most important upgrades you should adopt:

1. **Fix panel height**

   * 100px → ❌ unusable
   * Full height → ✅ correct

2. **Add loading / error states**

   * Missing but critical for UX

3. **Add copy + expand**

   * Essential for real usability

4. **Clarify node detection**

   * Biggest technical risk not covered

5. **Introduce caching**

   * Prevent excessive API calls

---

## 15. 🧭 Final Take

Your concept is strong and very practical—especially for teams working heavily with AI workflows in n8n.

The main thing you were missing wasn’t vision—it was **implementation depth and edge-case handling**. Now it’s much closer to something engineers can directly build from.
