# Setup Guide

## Prerequisites

| Tool | Version | Purpose |
|---|---|---|
| Node.js | ≥ 18 | Run MCP servers via npx |
| Claude Code | latest | AI agent CLI |
| Figma desktop app | latest | Figma MCP server (Dev Mode required) |
| Chrome | latest | Chrome DevTools MCP |

## 1. Clone the repo

```bash
git clone https://github.com/marcospoel/simplifier.git
cd simplifier
```

## 2. Configure environment

```bash
cp .env.example .env
```

Edit `.env`:

```bash
SIMPLIFIER_BASE_URL=https://tsystems-eval.simplifier.cloud
SIMPLIFIER_TOKEN=<paste from Simplifier user profile → "Copy current token">
```

> **Token expiry:** The token expires on logout. When it changes, update `.env` and restart Claude Code.
>
> **Alternative:** Instead of a token, set `SIMPLIFIER_CREDENTIALS_FILE=./.credentials` and create `.credentials`:
> ```json
> { "user": "youruser", "pass": "yourpassword" }
> ```

## 3. Start Figma desktop

Open the Figma desktop app and enable **Dev Mode** (Figma menu → Preferences → Enable Dev Mode). The Figma MCP server listens on `http://127.0.0.1:3845/mcp` automatically once Dev Mode is active.

## 4. Start Claude Code

```bash
claude
```

Claude Code picks up `.mcp.json` from the working directory automatically and starts all 6 MCP servers:

| MCP server | What it provides |
|---|---|
| `simplifier-mcp` | Platform operations (apps, BOs, connectors, workflows) |
| `Telecontext` | Jira and Confluence (Deutsche Telekom internal) |
| `ui5-mcp-server` | UI5 1.96.40 API reference and linting |
| `figma` | Read Figma designs (requires Figma desktop running) |
| `playwright` | Browser automation and UI testing |
| `chrome-devtools` | Runtime debugging and network inspection |

## 5. Authenticate Telecontext

Telecontext uses T-Systems SSO — no token needed in `.env`. On first use, Claude Code will open a browser window for you to log in with your DT/T-Systems account. See the [authentication guide](https://docs.devops.telekom.de/docs/AI/Telecontext/authentication#authenticating-in-claude-code).

> **Internal network users** (Nucleus/DevClient): The `.mcp.json` can alternatively point to `https://telecontext-internal.trap.prod.tap.telekom.de/mcp`.

## 6. Verify connectivity

Ask Claude: *"List the Simplifier projects on the instance."* — this exercises the Simplifier MCP and confirms the token is valid.

## Refreshing the Simplifier token

1. Log in to `https://tsystems-eval.simplifier.cloud`
2. Go to your user profile → copy the current token
3. Update `SIMPLIFIER_TOKEN` in `.env`
4. Restart Claude Code
