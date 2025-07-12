# PBI-12: Cross-Device Sync

## Overview
Implement cross-device synchronization for conversations and settings using Chrome's sync storage API, with selective sync options and proper conflict resolution.

## Problem Statement
Users who use Chrome on multiple devices expect their extension data to be available everywhere. The system needs to sync the right data while respecting privacy preferences and handling conflicts gracefully.

## User Stories
As a user, I want my conversations and settings to sync across devices so that I have a consistent experience.

## Technical Approach

### Sync Architecture

```mermaid
graph TB
    subgraph Local Device
        LE[Local Extension]
        LS[Local Storage]
        SS[Sync Storage]
        SM[Sync Manager]
    end
    
    subgraph Chrome Sync
        CS[Chrome Sync Service]
        CQ[Conflict Queue]
    end
    
    subgraph Other Devices
        D1[Device 1]
        D2[Device 2]
        D3[Device N]
    end
    
    LE <--> SM
    SM <--> LS
    SM <--> SS
    SS <--> CS
    CS <--> CQ
    CS <--> D1
    CS <--> D2
    CS <--> D3
```

### Sync Flow

```mermaid
sequenceDiagram
    actor User
    participant Ext as Extension
    participant SM as Sync Manager
    participant Local as Local Storage
    participant Sync as Sync Storage
    participant Chrome as Chrome Sync
    participant Remote as Other Devices
    
    User->>Ext: Change setting
    Ext->>SM: Update data
    
    SM->>Local: Save locally
    SM->>SM: Check sync preferences
    
    alt Sync enabled
        SM->>Sync: Write to sync
        Sync->>Chrome: Propagate
        Chrome->>Remote: Sync to devices
        Remote-->>User: Updated everywhere
    else Sync disabled
        Note over SM: Local only
    end
```

### Data Classification

```mermaid
graph TD
    subgraph Data Types
        Settings[User Settings]
        Conversations[Conversations]
        Tools[Tool Configs]
        Auth[Auth Data]
        History[Action History]
        Cache[Temporary Cache]
    end
    
    subgraph Sync Strategy
        Always[Always Sync]
        Optional[Optional Sync]
        Never[Never Sync]
    end
    
    Settings --> Always
    Tools --> Always
    Conversations --> Optional
    History --> Optional
    Auth --> Never
    Cache --> Never
    
    Always --> SyncStorage[Sync Storage<br/>100KB limit]
    Optional --> UserChoice{User Choice}
    UserChoice -->|Yes| SyncStorage
    UserChoice -->|No| LocalOnly[Local Storage Only]
    Never --> LocalOnly
```

### Selective Sync Configuration

```mermaid
sequenceDiagram
    participant User
    participant UI as Settings UI
    participant SM as Sync Manager
    participant Pref as Preferences
    
    User->>UI: Open sync settings
    UI->>SM: Get sync status
    SM-->>UI: Current config
    
    UI-->>User: Show options
    Note over UI: ☑ Settings<br/>☑ Tool configs<br/>☐ Conversations<br/>☐ Action history
    
    User->>UI: Toggle options
    UI->>SM: Update preferences
    
    SM->>Pref: Save choices
    SM->>SM: Apply filters
    
    alt Disabled item
        SM->>SM: Remove from sync
        SM->>Local: Move to local only
    else Enabled item
        SM->>SM: Add to sync
        SM->>Sync: Upload data
    end
    
    SM-->>UI: Config updated
    UI-->>User: Show confirmation
```

### Conflict Resolution

```mermaid
sequenceDiagram
    participant D1 as Device 1
    participant D2 as Device 2
    participant CS as Chrome Sync
    participant CR as Conflict Resolver
    participant User as User
    
    Note over D1,D2: Offline editing
    
    D1->>D1: Edit setting A→B
    D2->>D2: Edit setting A→C
    
    Note over D1,D2: Both come online
    
    D1->>CS: Sync change (A→B)
    D2->>CS: Sync change (A→C)
    
    CS->>CR: Conflict detected
    
    CR->>CR: Analyze conflict
    
    alt Auto-resolvable
        CR->>CR: Apply strategy
        Note over CR: Strategies:<br/>- Last write wins<br/>- Merge values<br/>- Union sets
        CR-->>CS: Resolved value
        CS->>D1: Update to resolved
        CS->>D2: Update to resolved
    else Manual resolution needed
        CR->>User: Show conflict UI
        User->>CR: Choose resolution
        CR-->>CS: User's choice
        CS->>D1: Apply choice
        CS->>D2: Apply choice
    end
```

### Storage Quota Management

```mermaid
sequenceDiagram
    participant SM as Sync Manager
    participant QM as Quota Monitor
    participant Comp as Compressor
    participant Arch as Archiver
    participant Sync as Sync Storage
    
    SM->>Sync: Write data
    Sync-->>SM: Quota warning (80%)
    
    SM->>QM: Handle quota
    QM->>QM: Analyze usage
    
    QM->>Comp: Compress data
    Comp->>Comp: Remove redundancy
    Comp->>Comp: Minify JSON
    Comp-->>QM: Reduced size
    
    alt Still over quota
        QM->>Arch: Archive old data
        Arch->>Arch: Move to local
        Arch->>Arch: Keep summary only
        Arch-->>QM: Freed space
    end
    
    QM->>Sync: Write optimized
    Sync-->>QM: Success
```

### Privacy Controls

```mermaid
stateDiagram-v2
    [*] --> SyncDisabled: Default
    
    SyncDisabled --> SetupSync: User enables
    SetupSync --> ChooseData: Select what to sync
    
    ChooseData --> BasicSync: Minimal data
    ChooseData --> FullSync: All permitted data
    ChooseData --> CustomSync: User selection
    
    BasicSync --> SyncActive
    FullSync --> SyncActive
    CustomSync --> SyncActive
    
    SyncActive --> PauseSync: Temporary disable
    PauseSync --> SyncActive: Resume
    
    SyncActive --> ClearRemote: Clear cloud data
    ClearRemote --> SyncDisabled: Confirm clear
    
    SyncActive --> SyncDisabled: Disable sync
    
    state SyncActive {
        [*] --> Monitoring
        Monitoring --> Syncing: Changes detected
        Syncing --> Monitoring: Sync complete
        Syncing --> ConflictUI: Conflict found
        ConflictUI --> Monitoring: Resolved
    }
```

### Sync Status Indicators

```mermaid
sequenceDiagram
    participant UI as Status UI
    participant SM as Sync Manager
    participant Mon as Sync Monitor
    participant User as User
    
    SM->>Mon: Start monitoring
    
    loop Continuous monitoring
        Mon->>Mon: Check sync status
        
        alt Syncing
            Mon->>UI: Show sync icon
            UI-->>User: "↻ Syncing..."
        else Synced
            Mon->>UI: Show checkmark
            UI-->>User: "✓ Synced"
        else Error
            Mon->>UI: Show error
            UI-->>User: "⚠ Sync error"
            UI->>UI: Show details on hover
        else Offline
            Mon->>UI: Show offline
            UI-->>User: "○ Offline"
        end
        
        Mon->>Mon: Wait interval
    end
```

### Data Migration for Sync

```mermaid
sequenceDiagram
    participant User
    participant SM as Sync Manager
    participant Mig as Migration Tool
    participant Local as Local Storage
    participant Sync as Sync Storage
    
    User->>SM: Enable sync first time
    SM->>Mig: Prepare for sync
    
    Mig->>Local: Read all data
    Local-->>Mig: Existing data
    
    Mig->>Mig: Filter syncable
    Mig->>Mig: Check size limits
    
    alt Under quota
        Mig->>Sync: Upload all
        Sync-->>Mig: Success
    else Over quota
        Mig->>Mig: Prioritize data
        Note over Mig: Priority:<br/>1. Settings<br/>2. Recent convos<br/>3. Tool configs
        Mig->>Sync: Upload priority
        Mig->>User: Notify partial sync
    end
    
    Mig->>SM: Migration complete
    SM-->>User: Sync enabled
```

## Sync Features

### 1. What Syncs
- **Always**: Extension settings, themes, shortcuts
- **Optional**: Conversations, tool configurations
- **Never**: Auth tokens, temporary data, caches

### 2. Conflict Resolution
- **Automatic**: Last-write-wins for simple values
- **Smart Merge**: Combine lists and sets
- **Manual**: User chooses for complex conflicts

### 3. Privacy Options
- Selective sync categories
- Clear remote data option
- Pause sync temporarily
- Local-only mode

### 4. Quota Management
- Automatic compression
- Old data archival
- Smart data prioritization
- Usage indicators

## UX/UI Considerations
- Clear sync status in toolbar
- Simple on/off toggle
- Advanced settings for power users
- Conflict resolution dialogs
- Storage usage visualization
- Sync history log

## Acceptance Criteria
- [ ] Chrome sync API integration complete
- [ ] Selective sync for different data types
- [ ] Conflict resolution for concurrent edits
- [ ] Clear sync status indication
- [ ] Option to disable sync
- [ ] Privacy controls implemented
- [ ] Quota management with compression
- [ ] Data migration from local to sync
- [ ] Sync error recovery

## Dependencies
- PBI-11: State management system
- Chrome sync storage API
- Chrome identity for user info
- Compression library

## Open Questions
- Should we encrypt data before syncing?
- How long to retain sync history?
- Should we support manual conflict resolution?
- Do we need sync analytics?

## Related Tasks
Tasks will be created once this PBI is approved and moved to "Agreed" status.

[View in Backlog](../backlog.md#user-content-12)