# PBI-7: Tool System Architecture

## Overview
Design and implement a robust, extensible tool system that provides a consistent interface for all automation capabilities, with built-in validation, error handling, and execution tracking.

## Problem Statement
The extension needs a well-architected tool system that can handle various automation tasks consistently, validate parameters, manage permissions, and provide a foundation for future tool development.

## User Stories
As a developer, I want a robust tool system architecture so that I can easily add new automation capabilities.

## Technical Approach

### Tool System Architecture

```mermaid
classDiagram
    class Tool {
        <<interface>>
        +name: string
        +displayName: string
        +description: string
        +parameters: JSONSchema
        +permissions: string[]
        +execute(params, context): Promise~Result~
        +validate(params): ValidationResult
    }
    
    class BaseTool {
        <<abstract>>
        +schema: ToolSchema
        +validateParams(params): ValidationResult
        +checkPermissions(): Promise~boolean~
        +requestPermissions(): Promise~boolean~
    }
    
    class ToolRegistry {
        -tools: Map~string, Tool~
        -enabledTools: Set~string~
        +register(tool): void
        +unregister(name): void
        +getTool(name): Tool
        +listTools(): ToolInfo[]
        +discover(): Promise~void~
    }
    
    class ToolExecutor {
        -registry: ToolRegistry
        -validator: ParamValidator
        -permissionChecker: PermissionChecker
        +execute(name, params, context): Promise~Result~
        +validateExecution(tool, params): ValidationResult
    }
    
    class ExecutionContext {
        +tabId: number
        +frameId: number
        +messageBus: MessageBus
        +user: UserInfo
        +settings: Settings
    }
    
    Tool <|-- BaseTool
    BaseTool <|-- DOMQueryTool
    BaseTool <|-- ClickTool
    BaseTool <|-- FormFillTool
    ToolRegistry --> Tool
    ToolExecutor --> ToolRegistry
    ToolExecutor --> ExecutionContext
```

### Tool Registration Flow

```mermaid
sequenceDiagram
    participant SW as Service Worker
    participant TR as Tool Registry
    participant Tool as Tool Implementation
    participant Storage as Chrome Storage
    
    SW->>TR: Initialize registry
    TR->>TR: Register built-in tools
    
    loop For each tool
        TR->>Tool: Validate schema
        Tool-->>TR: Schema valid
        TR->>TR: Add to registry
    end
    
    TR->>Storage: Load user preferences
    Storage-->>TR: Enabled tools list
    TR->>TR: Apply preferences
    
    SW->>TR: Discover external tools
    TR->>TR: Scan for tool modules
    TR-->>SW: Registry ready
```

### Tool Execution Flow

```mermaid
sequenceDiagram
    participant Gemini as Gemini AI
    participant TE as Tool Executor
    participant PV as Param Validator
    participant PC as Permission Checker
    participant Tool as Tool Instance
    participant EH as Error Handler
    
    Gemini->>TE: Execute tool(name, params)
    TE->>TE: Get tool from registry
    
    TE->>PV: Validate parameters
    PV->>PV: Check against schema
    
    alt Invalid params
        PV-->>TE: Validation errors
        TE-->>Gemini: Parameter error
    else Valid params
        PV-->>TE: Validation passed
        
        TE->>PC: Check permissions
        PC->>PC: Verify tool permissions
        
        alt Permissions denied
            PC-->>TE: Permission error
            TE-->>Gemini: Insufficient permissions
        else Permissions granted
            PC-->>TE: Authorized
            
            TE->>Tool: Execute(params, context)
            
            alt Execution success
                Tool-->>TE: Result data
                TE-->>Gemini: Success result
            else Execution error
                Tool-->>EH: Error details
                EH->>EH: Categorize error
                EH-->>TE: Error response
                TE-->>Gemini: Error result
            end
        end
    end
```

### Parameter Validation

```mermaid
sequenceDiagram
    participant TE as Tool Executor
    participant Val as JSON Schema Validator
    participant Tool as Tool
    participant Types as Type Checker
    
    TE->>Tool: Get parameter schema
    Tool-->>TE: JSONSchema definition
    
    TE->>Val: Validate(params, schema)
    
    Val->>Val: Check required fields
    Val->>Val: Validate types
    Val->>Val: Check constraints
    
    loop For each param
        Val->>Types: Validate type
        
        alt String param
            Types->>Types: Check length
            Types->>Types: Check pattern
        else Number param
            Types->>Types: Check range
            Types->>Types: Check precision
        else Array param
            Types->>Types: Check items
            Types->>Types: Check length
        else Object param
            Types->>Types: Validate nested
        end
        
        Types-->>Val: Type result
    end
    
    alt All valid
        Val-->>TE: Success
    else Validation errors
        Val-->>TE: Error details
    end
```

### Permission Management

```mermaid
sequenceDiagram
    participant Tool as Tool
    participant PM as Permission Manager
    participant Chrome as Chrome Permissions API
    participant User as User
    
    Tool->>PM: Request permissions
    PM->>PM: Get required permissions
    
    PM->>Chrome: Check current permissions
    Chrome-->>PM: Current state
    
    alt Has all permissions
        PM-->>Tool: Already granted
    else Missing permissions
        PM->>Chrome: Request permissions
        Chrome-->>User: Permission prompt
        
        alt User grants
            User->>Chrome: Allow
            Chrome-->>PM: Granted
            PM->>PM: Cache result
            PM-->>Tool: Success
        else User denies
            User->>Chrome: Deny
            Chrome-->>PM: Denied
            PM-->>Tool: Failed
        end
    end
```

### Error Handling Architecture

```mermaid
stateDiagram-v2
    [*] --> Executing: Start execution
    
    Executing --> ValidationError: Invalid params
    Executing --> PermissionError: No permission
    Executing --> ExecutionError: Runtime error
    Executing --> Success: Completed
    
    ValidationError --> [*]: Return error
    PermissionError --> RequestPermission: Can request
    PermissionError --> [*]: Cannot request
    
    RequestPermission --> Executing: Granted
    RequestPermission --> [*]: Denied
    
    ExecutionError --> Retry: Retryable
    ExecutionError --> [*]: Non-retryable
    
    Retry --> Executing: Attempt retry
    Retry --> [*]: Max retries
    
    Success --> [*]: Return result
    
    state ExecutionError {
        [*] --> NetworkError
        [*] --> TimeoutError
        [*] --> DOMError
        [*] --> UnknownError
    }
```

### Tool Discovery

```mermaid
sequenceDiagram
    participant TR as Tool Registry
    participant FS as File System
    participant Loader as Tool Loader
    participant Val as Tool Validator
    
    TR->>FS: Scan tool directories
    FS-->>TR: Tool file list
    
    loop For each file
        TR->>Loader: Load tool module
        Loader->>Loader: Import module
        
        alt Valid tool
            Loader-->>TR: Tool class
            TR->>Val: Validate tool
            Val->>Val: Check interface
            Val->>Val: Validate schema
            Val-->>TR: Valid
            TR->>TR: Register tool
        else Invalid tool
            Loader-->>TR: Load error
            TR->>TR: Log error
        end
    end
    
    TR->>TR: Sort by priority
    TR-->>System: Discovery complete
```

## Core Components

### 1. Base Tool Class
- Standard interface implementation
- Parameter validation
- Permission checking
- Error handling
- Execution context

### 2. Tool Registry
- Dynamic tool registration
- Tool discovery
- Enable/disable management
- Tool metadata
- Search and filtering

### 3. Tool Executor
- Execution orchestration
- Context management
- Result formatting
- Error propagation
- Execution tracking

### 4. Parameter Validator
- JSON Schema validation
- Type checking
- Custom validators
- Error messages
- Default values

### 5. Permission Manager
- Permission checking
- Dynamic requests
- User preferences
- Permission caching
- Audit logging

## UX/UI Considerations
- Clear tool descriptions in UI
- Parameter hints and examples
- Validation error messages
- Permission request dialogs
- Tool execution feedback

## Acceptance Criteria
- [ ] Base tool class with standard interface
- [ ] Tool registry with dynamic registration
- [ ] Parameter validation using JSON Schema
- [ ] Permission management system
- [ ] Consistent error handling
- [ ] Tool execution tracking
- [ ] Tool discovery mechanism
- [ ] Cancellation support
- [ ] Comprehensive logging

## Dependencies
- PBI-1: Extension foundation
- TypeScript for type safety
- JSON Schema library
- Chrome permissions API

## Open Questions
- Should tools support progress reporting?
- How should we handle tool versioning?
- Do we need tool categories/tags?
- Should tools have priority levels?

## Related Tasks
Tasks will be created once this PBI is approved and moved to "Agreed" status.

[View in Backlog](../backlog.md#user-content-7)