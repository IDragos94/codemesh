# CodeMode MCP Implementation

This project implements a "CodeMode" MCP server inspired by Cloudflare's approach - instead of many granular MCP tools, we provide a single tool that executes TypeScript code against discovered MCP tools.

## Project Structure

```
packages/
├── client/              # MCP client with CLI and interactive modes
├── example-server/      # Demo MCP server for testing
└── codemode-server/     # Main CodeMode MCP server
```

## Current Implementation Status

### ✅ Completed
1. **MCP Configuration System** (`packages/codemode-server/src/config.ts`)
   - VSCode-compatible MCP configuration format
   - ✅ **Multi-server support**: HTTP, stdio, and websocket server types
   - Singleton ConfigLoader with Zod validation

2. **Enhanced Client CLI Mode** (`packages/client/`)
   - Works in both interactive and CLI modes
   - ✅ `--tool-args-file` and `--code-file` options for clean argument passing
   - ✅ Supports HEREDOC patterns for complex JSON/TypeScript
   - Can connect to MCP servers and call tools without bash escaping hell

3. **Multi-Server Tool Discovery** (`packages/codemode-server/src/toolDiscovery.ts`)
   - ✅ **HTTP servers**: Full StreamableHTTPClientTransport support
   - ✅ **Stdio servers**: Complete StdioClientTransport integration
   - ✅ Connects to and enumerates tools from multiple server types simultaneously
   - ✅ Extracts tool schemas, descriptions, and metadata from all configured servers
   - ✅ Proper connection management with error handling and cleanup

4. **TypeScript Type Generation** (`packages/codemode-server/src/typeGenerator.ts`)
   - ✅ Converts JSON schemas to TypeScript interfaces using json-schema-to-typescript
   - ✅ Generates type-safe function signatures for discovered tools
   - ✅ Creates comprehensive tool metadata for runtime resolution

5. **Tiered Discovery Workflow** (`packages/codemode-server/src/index.ts`)
   - ✅ `discover-tools` - High-level tool overview (context-efficient)
   - ✅ `get-tool-apis` - Selective TypeScript API loading for specific tools
   - ✅ `generate-types` - File-based type generation for development/debugging

6. **Multi-Server Runtime Wrapper** (`packages/codemode-server/src/runtimeWrapper.ts`)
   - ✅ **HTTP & Stdio support**: Works with both transport types seamlessly
   - ✅ Creates runtime that makes generated TypeScript functions callable
   - ✅ Proxies function calls to actual MCP tools with proper connection management
   - ✅ In-memory execution environment for agent code
   - ✅ Safe function name generation (tool_serverId pattern)

7. **Complete Code Execution System** (`packages/codemode-server/src/codeExecutor.ts`)
   - ✅ **VM2 integration**: Sandboxed TypeScript execution with timeout
   - ✅ **Tool injection**: Runtime tools available as callable functions
   - ✅ **TypeScript compilation**: Compiles TypeScript to JavaScript before execution
   - ✅ **Error handling**: Proper error capture and formatting
   - ✅ **Multi-server execution**: Can call tools from multiple MCP servers in single code block

8. **Enhanced Execute-Code Tool**
   - ✅ Integrated runtime API wrapper with code execution tool
   - ✅ Injects tool APIs into TypeScript execution context
   - ✅ **LIVE EXECUTION**: Full TypeScript code execution with real tool calls
   - ✅ **Multi-server coordination**: Executes code that calls tools from both HTTP and stdio servers
   - ✅ **Intelligent data processing**: CodeMode enables advanced filtering, analysis, and transformation

### 🎯 **BREAKTHROUGH ACHIEVED**
✅ **Full Multi-Server CodeMode Implementation Complete!**
   - HTTP servers (example-server) + Stdio servers (weather-server) working together
   - Live TypeScript execution with real tool calls across multiple server types
   - Intelligent data processing capabilities (severity filtering, etc.)
   - Context-efficient tiered discovery prevents tool pollution
   - This is a "poor man's Code Mode" that rivals Cloudflare's approach!

## Key Files

- `mcp-config.json` - MCP server configuration (VSCode format)
- `packages/codemode-server/src/index.ts` - Main server with tools
- `packages/codemode-server/src/config.ts` - Configuration loader
- `packages/codemode-server/src/toolDiscovery.ts` - Tool discovery service
- `packages/client/index.ts` - CLI client for testing

## Development Commands

```bash
# Start example server (for testing)
pnpm dev:example-server

# Start codemode server
pnpm dev:codemode-server:watch

# Test with CLI client - old way
pnpm client -- --connect http://localhost:3002/mcp --call-tool discover-tools

# Test with new file-based CLI - much cleaner!
npx tsx packages/client/index.ts --connect http://localhost:3002/mcp --call-tool execute-code --tool-args-file tmp/execute-args.json --code-file tmp/greet-claudia.ts

# Test with HEREDOC (no files needed)
npx tsx packages/client/index.ts --connect http://localhost:3002/mcp --call-tool execute-code --tool-args "$(cat <<'EOF'
{
  "code": "const result = await greet_example_server({ name: 'Claudia' }); return result;",
  "toolNames": ["greet"],
  "configPath": "/Users/michael/Projects/learn/mcp/codemode/mcp-config.json"
}
EOF
)"

# Build all packages
pnpm build
```

## Major Breakthrough

The **tiered discovery workflow** solves the major UX issue with MCP:

**Problem**: Traditional MCP clutters agent context with ALL available tools, even unused ones.

**Solution**: Our three-tier approach:
1. **`discover-tools`** - High-level overview of what's available
2. **`get-tool-apis`** - Load TypeScript APIs only for tools you'll actually use
3. **`execute-code`** - Execute code with access to only the loaded tools

This drastically reduces context usage while maintaining full functionality!

## Architecture Notes

- Uses pnpm monorepo structure
- TypeScript with ESM modules (.js extensions required)
- Express server for HTTP MCP transport with debug logging
- Streamable HTTP transport with SSE support
- Context-efficient tiered discovery prevents tool pollution
- File-based CLI arguments eliminate bash escaping hell

## Current Status

✅ **Multi-Server CodeMode Complete** - Full implementation working:
- Multi-server tool discovery (HTTP + Stdio) ✅
- Type generation for all server types ✅
- Multi-server runtime wrapper ✅
- Live TypeScript code execution ✅
- Multi-server coordination in single code blocks ✅
- Enhanced CLI with file support ✅

## Multi-Server Examples

### Example 1: Multi-Server Greeting + Weather
```typescript
// Uses HTTP server (example-server) + Stdio server (weather-server)
const greeting = await greet_example_server({ name: "Michael" });
console.log("🎉", greeting.content[0].text);

const alerts = await get_alerts_weather_server({ state: "NC" });
const alertsData = JSON.parse(alerts.content[0].text);
console.log(`🌦️ Found ${alertsData.features.length} weather alerts`);

return "Successfully used both HTTP and stdio MCP servers!";
```

### Example 2: Intelligent Data Processing
```typescript
// CodeMode's superpower: intelligent filtering and analysis
const alerts = await get_alerts_weather_server({ state: "NC" });
const alertsData = JSON.parse(alerts.content[0].text);

// Find highest severity level present
const severityHierarchy = ['Extreme', 'Severe', 'Moderate', 'Minor'];
const highestSeverity = severityHierarchy.find(severity =>
  alertsData.features.some(alert => alert.properties.severity === severity)
);

// Filter to show only most severe alerts
const mostSevereAlerts = alertsData.features.filter(alert =>
  alert.properties.severity === highestSeverity
);

return {
  totalAlerts: alertsData.features.length,
  highestSeverityLevel: highestSeverity,
  mostSevereAlerts: mostSevereAlerts.length,
  summary: `Filtered ${alertsData.features.length} to ${mostSevereAlerts.length} most severe`
};
```

### Running Multi-Server Examples
```bash
# Multi-server greeting + weather
node packages/client/index.ts --connect http://localhost:3002/mcp --call-tool execute-code --code-file tmp/greet-and-weather.ts --tool-args-file tmp/multi-server-args.json

# Intelligent severity filtering
node packages/client/index.ts --connect http://localhost:3002/mcp --call-tool execute-code --code-file tmp/most-severe-alerts.ts --tool-args-file tmp/alerts-args.json
```