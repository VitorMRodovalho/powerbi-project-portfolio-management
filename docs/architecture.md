# Data Architecture

## System Overview

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         DATA SOURCES                                         │
│                                                                              │
│  ┌─────────────────────┐  ┌──────────────────┐  ┌────────────────────────┐  │
│  │  Azure Databricks   │  │  MS Project      │  │  SQL Server            │  │
│  │  SQL Warehouse      │  │  Online (PWA)    │  │  (ERP)                 │  │
│  │                     │  │                  │  │                        │  │
│  │  Databricks.Query() │  │  OData API       │  │  Sql.Database()        │  │
│  │  /sql/1.0/          │  │  _api/           │  │  Direct SQL queries    │  │
│  │  warehouses/...     │  │  ProjectData     │  │  to "uau" database     │  │
│  └────────┬────────────┘  └────────┬─────────┘  └───────────┬────────────┘  │
│           │                        │                        │               │
│  ┌────────┴────────────────────────┴────────────────────────┴────────────┐  │
│  │                                                                       │  │
│  │  ┌──────────────────┐                                                 │  │
│  │  │  SharePoint      │                                                 │  │
│  │  │  Lists & Sites   │                                                 │  │
│  │  │                  │                                                 │  │
│  │  │  SharePoint.     │                                                 │  │
│  │  │  Tables()        │                                                 │  │
│  │  └────────┬─────────┘                                                 │  │
│  │           │                                                           │  │
│  └───────────┤───────────────────────────────────────────────────────────┘  │
│              │                                                              │
└──────────────┼──────────────────────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         POWER BI DASHBOARDS                                  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  PPM Project (1,186 measures)                                         │  │
│  │  Source: Azure Databricks SQL Warehouse                               │  │
│  │  Features: SVG gauges, HTML tables, Unicode status icons              │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  Portfolio Obra Fisico Financeiro (58 measures)                        │  │
│  │  Sources: PWA OData + SQL Server (ERP)                            │  │
│  │  Features: Physical-financial variance, cumulative cost S-curves      │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  Briefing One Page (51 measures)                                      │  │
│  │  Sources: PWA OData + SQL Server (ERP) + SharePoint               │  │
│  │  Features: Executive briefing, contract tracking, budget execution    │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  Governanca e Sistema (43 measures)                                   │  │
│  │  Sources: PWA OData + SharePoint                                      │  │
│  │  Features: Governance compliance, risk register, project indicators   │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

## Data Source Connection Patterns

### 1. Azure Databricks (PPM Project Dashboard)

The flagship PPM Project dashboard connects to Azure Databricks via the `Databricks.Query()` Power Query function, targeting a SQL Warehouse endpoint:

```
Databricks.Query(
    "adb-example.azuredatabricks.net",
    "/sql/1.0/warehouses/example-id",
    []
)
```

This pattern enables:
- Pushdown of SQL queries to the Databricks SQL Warehouse for optimized performance
- Access to the full lakehouse data model (Delta tables)
- Centralized data governance through Databricks Unity Catalog

The dashboard queries multiple Databricks tables/views covering projects, tasks, milestones, costs, and schedule data that have been pre-processed and joined in the lakehouse layer.

### 2. MS Project Online / PWA via OData (Portfolio, Briefing, Governanca)

Three dashboards connect to MS Project Online (Project Web App) through the OData REST API:

```
OData.Feed(PWAURL & "/_api/ProjectData")
```

Where `PWAURL` is a Power Query parameter pointing to the PWA site. Common OData entities consumed:

| Entity | Portuguese Name | Purpose |
|--------|----------------|---------|
| Projects | Projetos | Project-level metadata, completion %, schedule indicators |
| Tasks | Tarefas | Task-level detail, percent complete, schedule status |
| TaskBaselines | LinhasdeBasedeTarefa | Baseline schedule for variance analysis |
| ProjectBaselines | LinhasdeBasedeProjeto | Baseline cost for budget comparisons |
| Risks | Riscos | Risk register entries per project |
| Issues | Problemas | Issue tracking per project |

Some queries apply OData filters for efficiency:
```
"_api/ProjectData/Tarefas?$filter=TarefaEstáAtiva eq true"
```

### 3. SQL Server -- ERP (Briefing, Portfolio)

The Briefing and Portfolio dashboards connect directly to a SQL Server instance hosting the ERP database for financial data:

```
Sql.Database("db-server.example.com", "uau", [Query="..."])
```

Key ERP tables queried:
- **cntdesembolso** -- Monthly disbursement control (budget vs. actual)
- **Contratos** -- Contract details (vendor, value, measurement)
- **OrcamentoAprov** -- Approved budget versions
- **contaspagas** -- Accounts paid
- **pagamentossi** -- Payment projections
- **ControleValorLimitePlanejado** -- Budget limit control

### 4. SharePoint (Governanca, Briefing)

SharePoint Lists provide reference data and supplemental information:

```
SharePoint.Tables("sharepoint.example.com/sites/pwa", [ApiVersion = 15])
```

Used for: project reference data, governance checklists, DIM_Obras (construction reference dimension), and risk link generation.

## Key DAX Techniques

### SVG Gauge Generation in DAX

The PPM Project dashboard generates SVG gauge (speedometer) visualizations entirely within DAX, using the HTML Content visual. The technique works as follows:

1. **VAR declarations** define the gauge parameters:
   - `var_value_needle` -- progress value (0-100) from the data model
   - `var_bar_base_color`, `var_bar_fill_color`, `var_needle_fill_color` -- configurable colors

2. **SVG path construction** builds a half-circle gauge with:
   - A `<clipPath>` element that rotates based on the fill value (`(var_value_color_fill/100)*180` degrees)
   - An arc path (`d='M204.71,0C91.65...'`) defining the gauge track
   - A needle group (`<g transform='rotate(...)'>`) that rotates based on progress
   - A text element displaying the percentage value

3. **Dynamic rotation** uses the formula `(value/100)*180` to map 0-100% to 0-180 degrees of rotation, creating the visual effect of a moving needle on a semicircular gauge.

This approach eliminates the need for custom Power BI visuals or external SVG libraries.

### HTML Table Rendering with Status Indicators

The `Table Proects` measure constructs a full HTML table in DAX, with:
- Conditional coloring (`var_time`, `var_costs`, `var_workload` mapped to green/red)
- Dynamic content injection via VAR concatenation
- Embedded within an `<html><head><style>...</style></head><body>` structure

### Unicode Status Icons for Milestone Gates

The `Table Joao` measure uses `UNICHAR()` to render colored circle indicators:
- `UNICHAR(128994)` -- green circle (on track)
- `UNICHAR(128992)` -- red circle (delayed)
- `UNICHAR(128993)` -- yellow circle (at risk)
- `UNICHAR(128998)` -- gray circle (no baseline)

This maps 14+ project milestones (SPE incorporation, building permits, construction start, etc.) to a single visual status row.

### Physical-Financial Variance Tracking

The Portfolio dashboard computes physical-financial variance using:
```dax
DIVIDE(
    (AVERAGE('PPM Projetos'[PorcentagemConcluídadoProjeto])
     - DIVIDE([Soma R$ Compromisso Pagar/Pago], [Soma R$ ControleDesembolso])),
    AVERAGE('PPM Projetos'[PorcentagemConcluídadoProjeto])
)
```
This compares physical progress (% complete from PWA) against financial progress (% budget consumed from ERP) to flag projects where spending outpaces construction.

### Cumulative Cost Tracking with ALL/FILTER

Cumulative (running total) measures use the `CALCULATE` + `FILTER(ALL(...))` pattern:
```dax
CALCULATE(
    [Soma R$ Compromisso Pagar/Pago],
    FILTER(ALL('dCalendario'), 'dCalendario'[DATA] <= MAX('dCalendario'[DATA]))
)
```
This produces S-curve charts showing cumulative disbursement over time.

### Month-over-Month Variance

Previous month comparisons use date-based FILTER expressions with `MONTH(TODAY())-1` logic to compute prior-period values for trend analysis:
```dax
CALCULATE(
    [Sum],
    FILTER('dCalendario',
        MONTH('dCalendario'[DATA]) = MONTH(TODAY()) - 1),
    FILTER('dCalendario',
        YEAR('dCalendario'[DATA]) = IF(MONTH(TODAY()) = 1, YEAR(TODAY()) - 1, YEAR(TODAY())))
)
```
