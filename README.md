# Power BI Project Portfolio Management Suite

**4 dashboards for project portfolio tracking using Azure Databricks and MS Project Online**

Author: **Vitor Rodovalho**

---

## Overview

This repository contains the DAX measures, Power Query (M) transformations, data model schemas, and relationship definitions extracted from a suite of 4 Power BI dashboards built for Project Portfolio Management (PPM) in the Brazilian real estate development and construction industry.

### Key Highlights

- **Azure Databricks** as the primary data source for the flagship dashboard, implementing a modern lakehouse architecture with SQL Warehouse endpoints
- **MS Project Online (PWA)** integration via OData for schedule tracking, task management, and baseline comparisons across all portfolio dashboards
- **SVG gauge generation directly in DAX** -- an innovative technique that constructs full SVG markup with dynamic needle rotation using VAR-based path construction, eliminating the need for custom visuals
- **1,338 total DAX measures** across 4 dashboards, demonstrating advanced Power BI development patterns
- **PPM Project** alone contains **1,186 DAX measures**, including HTML table rendering and Unicode-based status indicators

## Dashboard Inventory

| # | Dashboard | DAX Measures | Primary Data Source | Key Technique |
|---|-----------|-------------|---------------------|---------------|
| 1 | PPM Project | 1,186 | Azure Databricks SQL Warehouse | SVG gauge, HTML tables, Unicode status icons |
| 2 | Portfolio Obra (Construction Projects) Fisico Financeiro | 58 | MS Project Online (PWA/OData) + SQL Server (ERP) | Physical-financial variance, cumulative cost tracking |
| 3 | Briefing One Page | 51 | MS Project Online (PWA/OData) + SQL Server (ERP) | Executive one-page briefing, contract analysis |
| 4 | Governanca (Governance) e Sistema | 43 | MS Project Online (PWA/OData) + SharePoint | Governance tracking, risk management |

## Repository Structure

```
powerbi-project-portfolio-management/
├── README.md
├── LICENSE
├── ANONYMIZATION_RULES.md
├── .gitignore
├── docs/
│   ├── architecture.md          # Data architecture and connection patterns
│   ├── business-case.md         # Business context and value proposition
│   └── dashboard-catalog.md     # Detailed catalog of all 4 dashboards
├── dax/
│   ├── ppm-project/             # PPM Project (1,186 DAX measures)
│   │   ├── measures.md
│   │   └── columns.csv
│   ├── portfolio-fisico-financeiro/  # Portfolio Obra Fisico Financeiro (58 DAX)
│   │   ├── measures.md
│   │   └── columns.csv
│   ├── briefing-one-page/       # Briefing One Page (51 DAX)
│   │   ├── measures.md
│   │   └── columns.csv
│   └── governanca-sistema/      # Governanca e Sistema (43 DAX)
│       ├── measures.md
│       └── columns.csv
├── power-query/
│   ├── etl-patterns/
│   ├── connections/
│   ├── ppm-project.md
│   ├── portfolio-fisico-financeiro.md
│   ├── briefing-one-page.md
│   └── governanca-sistema.md
├── data-model/
│   ├── schema/
│   │   ├── ppm-project.csv
│   │   ├── portfolio-fisico-financeiro.csv
│   │   ├── briefing-one-page.csv
│   │   └── governanca-sistema.csv
│   └── relationships/
│       ├── ppm-project.csv
│       ├── portfolio-fisico-financeiro.csv
│       ├── briefing-one-page.csv
│       └── governanca-sistema.csv
└── assets/
    └── screenshots/
```

## How to Use

1. **Explore DAX measures**: Browse `dax/{dashboard}/measures.md` for the full catalog of DAX expressions per dashboard. Each file is a markdown table with table name, measure name, and full expression.

2. **Review Power Query logic**: The `power-query/` directory contains the M code for all data source connections and ETL transformations, including OData feeds, Databricks queries, and SQL Server connections.

3. **Understand the data model**: The `data-model/schema/` and `data-model/relationships/` directories contain CSV exports of table schemas and relationship definitions.

4. **Read the documentation**: The `docs/` directory provides architectural context, the business case, and a detailed catalog of each dashboard.

## Anonymization

All sensitive information (company names, IP addresses, cloud endpoints, personal identifiers, and connection strings) has been replaced with generic placeholders. See [ANONYMIZATION_RULES.md](ANONYMIZATION_RULES.md) for the full list of substitutions and validation methodology.

> **Note:** Original DAX and Power Query (M) code preserves Portuguese identifiers as-is from the source Power BI models.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
