# Business Case: Project Portfolio Management Suite

## Industry Context

Brazilian real estate development and construction. The organization manages dozens of active construction projects simultaneously, each with its own schedule, budget, contracts, and risk profile. Portfolio managers need consolidated visibility across all projects to make timely decisions.

## Problem Statement

Portfolio managers lacked a unified, real-time view of project health across the entire portfolio. Key challenges included:

- **Schedule fragmentation**: Project schedules lived in MS Project Online (PWA) with no consolidated dashboard view across projects
- **Financial opacity**: Disbursement data resided in the UAU ERP system (SQL Server), disconnected from schedule progress
- **Physical-financial misalignment**: No easy way to compare physical progress (% complete from PWA) with financial progress (% budget consumed from ERP)
- **Executive reporting burden**: Preparing one-page briefings and governance reports required manual data compilation from multiple systems
- **Risk blind spots**: Project risks tracked in PWA/SharePoint were not surfaced alongside schedule and cost data

## Solution

A suite of 4 Power BI dashboards that combine MS Project Online (PWA) schedule data with Azure Databricks analytics and SQL Server ERP data:

1. **PPM Project** -- The flagship dashboard with 1,186 DAX measures, providing deep project-level analytics including SVG gauge visualizations, HTML status tables, and milestone tracking. Powered by Azure Databricks SQL Warehouse.

2. **Portfolio Obra (Construction Projects) Fisico Financeiro** -- Physical-financial progress tracking that overlays construction completion percentages against budget consumption, enabling variance detection at the portfolio level.

3. **Briefing One Page** -- Executive-ready one-page summaries per project, combining schedule indicators, contract values, budget execution, and key risk counts into a concise format.

4. **Governanca (Governance) e Sistema** -- Governance and systems oversight dashboard tracking project status, risk registers, and SharePoint-based reference data for compliance monitoring.

## Value Delivered

- **Real-time portfolio health monitoring**: Automated data refresh from PWA, Databricks, and SQL Server provides always-current status across all projects
- **Physical-financial progress tracking**: Side-by-side comparison of schedule progress vs. budget consumption reveals projects that are over-spending relative to completion
- **Executive briefings on demand**: One-page briefing dashboard eliminates hours of manual report preparation
- **Risk visibility**: Consolidated risk counts and status indicators from PWA risk registers surface potential issues early
- **Milestone gate tracking**: Unicode-based status indicators (green/yellow/red circles) provide at-a-glance gate status for 14+ project milestones

## Technology Stack

| Component | Role |
|-----------|------|
| **Power BI** | Dashboard platform, DAX calculations, data visualization |
| **Azure Databricks** | Primary data source for PPM Project dashboard (lakehouse architecture, SQL Warehouse endpoint) |
| **MS Project Online (PWA)** | Project schedules, tasks, baselines, risks via OData API (`_api/ProjectData`) |
| **SQL Server (ERP)** | Financial data -- disbursements, contracts, budgets, accounts payable/paid |
| **SharePoint** | Reference data lists, project site metadata, governance documents |

## Architecture Evolution

The dashboards show a clear evolution in data architecture:

- **Early dashboards** (Governanca, Briefing, Portfolio) connect directly to MS Project Online via OData and to SQL Server for ERP data. This represents the initial "direct connect" pattern common in Power BI deployments.

- **Later dashboard** (PPM Project) migrated to Azure Databricks as the primary data source, using `Databricks.Query()` with a SQL Warehouse endpoint. This represents the modern lakehouse pattern where raw data is ingested into Databricks, transformed, and served to Power BI through an optimized SQL interface.

This evolution demonstrates a deliberate architectural maturation from point-to-point connections toward a centralized analytics platform.
