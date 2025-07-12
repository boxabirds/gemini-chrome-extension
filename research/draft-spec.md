
## Technical Architecture

### Core Components (Revised)

**1\. Extension Structure**

- **Background Script**: Handles Gemini API communication and orchestrates automation  
- **Content Script**: Injected into web pages for DOM manipulation and interaction  
- **Popup/Options UI**: Configuration interface and conversation management  
- **Service Worker**: Manages conversation state and tool execution

**2\. Simplified Authentication System** The extension will use Chrome's `chrome.identity.getAuthToken()` API directly, eliminating the need for OAuth client configuration entirely. [1](#7-0)

## Updated Implementation Plan

### Phase 1: Foundation Setup (Revised)

**Step 1: Extension Manifest and Structure (Updated)**

{

  "manifest\_version": 3,

  "name": "Gemini Web Automation",

  "version": "1.0.0",

  "description": "Conversational web page automation using Gemini AI",

  

  "permissions": \[

    "identity",

    "storage", 

    "activeTab",

    "scripting"

  \],

  

  "host\_permissions": \[

    "https://codeassist.googleapis.com/\*",

    "https://generativelanguage.googleapis.com/\*"

  \],

  

  "background": {

    "service\_worker": "background.js"

  },

  

  "content\_scripts": \[

    {

      "matches": \["\<all\_urls\>"\],

      "js": \["content.js"\],

      "run\_at": "document\_idle"

    }

  \],

  

  "action": {

    "default\_popup": "popup.html",

    "default\_title": "Gemini Web Automation"

  },

  

  "options\_page": "options.html"

}

**Key Changes:**

- Removed `oauth2` section entirely  
- No client ID configuration needed  
- Simplified permissions structure

**Step 2: Authentication Integration (Simplified)**

Instead of the complex OAuth flow the CLI uses [2](#7-1) , implement a simple Chrome identity wrapper:

// lib/auth-manager.js

async function getGoogleAuthToken() {

  return new Promise((resolve, reject) \=\> {

    chrome.identity.getAuthToken({

      scopes: \[

        'https://www.googleapis.com/auth/cloud-platform',

        'https://www.googleapis.com/auth/userinfo.email', 

        'https://www.googleapis.com/auth/userinfo.profile'

      \]

    }, (token) \=\> {

      if (chrome.runtime.lastError) {

        reject(chrome.runtime.lastError);

      } else {

        resolve(token);

      }

    });

  });

}

**Step 3: Core API Client Adaptation (Streamlined)**

Adapt the `CodeAssistServer` class [3](#7-2)  to work with Chrome's token instead of OAuth2Client:

- Replace OAuth2Client dependency with direct token usage  
- Eliminate credential caching since Chrome handles token management  
- Remove user setup complexity [4](#7-3)

### Updated Phase 2-5 (No Changes)

The remaining phases (Tool System Development, Conversation Interface, Advanced Features, Testing) remain the same as the original plan since they don't depend on the authentication mechanism.

## Key Simplifications

**Authentication Flow**:

- **Before**: Complex OAuth flow with local servers, credential caching, and token refresh  
- **After**: Single `chrome.identity.getAuthToken()` call leveraging browser's existing auth

**Configuration**:

- **Before**: OAuth client ID, client secret, redirect URIs  
- **After**: Just permission declarations in manifest

**Error Handling**:

- **Before**: Handle OAuth errors, token expiry, refresh logic  
- **After**: Chrome handles all token lifecycle management

## Notes

This simplification removes significant complexity from the authentication system while maintaining full compatibility with the Code Assist API. The extension can focus on web automation tools and conversation management rather than authentication infrastructure. The same scopes used by the CLI [1](#7-0)  are still required, but Chrome's identity API handles all the OAuth complexity behind the scenes.

Wiki pages you might want to explore:

- [Authentication (google-gemini/gemini-cli)](http:///wiki/google-gemini/gemini-cli#2.2)  
- [AI Integration (google-gemini/gemini-cli)](http:///wiki/google-gemini/gemini-cli#4.2)

You're asking me to flesh out **Phase 2: Tool System Development** for the Chrome extension version of Gemini CLI. This phase focuses on creating web automation tools to replace the CLI's file system tools.

## Phase 2: Tool System Development

### Step 4: Web Automation Tool Registry

Create an extension-specific tool registry that replaces the CLI's file system tools with DOM manipulation capabilities. The CLI's tool system is built around [1](#10-0)  which manages built-in tools like `ReadFileTool` and `ShellTool`.

For the Chrome extension, create these web automation tools:

// tools/dom-tools.js

class DOMQueryTool extends BaseTool {

  constructor() {

    super(

      'dom\_query',

      'DOM Query',

      'Query DOM elements using CSS selectors',

      {

        properties: {

          selector: { type: 'string', description: 'CSS selector to query' },

          action: { type: 'string', enum: \['find', 'count', 'text', 'attributes'\] }

        },

        required: \['selector', 'action'\]

      }

    );

  }

  async execute(params) {

    // Send message to content script to execute DOM query

    const result \= await chrome.tabs.sendMessage(tabId, {

      type: 'DOM\_QUERY',

      selector: params.selector,

      action: params.action

    });

    

    return {

      llmContent: \`Found ${result.count} elements matching "${params.selector}"\`,

      returnDisplay: result.data

    };

  }

}

class ClickTool extends BaseTool {

  constructor() {

    super(

      'click\_element',

      'Click Element', 

      'Click on DOM elements',

      {

        properties: {

          selector: { type: 'string', description: 'CSS selector for element to click' },

          waitFor: { type: 'number', description: 'Milliseconds to wait after click' }

        },

        required: \['selector'\]

      }

    );

  }

  shouldConfirmExecute(params) {

    return {

      type: 'warning',

      title: 'Confirm Click Action',

      message: \`Click element: ${params.selector}\`,

      onConfirm: () \=\> {}

    };

  }

}

### Step 5: Content Script Communication

Implement secure message passing between the background script and content scripts, similar to how the CLI handles tool execution through [2](#10-1) .

// content.js \- Content script for DOM manipulation

chrome.runtime.onMessage.addListener((message, sender, sendResponse) \=\> {

  switch (message.type) {

    case 'DOM\_QUERY':

      const elements \= document.querySelectorAll(message.selector);

      sendResponse({

        count: elements.length,

        data: Array.from(elements).map(el \=\> ({

          tagName: el.tagName,

          textContent: el.textContent?.substring(0, 100),

          attributes: Object.fromEntries(

            Array.from(el.attributes).map(attr \=\> \[attr.name, attr.value\])

          )

        }))

      });

      break;

      

    case 'CLICK\_ELEMENT':

      try {

        const element \= document.querySelector(message.selector);

        if (element) {

          element.click();

          sendResponse({ success: true });

        } else {

          sendResponse({ success: false, error: 'Element not found' });

        }

      } catch (error) {

        sendResponse({ success: false, error: error.message });

      }

      break;

  }

  return true; // Keep message channel open for async response

});

### Step 6: Site-Specific Automation

Develop YouTube-specific tools for playlist management, leveraging the CLI's pattern of specialized tools like [3](#10-2)  for web operations.

// tools/youtube-tools.js

class YouTubePlaylistTool extends BaseTool {

  constructor() {

    super(

      'youtube\_playlist\_action',

      'YouTube Playlist Manager',

      'Manage YouTube playlist items with rate limiting',

      {

        properties: {

          action: { 

            type: 'string', 

            enum: \['list\_items', 'remove\_items', 'get\_count'\] 

          },

          itemSelector: { 

            type: 'string', 

            description: 'CSS selector for playlist items' 

          },

          batchSize: { 

            type: 'number', 

            default: 10,

            description: 'Number of items to process per batch' 

          },

          delayMs: { 

            type: 'number', 

            default: 1000,

            description: 'Delay between batches to avoid rate limiting' 

          }

        },

        required: \['action'\]

      }

    );

  }

  async execute(params) {

    switch (params.action) {

      case 'list\_items':

        return await this.listPlaylistItems(params);

      case 'remove\_items':

        return await this.removePlaylistItems(params);

      case 'get\_count':

        return await this.getPlaylistCount(params);

    }

  }

  async removePlaylistItems(params) {

    const { batchSize \= 10, delayMs \= 1000 } \= params;

    let totalRemoved \= 0;

    let hasMore \= true;

    while (hasMore && totalRemoved \< 1000\) {

      const result \= await chrome.tabs.sendMessage(tabId, {

        type: 'YOUTUBE\_REMOVE\_BATCH',

        batchSize: Math.min(batchSize, 1000 \- totalRemoved)

      });

      totalRemoved \+= result.removed;

      hasMore \= result.hasMore;

      if (hasMore) {

        await new Promise(resolve \=\> setTimeout(resolve, delayMs));

      }

    }

    return {

      llmContent: \`Removed ${totalRemoved} items from playlist\`,

      returnDisplay: \`Successfully removed ${totalRemoved} playlist items\`

    };

  }

}

The content script implementation for YouTube automation:

// YouTube-specific content script handlers

function handleYouTubeRemoveBatch(message) {

  const playlistItems \= document.querySelectorAll('\[data-testid="playlist-item"\]');

  const batchSize \= Math.min(message.batchSize, playlistItems.length);

  let removed \= 0;

  for (let i \= 0; i \< batchSize; i++) {

    const item \= playlistItems\[i\];

    const removeButton \= item.querySelector('\[aria-label\*="Remove"\]');

    

    if (removeButton) {

      removeButton.click();

      removed++;

      

      // Wait for removal animation

      setTimeout(() \=\> {}, 500);

    }

  }

  return {

    removed,

    hasMore: document.querySelectorAll('\[data-testid="playlist-item"\]').length \> 0

  };

}

## Key Implementation Notes

**Tool Registry Integration**: The extension's tool registry follows the same pattern as the CLI's [4](#10-3)  but registers web automation tools instead of file system tools.

**Safety and Confirmation**: Like the CLI's approach with [5](#10-4) , destructive operations require user confirmation before execution.

**Rate Limiting**: YouTube automation includes built-in rate limiting to prevent the extension from being blocked, similar to how the CLI handles [6](#10-5)  with confirmation dialogs.

**Error Handling**: Comprehensive error handling ensures graceful degradation when page structures change or elements aren't found, following the CLI's robust error handling patterns.

This tool system provides the foundation for conversational web automation while maintaining the safety and extensibility principles of the original Gemini CLI architecture.

Wiki pages you might want to explore:

- [Overview (google-gemini/gemini-cli)](http:///wiki/google-gemini/gemini-cli#1)

You're asking me to flesh out **Phase 3: Conversation Interface** for the Chrome extension version of Gemini CLI. This phase focuses on adapting the CLI's conversation management system for the browser extension environment.

## Phase 3: Conversation Interface

### Step 7: Chat Management System

Adapt the CLI's conversation management for extension popup, leveraging the existing streaming architecture from [1](#11-0) .

// popup/chat-manager.js

class ExtensionChatManager {

  constructor() {

    this.history \= \[\];

    this.streamingState \= 'idle';

    this.abortController \= null;

    this.geminiClient \= null;

  }

  async initialize() {

    this.geminiClient \= await this.createGeminiClient();

  }

  async sendMessage(message) {

    this.streamingState \= 'responding';

    this.abortController \= new AbortController();

    

    try {

      const response \= await this.geminiClient.sendMessageStream(

        message,

        this.abortController.signal

      );

      

      return this.processStreamingResponse(response);

    } catch (error) {

      this.streamingState \= 'error';

      throw error;

    }

  }

  async processStreamingResponse(responseStream) {

    const chunks \= \[\];

    

    for await (const chunk of responseStream) {

      if (this.abortController.signal.aborted) break;

      

      chunks.push(chunk);

      this.onStreamChunk(chunk);

    }

    

    this.streamingState \= 'idle';

    return chunks;

  }

}

### Step 8: System Prompts and Context

Develop web automation-focused system prompts, adapting the CLI's comprehensive prompt system from [2](#11-1) .

// lib/extension-prompts.js

function getWebAutomationSystemPrompt() {

  return \`

You are an interactive Chrome extension agent specializing in web page automation tasks. Your primary goal is to help users safely and efficiently automate web interactions, adhering strictly to the following instructions and utilizing your available web automation tools.

\# Core Mandates

\- \*\*Page Context:\*\* Always analyze the current page structure and content before taking actions. Use DOM query tools to understand the page layout.

\- \*\*Rate Limiting:\*\* Implement appropriate delays between actions to avoid being blocked by websites, especially for bulk operations.

\- \*\*User Safety:\*\* Always confirm destructive operations (like deleting items) before execution.

\- \*\*Site Compatibility:\*\* Adapt to different website structures and handle dynamic content loading gracefully.

\- \*\*Error Handling:\*\* Provide clear feedback when elements cannot be found or actions fail.

\# Primary Workflows

\#\# Web Automation Tasks

When requested to perform web automation tasks, follow this sequence:

1\. \*\*Understand:\*\* Analyze the current page using DOM query tools to understand structure and available elements.

2\. \*\*Plan:\*\* Build a coherent plan for the automation task, considering rate limiting and error handling.

3\. \*\*Implement:\*\* Use available web automation tools (DOM query, click, form fill, etc.) to execute the plan.

4\. \*\*Verify:\*\* Check that actions completed successfully and provide feedback to the user.

\# Available Tools

\- \*\*dom\_query:\*\* Query and analyze page elements using CSS selectors

\- \*\*click\_element:\*\* Click on page elements with confirmation for destructive actions

\- \*\*form\_fill:\*\* Fill out forms and input fields

\- \*\*scroll\_page:\*\* Handle page scrolling and pagination

\- \*\*wait\_for\_element:\*\* Wait for dynamic content to load

\- \*\*youtube\_playlist\_action:\*\* Specialized YouTube playlist management with rate limiting

\# Operational Guidelines

\#\# Tone and Style (Extension Interaction)

\- \*\*Concise & Direct:\*\* Adopt a helpful, direct tone suitable for browser automation.

\- \*\*Clear Feedback:\*\* Always provide clear status updates during long-running operations.

\- \*\*Safety First:\*\* Confirm potentially destructive operations before execution.

\#\# Security and Safety Rules

\- \*\*Explain Actions:\*\* Before executing actions that modify page content, provide a brief explanation.

\- \*\*Respect Rate Limits:\*\* Implement appropriate delays to avoid being blocked by websites.

\- \*\*Handle Errors Gracefully:\*\* Provide helpful error messages when automation fails.

\`;

}

### Step 9: User Interface

Build popup interface for conversation and configuration, adapting the CLI's React-based UI architecture from [3](#11-2) .

\<\!-- popup.html \--\>

\<\!DOCTYPE html\>

\<html\>

\<head\>

  \<meta charset="utf-8"\>

  \<style\>

    body {

      width: 400px;

      height: 600px;

      margin: 0;

      font-family: \-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;

    }

    

    .chat-container {

      display: flex;

      flex-direction: column;

      height: 100%;

    }

    

    .chat-history {

      flex: 1;

      overflow-y: auto;

      padding: 16px;

      border-bottom: 1px solid \#e0e0e0;

    }

    

    .message {

      margin-bottom: 12px;

      padding: 8px 12px;

      border-radius: 8px;

    }

    

    .message.user {

      background-color: \#e3f2fd;

      margin-left: 20px;

    }

    

    .message.assistant {

      background-color: \#f5f5f5;

      margin-right: 20px;

    }

    

    .message.tool {

      background-color: \#fff3e0;

      font-family: monospace;

      font-size: 12px;

    }

    

    .input-container {

      padding: 16px;

      border-top: 1px solid \#e0e0e0;

    }

    

    .input-field {

      width: 100%;

      padding: 8px;

      border: 1px solid \#ccc;

      border-radius: 4px;

      resize: vertical;

      min-height: 60px;

    }

    

    .send-button {

      margin-top: 8px;

      padding: 8px 16px;

      background-color: \#1976d2;

      color: white;

      border: none;

      border-radius: 4px;

      cursor: pointer;

    }

    

    .streaming-indicator {

      padding: 8px;

      text-align: center;

      font-style: italic;

      color: \#666;

    }

  \</style\>

\</head\>

\<body\>

  \<div class="chat-container"\>

    \<div class="chat-history" id="chatHistory"\>

      \<\!-- Messages will be dynamically added here \--\>

    \</div\>

    

    \<div class="streaming-indicator" id="streamingIndicator" style="display: none;"\>

      Gemini is thinking...

    \</div\>

    

    \<div class="input-container"\>

      \<textarea 

        id="messageInput" 

        class="input-field" 

        placeholder="Ask Gemini to automate something on this page..."

        rows="3"\>

      \</textarea\>

      \<button id="sendButton" class="send-button"\>Send\</button\>

    \</div\>

  \</div\>

  

  \<script src="popup.js"\>\</script\>

\</body\>

\</html\>

// popup.js

class ExtensionPopupUI {

  constructor() {

    this.chatManager \= new ExtensionChatManager();

    this.chatHistory \= document.getElementById('chatHistory');

    this.messageInput \= document.getElementById('messageInput');

    this.sendButton \= document.getElementById('sendButton');

    this.streamingIndicator \= document.getElementById('streamingIndicator');

    

    this.setupEventListeners();

    this.loadChatHistory();

  }

  setupEventListeners() {

    this.sendButton.addEventListener('click', () \=\> this.sendMessage());

    this.messageInput.addEventListener('keydown', (e) \=\> {

      if (e.key \=== 'Enter' && (e.ctrlKey || e.metaKey)) {

        this.sendMessage();

      }

    });

  }

  async sendMessage() {

    const message \= this.messageInput.value.trim();

    if (\!message) return;

    this.addMessageToHistory('user', message);

    this.messageInput.value \= '';

    this.setStreamingState(true);

    try {

      const response \= await this.chatManager.sendMessage(message);

      this.addMessageToHistory('assistant', response.text);

      

      if (response.toolCalls) {

        response.toolCalls.forEach(toolCall \=\> {

          this.addToolCallToHistory(toolCall);

        });

      }

    } catch (error) {

      this.addMessageToHistory('error', \`Error: ${error.message}\`);

    } finally {

      this.setStreamingState(false);

    }

  }

  addMessageToHistory(type, content) {

    const messageDiv \= document.createElement('div');

    messageDiv.className \= \`message ${type}\`;

    messageDiv.textContent \= content;

    this.chatHistory.appendChild(messageDiv);

    this.chatHistory.scrollTop \= this.chatHistory.scrollHeight;

    

    this.saveChatHistory();

  }

  addToolCallToHistory(toolCall) {

    const toolDiv \= document.createElement('div');

    toolDiv.className \= 'message tool';

    toolDiv.innerHTML \= \`

      \<strong\>${toolCall.name}\</strong\>\<br\>

      \<pre\>${JSON.stringify(toolCall.args, null, 2)}\</pre\>

      \<div\>Result: ${toolCall.result}\</div\>

    \`;

    this.chatHistory.appendChild(toolDiv);

    this.chatHistory.scrollTop \= this.chatHistory.scrollHeight;

  }

  setStreamingState(isStreaming) {

    this.streamingIndicator.style.display \= isStreaming ? 'block' : 'none';

    this.sendButton.disabled \= isStreaming;

    this.messageInput.disabled \= isStreaming;

  }

  async loadChatHistory() {

    const stored \= await chrome.storage.local.get(\['chatHistory'\]);

    if (stored.chatHistory) {

      stored.chatHistory.forEach(msg \=\> {

        this.addMessageToHistory(msg.type, msg.content);

      });

    }

  }

  async saveChatHistory() {

    const messages \= Array.from(this.chatHistory.children).map(div \=\> ({

      type: div.className.replace('message ', ''),

      content: div.textContent

    }));

    

    await chrome.storage.local.set({ chatHistory: messages });

  }

}

// Initialize the popup UI when the DOM is loaded

document.addEventListener('DOMContentLoaded', () \=\> {

  new ExtensionPopupUI();

});

## Key Implementation Notes

**Streaming Integration**: The popup UI adapts the CLI's streaming response handling from [4](#11-3)  to provide real-time feedback during automation tasks.

**History Management**: Persistent conversation history using Chrome's storage API, similar to how the CLI manages conversation state in [5](#11-4) .

**Tool Execution Display**: Visual feedback for tool execution results, adapting the CLI's tool display patterns from the conversation interface.

**Safety Confirmations**: User confirmation dialogs for destructive operations, following the CLI's safety-first approach from [6](#11-5) .

This conversation interface provides a familiar chat-based interaction model while being optimized for web automation tasks within the Chrome extension environment.

Wiki pages you might want to explore:

- [Overview (google-gemini/gemini-cli)](http:///wiki/google-gemini/gemini-cli#1)  
- [Architecture (google-gemini/gemini-cli)](http:///wiki/google-gemini/gemini-cli#1.1)
