# PBI-3: Conversation Interface

## Overview
Create an intuitive chat interface within the extension popup that enables users to have streaming conversations with Gemini AI, with persistent history and real-time response rendering.

## Problem Statement
Users need a familiar chat-like interface to interact with Gemini AI while browsing. The interface should support streaming responses, maintain conversation history, and provide clear visual feedback during AI processing.

## User Stories
As a user, I want to have conversations with Gemini through the extension popup so that I can get AI assistance while browsing.

## Technical Approach

### UI Architecture

```mermaid
graph TB
    subgraph Popup Window
        CI[Chat Interface]
        ML[Message List]
        II[Input Interface]
        TB[Tool Bar]
    end
    
    subgraph React Components
        CM[ChatManager]
        MR[MessageRenderer]
        SR[StreamRenderer]
        TC[ToolCallDisplay]
    end
    
    subgraph State Management
        CS[Conversation State]
        SS[Streaming State]
        HS[History State]
    end
    
    CI --> ML
    CI --> II
    CI --> TB
    ML --> MR
    MR --> SR
    MR --> TC
    CM --> CS
    CM --> SS
    CM --> HS
```

### Message Flow

```mermaid
sequenceDiagram
    actor User
    participant Input as Input Field
    participant UI as Chat UI
    participant CM as Chat Manager
    participant SW as Service Worker
    participant API as Gemini API
    participant Store as Chrome Storage
    
    User->>Input: Type message
    User->>Input: Press Enter
    Input->>CM: Send message
    CM->>UI: Add user message
    CM->>UI: Show typing indicator
    CM->>SW: Forward to Gemini
    
    SW->>API: generateContentStream()
    API-->>SW: Stream start
    
    loop Streaming chunks
        API-->>SW: Content chunk
        SW-->>CM: Forward chunk
        CM->>UI: Update AI message
        UI-->>User: Render partial response
    end
    
    API-->>SW: Stream complete
    SW-->>CM: End of stream
    CM->>UI: Finalize message
    CM->>Store: Save to history
    UI-->>User: Show complete response
```

### Tool Execution Display

```mermaid
sequenceDiagram
    participant UI as Chat UI
    participant TC as Tool Call Display
    participant SW as Service Worker
    participant Tool as Tool Executor
    
    SW->>UI: Tool call started
    UI->>TC: Create tool card
    TC-->>User: Show "Executing: dom_query"
    
    SW->>Tool: Execute tool
    Tool-->>SW: Progress update
    SW-->>TC: Update progress
    TC-->>User: Show progress bar
    
    alt Success
        Tool-->>SW: Result data
        SW-->>TC: Tool completed
        TC-->>User: Show success + result
    else Error
        Tool-->>SW: Error details
        SW-->>TC: Tool failed
        TC-->>User: Show error + retry
    end
```

### History Management

```mermaid
sequenceDiagram
    participant UI as Chat UI
    participant CM as Chat Manager
    participant Store as Chrome Storage
    participant Sync as Sync Manager
    
    Note over UI,Store: On popup open
    UI->>CM: Initialize
    CM->>Store: Load conversation history
    Store-->>CM: Historical messages
    CM->>UI: Render messages
    
    Note over UI,Store: During conversation
    loop Each message
        CM->>Store: Append to history
        CM->>Sync: Check storage quota
        alt Quota exceeded
            Sync->>Store: Trim old messages
            Sync->>UI: Show storage warning
        end
    end
    
    Note over UI,Store: On clear history
    User->>UI: Clear history
    UI->>CM: Confirm action
    CM->>Store: Delete messages
    CM->>UI: Reset view
```

### Streaming Response Handling

```mermaid
sequenceDiagram
    participant API as Gemini API
    participant SW as Service Worker
    participant SR as Stream Renderer
    participant UI as UI Component
    
    API->>SW: SSE: Start stream
    SW->>SR: Initialize renderer
    SR->>UI: Create message container
    
    loop While streaming
        API->>SW: SSE: Content chunk
        SW->>SR: Process chunk
        
        alt Text content
            SR->>SR: Append to buffer
            SR->>UI: Update text
        else Tool call start
            SR->>UI: Create tool card
        else Tool call result
            SR->>UI: Update tool card
        end
        
        UI-->>User: Render update
    end
    
    API->>SW: SSE: [DONE]
    SW->>SR: Finalize
    SR->>UI: Enable actions
    UI-->>User: Show copy/retry buttons
```

### Error States

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Sending: User sends message
    Sending --> Streaming: API responds
    Sending --> Error: API error
    
    Streaming --> Receiving: Chunks arriving
    Receiving --> Streaming: More chunks
    Receiving --> Complete: Stream ends
    Receiving --> Error: Stream error
    
    Error --> Idle: User dismisses
    Error --> Retrying: User retries
    Retrying --> Sending
    
    Complete --> Idle: Ready for next
    
    state Error {
        [*] --> NetworkError: No connection
        [*] --> QuotaError: Rate limited  
        [*] --> AuthError: Token expired
        [*] --> ServerError: API down
    }
```

## UX/UI Considerations
- Familiar chat interface similar to popular messaging apps
- Real-time streaming with smooth text rendering
- Clear visual distinction between user and AI messages
- Loading states and progress indicators
- Tool execution visualization
- Error messages with actionable recovery options
- Keyboard shortcuts (Enter to send, Shift+Enter for newline)
- Auto-scroll to latest message
- Message timestamps on hover

## Acceptance Criteria
- [ ] Chat interface displays in popup with message history
- [ ] Real-time streaming responses render smoothly
- [ ] Conversation persists across popup open/close sessions
- [ ] Clear visual indication when AI is "thinking" or streaming
- [ ] Graceful error handling with retry options
- [ ] Tool executions display with progress and results
- [ ] Messages are stored in chrome.storage.local
- [ ] Copy message functionality
- [ ] Clear history option with confirmation

## Dependencies
- PBI-1: Extension foundation must be complete
- PBI-2: Authentication must be implemented
- React and TypeScript setup
- Chrome storage API permissions

## Open Questions
- Should we implement message search functionality?
- How many messages should we store in history?
- Should we support message editing?
- Do we need conversation branching/threads?

## Related Tasks
Tasks will be created once this PBI is approved and moved to "Agreed" status.

[View in Backlog](../backlog.md#user-content-3)