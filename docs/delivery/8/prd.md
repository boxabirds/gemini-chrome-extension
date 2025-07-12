# PBI-8: Permission and Safety System

## Overview
Implement a comprehensive permission and safety system that gives users fine-grained control over AI actions, with site-specific rules, action confirmations, and detailed audit logging.

## Problem Statement
Users need to maintain control over what actions the AI can perform on their behalf. The system must balance automation convenience with safety, providing clear visibility and control over all automated actions.

## User Stories
As a user, I want fine-grained control over what actions the AI can perform so that I maintain control over my browsing experience.

## Technical Approach

### Permission System Architecture

```mermaid
graph TB
    subgraph Permission Core
        PM[Permission Manager]
        PS[Permission Store]
        PR[Permission Rules]
        PE[Permission Evaluator]
    end
    
    subgraph Safety Controls
        CF[Confirmation System]
        WL[Whitelist Manager]
        BL[Blacklist Manager]
        AL[Action Logger]
    end
    
    subgraph User Interface
        PUI[Permission UI]
        CD[Confirmation Dialogs]
        AV[Audit Viewer]
        QT[Quick Toggle]
    end
    
    PM --> PS
    PM --> PR
    PM --> PE
    PE --> CF
    PE --> WL
    PE --> BL
    PM --> AL
    
    PUI --> PM
    CD --> CF
    AV --> AL
    QT --> PM
```

### Permission Evaluation Flow

```mermaid
sequenceDiagram
    participant Tool as Tool
    participant PE as Permission Evaluator
    participant PS as Permission Store
    participant WL as Whitelist
    participant BL as Blacklist
    participant CF as Confirmation
    participant User as User
    
    Tool->>PE: Request permission(action, site)
    
    PE->>BL: Check blacklist
    alt Site blacklisted
        BL-->>PE: Blocked
        PE-->>Tool: Permission denied
    else Not blacklisted
        PE->>WL: Check whitelist
        
        alt Whitelisted
            WL-->>PE: Auto-approved
            PE-->>Tool: Permission granted
        else Not whitelisted
            PE->>PS: Get permission rules
            PS-->>PE: Rules for action/site
            
            alt Rules allow
                PE-->>Tool: Permission granted
            else Rules require confirmation
                PE->>CF: Request confirmation
                CF-->>User: Show dialog
                User->>CF: Decision
                CF-->>PE: User response
                
                alt User approved
                    PE-->>Tool: Permission granted
                else User denied
                    PE-->>Tool: Permission denied
                end
            else Rules deny
                PE-->>Tool: Permission denied
            end
        end
    end
    
    PE->>AL: Log decision
```

### Confirmation Dialog Flow

```mermaid
sequenceDiagram
    participant Tool as Tool
    participant CF as Confirmation System
    participant UI as Confirmation UI
    participant User as User
    participant PS as Permission Store
    
    Tool->>CF: Need confirmation(action, details)
    CF->>CF: Check recent confirmations
    
    alt Recently approved similar
        CF-->>Tool: Auto-approve
    else Needs user input
        CF->>UI: Show dialog
        UI->>UI: Format action details
        UI-->>User: Display confirmation
        
        Note over User,UI: Dialog shows:<br/>- Action type<br/>- Target element<br/>- Site name<br/>- Potential impact
        
        User->>UI: Make choice
        
        alt Approve once
            UI->>CF: Approved
            CF-->>Tool: Proceed
        else Always allow this action
            UI->>CF: Always allow
            CF->>PS: Add to whitelist
            CF-->>Tool: Proceed
        else Always allow on this site
            UI->>CF: Site whitelist
            CF->>PS: Add site rule
            CF-->>Tool: Proceed
        else Deny
            UI->>CF: Denied
            CF-->>Tool: Cancelled
        end
    end
    
    CF->>AL: Log user decision
```

### Site-Specific Rules

```mermaid
sequenceDiagram
    participant PM as Permission Manager
    participant SR as Site Rules
    participant Pattern as URL Pattern Matcher
    participant Store as Rule Store
    
    PM->>SR: Evaluate site(url)
    SR->>Pattern: Match URL patterns
    
    loop Check patterns
        Pattern->>Store: Get rules for pattern
        Store-->>Pattern: Matching rules
        
        alt Exact match
            Pattern-->>SR: Use exact rules
        else Domain match
            Pattern-->>SR: Use domain rules
        else Wildcard match
            Pattern-->>SR: Use wildcard rules
        else Default
            Pattern-->>SR: Use default rules
        end
    end
    
    SR->>SR: Merge rule hierarchy
    SR-->>PM: Effective rules
    
    Note over SR: Rule precedence:<br/>1. Exact URL<br/>2. Subdomain<br/>3. Domain<br/>4. Wildcard<br/>5. Default
```

### Action Audit Log

```mermaid
sequenceDiagram
    participant Tool as Tool
    participant AL as Action Logger
    participant Store as Log Store
    participant Index as Search Index
    participant UI as Audit UI
    
    Tool->>AL: Log action(type, details, result)
    AL->>AL: Add metadata
    Note over AL: Metadata:<br/>- Timestamp<br/>- User decision<br/>- Site/URL<br/>- Duration<br/>- Success/Error
    
    AL->>Store: Save log entry
    AL->>Index: Update search index
    
    User->>UI: View audit log
    UI->>Store: Query logs
    
    alt Filter by date
        UI->>Store: Date range query
    else Filter by site
        UI->>Index: Site search
    else Filter by action
        UI->>Index: Action type search
    else Search text
        UI->>Index: Full text search
    end
    
    Store-->>UI: Matching entries
    UI-->>User: Display results
```

### Quick Toggle System

```mermaid
stateDiagram-v2
    [*] --> Enabled: Default
    
    Enabled --> Paused: User pauses
    Enabled --> Disabled: User disables
    
    Paused --> Enabled: Resume
    Paused --> Disabled: Disable
    
    Disabled --> Enabled: Re-enable
    
    state Enabled {
        [*] --> FullAuto: Whitelist match
        [*] --> Confirmations: Normal mode
        [*] --> ReadOnly: Safe mode
        
        FullAuto --> Confirmations: Remove whitelist
        Confirmations --> FullAuto: Add whitelist
        Confirmations --> ReadOnly: Enable safe mode
        ReadOnly --> Confirmations: Disable safe mode
    }
    
    state Paused {
        note: All actions blocked<br/>Temporary pause
    }
    
    state Disabled {
        note: Extension inactive<br/>No automation
    }
```

### Permission Inheritance

```mermaid
graph TD
    Global[Global Permissions]
    Domain[Domain Permissions]
    Subdomain[Subdomain Permissions]
    Page[Page Permissions]
    Action[Action Permissions]
    
    Global --> Domain
    Domain --> Subdomain
    Subdomain --> Page
    Page --> Action
    
    Global -.- G1[Default: Confirm all]
    Domain -.- D1[youtube.com: Auto-allow reads]
    Subdomain -.- S1[mail.google.com: Restrict writes]
    Page -.- P1[Specific URL: Custom rules]
    Action -.- A1[Final permission decision]
```

## Safety Features

### 1. Action Categories
- **Read-only**: DOM queries, page analysis
- **Low-risk**: Scrolling, hovering
- **Medium-risk**: Clicking, form filling
- **High-risk**: Submitting forms, navigation
- **Critical**: Payments, deletions

### 2. Confirmation Levels
- **Always**: Every action needs confirmation
- **Smart**: Based on risk assessment
- **Trusted**: Only high-risk actions
- **None**: Fully automated (not recommended)

### 3. Site Categories
- **Trusted**: User's own sites
- **Financial**: Banks, payment processors
- **Social**: Social media platforms
- **Shopping**: E-commerce sites
- **Unknown**: Unrecognized sites

## UX/UI Considerations
- Clear permission status in toolbar icon
- Quick toggle for enable/disable
- Unobtrusive confirmation dialogs
- Detailed but readable audit logs
- Bulk permission management
- Export/import permission settings

## Acceptance Criteria
- [ ] Permission system for different action types
- [ ] Site-specific whitelists and blacklists
- [ ] Confirmation dialogs with remember options
- [ ] Comprehensive audit log with search
- [ ] Quick toggle to pause/resume automation
- [ ] Permission inheritance and precedence
- [ ] Risk-based confirmation levels
- [ ] Settings export/import
- [ ] Clear permission status indicators

## Dependencies
- PBI-1: Extension foundation
- PBI-5: Web automation tools
- Chrome storage for persistence
- Chrome notifications API

## Open Questions
- Should permissions sync across devices?
- How long should audit logs be retained?
- Should we support regex patterns for URLs?
- Do we need permission templates?

## Related Tasks
Tasks will be created once this PBI is approved and moved to "Agreed" status.

[View in Backlog](../backlog.md#user-content-8)