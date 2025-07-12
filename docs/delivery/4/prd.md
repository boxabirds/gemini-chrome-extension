# PBI-4: Page Content Analysis

## Overview
Enable Gemini to analyze and understand the content of the current web page, allowing users to ask questions and get insights about what they're viewing.

## Problem Statement
Users want to leverage AI to understand, summarize, and interact with web page content. The extension needs to safely extract page content, respect security boundaries, and provide this context to Gemini for analysis.

## User Stories
As a user, I want Gemini to analyze the current web page content so that I can get insights and assistance about what I'm viewing.

## Technical Approach

### Content Extraction Architecture

```mermaid
graph TB
    subgraph Content Script
        CE[Content Extractor]
        TI[Text Identifier]
        II[Image Identifier]
        MD[Metadata Collector]
        SF[Security Filter]
    end
    
    subgraph Processing
        CP[Content Processor]
        CS[Content Serializer]
        CC[Content Chunker]
    end
    
    subgraph Analysis
        GA[Gemini Analyzer]
        CR[Context Builder]
        RP[Response Processor]
    end
    
    CE --> TI
    CE --> II
    CE --> MD
    TI --> SF
    II --> SF
    MD --> SF
    SF --> CP
    CP --> CS
    CS --> CC
    CC --> GA
    GA --> CR
    CR --> RP
```

### Content Extraction Flow

```mermaid
sequenceDiagram
    actor User
    participant Popup as Popup UI
    participant SW as Service Worker
    participant CS as Content Script
    participant DOM as Web Page
    participant Gemini as Gemini API
    
    User->>Popup: "What is this page about?"
    Popup->>SW: Analyze page request
    SW->>CS: Extract page content
    
    CS->>DOM: document.title
    DOM-->>CS: Page title
    CS->>DOM: document.body.innerText
    DOM-->>CS: Text content
    CS->>DOM: document.images
    DOM-->>CS: Image elements
    CS->>DOM: meta tags
    DOM-->>CS: Metadata
    
    CS->>CS: Filter sensitive content
    CS->>CS: Serialize to JSON
    CS-->>SW: Page content data
    
    SW->>Gemini: Analyze with context
    Gemini-->>SW: Analysis response
    SW-->>Popup: Display insights
    Popup-->>User: Show analysis
```

### Security Filtering

```mermaid
sequenceDiagram
    participant CS as Content Script
    participant SF as Security Filter
    participant DOM as DOM Elements
    
    CS->>DOM: Query all elements
    DOM-->>CS: Element list
    
    loop For each element
        CS->>SF: Check element
        
        alt Password field
            SF-->>CS: Skip element
        else Credit card field
            SF-->>CS: Skip element
        else Hidden element
            SF-->>CS: Check visibility rules
        else Normal content
            SF-->>CS: Include element
        end
    end
    
    CS->>SF: Filter complete content
    SF->>SF: Remove PII patterns
    SF->>SF: Remove auth tokens
    SF->>SF: Sanitize URLs
    SF-->>CS: Safe content
```

### Image Handling

```mermaid
sequenceDiagram
    participant CS as Content Script
    participant IH as Image Handler
    participant DOM as DOM
    participant SW as Service Worker
    
    CS->>DOM: Get all images
    DOM-->>CS: Image elements
    
    loop For each image
        CS->>IH: Process image
        
        alt Data URL
            IH->>IH: Extract base64
            IH-->>CS: Image data
        else External URL
            IH->>IH: Check CORS
            alt CORS allowed
                IH->>SW: Fetch image
                SW-->>IH: Image blob
                IH->>IH: Convert to base64
                IH-->>CS: Image data
            else CORS blocked
                IH-->>CS: Image URL only
            end
        else SVG
            IH->>IH: Serialize SVG
            IH-->>CS: SVG data
        end
    end
    
    CS-->>SW: Images with metadata
```

### Dynamic Content Handling

```mermaid
sequenceDiagram
    participant User as User
    participant CS as Content Script
    participant MO as Mutation Observer
    participant DOM as DOM
    participant SW as Service Worker
    
    User->>CS: Enable page monitoring
    CS->>MO: Create observer
    MO->>DOM: Observe mutations
    
    loop Page changes
        DOM-->>MO: Mutation event
        MO->>CS: Content changed
        
        alt Significant change
            CS->>CS: Debounce (500ms)
            CS->>DOM: Re-extract content
            CS->>SW: Update context
            SW->>SW: Update Gemini context
        else Minor change
            CS->>CS: Ignore
        end
    end
    
    User->>CS: Disable monitoring
    CS->>MO: Disconnect observer
```

### Error Handling

```mermaid
sequenceDiagram
    participant SW as Service Worker
    participant CS as Content Script
    participant EH as Error Handler
    
    SW->>CS: Extract content
    
    alt No content script
        CS--xSW: No response
        SW->>EH: Handle missing script
        EH->>Chrome: Inject content script
        Chrome-->>EH: Injection result
        alt Success
            EH->>SW: Retry extraction
        else Failed
            EH-->>User: "Refresh page"
        end
    else Script blocked by CSP
        CS-->>SW: CSP error
        SW->>EH: Handle CSP
        EH-->>User: "Page security prevents analysis"
    else Timeout
        CS--xSW: Timeout (30s)
        SW->>EH: Handle timeout
        EH-->>User: "Page too complex"
    end
```

## UX/UI Considerations
- Clear indication of what content is being analyzed
- Privacy-focused messaging about data handling
- Visual feedback during extraction
- Option to exclude certain page elements
- Summary of extracted content before sending

## Acceptance Criteria
- [ ] Extract page text content accurately
- [ ] Handle images with proper CORS compliance
- [ ] Extract relevant metadata (title, description, etc.)
- [ ] Filter out sensitive information (passwords, credit cards)
- [ ] Respect page security boundaries (CSP, X-Frame-Options)
- [ ] Handle dynamic content with mutation observers
- [ ] Provide clear feedback about extraction process
- [ ] Support common content types (HTML, text, images)
- [ ] Graceful handling of extraction failures

## Dependencies
- PBI-1: Extension foundation
- PBI-3: Conversation interface for displaying results
- Content script permissions in manifest
- ActiveTab permission for current page access

## Open Questions
- Should we support PDF content extraction?
- How should we handle very large pages?
- Should extracted content be cached?
- Do we need user controls for extraction depth?

## Related Tasks
Tasks will be created once this PBI is approved and moved to "Agreed" status.

[View in Backlog](../backlog.md#user-content-4)