<div align="center">

<h1>🚀 NetSuite MCP AI Integration<br>Architecture & Key Components Guide</h1>

<p><strong>Understanding how to connect your AI assistant to NetSuite using Model Context Protocol (MCP)</strong></p>

<span style="background: #667eea; color: white; padding: 8px 16px; border-radius: 20px; font-size: 0.9rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.5px;">BYOAI - Bring Your Own AI</span>

<sub>Author: Caleb Moore, Certified SuiteCloud Dev, Aug 15 2025</sub>

</div>

---

<div align="center">

> ⚠️ **Rapidly Evolving Technology:** This document reflects the current state of NetSuite MCP and AI integration as of the date above. Given the fast-paced nature of AI technology and NetSuite updates, portions of this guide may become outdated quickly. Always verify current NetSuite documentation and AI service capabilities.

</div>

---

## 🎯 What This Guide Provides

This is a **conceptual guide** that explains the key components and architecture needed to connect your AI assistant to NetSuite MCP. It's not a copy/paste tutorial, but rather helps you understand what you need to build and how the pieces fit together.

## 🔧 Technology Implementation Note

**This concept was proven using:** Google Gemini AI + React-based chatbot interface

**However, there are many ways to leverage NetSuite's AI Connector:** You could use Claude, GPT-4, Azure OpenAI, or any other AI service. The architecture principles remain the same regardless of your AI choice.

**Current State:** While this integration concept has been proven to work, it isn't perfect and would still benefit from further refinement. Expect some bugs and edge cases that need addressing.

## 📋 Table of Contents

- [Overview & Architecture](#overview)
- [Prerequisites](#prerequisites)
- [Part 1: NetSuite MCP Setup](#part-1-netsuite-mcp-setup)
- [Part 2: Core Components](#part-2-core-components)
- [Part 3: AI Integration](#part-3-ai-integration)
- [Part 4: Frontend React Application](#part-4-frontend-react-application)
- [Part 5: Development & Testing](#part-5-development--testing)
- [Part 6: Getting Started](#part-6-getting-started)
- [Part 7: Troubleshooting](#part-7-troubleshooting)
- [Part 8: Production Considerations](#part-8-production-considerations)

---

## Overview

This guide explains the architecture and key components needed to connect your AI assistant to NetSuite using Model Context Protocol (MCP). The goal is to create a system where users can ask natural language questions and get real-time data from NetSuite without needing to know SuiteQL or navigate complex interfaces.

### 🎯 What You'll Understand

How to architect an AI-powered system that bridges natural language queries with NetSuite's MCP tools, enabling conversational data access to your business data.

### 🔄 System Flow Overview

```
👤 User Question → 🧠 AI Analysis → 🔧 MCP Execution → 💬 AI Interpretation
```

**Example Flow:**

1. **User Question:** "How many customers do we have?"
2. **AI Analysis:** Gemini determines: use runCustomSuiteQL with COUNT query
3. **MCP Execution:** NetSuite MCP tool runs: `SELECT COUNT(*) FROM customer`
4. **AI Interpretation:** Gemini converts raw data to: "You have 4,214 customers"

### 🚀 Key Features

| Feature                           | Description                                                                           |
| --------------------------------- | ------------------------------------------------------------------------------------- |
| 🧠 **AI Query Analysis**          | Your AI analyzes user questions and selects the right MCP tool with proper parameters |
| 🔧 **MCP Tool Execution**         | Executes the selected tool against NetSuite and retrieves raw data                    |
| 💬 **AI Response Interpretation** | Your AI interprets the NetSuite results and provides conversational answers           |
| 🛡️ **Smart Fallbacks**            | Intelligent parsing when AI interpretation fails                                      |

---

## Prerequisites

<div>

> **Note:** The examples in this guide use Google Gemini AI + React + Node.js, but this architecture is flexible and can be adapted to your preferred AI service, frontend framework, or backend technology.

</div>

### Before You Begin

- **NetSuite Account:** Administrative access required
- **AI Service:** API key for your chosen AI provider (Gemini, Claude, GPT-4, Azure OpenAI, etc.)
- **Development Environment:** Your preferred frontend framework (React, Vue, Angular, etc.) and backend technology (Node.js, Python, .NET, etc.)
- **Development Skills:** Basic knowledge of your chosen technologies and REST API integration

---

## Part 1: NetSuite MCP Setup

Before you can connect your AI to NetSuite, you need to set up the MCP (Model Context Protocol) infrastructure on the NetSuite side. This involves installing the MCP Tools SuiteApp and configuring the necessary permissions.

### 1.1 Install MCP Tools SuiteApp

The MCP Tools SuiteApp provides the interface between your AI and NetSuite data. You'll need to:

1. Navigate to **Customization → SuiteCloud Development → SuiteApp Marketplace**
2. Search for and install **MCP Tools**
3. Wait for the installation to complete

### 1.2 Create Custom Role

> ⚠️ **Important**
>
> Administrators are not allowed to work with MCP - you must create a custom role with specific permissions.

You'll need to create a custom role with these key permissions:

- **MCP Server Connection** - Allows connection to the MCP server
- **OAuth 2.0 Authorized Applications Management** - Manages OAuth integrations
- **Log in using OAuth 2.0 Access Tokens** - Enables token-based authentication

This role will be assigned to the user account that your AI integration will use to connect to NetSuite.

### 1.3 Assign Role to User

Assign the custom MCP role to the user account that will be used for your AI integration. This user will authenticate with NetSuite on behalf of your AI system.

### 1.4 Create Integration Record

Create an OAuth 2.0 integration record that will allow your AI system to authenticate with NetSuite. Key configuration points:

- **Application Type:** Public Client
- **State:** Enabled
- **Authorization Code Grant:** Enabled for secure authentication
- **Redirect URI:** Your application's callback URL
- **Scopes:** Leave all unchecked - the `mcp` scope is handled in the OAuth authorization call, not in the NetSuite integration record

> ⚠️ **Important: Disable These Features**
>
> Make sure these features are **disabled** in your integration record:
>
> - **Token-based Authentication**
> - **TBA: issuetoken Endpoint**
> - **TBA: Authorization Flow**

This integration record will provide you with a Client ID that your AI system will use during the OAuth flow.

---

## Part 2: Core Components

Your AI integration system will need several core components to function properly. Here's what you'll need to build or integrate:

### 2.1 Backend Server

You'll need a backend server that handles:

- **OAuth Flow:** Manages authentication with NetSuite
- **MCP Proxy:** Forwards requests to NetSuite's MCP server
- **Token Management:** Handles access token refresh and storage
- **Error Handling:** Manages API failures and retries

### 2.2 Frontend Interface

Your user interface should provide:

- **Configuration Panel:** For NetSuite credentials and settings
- **Chat Interface:** Where users ask questions
- **Status Indicators:** Show connection and tool availability
- **Response Display:** Show AI-generated answers

### 2.3 AI Integration

The core intelligence layer includes:

- **Query Analysis:** AI that understands user questions
- **Tool Selection:** Maps questions to appropriate MCP tools
- **Response Generation:** Converts raw data to conversational answers
- **Fallback Logic:** Handles cases where AI interpretation fails

---

## Part 3: AI Integration

This is where the magic happens! Your AI needs to understand user questions, select the right MCP tools, and interpret the results conversationally. Here's how the AI integration works:

### 3.1 AI Query Analysis

Your AI (Gemini in our case) needs to:

- **Parse Natural Language:** Understand what the user is asking
- **Map to MCP Tools:** Determine which NetSuite MCP tool to use
- **Generate Parameters:** Create the right parameters for the tool
- **Assess Confidence:** Determine if it's confident enough to proceed

> 💡 **Example AI Analysis**
>
> **User:** "How many customers do we have?"
>
> **AI Analysis:** Use `runCustomSuiteQL` with `SELECT COUNT(*) FROM customer`

### 3.2 MCP Tool Execution

Once your AI determines the right tool and parameters, you need to:

- **Authenticate:** Use OAuth tokens to access NetSuite's MCP server
- **Format Requests:** Structure JSON-RPC calls to the MCP endpoint
- **Handle Responses:** Process the raw data returned from NetSuite
- **Manage Errors:** Handle authentication failures and API errors

> 🔧 **MCP Request Format**
>
> Your requests to NetSuite's MCP server should follow the JSON-RPC 2.0 specification:
>
> ```json
> {
>   "jsonrpc": "2.0",
>   "id": "unique_request_id",
>   "method": "tools/call",
>   "params": {
>     "name": "toolName",
>     "arguments": { ... }
>   }
> }
> ```

### 3.3 AI Response Interpretation

This is the key component that makes your AI conversational! Instead of returning raw JSON data, your AI needs to:

- **Parse NetSuite Responses:** Extract meaningful data from the MCP tool results
- **Generate Natural Language:** Convert technical data into conversational answers
- **Handle Edge Cases:** Deal with errors, missing data, or unexpected responses
- **Provide Context:** Give users relevant information based on their question

> 💬 **Example Response Transformation**
>
> **Raw NetSuite Response:** `{"expr1": 4214}`
>
> **AI Interpretation:** "You have 4,214 customers in your system."

### 3.4 MCP Tools Discovery Endpoint

```javascript
app.post("/netsuite/mcp/tools", async (req, res) => {
  try {
    const accountId = req.headers["x-netsuite-account"];
    const authorization = req.headers.authorization;

    if (!authorization) {
      return res.status(401).json({
        error: "Missing Authorization header",
      });
    }

    const mcpUrl = `https://${accountId}.suitetalk.api.netsuite.com/services/mcp/v1/all`;

    const requestBody = {
      jsonrpc: "2.0",
      id: "2",
      method: "tools/list",
      params: {},
    };

    const response = await fetch(mcpUrl, {
      method: "POST",
      headers: {
        Authorization: authorization,
        "Content-Type": "application/json",
      },
      body: JSON.stringify(requestBody),
    });

    if (response.ok) {
      const data = await response.json();
      res.json({
        success: true,
        message: "MCP tools fetched successfully",
        status: response.status,
        data: data,
        url: mcpUrl,
      });
    } else {
      const errorText = await response.text();
      res.status(response.status).json({
        success: false,
        message: "MCP tools fetch failed",
        status: response.status,
        error: errorText,
        url: mcpUrl,
      });
    }
  } catch (error) {
    console.error("MCP tools endpoint error:", error);
    res.status(500).json({
      error: "MCP tools fetch failed",
      details: error.message,
    });
  }
});
```

### 3.5 MCP Tool Execution Endpoint

```javascript
app.post("/netsuite/mcp/execute", async (req, res) => {
  try {
    const { jsonrpc, id, method, params } = req.body;
    const accountId = req.headers["x-netsuite-account"];
    const authorization = req.headers.authorization;

    if (!authorization) {
      return res.status(401).json({
        error: "Missing Authorization header",
      });
    }

    if (!jsonrpc || !id || !method) {
      return res.status(400).json({
        error: "Missing required parameters: jsonrpc, id, method",
      });
    }

    const mcpUrl = `https://${accountId}.suitetalk.api.netsuite.com/services/mcp/v1/all`;

    const requestBody = {
      jsonrpc,
      id,
      method,
      params: params || {},
    };

    const response = await fetch(mcpUrl, {
      method: "POST",
      headers: {
        Authorization: authorization,
        "Content-Type": "application/json",
      },
      body: JSON.stringify(requestBody),
    });

    if (response.ok) {
      const data = await response.json();

      if (data.error) {
        return res.status(400).json({
          success: false,
          message: "MCP tool execution failed",
          status: response.status,
          error: data.error,
          url: mcpUrl,
        });
      }

      res.json({
        success: true,
        message: "MCP tool executed successfully",
        status: response.status,
        data: data,
        url: mcpUrl,
      });
    } else {
      const errorText = await response.text();
      let errorDetails = errorText;

      try {
        const errorJson = JSON.parse(errorText);
        if (errorJson.error) {
          errorDetails = errorJson.error;
        }
      } catch (e) {
        // If not JSON, use the raw text
      }

      res.status(response.status).json({
        success: false,
        message: "MCP tool execution failed",
        status: response.status,
        error: errorDetails,
        url: mcpUrl,
      });
    }
  } catch (error) {
    console.error("MCP tool execution endpoint error:", error);
    res.status(500).json({
      error: "MCP tool execution failed",
      details: error.message,
    });
  }
});
```

### 3.6 Start Server

```javascript
app.listen(PORT, () => {
  console.log(`Backend server running on port ${PORT}`);
  console.log(`CORS enabled for origin: http://localhost:3000`);
});
```

---

## Part 4: Frontend React Application

### 4.1 OAuth Utility Functions (src/utils/oauth.js)

```javascript
// Generate PKCE code verifier
export function generateCodeVerifier() {
  const array = new Uint8Array(32);
  crypto.getRandomValues(array);
  return base64URLEncode(array);
}

// Generate PKCE code challenge
export async function generateCodeChallenge(codeVerifier) {
  const hash = await crypto.subtle.digest(
    "SHA-256",
    new TextEncoder().encode(codeVerifier)
  );
  return base64URLEncode(new Uint8Array(hash));
}

// Base64 URL encoding
function base64URLEncode(buffer) {
  return btoa(String.fromCharCode(...buffer))
    .replace(/\+/g, "-")
    .replace(/\//g, "_")
    .replace(/=/g, "");
}
```

### 4.2 Main App Component (src/App.js)

The main React component handles:

- OAuth flow with NetSuite
- MCP tool discovery and execution
- AI-powered query analysis and tool selection
- AI-powered response interpretation
- User interface for configuration and chat

| Feature                     | Description                                                                     |
| --------------------------- | ------------------------------------------------------------------------------- |
| 🔐 **OAuth 2.0 PKCE Flow**  | Secure authentication with NetSuite using industry-standard OAuth 2.0 with PKCE |
| 🔍 **MCP Tool Discovery**   | Automatically fetches available tools from NetSuite's MCP server                |
| 🤖 **AI-Powered Responses** | Uses Google Gemini to interpret NetSuite data naturally                         |
| 🛡️ **Smart Fallbacks**      | Handles errors gracefully with intelligent parsing                              |

### 4.3 Key State Variables

```javascript
const [netsuiteConfig, setNetsuiteConfig] = useState({
  accountId: "",
  consumerKey: "",
});

const [accessToken, setAccessToken] = useState(null);
const [refreshToken, setRefreshToken] = useState(null);
const [mcpTools, setMcpTools] = useState([]);
```

### 4.4 AI Query Analysis

Before executing any MCP tools, the AI analyzes the user's question to determine the right tool and parameters:

```javascript
const analyzeUserQuery = async (userInput) => {
  try {
    const genAI = new GoogleGenerativeAI(process.env.REACT_APP_GEMINI_API_KEY);
    const model = genAI.getGenerativeModel({ model: "gemini-2.0-flash" });

    const analysisPrompt = `You are an AI assistant that helps users interact with NetSuite data through MCP tools.

Available MCP tools:
${mcpTools.map((tool) => `- ${tool.name}: ${tool.description}`).join("\n")}

User question: "${userInput}"

Analyze this question and determine:
1. Which MCP tool should be used
2. What parameters to pass to the tool
3. Your confidence level (0-100)

Respond with JSON in this exact format:
{
  "confidence_score": 85,
  "action": "execute_tool",
  "tool_name": "runCustomSuiteQL",
  "tool_params": {
    "sqlQuery": "SELECT COUNT(*) FROM customer",
    "description": "Count all customers"
  },
  "reason": "User wants to count customers, so I'll use runCustomSuiteQL with a COUNT query"
}

If you're not confident or need clarification, use:
{
  "confidence_score": 45,
  "action": "ask_clarification",
  "reason": "The request is ambiguous. Please specify what type of customer information you need."
}`;

    const result = await model.generateContent(analysisPrompt);
    const response = await result.response;
    const responseText = response.text();

    // Extract JSON from the response
    const jsonMatch = responseText.match(/\{[\s\S]*\}/);
    if (jsonMatch) {
      return JSON.parse(jsonMatch[0]);
    }

    throw new Error("Failed to parse AI analysis response");
  } catch (error) {
    console.error("AI query analysis failed:", error);
    return {
      confidence_score: 0,
      action: "ask_clarification",
      reason:
        "I'm having trouble understanding your request. Please try rephrasing it.",
    };
  }
};
```

### 4.5 MCP Tool Execution

```javascript
const executeMcpTool = async (toolName, params) => {
  try {
    let currentAccessToken = accessToken;

    if (!currentAccessToken) {
      currentAccessToken = localStorage.getItem("netsuite_access_token");
      if (currentAccessToken) {
        setAccessToken(currentAccessToken);
      }
    }

    if (!currentAccessToken) {
      throw new Error(
        "No access token available. Please authorize with NetSuite first."
      );
    }

    const response = await fetch("http://localhost:3001/netsuite/mcp/execute", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${currentAccessToken}`,
        "X-NetSuite-Account": netsuiteConfig.accountId,
      },
      body: JSON.stringify({
        jsonrpc: "2.0",
        id: Date.now().toString(),
        method: "tools/call",
        params: {
          name: toolName,
          arguments: params,
        },
      }),
    });

    if (response.status === 401) {
      // Token expired, try to refresh automatically
      try {
        const newToken = await refreshAccessToken();

        // Retry with new token
        const retryResponse = await fetch(
          "http://localhost:3001/netsuite/mcp/execute",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${newToken}`,
              "X-NetSuite-Account": netsuiteConfig.accountId,
            },
            body: JSON.stringify({
              jsonrpc: "2.0",
              id: Date.now().toString(),
              method: "tools/call",
              params: {
                name: toolName,
                arguments: params,
              },
            }),
          }
        );

        if (!retryResponse.ok) {
          const errorData = await retryResponse.json();
          throw new Error(
            `MCP tool execution error after token refresh: ${
              retryResponse.status
            } - ${errorData.error || "Unknown error"}`
          );
        }

        const result = await retryResponse.json();
        return result;
      } catch (refreshError) {
        throw new Error(
          `Authentication failed: ${refreshError.message}. Please re-authorize with NetSuite.`
        );
      }
    }

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(
        `MCP tool execution error: ${response.status} - ${
          errorData.error || "Unknown error"
        }`
      );
    }

    const result = await response.json();
    return result;
  } catch (error) {
    console.error("MCP tool execution error:", error);
    throw error;
  }
};
```

### 4.6 OAuth Initialization

```javascript
const initializeOAuth = async (e) => {
  if (e) {
    e.preventDefault();
    e.stopPropagation();
  }

  try {
    if (!netsuiteConfig.accountId || !netsuiteConfig.consumerKey) {
      throw new Error("Missing required credentials");
    }

    setIsInitializing(true);

    // Generate PKCE code verifier and challenge
    const codeVerifier = generateCodeVerifier();
    const codeChallenge = await generateCodeChallenge(codeVerifier);

    // Generate state parameter
    const state =
      Math.random().toString(36).substring(2) +
      Math.random().toString(36).substring(2) +
      Math.random().toString(36).substring(2) +
      Math.random().toString(36).substring(2);

    // Store PKCE code verifier and state
    localStorage.setItem("code_verifier", codeVerifier);
    localStorage.setItem("oauth_state", state);

    // Build authorization URL with MCP scope only
    const authUrl = `https://${
      netsuiteConfig.accountId
    }.app.netsuite.com/app/login/oauth2/authorize.nl?response_type=code&client_id=${
      netsuiteConfig.consumerKey
    }&redirect_uri=${encodeURIComponent(
      "http://localhost:3000"
    )}&scope=mcp&code_challenge=${codeChallenge}&code_challenge_method=S256&state=${state}`;

    // Redirect to NetSuite authorization
    window.location.href = authUrl;
  } catch (error) {
    console.error("OAuth initialization failed:", error);
    setIsInitializing(false);
    alert(`OAuth initialization failed: ${error.message}`);
  }
};
```

### 4.7 AI-Powered Response Interpretation

This is the key component that makes the bot intelligent and conversational. Instead of returning raw JSON data, the AI interprets NetSuite results and provides natural language answers.

````javascript
// AI interpretation prompt for Gemini
const interpretationPrompt = `You are a helpful NetSuite assistant. The user asked: "${inputText}"
I successfully retrieved data from NetSuite using the MCP tool "${toolName}". The tool executed successfully and returned data. Here is the complete response:
${JSON.stringify(toolResult, null, 2)}

CRITICAL: The MCP tool executed SUCCESSFULLY. Do NOT say there was an error or that the system doesn't recognize terms. The data IS available in this response.

Your task:
1. Look through this entire response structure to find the actual data
2. The data is likely nested in the response structure
3. Answer the user's original question based on the data you find
4. Give a natural, conversational answer

REQUIREMENTS:
- The tool executed successfully, so the data IS available
- Answer the question directly and briefly
- Don't mention technical details like "MCP tool", "NetSuite", or show raw data
- Don't explain HOW you found the information
- Just give the answer a human would give
- If it's a count, format numbers nicely (e.g., "4,214" not "4214")
- Keep it conversational and helpful

IMPORTANT: Look for success indicators like "status: 200" and "success: true" - these mean the data retrieval was successful.`;

// Use Gemini AI to interpret the results
const genAI = new GoogleGenerativeAI(process.env.REACT_APP_GEMINI_API_KEY);
const model = genAI.getGenerativeModel({ model: "gemini-2.0-flash" });

try {
  const result = await model.generateContent(interpretationPrompt);
  const response = await result.response;
  const responseText = response.text();

  // Clean up the response
  return responseText.replace(/^```\w*\n?|\n?```$/g, "").trim();
} catch (interpretationError) {
  console.error("AI interpretation failed:", interpretationError);

  // Fallback to intelligent parsing if AI fails
  return extractAnswerFromData(toolResult, inputText);
}
````

### 4.8 Fallback Data Extraction

When AI interpretation fails, you need intelligent fallback mechanisms to ensure users still get helpful responses. This is crucial for production reliability.

> 🛡️ **Why Fallbacks Are Essential**
>
> AI interpretation can fail for many reasons:
>
> - **API Rate Limits:** Gemini or other AI services may be temporarily unavailable
> - **Unexpected Data Formats:** NetSuite responses might not match expected patterns
> - **Network Issues:** Intermittent connectivity problems
> - **AI Model Limitations:** Complex or ambiguous responses may confuse the AI

**Your fallback strategy should include:**

- **Pattern Recognition:** Use regex or string matching to identify common response types
- **Data Extraction:** Parse JSON responses to find key values (counts, balances, names, etc.)
- **Graceful Degradation:** Provide useful information even when full AI interpretation fails
- **User Communication:** Clearly indicate when fallback logic is being used

> 💡 **Example Fallback Approach**
>
> Instead of hardcoding specific field names like `expr1`, design your fallback to:
>
> - Look for numeric values that might represent counts
> - Identify currency patterns for financial data
> - Extract text fields that could be names or descriptions
> - Provide generic but helpful responses when specific parsing fails

### 4.9 Complete Message Handling Flow

Here's how the complete flow works when a user asks a question:

```javascript
const handleSendMessage = async (inputText) => {
  try {
    // 1. AI determines the right MCP tool and parameters
    const aiAnalysis = await analyzeUserQuery(inputText);

    if (aiAnalysis.action === "execute_tool") {
      // 2. Execute the MCP tool against NetSuite
      const toolResult = await executeMcpTool(
        aiAnalysis.tool_name,
        aiAnalysis.tool_params
      );

      // 3. Use AI to interpret the results conversationally
      const responseText = await interpretNetSuiteResults(
        toolResult,
        inputText,
        aiAnalysis.tool_name
      );

      // 4. Add the response to the chat
      addMessage("bot", responseText);
    } else {
      // Handle clarification requests
      addMessage("bot", aiAnalysis.reason);
    }
  } catch (error) {
    console.error("Message handling failed:", error);
    addMessage("bot", `I encountered an error: ${error.message}`);
  }
};
```

---

## Part 5: Development & Testing

> 🔧 **Implementation Approach**
>
> **Remember:** This guide shows one way to implement the NetSuite MCP integration. You can adapt these concepts to your preferred technology stack and development approach.

### 5.1 Development Environment

You'll need to set up a development environment that supports:

- **Backend Server:** For OAuth handling and MCP proxy functionality
- **Frontend Interface:** For user interaction and configuration
- **AI Integration:** For query analysis and response interpretation
- **Development Tools:** For debugging and testing your integration

### 5.2 Testing Strategy

Test your integration systematically:

- **OAuth Flow:** Ensure authentication works end-to-end
- **MCP Tool Discovery:** Verify you can retrieve available tools
- **Tool Execution:** Test basic MCP tool calls
- **AI Interpretation:** Validate that AI responses make sense
- **Error Handling:** Test fallback mechanisms and error scenarios

---

## Part 6: Getting Started

### 6.1 Basic Workflow

Once your system is set up, the basic workflow is:

- User asks a question in natural language
- Your AI analyzes the question and selects an MCP tool
- System executes the tool against NetSuite
- AI interprets the results and provides a conversational answer

### 6.2 Configuration Requirements

You'll need to configure:

- **NetSuite Credentials:** Account ID and Client ID from your integration record
- **AI Service:** API keys and configuration for your chosen AI provider
- **Environment Settings:** URLs, ports, and other configuration values

### 6.3 Validation Steps

Verify your integration works by:

- Testing the OAuth authentication flow
- Discovering available MCP tools
- Executing a simple query (like counting customers)
- Ensuring AI interpretation provides meaningful answers

---

## Part 7: Troubleshooting

### 7.1 Common Issues

**"MCP tools not available"**

- Ensure the MCP Tools SuiteApp is installed
- Check that your user has the custom MCP role assigned
- Verify the OAuth scope includes "mcp"

**"Authentication failed"**

- Check that your Client ID is correct
- Ensure the redirect URI matches exactly: `http://localhost:3000`
- Verify the integration record is enabled

**"Gemini API key not configured"**

- Add your Gemini API key to the `.env` file
- Restart the development server

**Port conflicts**

- Ensure ports 3000 and 3001 are available
- Check for other running Node.js processes

### 7.2 Debug Logging

The application includes comprehensive console logging. Check the browser console and terminal for:

- OAuth flow progress
- MCP tool discovery results
- API request/response details
- AI interpretation steps

---

## Part 8: Production Considerations

### 8.1 Security

- **Store sensitive configuration in environment variables**
- **Implement proper session management**
- **Use HTTPS in production**
- **Consider implementing rate limiting**

### 8.2 Deployment

- **Build the React app:** `npm run build`
- **Deploy the backend to a cloud service** (Heroku, AWS, etc.)
- **Update redirect URIs for production domains**
- **Configure CORS for production origins**

### 8.3 Monitoring

- **Implement logging for production environments**
- **Monitor API usage and performance**
- **Set up error tracking and alerting**

---

## 🎯 Key Takeaway

**Yes, you CAN connect your AI assistant to NetSuite using MCP!** While it may have some bugs and challenges (as any integration does), the core architecture works and provides a powerful way to give users conversational access to their NetSuite data.

### 🚀 What We've Proven

Using Gemini AI, we successfully created a system that:

1. **Understands natural language questions** about NetSuite data
2. **Automatically selects and executes** the right MCP tools
3. **Converts raw NetSuite responses** into conversational answers
4. **Provides a user-friendly interface** for business intelligence

### 🔒 Key Features

| Feature            | Description                                       |
| ------------------ | ------------------------------------------------- |
| 🔒 **Secure**      | OAuth 2.0 PKCE authentication with NetSuite       |
| 🧠 **Intelligent** | AI-powered query analysis and response generation |
| ⚡ **Real-time**   | Live data access through NetSuite's MCP server    |
| 🔧 **Extensible**  | Easy to add new MCP tools and capabilities        |

The integration works, it's powerful, and it opens up new possibilities for how users interact with their NetSuite data. While there are challenges to overcome, the foundation is solid and the potential is enormous.

---

_This guide represents the current state of NetSuite MCP AI integration as of August 15, 2025. Given the rapidly evolving nature of this technology, always verify current NetSuite documentation and AI service capabilities._
