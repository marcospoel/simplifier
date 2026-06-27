# Agents

This project uses specialized subagents for distinct concerns. Agents are invoked proactively — no user prompt needed. Independent agents run in parallel.

## Core Principles

- **Requirements first**: Every feature starts with `business-analyst` reading Jira/Confluence, then `solution-architect` producing a TSD before any builder runs.
- **Simplifier-first**: Always use the Simplifier MCP server before calling the REST API directly.
- **Figma as truth**: UI must match Figma designs. Verify with Playwright after every screen change.
- **OpenUI5 1.96.40 only**: No other UI5 version is permitted anywhere in generated code or config.
- **Test after build**: `ui-tester` runs after every screen or BO change.
- **Review after every change**: `code-reviewer` is the last agent in every chain.

## Agent Index

| Agent | File | Role | Auto-trigger |
|---|---|---|---|
| business-analyst | `agents/business-analyst.md` | Read Jira/Confluence, clarify requirements, produce BRD in `docs/requirements/` | Any new feature request; when a Jira ticket is referenced |
| solution-architect | `agents/solution-architect.md` | Convert BRD into Technical Solution Design (TSD) in `docs/designs/` — artifact graph, data model, BO contracts, connector design, build order | After business-analyst; before planner and all builders |
| planner | `agents/planner.md` | Break down TSD build order into an ordered agent execution plan | After solution-architect; on any new feature |
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
| doc-writer | `agents/doc-writer.md` | Create and keep `docs/` current — architecture, artifact specs, screen specs, setup guide, changelog | After any artifact is created or changed; after `retrospective` edits agent files |

## Orchestration Patterns

### Full Feature (standard — from Jira ticket to deployed code)
```
business-analyst (read Jira/Confluence → BRD)
  ↓
solution-architect (BRD → TSD: artifact graph, data model, BO contracts)
  ↓
planner (TSD → ordered agent execution plan)
  ↓ (parallel)
simplifier-connector-manager   simplifier-bo-developer   simplifier-app-builder + ui5-developer
  ↓ (all complete)
figma-inspector → ui-tester → code-reviewer → doc-writer → retrospective
```

### New Screen from Figma (UI only, no new backend)
```
business-analyst → solution-architect → planner
  ↓
figma-inspector → simplifier-app-builder + ui5-developer (parallel) → ui-tester → code-reviewer → doc-writer
```

### New Backend Integration (no new screens)
```
business-analyst → solution-architect → planner
  ↓
simplifier-connector-manager → simplifier-bo-developer → ui-tester → code-reviewer → doc-writer
```

### Bug Fix
```
debugger → (simplifier-bo-developer | ui5-developer) → ui-tester → code-reviewer → doc-writer
```

### Standing Improvement Cycle (every 5 features)
```
retrospective → (edits to agents/*.md, AGENTS.md, CLAUDE.md) → doc-writer → git commit
```
