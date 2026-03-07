# Testing Tools

> **Category:** Testing | **Tools:** 1 | **Auth required:** Yes

Simple connectivity verification tool.

---

## hello_world

Returns a simple Hello World greeting. Used to verify MCP connection is working without requiring any account permissions.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `name` | string | No | Optional name to include in the greeting |

**Returns:** Greeting message string.

**Example:**
```
Input:  { "name": "Sweeppea" }
Output: "Hello, Sweeppea! Welcome to Sweeppea MCP Server."
```

---

## When to Use

- First connection test when setting up a new MCP client
- Debugging connectivity issues before running `health_check`
- Verifying that the transport layer is working correctly (before auth is validated)

**Note:** `health_check` validates your API key and is the proper tool for session-start verification. Use `hello_world` only to confirm the MCP transport connection itself.
