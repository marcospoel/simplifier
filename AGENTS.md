# Agents

This project uses specialized subagents for distinct concerns. Agents are invoked proactively — no user prompt needed. Independent agents run in parallel.

## Core Principles

- **Simplifier-first**: Always use the Simplifier MCP server before calling the REST API directly.
- **Figma as truth**: UI must match Figma designs. Verify with Playwright after every screen change.
- **OpenUI5 1.96.40 only**: No other UI5 version is permitted anywhere in generated code or config.
- **Test after build**: `ui-tester` runs after every screen or BO change.
- **Review after every change**: `code-reviewer` is the last agent in every chain.

## Agent Index

| Agent | File | Role | Auto-trigger |
|---|---|---|---|
| planner | `agents/planner.md` | Break down features into Simplifier artifacts + ordered build steps | New feature, multi-screen work, architecture question |
| simplifier-app-builder | `agents/simplifier-app-builder.md` | Create/modify Simplifier Apps, Screens, UI widgets, navigation | Building or changing any app screen |
| simplifier-bo-developer | `agents/simplifier-bo-developer.md` | Create/edit Business Objects and their JavaScript functions | Any server-side logic change |
| simplifier-connector-manager | `agents/simplifier-connector-manager.md` | Create/configure Connectors, Logins, Connector Calls | New integration, connector change |
| simplifier-workflow-builder | `agents/simplifier-workflow-builder.md` | Create/edit Workflows, tasks, conditions, sub-workflows | Workflow creation or modification |
| ui5-developer | `agents/ui5-developer.md` | Write OpenUI5 1.96.40 views, controllers, fragments; apply best practices | Any UI code file touched |
| figma-inspector | `agents/figma-inspector.md` | Read Figma frames, extract layout/styles, report deviations from live app | Before building a screen; after building to verify |
| ui-tester | `agents/ui-tester.md` | Write and run Playwright tests; inspect live UI; report Figma deviations | After any screen or BO change |
| debugger | `agents/debugger.md` | Debug runtime errors via Chrome DevTools MCP; inspect network, console, performance | On any error report or unexpected behavior |
| code-reviewer | `agents/code-reviewer.md` | Review all changes for correctness, UI5 best practices, security, Simplifier patterns | After every code or config change |
| retrospective | `agents/retrospective.md` | Evaluate completed work and improve agent instructions, orchestration patterns, and constraints | After every completed feature; after repeated agent failures; every 5 features as a standing cycle |

## Orchestration Patterns

### New Screen from Figma
```
planner → figma-inspector (read design) → simplifier-app-builder + ui5-developer (parallel) → ui-tester → code-reviewer
```

### New Backend Integration
```
planner → simplifier-connector-manager → simplifier-bo-developer → ui-tester → code-reviewer
```

### Bug Fix
```
debugger → (simplifier-bo-developer | ui5-developer) → ui-tester → code-reviewer
```

### Full Feature
```
planner
  ↓ (parallel)
simplifier-connector-manager  simplifier-bo-developer  simplifier-app-builder
  ↓ (all complete)
figma-inspector → ui-tester → code-reviewer → retrospective
```

### Standing Improvement Cycle (every 5 features)
```
retrospective → (edits to agents/*.md, AGENTS.md, CLAUDE.md) → git commit
```
