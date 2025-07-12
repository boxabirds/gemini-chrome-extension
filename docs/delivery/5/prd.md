# PBI-5: Web Automation Tools

## Overview
Implement core web automation tools that allow Gemini to perform actions on web pages, including clicking elements, filling forms, and navigating, with appropriate safety confirmations.

## Problem Statement
Users want to automate repetitive web tasks through natural language commands. The system needs to safely execute DOM manipulations while providing clear feedback and maintaining user control.

## User Stories
As a user, I want Gemini to perform automated actions on web pages so that I can save time on repetitive tasks.

## Technical Approach

### Tool System Architecture

```mermaid
graph TB
    subgraph Tool Registry
        TR[Tool Registry]
        BT[Base Tool Class]
        DQ[DOM Query Tool]
        CT[Click Tool]
        FT[Form Fill Tool]
        ST[Scroll Tool]
        WT[Wait Tool]
    end
    
    subgraph Execution Engine
        TE[Tool Executor]
        PV[Param Validator]
        CP[Confirmation Prompt]
        RL[Rate Limiter]
    end
    
    subgraph Safety Layer
        PC[Permission Checker]
        SC[Site Checker]
        AC[Action Logger]
    end
    
    TR --> BT
    BT --> DQ
    BT --> CT
    BT --> FT
    BT --> ST
    BT --> WT
    
    TE --> PV
    TE --> CP
    TE --> RL
    TE --> PC
    PC --> SC
    TE --> AC
```

### Tool Execution Flow

```mermaid
sequenceDiagram
    actor User
    participant Chat as Chat UI
    participant Gemini as Gemini AI
    participant SW as Service Worker
    participant TR as Tool Registry
    participant TE as Tool Executor
    participant CS as Content Script
    participant DOM as Web Page
    
    User->>Chat: "Click the submit button"
    Chat->>Gemini: Process request
    Gemini->>SW: Execute tool: click_element
    SW->>TR: Get tool definition
    TR-->>SW: ClickTool
    
    SW->>TE: Validate parameters
    TE->>TE: Check permissions
    
    alt Requires confirmation
        TE->>Chat: Show confirmation
        Chat-->>User: "Click submit button?"
        User->>Chat: Approve
        Chat->>TE: User approved
    end
    
    TE->>CS: Execute click
    CS->>DOM: Find element
    CS->>DOM: element.click()
    DOM-->>CS: Event fired
    CS-->>TE: Success
    TE-->>SW: Result
    SW-->>Gemini: Tool completed
    Gemini-->>Chat: Response
    Chat-->>User: "Clicked submit button"
```

### Safety Confirmation Flow

```mermaid
sequenceDiagram
    participant TE as Tool Executor
    participant CP as Confirmation Prompt
    participant UI as Popup UI
    participant User as User
    participant AL as Action Log
    
    TE->>CP: Request confirmation
    CP->>CP: Check auto-approve list
    
    alt Auto-approved
        CP-->>TE: Proceed
    else Needs confirmation
        CP->>UI: Show dialog
        UI-->>User: Display action details
        
        alt User approves
            User->>UI: Approve
            UI->>CP: Approved
            
            alt Remember choice
                User->>UI: "Always allow"
                UI->>CP: Add to auto-approve
            end
            
            CP-->>TE: Proceed
        else User denies
            User->>UI: Deny
            UI->>CP: Denied
            CP-->>TE: Cancelled
        end
    end
    
    TE->>AL: Log action + result
```

### Form Filling Flow

```mermaid
sequenceDiagram
    participant Gemini as Gemini AI
    participant FT as Form Fill Tool
    participant CS as Content Script
    participant DOM as DOM
    participant Val as Validator
    
    Gemini->>FT: Fill form data
    FT->>CS: Get form fields
    CS->>DOM: Find form elements
    DOM-->>CS: Input fields
    
    CS->>FT: Field analysis
    FT->>Val: Validate data types
    
    loop For each field
        FT->>CS: Fill field
        
        alt Text input
            CS->>DOM: Set value
            CS->>DOM: Dispatch input event
        else Select dropdown
            CS->>DOM: Set selectedIndex
            CS->>DOM: Dispatch change event
        else Checkbox
            CS->>DOM: Set checked
            CS->>DOM: Dispatch click event
        else Radio button
            CS->>DOM: Set checked
            CS->>DOM: Dispatch change event
        end
        
        DOM-->>CS: Events fired
    end
    
    CS-->>FT: Form filled
    FT-->>Gemini: Success
```

### Rate Limiting

```mermaid
sequenceDiagram
    participant TE as Tool Executor
    participant RL as Rate Limiter
    participant Queue as Action Queue
    participant Timer as Timer
    
    TE->>RL: Execute action
    RL->>RL: Check rate limit
    
    alt Under limit
        RL->>Queue: Execute immediately
        Queue-->>TE: Action completed
        RL->>RL: Update counter
    else Over limit
        RL->>Queue: Add to queue
        RL->>Timer: Start delay
        Note over Timer: Wait period
        Timer-->>RL: Delay complete
        RL->>Queue: Process next
        Queue-->>TE: Action completed
    end
    
    Note over RL: Sliding window:<br/>10 actions per minute<br/>Site-specific limits
```

### Error Recovery

```mermaid
sequenceDiagram
    participant TE as Tool Executor
    participant CS as Content Script
    participant DOM as DOM
    participant ER as Error Recovery
    
    TE->>CS: Execute action
    CS->>DOM: Find element
    
    alt Element not found
        DOM-->>CS: null
        CS->>ER: Element not found
        ER->>CS: Wait 1s and retry
        CS->>DOM: Find element (retry)
        
        alt Found after wait
            DOM-->>CS: Element
            CS->>DOM: Execute action
        else Still not found
            CS-->>TE: Error: Element not found
            TE-->>User: "Cannot find element"
        end
        
    else Element not visible
        CS->>ER: Element hidden
        ER->>CS: Scroll to element
        CS->>DOM: scrollIntoView()
        CS->>CS: Wait for visibility
        CS->>DOM: Execute action
        
    else Element disabled
        CS-->>TE: Error: Element disabled
        TE-->>User: "Element is disabled"
    end
```

## Available Tools

### 1. DOM Query Tool
- Find elements by CSS selector
- Count matching elements
- Extract text content
- Get element attributes

### 2. Click Tool
- Click buttons and links
- Support for dynamic elements
- Handle navigation after click

### 3. Form Fill Tool
- Fill text inputs
- Select dropdowns
- Check/uncheck boxes
- Handle radio buttons

### 4. Scroll Tool
- Scroll to elements
- Scroll by pixels
- Scroll to top/bottom

### 5. Wait Tool
- Wait for elements to appear
- Wait for specific conditions
- Custom wait times

## UX/UI Considerations
- Clear confirmation dialogs for actions
- Visual highlighting of target elements
- Progress indicators for multi-step actions
- Ability to stop automation mid-process
- Action history and undo capabilities

## Acceptance Criteria
- [ ] DOM element selection using CSS selectors
- [ ] Click automation with element highlighting
- [ ] Form filling with type validation
- [ ] Safety confirmations for destructive actions
- [ ] Visual feedback during automation
- [ ] Rate limiting to prevent abuse
- [ ] Error handling with retry logic
- [ ] Action logging for audit trail
- [ ] Ability to cancel running automations

## Dependencies
- PBI-1: Extension foundation
- PBI-3: Conversation interface
- PBI-7: Tool system architecture
- Content script permissions
- ActiveTab permission

## Open Questions
- Should we support XPath selectors in addition to CSS?
- How should we handle file upload inputs?
- Do we need recording/playback functionality?
- Should we support keyboard event simulation?

## Related Tasks
Tasks will be created once this PBI is approved and moved to "Agreed" status.

[View in Backlog](../backlog.md#user-content-5)