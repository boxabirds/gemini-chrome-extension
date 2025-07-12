# PBI-6: YouTube Playlist Management Tools

## Overview
Create specialized tools for YouTube playlist management that handle the platform's dynamic content loading, implement proper rate limiting, and provide bulk operations with safety measures.

## Problem Statement
YouTube playlists can contain thousands of items, and managing them manually is time-consuming. Users need automated tools that can handle YouTube's dynamic loading, respect rate limits, and safely perform bulk operations.

## User Stories
As a user, I want specialized tools for YouTube playlist management so that I can efficiently organize my playlists.

## Technical Approach

### YouTube Tool Architecture

```mermaid
graph TB
    subgraph YouTube Tools
        YPT[YouTube Playlist Tool]
        YDH[YouTube DOM Handler]
        YRL[YouTube Rate Limiter]
        YMO[YouTube Mutation Observer]
    end
    
    subgraph Operations
        LI[List Items]
        RI[Remove Items]
        RE[Reorder Items]
        FI[Filter Items]
    end
    
    subgraph Safety
        BC[Batch Controller]
        PG[Progress Monitor]
        CF[Confirmation]
    end
    
    YPT --> YDH
    YPT --> YRL
    YDH --> YMO
    YPT --> LI
    YPT --> RI
    YPT --> RE
    YPT --> FI
    BC --> PG
    BC --> CF
```

### Playlist Analysis Flow

```mermaid
sequenceDiagram
    actor User
    participant Chat as Chat UI
    participant YT as YouTube Tool
    participant CS as Content Script
    participant DOM as YouTube Page
    participant MO as Mutation Observer
    
    User->>Chat: "How many videos in this playlist?"
    Chat->>YT: Analyze playlist
    YT->>CS: Setup observer
    CS->>MO: Watch for items
    MO->>DOM: Observe container
    
    YT->>CS: Scroll to load all
    loop Load more items
        CS->>DOM: Scroll down
        DOM-->>MO: DOM mutation
        MO-->>CS: New items loaded
        CS->>CS: Count total items
        
        alt All loaded
            CS-->>YT: Complete count
        else More to load
            CS->>CS: Continue scrolling
        end
    end
    
    YT-->>Chat: "Playlist has 847 videos"
    Chat-->>User: Display count
```

### Bulk Remove Flow

```mermaid
sequenceDiagram
    actor User
    participant Chat as Chat UI
    participant YT as YouTube Tool
    participant BC as Batch Controller
    participant CS as Content Script
    participant DOM as YouTube DOM
    
    User->>Chat: "Remove first 100 videos"
    Chat->>YT: Remove items request
    YT->>BC: Initialize batch job
    
    BC->>Chat: Request confirmation
    Chat-->>User: "Remove 100 videos?"
    User->>Chat: Confirm
    
    BC->>CS: Start removal
    
    loop Batch processing
        BC->>CS: Get next batch (10 items)
        CS->>DOM: Find remove buttons
        
        loop For each item
            CS->>DOM: Click menu button
            CS->>CS: Wait for menu
            CS->>DOM: Click remove option
            CS->>CS: Wait for confirmation
            CS->>DOM: Confirm removal
            CS->>BC: Update progress
        end
        
        BC->>Chat: Progress update
        Chat-->>User: "40/100 removed"
        
        BC->>BC: Rate limit delay (1s)
    end
    
    BC-->>YT: Removal complete
    YT-->>Chat: "Removed 100 videos"
```

### Dynamic Content Handling

```mermaid
sequenceDiagram
    participant CS as Content Script
    participant MO as Mutation Observer
    participant DOM as YouTube DOM
    participant YDH as DOM Handler
    
    CS->>MO: Setup observer
    MO->>DOM: Watch playlist container
    
    Note over DOM: User scrolls or<br/>YouTube loads content
    
    DOM-->>MO: Mutation detected
    MO->>YDH: Process mutations
    
    alt New items added
        YDH->>YDH: Update item cache
        YDH->>YDH: Update selectors
        YDH-->>CS: Items ready
    else Items removed
        YDH->>YDH: Clean stale refs
        YDH-->>CS: Update count
    else Structure changed
        YDH->>YDH: Relearn selectors
        YDH->>DOM: Requery elements
        YDH-->>CS: Selectors updated
    end
```

### Rate Limiting Strategy

```mermaid
sequenceDiagram
    participant BC as Batch Controller
    participant RL as Rate Limiter
    participant Queue as Action Queue
    participant CS as Content Script
    
    BC->>RL: Check rate limit
    
    Note over RL: Limits:<br/>- 10 actions/minute<br/>- 1s between actions<br/>- 5s after errors
    
    RL->>RL: Calculate delay
    
    alt Under limit
        RL-->>BC: Proceed (0ms)
    else Near limit
        RL-->>BC: Delay (1000ms)
        BC->>BC: Wait
    else Over limit
        RL-->>BC: Delay (5000ms)
        BC->>BC: Extended wait
    end
    
    BC->>Queue: Execute action
    Queue->>CS: Perform operation
    
    alt Success
        CS-->>Queue: Complete
        Queue->>RL: Update success count
    else Error
        CS-->>Queue: Failed
        Queue->>RL: Update error count
        RL->>RL: Increase delay
    end
```

### Error Recovery

```mermaid
sequenceDiagram
    participant YT as YouTube Tool
    participant CS as Content Script
    participant ER as Error Recovery
    participant DOM as YouTube DOM
    
    YT->>CS: Remove playlist item
    CS->>DOM: Click menu
    
    alt Menu not found
        DOM-->>CS: Element missing
        CS->>ER: Handle stale element
        ER->>CS: Refresh selectors
        ER->>CS: Retry with new selector
        CS->>DOM: Click menu (retry)
    else YouTube error
        DOM-->>CS: Error message shown
        CS->>ER: YouTube API error
        ER->>CS: Wait 5s
        ER->>CS: Refresh page section
        ER->>CS: Resume from last position
    else Rate limited
        DOM-->>CS: 429 response
        CS->>ER: Rate limit hit
        ER->>CS: Exponential backoff
        ER->>CS: Resume after delay
    end
    
    ER-->>YT: Recovery result
```

### Progress Monitoring

```mermaid
sequenceDiagram
    participant User
    participant UI as Progress UI
    participant BC as Batch Controller
    participant PM as Progress Monitor
    
    BC->>PM: Start job (100 items)
    PM->>UI: Initialize progress bar
    UI-->>User: "0/100 complete"
    
    loop Processing
        BC->>PM: Item completed
        PM->>PM: Update stats
        PM->>UI: Update display
        
        UI-->>User: "25/100 complete"
        UI-->>User: "~3 min remaining"
        UI-->>User: Show progress bar
        
        alt User cancels
            User->>UI: Click cancel
            UI->>BC: Cancel signal
            BC->>BC: Stop processing
            BC->>PM: Job cancelled
            PM->>UI: Show final stats
            UI-->>User: "Cancelled: 25/100"
        end
    end
    
    PM->>UI: Job complete
    UI-->>User: "✓ 100/100 complete"
```

## YouTube-Specific Features

### 1. Smart Selection
- Select by date range
- Select by video duration
- Select by channel
- Select by title pattern

### 2. Batch Operations
- Remove in batches
- Reorder by criteria
- Move between playlists
- Export playlist data

### 3. Safety Features
- Dry run mode
- Undo capability (within session)
- Operation preview
- Automatic backups

## UX/UI Considerations
- Clear progress indicators with time estimates
- Ability to pause/resume operations
- Preview of items to be affected
- Confirmation with item count
- Real-time progress updates
- Cancel button always visible

## Acceptance Criteria
- [ ] List all playlist items with metadata
- [ ] Bulk remove with batching and rate limiting
- [ ] Handle YouTube's infinite scroll
- [ ] Progress indication with time estimates
- [ ] Confirmation dialogs show item count
- [ ] Graceful handling of YouTube errors
- [ ] Resume capability after interruption
- [ ] Respect YouTube's rate limits
- [ ] Work with different playlist sizes

## Dependencies
- PBI-5: Web automation tools foundation
- PBI-7: Tool system architecture
- Content script access to YouTube
- Mutation Observer API support

## Open Questions
- Should we support moving items between playlists?
- How should we handle private/deleted videos?
- Do we need playlist backup functionality?
- Should we support collaborative playlists?

## Related Tasks
Tasks will be created once this PBI is approved and moved to "Agreed" status.

[View in Backlog](../backlog.md#user-content-6)