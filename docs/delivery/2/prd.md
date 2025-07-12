# PBI-2: Google Account Authentication

## Overview
Implement Google OAuth2 authentication to enable users to access Gemini AI capabilities through their existing Google account, leveraging Chrome's identity API for seamless integration.

## Problem Statement
Users need to authenticate with Google to access the generous free tier of Gemini API. The authentication should be seamless, secure, and leverage the user's existing Chrome session without requiring a separate login.

## User Stories
As a user, I want to authenticate with my Google account so that I can access Gemini AI capabilities.

## Technical Approach

### Authentication Architecture

```mermaid
graph LR
    subgraph Chrome Extension
        UI[Popup UI]
        SW[Service Worker]
        AM[Auth Manager]
    end
    
    subgraph Chrome APIs
        IA[Identity API]
        SA[Storage API]
    end
    
    subgraph External
        GO[Google OAuth]
        GA[Gemini API]
    end
    
    UI --> SW
    SW --> AM
    AM --> IA
    AM --> SA
    IA --> GO
    AM --> GA
```

### Core Authentication Flow

```mermaid
sequenceDiagram
    actor User
    participant Popup as Popup UI
    participant SW as Service Worker
    participant AM as Auth Manager
    participant Chrome as Chrome Identity
    participant Google as Google OAuth
    participant Storage as Chrome Storage
    
    User->>Popup: Open extension
    Popup->>SW: Check auth status
    SW->>AM: isAuthenticated()
    AM->>Storage: Get cached token
    
    alt No token or expired
        Storage-->>AM: No valid token
        AM-->>SW: Not authenticated
        SW-->>Popup: Show login button
        User->>Popup: Click "Sign in with Google"
        Popup->>SW: Request authentication
        SW->>AM: authenticate(interactive: true)
        AM->>Chrome: getAuthToken({interactive: true})
        Chrome->>Google: OAuth flow
        Google-->>User: Consent screen
        User->>Google: Grant permissions
        Google-->>Chrome: Auth code
        Chrome-->>AM: Access token
        AM->>Storage: Cache token
        AM-->>SW: Auth success
        SW-->>Popup: Update UI
    else Token valid
        Storage-->>AM: Valid token
        AM-->>SW: Authenticated
        SW-->>Popup: Show chat interface
    end
```

### Token Refresh Flow

```mermaid
sequenceDiagram
    participant SW as Service Worker
    participant AM as Auth Manager
    participant Chrome as Chrome Identity
    participant API as Gemini API
    
    SW->>API: Make API request
    API-->>SW: 401 Unauthorized
    SW->>AM: Token expired
    AM->>Chrome: getAuthToken({interactive: false})
    
    alt Refresh successful
        Chrome-->>AM: New token
        AM->>AM: Update cached token
        AM-->>SW: Token refreshed
        SW->>API: Retry request
        API-->>SW: Success
    else Refresh failed
        Chrome-->>AM: Error
        AM->>Chrome: removeCachedAuthToken()
        AM->>Chrome: getAuthToken({interactive: true})
        Note over Chrome: User re-authentication required
    end
```

### Error Handling

```mermaid
sequenceDiagram
    participant User
    participant Popup as Popup UI
    participant AM as Auth Manager
    participant Chrome as Chrome APIs
    
    User->>Popup: Sign in
    Popup->>AM: authenticate()
    
    alt Network error
        AM--xChrome: Network timeout
        AM-->>Popup: Show "Check connection"
    else User cancelled
        Chrome-->>AM: User cancelled
        AM-->>Popup: Show "Sign in to continue"
    else Invalid scopes
        Chrome-->>AM: Scope error
        AM-->>Popup: Show "Update extension"
    else Chrome not signed in
        Chrome-->>AM: No Chrome user
        AM-->>Popup: Show "Sign in to Chrome first"
    end
    
    Popup-->>User: Display error message
```

### Logout Flow

```mermaid
sequenceDiagram
    actor User
    participant Popup as Popup UI
    participant SW as Service Worker
    participant AM as Auth Manager
    participant Chrome as Chrome APIs
    participant Storage as Chrome Storage
    
    User->>Popup: Click logout
    Popup->>SW: Request logout
    SW->>AM: logout()
    
    par Clear tokens
        AM->>Chrome: removeCachedAuthToken()
        AM->>Storage: Remove stored token
    and Clear user data
        AM->>Storage: Remove user info
        AM->>Storage: Clear conversation history
    end
    
    AM-->>SW: Logout complete
    SW-->>Popup: Update UI
    Popup-->>User: Show login screen
```

## UX/UI Considerations
- One-click authentication using existing Chrome session
- Clear visual feedback during authentication
- Persistent login state indicator
- Graceful error messages
- Easy logout option

## Acceptance Criteria
- [ ] One-click Google authentication using Chrome identity API
- [ ] Tokens stored securely in chrome.storage.local
- [ ] Automatic token refresh before expiration
- [ ] Clear, user-friendly error messages for auth failures
- [ ] Logout functionality clears all auth data
- [ ] Auth state persists across browser sessions

## Dependencies
- PBI-1: Chrome Extension Foundation must be completed
- Chrome identity API permissions in manifest
- Google OAuth2 client configuration

## Open Questions
- Should we implement a token refresh buffer (e.g., refresh 5 minutes before expiry)?
- How should we handle users with multiple Google accounts?
- Should auth state sync across devices via chrome.storage.sync?

## Related Tasks
Tasks will be created once this PBI is approved and moved to "Agreed" status.

[View in Backlog](../backlog.md#user-content-2)