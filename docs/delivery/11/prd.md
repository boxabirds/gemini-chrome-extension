# PBI-11: State Management System

## Overview
Implement a robust state management system that synchronizes data across popup, service worker, and content scripts, with persistence and proper handling of concurrent updates.

## Problem Statement
Chrome extensions have multiple execution contexts that need to share state. Managing state consistency across these contexts while handling persistence and concurrent updates is complex and error-prone without a proper system.

## User Stories
As a developer, I want proper state management across extension contexts so that data stays synchronized.

## Technical Approach

### State Architecture

```mermaid
graph TB
    subgraph State Store
        CS[Central State]
        PS[Persistent State]
        TS[Transient State]
        DS[Derived State]
    end
    
    subgraph Contexts
        SW[Service Worker]
        POP[Popup UI]
        OPT[Options Page]
        CNT[Content Scripts]
    end
    
    subgraph Sync Layer
        MB[Message Bus]
        SM[Sync Manager]
        CR[Conflict Resolver]
    end
    
    CS --> PS
    CS --> TS
    CS --> DS
    
    SW <--> MB
    POP <--> MB
    OPT <--> MB
    CNT <--> MB
    
    MB <--> SM
    SM <--> CR
    SM <--> Chrome.Storage
```

### State Synchronization Flow

```mermaid
sequenceDiagram
    participant Popup as Popup UI
    participant SW as Service Worker
    participant SM as State Manager
    participant MB as Message Bus
    participant Storage as Chrome Storage
    participant CS as Content Script
    
    Popup->>SM: Update state
    SM->>SM: Validate change
    SM->>Storage: Persist state
    SM->>MB: Broadcast change
    
    par Notify all contexts
        MB->>SW: State updated
        SW->>SW: Update local copy
    and
        MB->>CS: State updated
        CS->>CS: Update local copy
    and
        MB->>OtherPopup: State updated
        OtherPopup->>OtherPopup: Update UI
    end
    
    Storage-->>SM: Write confirmed
    SM-->>Popup: Update complete
```

### State Model

```mermaid
classDiagram
    class ExtensionState {
        +auth: AuthState
        +conversation: ConversationState
        +tools: ToolsState
        +settings: SettingsState
        +ui: UIState
        +version: number
    }
    
    class StateManager {
        -state: ExtensionState
        -listeners: Map
        -storage: StorageAdapter
        +get(path): any
        +set(path, value): Promise
        +subscribe(path, callback): Unsubscribe
        +transaction(fn): Promise
    }
    
    class StorageAdapter {
        +get(keys): Promise
        +set(data): Promise
        +remove(keys): Promise
        +onChange(callback): void
    }
    
    class MessageBus {
        +broadcast(event): void
        +subscribe(event, handler): void
        +request(event, data): Promise
    }
    
    class ConflictResolver {
        +resolve(local, remote): State
        +merge(states): State
        +detectConflict(v1, v2): boolean
    }
    
    ExtensionState --* StateManager
    StateManager --> StorageAdapter
    StateManager --> MessageBus
    StateManager --> ConflictResolver
```

### Concurrent Update Handling

```mermaid
sequenceDiagram
    participant C1 as Context 1
    participant C2 as Context 2
    participant SM as State Manager
    participant CR as Conflict Resolver
    participant Storage as Chrome Storage
    
    Note over C1,C2: Concurrent updates
    
    C1->>SM: Update A
    C2->>SM: Update B
    
    SM->>SM: Queue updates
    
    SM->>Storage: Read version
    Storage-->>SM: Version 1
    
    SM->>CR: Check conflict(A, B)
    
    alt No conflict
        CR-->>SM: Merge changes
        SM->>Storage: Write merged
        Storage-->>SM: Version 2
        SM-->>C1: Success
        SM-->>C2: Success
    else Conflict exists
        CR->>CR: Apply resolution
        alt Last-write-wins
            CR-->>SM: Use update B
        else Merge possible
            CR-->>SM: Merged state
        else Manual resolution
            CR-->>C1: Conflict error
            CR-->>C2: Conflict error
        end
    end
```

### State Persistence Strategy

```mermaid
sequenceDiagram
    participant SM as State Manager
    participant Cache as Memory Cache
    participant Local as Local Storage
    participant Sync as Sync Storage
    participant Session as Session Storage
    
    SM->>SM: Categorize data
    
    alt Auth tokens
        SM->>Session: Store securely
        Session-->>SM: Stored
    else User preferences
        SM->>Sync: Sync across devices
        Sync-->>SM: Stored
    else Conversation history
        SM->>Local: Store locally
        Local-->>SM: Stored
    else Temporary UI state
        SM->>Cache: Keep in memory
        Cache-->>SM: Cached
    end
    
    Note over SM: Storage limits:<br/>Local: 10MB<br/>Sync: 100KB<br/>Session: 10MB
```

### State Migration

```mermaid
sequenceDiagram
    participant Ext as Extension
    participant SM as State Manager
    participant Mig as Migration Engine
    participant Storage as Chrome Storage
    
    Ext->>SM: Initialize
    SM->>Storage: Get stored state
    Storage-->>SM: State + version
    
    SM->>Mig: Check version
    
    alt Version outdated
        Mig->>Mig: Find migrations
        
        loop Each migration
            Mig->>Mig: Apply migration
            Note over Mig: v1→v2: Add new field<br/>v2→v3: Rename field<br/>v3→v4: Change structure
        end
        
        Mig-->>SM: Migrated state
        SM->>Storage: Save new version
    else Version current
        Mig-->>SM: No migration needed
    else Version newer
        Mig-->>SM: Error: Downgrade
    end
    
    SM-->>Ext: State ready
```

### Selective Sync

```mermaid
sequenceDiagram
    participant UI as UI Component
    participant SM as State Manager
    participant Sub as Subscription Manager
    participant MB as Message Bus
    
    UI->>SM: Subscribe('conversation.messages')
    SM->>Sub: Register listener
    
    Note over Sub: Subscription tree:<br/>conversation.*<br/>conversation.messages<br/>conversation.messages[0]
    
    Someone->>SM: Update state
    SM->>Sub: Find affected paths
    
    alt Path matches
        Sub->>UI: Notify change
        UI->>SM: Get new value
        SM-->>UI: Updated data
        UI->>UI: Re-render
    else Path doesn't match
        Note over UI: No update needed
    end
    
    UI->>Unsub: Component unmount
    Unsub->>Sub: Remove listener
```

### State Debugging

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant DT as Dev Tools
    participant SM as State Manager
    participant TM as Time Machine
    
    Dev->>DT: Open devtools
    DT->>SM: Enable debug mode
    
    SM->>TM: Start recording
    
    loop State changes
        Action->>SM: Modify state
        SM->>TM: Record change
        TM->>TM: Store snapshot
        TM->>DT: Emit event
        DT-->>Dev: Show in panel
    end
    
    Dev->>DT: Time travel
    DT->>TM: Restore snapshot
    TM->>SM: Apply state
    SM-->>All: Broadcast change
    
    Dev->>DT: Export state
    DT->>TM: Serialize
    TM-->>Dev: JSON export
```

## State Categories

### 1. Persistent State
- User authentication
- Conversation history
- User preferences
- Tool configurations

### 2. Transient State
- Current tab info
- Active tool execution
- Streaming responses
- UI element states

### 3. Synced State
- User settings
- Whitelisted sites
- Custom shortcuts
- Theme preferences

### 4. Session State
- Auth tokens
- Temporary caches
- Active sessions
- Rate limit counters

## UX/UI Considerations
- Loading states during sync
- Conflict resolution UI
- State reset options
- Debug panel for developers
- Storage usage indicators

## Acceptance Criteria
- [ ] State syncs across popup, background, and content scripts
- [ ] State persists across browser restarts
- [ ] Concurrent updates handled properly
- [ ] Selective state subscriptions work
- [ ] State migrations implemented
- [ ] Clear state reset functionality
- [ ] State debugging tools available
- [ ] Storage quota management
- [ ] Conflict resolution strategies

## Dependencies
- PBI-1: Extension foundation
- Chrome storage APIs
- Message passing system
- TypeScript for type safety

## Open Questions
- Should we use Redux/MobX or custom solution?
- How do we handle storage quota exceeded?
- Should state sync across devices by default?
- Do we need state versioning for migrations?

## Related Tasks
Tasks will be created once this PBI is approved and moved to "Agreed" status.

[View in Backlog](../backlog.md#user-content-11)