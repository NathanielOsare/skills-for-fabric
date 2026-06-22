# Microsoft Fabric Development Instructions

> **Update Check**: At session start, check for skills-for-fabric updates by reading the remote `package.json` version from `https://github.com/microsoft/skills-for-fabric` (via `git fetch origin main --quiet && git show origin/main:package.json` or GitHub API with authentication) and comparing with the local `package.json` version. Show changelog if update available.

This project uses Microsoft Fabric for data engineering, warehousing, and analytics.

## Architecture Mode

- Use the hybrid layering model: **Agents → Skills → Common**.
- For cross-workload orchestration, start with `agents/FabricDataEngineer.agent.md`.
- Delegate deep endpoint implementation to relevant skills under `skills/`.

## Authentication

All Fabric operations require Azure AD authentication. For development:

```bash
# Login to Azure
az login

# Get token for Fabric REST API
az account get-access-token --resource https://api.fabric.microsoft.com

# Get token for SQL connections (Warehouse, Lakehouse SQL Endpoint)
az account get-access-token --resource https://database.windows.net
```

## Fabric REST APIs

All Fabric operations use the REST APIs documented at:
https://learn.microsoft.com/en-us/rest/api/fabric/articles/

## Developer vs Consumer Patterns

### Developers
- Use **REST APIs** to create/manage artifacts (workspaces, warehouses, lakehouses)
- Use **protocol-specific** connections to access data:
  - ODBC/JDBC for Warehouse queries
  - Spark/PySpark for Lakehouse data
  - XMLA/DAX for Semantic Models
  - KQL for Real-Time Intelligence

### Consumers
- Use **MCP servers** for natural language queries
- Limited to: Semantic Models, Warehouses, Lakehouse SQL Endpoints
- No ODBC/JDBC setup needed - MCP handles connections

## Workloads

### Data Engineering
- **Lakehouse**: Delta tables, Spark, file management
  - Docs: https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-overview
  - Authoring skill: `skills/spark-authoring-cli/SKILL.md` — notebook authoring, Lakehouse authoring, Materialized Lake Views, and refresh-friendly Spark patterns.
- **Notebooks**: PySpark notebooks with mssparkutils
  - Docs: https://learn.microsoft.com/en-us/fabric/data-engineering/how-to-use-notebook
- **Spark Jobs**: Production Spark workloads
  - Docs: https://learn.microsoft.com/en-us/fabric/data-engineering/spark-job-definition
  - Operations skill: `skills/spark-operations-cli/SKILL.md` — read-only triage for failed jobs, stuck sessions, performance bottlenecks

### Data Warehouse
- **Warehouse**: T-SQL data warehouse
  - Docs: https://learn.microsoft.com/en-us/fabric/data-warehouse/data-warehousing
  - Note: Limited T-SQL surface area - check supported features
  - Authoring skill: `skills/sqldw-authoring-cli/SKILL.md` — DDL, DML, ingestion, schema changes
  - Consumption skill: `skills/sqldw-consumption-cli/SKILL.md` — read-only T-SQL queries
  - Operations skill: `skills/sqldw-operations-cli/SKILL.md` — performance diagnostics, slow queries, query insights

### Data Integration
- **Pipelines**: Orchestration and data movement
  - Docs: https://learn.microsoft.com/en-us/fabric/data-factory/data-factory-overview
- **Dataflows Gen2**: Low-code transformations with Power Query
  - Docs: https://learn.microsoft.com/en-us/fabric/data-factory/dataflows-gen2-overview
  - Authoring skill: `skills/dataflows-authoring-cli/SKILL.md` — dataflow lifecycle management, Power Query M mashup authoring
  - Consumption skill: `skills/dataflows-consumption-cli/SKILL.md` — read-only dataflow exploration, monitoring, status queries
  - Primary CLI tool: `az rest` via Fabric REST API

### Real-Time Intelligence
- **Eventstreams**: Real-time data ingestion
  - Docs: https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/overview
  - Authoring skill: `skills/eventstream-authoring-cli/SKILL.md` — create, configure, deploy Eventstream topologies (sources, operators, destinations)
  - Consumption skill: `skills/eventstream-consumption-cli/SKILL.md` — list, inspect, monitor Eventstreams
  - Primary CLI tool: `az rest` via Fabric REST API
- **Activator**: Alerts, notifications, and automated actions over Fabric data/events
  - Docs: https://learn.microsoft.com/en-us/fabric/real-time-intelligence/data-activator/activator-introduction
  - Authoring skill: `skills/activator-authoring-cli/SKILL.md` — create Activator items, sources, rules, conditions, and actions
  - Consumption skill: `skills/activator-consumption-cli/SKILL.md` — inspect Activator definitions, rules, sources, and actions
  - Primary CLI tool: `az rest` via Fabric REST API
- **KQL Database / Eventhouse**: Time-series queries with Kusto
  - Docs: https://learn.microsoft.com/en-us/fabric/real-time-intelligence/create-database
  - Authoring skill: `skills/eventhouse-authoring-cli/SKILL.md` — table management, ingestion, policies, materialized views
  - Consumption skill: `skills/eventhouse-consumption-cli/SKILL.md` — read-only KQL queries, schema discovery
  - Primary CLI tool: `az rest` via Kusto REST API (`/v1/rest/query` and `/v1/rest/mgmt`)
  - Token audience: `https://kusto.kusto.windows.net/.default`

### OneLake Catalog Search
- **Catalog Search API**: Cross-workspace item discovery
  - Docs: https://learn.microsoft.com/en-us/rest/api/fabric/core/catalog/search
  - Consumption skill: `skills/search-consumption-cli/SKILL.md` — find items by name, description, workspace name, or type
  - Primary CLI tool: `az rest` via `POST /v1/catalog/search`
  - Token audience: `https://api.fabric.microsoft.com/.default`

### Business Intelligence
- **Semantic Models**: DAX, XMLA, Power BI integration, TMDL
  - Docs: https://learn.microsoft.com/en-us/power-bi/connect-data/service-datasets-understand
  - Authoring skill: `skills/semantic-model-authoring/SKILL.md` — semantic model authoring
  - Consumption skill: `skills/semantic-model-consumption/SKILL.md` — raw DAX queries against semantic models via MCP ExecuteQuery tool
  - FabricIQ skill: `skills/fabriciq/SKILL.md` — multi-step Power BI data analysis (discover, inspect, resolve, generate, execute)
  - ⚠️ **MANDATORY**: Before calling any FabricIQ MCP tool, read `skills/fabriciq/SKILL.md` in full (see [`agents/FabricIQ.agent.md` § Pre-Flight](../agents/FabricIQ.agent.md#pre-flight--mandatory-skill-reading)).
- **Power BI Reports**: PBIR/PBIP report projects, visual design, Desktop validation, and Fabric report item management
  - Docs: https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-report
  - Skill docs: https://aka.ms/Report_Authoring_skill_LearnDocs
  - Planning skill: `skills/powerbi-report-planning/SKILL.md` — requirements, page plan, approval gate
  - Design skill: `skills/powerbi-report-design/SKILL.md` — archetype routing, layout, theme, accessibility
  - Authoring skill: `skills/powerbi-report-authoring/SKILL.md` — PBIR/PBIP file mechanics, Desktop reload/screenshot
  - Management skill: `skills/powerbi-report-management/SKILL.md` — Fabric report item CRUD via `az rest`

### Data Science
- **Data Agents**: Conversational AI over Fabric data sources
  - Docs: https://learn.microsoft.com/en-us/fabric/data-science/concept-data-agent
- **Data Agent Evaluation**: Testing and validating Data Agent accuracy
  - Docs: https://learn.microsoft.com/en-us/fabric/data-science/fabric-data-agent-sdk

## Best Practices

### Must
- Use Delta Lake format for Lakehouse tables
- Include time filters in KQL queries (`where Timestamp > ago(...)`)
- Use `has` over `contains` for indexed string search in KQL
- Use idempotent KQL commands (`.create-merge table`, `.create-or-alter function`)
- Handle credentials via environment variables or Key Vault
- Use parameterized notebooks and pipelines

### Prefer
- Medallion architecture (Bronze/Silver/Gold) for data organization
- REST APIs for programmatic management
- Incremental processing over full refreshes
- mssparkutils for Fabric-specific notebook operations

### Avoid
- Hardcoded workspace/item IDs
- SELECT * without LIMIT on large tables
- Long-running transactions in Warehouse
- Unbounded streaming queries

## Project Context & Resources — osare-cap-data Integration

This `skills-for-fabric` repository is cloned into the **osare-cap-data** project, which has comprehensive, production-tested Power BI agentic development documentation. Use these resources alongside the Microsoft Fabric skills for complete context:

### 1. Project Overview & Setup
**File:** [../README.md](../README.md)
- Project objectives, dependencies, and getting started guide
- Reference for environment setup and authentication configuration

### 2. Complete Power BI Agentic Development Briefing
**File:** [../src/powerbi/docs/agentic-powerbi-development-briefing.md](../src/powerbi/docs/agentic-powerbi-development-briefing.md)
- **Ecosystem map** — all MCP servers, CLIs, plugins, and their relationships (semantic model layer, report layer, CI/CD layer)
- **Version control** — `power-bi-agentic-development` marketplace v26.25 (stable; v26.26 breaking consolidation coming)
- **Anatsko product disambiguation** — `pbir-cli` (open-source) vs. Desktop MCP Server (commercial)
- **Microsoft official skills** — `powerbi-authoring` bundle (`planning`, `design`, `authoring`, `management`) with Desktop Bridge reload/screenshot validation
- **Data-goblin plugin ecosystem** — 6 plugins (tabular-editor, pbi-desktop, semantic-models, reports, pbip, fabric-cli) with deterministic hooks
- **Fabric Apps vs. native PBIR** — decision tree for custom web app dashboards (React/Vega-Lite)
- **Worked examples** — 8-phase MCP semantic model build, HTML wireframe → working Power BI report, dashboard optimization workflows
- **CI/CD, governance, theming, and custom visuals** — Deneb (Vega-Lite) over SVG DAX

### 3. Power BI Agent Tool Quick Reference (Cheat Sheet)
**File:** [../src/powerbi/docs/agent-tool-cheatsheet.md](../src/powerbi/docs/agent-tool-cheatsheet.md)
- **6-stage development cycle** — Plan & Design → Semantic Model → Report/PBIR → Bespoke Visuals → Validate → Deploy
- **Installed tools** — Power BI Modeling MCP v0.4.0, data-goblin plugins v26.25, pbir-cli, sequential-thinking MCP
- **Desktop Bridge integration** — reload/screenshot workflow for validation
- **pbir-cli gotchas** — UTF-8 BOM handling, mobile.json schema versions, parentGroupName orphans, visual auto-slug length limits
- **MCP known constraints** — calculated tables, isKey support, relationship best practices
- **Effort & model guidance** — when to use Opus-tier (judgment calls: wireframe interpretation, DAX diagnosis) vs. Sonnet 4.6 (mechanical: PBIR authoring, bulk edits)
- **Cost-reduction tactics** — MCP server management, targeted queries, prompt caching, conversation compacting

### Integration Pattern for This Project

When working on **Power BI report or semantic model tasks** in osare-cap-data:

1. **Start with the briefing** ([agentic-powerbi-development-briefing.md](../src/powerbi/docs/agentic-powerbi-development-briefing.md)) for ecosystem context and tool relationships.
2. **Use the cheat sheet** ([agent-tool-cheatsheet.md](../src/powerbi/docs/agent-tool-cheatsheet.md)) to pick the right tool for your task (MCP vs. pbir-cli vs. skill).
3. **Reference this file** (skills-for-fabric CLAUDE.md) for Fabric-specific workload operations (Warehouse, Lakehouse, Eventstreams, KQL) and REST API patterns.
4. **Cross-reference osare-cap-data/CLAUDE.md** — project-level rules for tool boundaries, MCP server setup, TMDL formatting, validation loops, and plugin usage decision trees.

### Key Distinctions (osare-cap-data context)

| Layer | Owner | Reference |
|---|---|---|
| PBIR report files (disk) | `pbir-cli` / `pbip` plugins | agent-tool-cheatsheet.md § Stage 3 |
| Semantic model (Desktop open) | `mcp__powerbi-modeling__*` (Modeling MCP v0.4.0) | agent-tool-cheatsheet.md § Stage 2; osare-cap-data/CLAUDE.md § MCP Server Setup |
| Semantic model (Desktop closed) | `te2-cli` (Tabular Editor 2) | agent-tool-cheatsheet.md § Stage 2; osare-cap-data/CLAUDE.md § Tool Boundaries |
| Fabric semantic model (Workspace / Fabric REST API) | `skills/semantic-model-authoring/SKILL.md` + `az rest` | skills-for-fabric CLAUDE.md § Business Intelligence |
| Embedded report (runtime, web app) | `powerbi-report-authoring` npm SDK | agent-tool-cheatsheet.md § REF C |
| Fabric workspace operations | `skills/fabric-*` + `az rest` | skills-for-fabric README.md § Install |

### When to Escalate

- **Cross-Fabric-Workload orchestration** → Use `agents/FabricDataEngineer.agent.md` from this skills-for-fabric repo
- **Power BI Desktop Bridge validation** → Use `pbi-desktop:connect-pbid` skill from osare-cap-data's installed data-goblin plugins
- **Live semantic model DAX/M changes** → Use Power BI Modeling MCP (osare-cap-data project) for fast iteration; publish to Fabric workspace via `powerbi-report-management` skill
- **Performance diagnostics** → Warehouse: use `skills/sqldw-operations-cli/SKILL.md`; Power BI: use DAX Studio + `skills/semantic-models:dax` skill
