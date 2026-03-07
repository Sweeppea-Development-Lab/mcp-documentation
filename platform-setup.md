# Platform Setup Guide

> How to connect the Sweeppea MCP Server to various AI clients and development environments.

**MCP Endpoint:** `https://mcp.sweeppea.com/`
**Protocol:** MCP 2025-11-25 (Streamable HTTP)
**Auth:** Bearer token (your Sweeppea API key)

---

## Claude Code (CLI)

```bash
claude mcp add sweeppea https://mcp.sweeppea.com/ \
  --transport http \
  --header "Authorization: Bearer YOUR_API_KEY" \
  --header "MCP-Protocol-Version: 2025-11-25"
```

**Verify the connection:**
```bash
claude mcp list
```

---

## Claude Desktop / Cowork

Add to your Claude Desktop configuration file:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "sweeppea": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.sweeppea.com/",
        "--header",
        "Authorization: Bearer YOUR_API_KEY",
        "--header",
        "MCP-Protocol-Version: 2025-11-25"
      ]
    }
  }
}
```

**Requirements:** Node.js must be installed.

---

## Cursor

**Global config:** `~/.cursor/mcp.json`
**Project config:** `.cursor/mcp.json` (in your project root)

```json
{
  "mcpServers": {
    "sweeppea": {
      "url": "https://mcp.sweeppea.com/",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY",
        "MCP-Protocol-Version": "2025-11-25"
      }
    }
  }
}
```

---

## Windsurf

**Config file:** `~/.codeium/windsurf/mcp_config.json`

```json
{
  "mcpServers": {
    "sweeppea": {
      "serverUrl": "https://mcp.sweeppea.com/",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY",
        "MCP-Protocol-Version": "2025-11-25"
      }
    }
  }
}
```

---

## GitHub Copilot (VS Code)

**Workspace config:** `.vscode/mcp.json`

```json
{
  "servers": {
    "sweeppea": {
      "type": "http",
      "url": "https://mcp.sweeppea.com/",
      "headers": {
        "Authorization": "Bearer ${input:sweeppeaApiKey}",
        "MCP-Protocol-Version": "2025-11-25"
      }
    }
  },
  "inputs": [
    {
      "type": "promptString",
      "id": "sweeppeaApiKey",
      "description": "Sweeppea API Key",
      "password": true
    }
  ]
}
```

The `promptString` input prompts you to enter the API key securely when VS Code loads the workspace.

---

## Gemini CLI

**Global config:** `~/.gemini/settings.json`
**Project config:** `.gemini/settings.json`

```json
{
  "mcpServers": {
    "sweeppea": {
      "httpUrl": "https://mcp.sweeppea.com/",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY",
        "MCP-Protocol-Version": "2025-11-25"
      }
    }
  }
}
```

---

## Agent Zero

1. Open Agent Zero
2. Go to **Settings > MCP/A2A > MCP Servers**
3. Add a new server with:

```json
{
  "type": "streamable-http",
  "url": "https://mcp.sweeppea.com/",
  "headers": {
    "Authorization": "Bearer YOUR_API_KEY",
    "MCP-Protocol-Version": "2025-11-25"
  }
}
```

---

## Antigravity by Google

**Config file:** `~/.gemini/antigravity/mcp_config.json`

```json
{
  "mcpServers": {
    "sweeppea": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.sweeppea.com/",
        "--header",
        "Authorization: Bearer YOUR_API_KEY",
        "--header",
        "MCP-Protocol-Version: 2025-11-25"
      ]
    }
  }
}
```

Uses `npx mcp-remote` as a bridge for SSE-to-stdio conversion.

---

## Finding Your API Key

1. Log in to your Sweeppea account at [app.sweeppea.com](https://app.sweeppea.com)
2. Navigate to **Settings > API**
3. Copy your API key
4. Replace `YOUR_API_KEY` in the configuration above

**Security:** Never commit your API key to version control. Use environment variables or secure secret storage.

---

## Verifying the Connection

After setup, test the connection with the `hello_world` tool:

```
Run: hello_world with name "test"
Expected: "Hello, test! Welcome to Sweeppea MCP Server."
```

Then verify your API key with `health_check`:
```
Run: health_check
Expected: { "status": "ok", "message": "API connection successful" }
```

---

## Troubleshooting

| Error | Solution |
|---|---|
| `401 Unauthorized` | Check your API key — it may be invalid or expired |
| `Protocol version error` | Ensure you're passing `MCP-Protocol-Version: 2025-11-25` |
| `Session ID required` | Your MCP client must persist the session ID header across requests |
| `404 Not Found` | Use POST method — GET only returns server info at root |
| Connection timeout | Check network connectivity to `mcp.sweeppea.com` |
