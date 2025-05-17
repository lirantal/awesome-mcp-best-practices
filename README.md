# Awesome MCP Best Practices ![Awesome](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)

A curated and opinionated list of awesome Model Context Protocol (MCP) best practices as they pertain to building MCP Servers and MCP Clients.

---

## MCP Servers

- 1 MCP Server Tools:
  - 1.1 [Tool Naming Standards](#11-Tool-Naming-Standards)
  - 1.2 [Tool Naming Aliases](#12-Tool-Naming-Aliases)
  - 1.3 [Avoid Not Found Responses](13-Avoid-Not-Found-Responses)
- 2 MCP Server Architecture
  -  2.1 [Abstract Server Capabilities](#21-Abstract-Server-Capabilities)
- MCP Server Deployment:
  - [Package Your MCP Server as a Docker Container]() 
- MCP Server Security: 
  - [Secure MCP Server Code]()
  - [Secure MCP Server Dependencies]()
- MCP Server Performance:
- MCP Server Errors and Observability:

## MCP Clients

// TBD

---

## 1 MCP Server Tools

### 🔵 1.1 Tool Naming Standards

Use consistent, compatible naming conventions for your MCP Server `Tools` to ensure they can be properly discovered and invoked by MCP Clients.

#### ❌ Avoid These Tool Naming Conventions

- Spaces: `get Npm Package Info`
- Dot notation: `get.Npm.Package.Info`
- Brackets/parentheses: `get(Npm)PackageInfo`
- 
#### ✅ Recommended Tool Naming Conventions

- ✅ camelCase (**preferred**): `getNpmPackageInfo`
- kebab-case: `get-npm-package-info`
- snake_case: `get_npm_package_info`

```javascript
server.tool(
  "getNpmPackageInfo",
  "Get information about an npm package",
  {
    packageName: z.string()
  },
  async ({ packageName }) => {    
    // Implementation details...
    return {
      content: [{ type: "text", text: output }],
    };
  }
);
```

#### 💡 Why It Matters

Using non-standard naming conventions can prevent or disrupt MCP Clients from properly discovering and surfacing your tools to end users. GPT-4o tokenization works best with `camelCase` naming conventions.

---

### 🔵 1.2 Tool Naming Aliases

When the Tool name can be intereted and accessed using different naming conventions, call out aliases in the Tool's description.

#### ❌ Problematic Pattern

For example, a `postMesage` tool name might be too specific:

```javascript
server.tool(
  "postMessage",
  "Post a message under your account",
   () => {}
)
```

The LLM might not invoke the tool if the users ask for "share a social post on Twitter", or "upload this picture to Instagram".

#### ✅ Recommended Practice

Specify aliases and elaborate description for LLMs to better understand when it is required to invoke your tool.

```javascript
server.tool(
  "postMessage",
  "Upload, share, and post messages on social media",
   () => {}
)
```

---

### 🔵 1.3 Avoid Not Found Responses

When implementing search-type tools in your MCP Server, avoid returning explicit "not found" messages even when exact matches aren't available.

#### ❌ Problematic Pattern

```javascript
// Don't do this
if (!exactMatch) {
  return {
    content: [
      { 
        type: "text", 
        text: `Module ${query} not found. Here are all available modules: ${allModules}` 
      }
    ]
  };
}
````

#### ✅ Recommended Practice

```javascript
// Do this instead
return {
  content: [
    { 
      type: "text",
      text: `Here are the available modules that may help with your query: ${relevantModules}`
    }
  ]
};
```

#### 💡 Why It Matters

Let the LLM determine relevance from the data provided rather than prematurely declaring failure in your tool response. LLMs can be overly influenced by negative statements like "not found" causing them to ignore the useful information that follows. By providing relevant data without negative framing, you enable the LLM to process and utilize all available information properly.

#### ⚠️ Important Exception

This approach isn't appropriate for all scenarios. When handling sensitive data (like user information), security and privacy concerns should take precedence over providing alternative data.

---

## MCP Server Architecture

### 🔵 2.1 Abstract Server Capabilities

Follow an inversion-of-control paradigm to allow your MCP Server capabilities to receive a `server` object and use it to apply `tools`, `resources`, `prompts`, and other capabilities.

#### 💡 Why It Matters

You may not know ahead of time if you will need to build for an STDIO or HTTP transports, nor you don't know which SDK or cloud infrastructure you will be deploying to (e.g: Vercel vs Cloudflare). To allow the underlying MCP Server logic implementation to supplement any of these future concerns, build it in a way that 

// TBD bad pattern

// TBD good pattern

// TBD example code

---

## MCP Server Deployment:

### 🔵 Package Your MCP Server as a Docker Container

Deploy your MCP Servers as Docker containers to eliminate environment setup challenges and ensure consistent operation across different systems.

#### 💡 Why It Matters

MCP Servers often require specific runtime environments (Node.js, Python) with particular versions and dependencies. Docker abstracts away these requirements, turning complex setup instructions into a simple container run command.

#### 💪 Key Benefits

- Consistency: Eliminates "works on my machine" problems
- Isolation: Prevents dependency conflicts with host system
- Portability: Runs identically across development, testing, and production
- Simplified Deployment: Reduces user setup to installing Docker and running a container
- Resource Management: Provides built-in tools for controlling CPU, memory, and network usage

Example `Dockerfile` implementation to package an MCP Server:

```
FROM node:18-slim

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

Users run it as follows:

```
docker run -p 3000:3000 your-mcp-server-image
```

---

## MCP Server Security: 
  
### 🔵 Secure MCP Server Dependencies

Ensure your MCP Server is free from known vulnerabilities in third-party dependencies to meet security requirements and facilitate organizational adoption.

#### 💡 Why It Matters

MCP Servers typically require broad access and integration capabilities, making any vulnerability a significant security risk. Organizational IT and security teams scrutinize these dependencies before approving adoption.

- MCP Servers must meet stringent security and compliance requirements
- Vulnerable dependencies create potential entry points for malicious actors
- Security is mandated by SBOM requirements following the SolarWinds attack

#### ✅ Recommended Practice

- Regularly scan dependencies for known vulnerabilities
- Keep all components updated to the latest secure versions
- Monitor security advisories related to your dependencies
- Ensure compliance with licensing and security standards

