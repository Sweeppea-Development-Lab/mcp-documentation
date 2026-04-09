# Sweeppea MCP Server — Documentation

> **Version:** 1.15.0 | **Tools:** 70 | **Categories:** 16 | **Protocol:** MCP 2025-11-25

Sweeppea MCP Server is a Model Context Protocol (MCP) server for AI-powered sweepstakes management. It provides 70 tools across 16 categories to manage sweepstakes, participants, official rules, winners, billing, calendar events, support tickets, and more — all accessible through any MCP-compatible AI client via Streamable HTTP transport.

> **Availability:** This MCP server is available exclusively for Sweeppea clients in the United States and Canada.

---

## Quick Start

### Claude Code (CLI)

```bash
claude mcp add sweeppea https://mcp.sweeppea.com/ \
  --transport http \
  --header "Authorization: Bearer YOUR_API_KEY" \
  --header "MCP-Protocol-Version: 2025-11-25"
```

### Connection Details

| Property | Value |
|---|---|
| Endpoint | `https://mcp.sweeppea.com/` |
| Protocol | MCP 2025-11-25 (Streamable HTTP) |
| Auth | Bearer token (Sweeppea API key) |
| Format | JSON-RPC 2.0 |

---

## Tool Categories

| # | Category | Tools | Description |
|---|---|---|---|
| 1 | [Account](./01-account-tools.md) | 4 | Profile, business info, plan details, health check |
| 2 | [Sweepstakes](./02-sweepstakes-tools.md) | 7 | Create, update, clone, pause, delete sweepstakes |
| 3 | [Participants](./03-participant-tools.md) | 6 | Add, search, list, count, delete participants |
| 4 | [Entry Page](./04-entry-page-tools.md) | 3 | Form fields, settings, configuration |
| 5 | [Groups](./05-group-tools.md) | 4 | Participant segmentation and organization |
| 6 | [Notes](./06-notes-tools.md) | 5 | Encrypted notes management |
| 7 | [Calendar](./07-calendar-tools.md) | 5 | Campaign milestone calendar events |
| 8 | [Rules](./08-rules-tools.md) | 5 | Official rules documents and wizard |
| 9 | [Billing & Wallet](./09-billing-wallet-tools.md) | 4 | Invoices, usage, wallet transactions |
| 10 | [Support Tickets](./10-support-ticket-tools.md) | 7 | Create and manage support requests |
| 11 | [Winners](./11-winner-tools.md) | 5 | Draw winners, schedule drawings |
| 12 | [Documentation](./12-documentation-tools.md) | 1 | Search help articles |
| 13 | [Utilities](./13-utility-tools.md) | 5 | States, ZIP codes, area codes, countries, timezones |
| 14 | [To-Do](./14-todo-tools.md) | 4 | Internal task management (admin only) |
| 15 | [Testing](./15-testing-tools.md) | 1 | Connection verification |
| 16 | [Files](./16-file-tools.md) | 4 | Upload, list, delete, and share files |

---

## Platform Setup

See [Platform Setup Guide](./platform-setup.md) for configuration instructions for:
- Claude Code (CLI)
- Claude Desktop / Cowork
- Cursor
- Windsurf
- GitHub Copilot (VS Code)
- Gemini CLI
- Agent Zero
- Antigravity by Google

---

## Legal & Compliance

See [Legal Compliance Guide](./legal-compliance.md) for:
- Mandatory checklist for a legally compliant sweepstakes
- AMOE requirements
- State registration and bonding rules
- Industry-specific restrictions (alcohol, cannabis)

---

## AI Agent Operational Guide

See [AI Agent Guide](./ai-agent-guide.md) for:
- Step-by-step workflow for AI agents
- Critical guardrails and prohibited promotions
- Automatic compliance decisions
- Tool chaining best practices

---

## Useful Links

- **MCP Docs (Web):** https://mcpdocs.sweeppea.com/
- **MCP Endpoint:** https://mcp.sweeppea.com/
- **Sweeppea Platform:** https://www.sweeppea.com
- **Tool Showcase:** https://mcpdocs.sweeppea.com/showcase.html
