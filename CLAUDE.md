# Simplifier Project — Claude Code Configuration

## Project Overview

This project builds and maintains applications on the **Simplifier Low-Code Platform** (SAP ecosystem). The UI is built with **OpenUI5 1.96.40 exclusively**. UI designs are defined in Figma. The backend integrates with SAP and other enterprise systems via Simplifier Connectors and Business Objects.

**Target instance:** `https://tsystems-eval.simplifier.cloud`

## Strict Constraints

- **OpenUI5 version: 1.96.40 only.** Never reference or suggest any other version. Widget names in Simplifier always include the UI5 version suffix, e.g. `Button (1.96)`.
- Always interact with Simplifier via the **Simplifier MCP server first**. Only fall back to direct API calls (`/client/api-docs/api.yaml`) when the MCP lacks a required capability.
- Never commit `.env`, `.credentials`, or any file containing `SIMPLIFIER_TOKEN`.

## MCP Server Priority Order

1. **simplifier-mcp** — all Simplifier platform operations (apps, screens, BOs, connectors, data types, workflows)
2. **telecontext** — Jira and Confluence: requirements, tickets, documentation, comments
3. **ui5-mcp-server** — UI5 API reference, linting, manifest validation, project scaffolding
4. **figma** — read Figma designs (components, layout, variables) as the source of truth for UI
5. **playwright** — UI inspection, deviation detection between Figma and live app, automated testing
6. **chrome-devtools** — runtime debugging, network inspection, performance tracing, console errors

## Agent Orchestration

Invoke agents **proactively** — do not wait for the user to ask. Run independent agents in parallel.

| Situation | Agent to invoke |
|---|---|
| New feature request or Jira ticket referenced | `business-analyst` → `solution-architect` → `planner` |
| Architecture or multi-system design question | `solution-architect` |
| Building/modifying a Simplifier App or Screen | `simplifier-app-builder` |
| Creating/editing a Business Object | `simplifier-bo-developer` |
| Setting up or modifying a Connector | `simplifier-connector-manager` |
| Creating/editing a Workflow | `simplifier-workflow-builder` |
| Writing or fixing UI5 view/controller code | `ui5-developer` |
| Comparing live UI against Figma design | `figma-inspector` |
| Writing or running E2E / UI tests | `ui-tester` |
| Debugging runtime errors or network issues | `debugger` |
| After any code change | `code-reviewer` → `doc-writer` |

## Development Commands

```bash
# Simplifier MCP server (npx, uses env vars from .env or .mcp.json)
npx @simplifierag/simplifier-mcp@latest

# UI5 linting
npx @ui5/linter

# Playwright tests
npx playwright test

# Install project dependencies
npm install
```

## Authentication

The `SIMPLIFIER_TOKEN` is a session token obtained from the Simplifier user profile. It expires on logout or session timeout. When it changes:
1. Update `SIMPLIFIER_TOKEN` in your `.env` or re-run the MCP `claude mcp add` command.
2. Restart Claude Code.

Alternatively use `SIMPLIFIER_CREDENTIALS_FILE` pointing to a JSON file `{ "user": "...", "pass": "..." }`.

## Key Platform Concepts

- **Apps / Screens** — OpenUI5-based UI, built in the Simplifier Screen Designer
- **Business Objects (BOs)** — server-side Node.js JavaScript, callable from app logic or other BOs
- **Connectors** — integration definitions (REST, SOAP, SQL, SAP RFC, OData V2/V4, etc.)
- **Data Types** — Domain, Struct, Collection — used for BO/connector parameter typing
- **Workflows** — process automation with Automated, User, and Notification tasks
- **Modules** — reusable app components with defined interfaces
- **Projects** — organizational containers for all artifacts

## Figma Integration

Figma is the **single source of truth for UI**. Before implementing any screen:
1. Use `figma` MCP to read the relevant frame/component.
2. Use `ui5-developer` to implement with OpenUI5 1.96.40 best practices.
3. Use `playwright` + `figma-inspector` to verify the live app matches the design.

## File Structure

```
.mcp.json               — MCP server configuration
.env                    — Local secrets (gitignored)
.env.example            — Template for env vars
CLAUDE.md               — This file
AGENTS.md               — Agent index and orchestration guide
agents/                 — Specialized subagent definitions
docs/
  architecture.md       — System overview and platform layers
  setup.md              — Onboarding and setup guide
  changelog.md          — Dated change history
  requirements/         — BRDs produced by business-analyst
  designs/              — TSDs produced by solution-architect
  artifacts/            — Simplifier artifact registry (apps, BOs, connectors, etc.)
  screens/              — Per-screen specs
```
