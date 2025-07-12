# PBI-9: Testing Infrastructure

## Overview
Establish comprehensive testing infrastructure including unit tests, integration tests, mocks for Chrome APIs, and end-to-end tests to ensure reliability and enable confident development.

## Problem Statement
Chrome extensions have unique testing challenges due to their multi-context architecture and dependency on Chrome APIs. We need a robust testing framework that covers all components and enables rapid development with confidence.

## User Stories
As a developer, I want comprehensive testing infrastructure so that I can ensure reliability and catch regressions.

## Technical Approach

### Testing Architecture

```mermaid
graph TB
    subgraph Test Types
        UT[Unit Tests]
        IT[Integration Tests]
        E2E[E2E Tests]
        VT[Visual Tests]
    end
    
    subgraph Test Infrastructure
        TF[Test Framework<br/>Jest/Vitest]
        MC[Mock Chrome APIs]
        TU[Test Utilities]
        FX[Fixtures]
    end
    
    subgraph CI/CD
        GH[GitHub Actions]
        LC[Lint & Format]
        TB[Test & Build]
        CR[Coverage Report]
    end
    
    UT --> TF
    IT --> TF
    E2E --> Puppeteer
    VT --> Percy
    
    TF --> MC
    TF --> TU
    TF --> FX
    
    GH --> LC
    GH --> TB
    TB --> CR
```

### Unit Test Strategy

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Test as Test Suite
    participant Mock as Mock APIs
    participant SUT as System Under Test
    participant Assert as Assertions
    
    Dev->>Test: Run unit test
    Test->>Mock: Setup Chrome mocks
    Mock->>Mock: Mock storage API
    Mock->>Mock: Mock identity API
    Mock->>Mock: Mock tabs API
    
    Test->>SUT: Create instance
    Test->>SUT: Execute method
    SUT->>Mock: Call Chrome API
    Mock-->>SUT: Return mock data
    SUT-->>Test: Return result
    
    Test->>Assert: Verify result
    Test->>Assert: Verify mock calls
    Assert-->>Dev: Test result
```

### Integration Test Flow

```mermaid
sequenceDiagram
    participant Test as Test Runner
    participant SW as Service Worker
    participant CS as Content Script
    participant Mock as Mock APIs
    participant MB as Message Bus
    
    Test->>Mock: Setup test environment
    Test->>SW: Load service worker
    Test->>CS: Inject content script
    
    Test->>SW: Send test message
    SW->>MB: Process message
    MB->>CS: Forward to content
    CS->>Mock: Interact with DOM
    Mock-->>CS: Mock response
    CS-->>MB: Return result
    MB-->>SW: Forward result
    SW-->>Test: Final response
    
    Test->>Test: Assert behavior
    Test->>Test: Verify side effects
```

### E2E Test Architecture

```mermaid
sequenceDiagram
    participant Test as E2E Test
    participant Puppet as Puppeteer
    participant Chrome as Chrome Browser
    participant Ext as Extension
    participant Page as Test Page
    
    Test->>Puppet: Launch Chrome
    Puppet->>Chrome: Start with extension
    Chrome->>Ext: Load extension
    
    Test->>Chrome: Navigate to test page
    Chrome->>Page: Load page
    
    Test->>Chrome: Click extension icon
    Chrome->>Ext: Open popup
    
    Test->>Ext: Interact with UI
    Ext->>Page: Execute automation
    Page-->>Ext: Result
    Ext-->>Test: Verify outcome
    
    Test->>Puppet: Close browser
```

### Mock Chrome APIs

```mermaid
classDiagram
    class ChromeMock {
        +storage: StorageMock
        +identity: IdentityMock
        +tabs: TabsMock
        +runtime: RuntimeMock
        +reset(): void
    }
    
    class StorageMock {
        -data: Map
        +local: LocalStorage
        +sync: SyncStorage
        +session: SessionStorage
    }
    
    class IdentityMock {
        -tokens: Map
        +getAuthToken(options): Promise
        +removeCachedAuthToken(token): Promise
        +getProfileUserInfo(): Promise
    }
    
    class TabsMock {
        -tabs: Map
        +query(options): Promise
        +sendMessage(tabId, message): Promise
        +create(options): Promise
    }
    
    class RuntimeMock {
        -messages: EventEmitter
        +sendMessage(message): Promise
        +onMessage: MessageListener
        +lastError: Error?
    }
    
    ChromeMock --> StorageMock
    ChromeMock --> IdentityMock
    ChromeMock --> TabsMock
    ChromeMock --> RuntimeMock
```

### Test Data Management

```mermaid
sequenceDiagram
    participant Test as Test Case
    participant Fix as Fixture Loader
    participant Build as Test Builder
    participant DB as Test Database
    
    Test->>Fix: Load fixture('conversation')
    Fix->>DB: Get template
    DB-->>Fix: Conversation template
    
    Fix->>Build: Build test data
    Build->>Build: Generate IDs
    Build->>Build: Set timestamps
    Build->>Build: Apply overrides
    Build-->>Test: Test data ready
    
    Test->>Test: Execute test
    Test->>DB: Cleanup test data
```

### Coverage Strategy

```mermaid
graph LR
    subgraph Coverage Targets
        Unit[Unit: 80%]
        Integration[Integration: 70%]
        E2E[E2E: Critical Paths]
        Overall[Overall: 75%]
    end
    
    subgraph Coverage Areas
        SW[Service Worker]
        CS[Content Scripts]
        UI[UI Components]
        Tools[Tool System]
        Auth[Authentication]
    end
    
    subgraph Reports
        Local[Local HTML]
        CI[CI Reports]
        PR[PR Comments]
        Badge[Coverage Badge]
    end
    
    Unit --> SW
    Unit --> Tools
    Unit --> Auth
    Integration --> CS
    Integration --> UI
    E2E --> CriticalPaths
    
    SW --> Local
    CS --> Local
    UI --> Local
    Local --> CI
    CI --> PR
    CI --> Badge
```

### CI/CD Pipeline

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant CI as GitHub Actions
    participant Test as Test Suite
    participant Build as Build System
    participant Report as Reports
    
    Dev->>GH: Push code
    GH->>CI: Trigger workflow
    
    par Parallel jobs
        CI->>Test: Run linting
        Test-->>CI: Lint results
    and
        CI->>Test: Run type check
        Test-->>CI: Type results
    and
        CI->>Test: Run unit tests
        Test-->>CI: Unit results
    end
    
    CI->>Test: Run integration tests
    Test-->>CI: Integration results
    
    CI->>Build: Build extension
    Build-->>CI: Build artifacts
    
    CI->>Test: Run E2E tests
    Test-->>CI: E2E results
    
    CI->>Report: Generate reports
    Report->>GH: Comment on PR
    Report->>GH: Update status
    
    alt All passed
        CI-->>Dev: ✅ Success
    else Failed
        CI-->>Dev: ❌ Failure details
    end
```

## Test Categories

### 1. Unit Tests
- Pure functions
- Class methods
- React components
- Tool implementations
- Utility functions

### 2. Integration Tests
- Message passing
- State synchronization
- API communication
- Tool execution
- Permission checks

### 3. E2E Tests
- Authentication flow
- Chat conversation
- Tool execution
- Permission dialogs
- Error recovery

### 4. Performance Tests
- Message latency
- Memory usage
- Storage efficiency
- API response times

## Testing Utilities

### 1. Test Builders
```typescript
const conversation = buildConversation()
  .withMessages(5)
  .withToolCalls(2)
  .build();
```

### 2. Async Helpers
```typescript
await waitForMessage('TOOL_COMPLETE');
await expectEventually(() => 
  screen.getByText('Success')
);
```

### 3. Chrome API Mocks
```typescript
mockChrome.storage.local.get
  .mockResolvedValue({ token: 'test' });
```

## UX/UI Considerations
- Test results in PR comments
- Coverage badges in README
- Local test runner UI
- Visual regression reports
- Performance benchmarks

## Acceptance Criteria
- [ ] Unit test framework configured (Jest/Vitest)
- [ ] Chrome API mocks implemented
- [ ] Integration test setup complete
- [ ] E2E tests with Puppeteer working
- [ ] CI/CD pipeline on GitHub Actions
- [ ] Coverage reporting integrated
- [ ] Test utilities and builders created
- [ ] Documentation for test patterns
- [ ] Pre-commit hooks for tests

## Dependencies
- Jest or Vitest test framework
- Puppeteer for E2E tests
- Testing Library for React
- GitHub Actions for CI/CD
- NYC for coverage reports

## Open Questions
- Should we use Jest or Vitest?
- Do we need visual regression testing?
- What's our target coverage percentage?
- Should we test across multiple Chrome versions?

## Related Tasks
Tasks will be created once this PBI is approved and moved to "Agreed" status.

[View in Backlog](../backlog.md#user-content-9)