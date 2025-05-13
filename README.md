# Awesome MCP Best Practices ![Awesome](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)

A curated and opinionated list of awesome Model Context Protocol (MCP) best practices as they pertain to building MCP Servers and MCP Clients.

---

## MCP Servers

- 1 MCP Server Tools:
  - 1.1 [Tool Naming Standards](#11-Tool-Naming-Standards)
  - 1.2 [Avoid Not Found Responses](12-Avoid-Not-Found-Responses)
- MCP Server Deployment:
  - [Package Your MCP Server as a Docker Container]() 
- MCP Server Security: 
  - [Secure MCP Server Code]()
  - [Secure MCP Server Dependencies]()
- MCP Server Performance:
- MCP Server Errors and Observability:

## MCP Clients

TBD

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

### 🔵 1.2 Avoid Not Found Responses

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

