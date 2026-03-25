# Dashboard Catalog

## 1. PPM Project

**Purpose**: The flagship project portfolio management dashboard, providing comprehensive project-level analytics for construction projects managed through Azure Databricks. This is the most complex dashboard in the suite with 1,186 DAX measures, featuring innovative SVG gauge generation, HTML table rendering with conditional formatting, and Unicode-based milestone gate tracking across 14+ project phases.

**Data Sources**:
- Azure Databricks SQL Warehouse (primary -- projects, tasks, milestones, costs)

**Complexity**: Very High (1,186 DAX measures)

**Top 5 Key DAX Measures**:

| Measure | Description |
|---------|-------------|
| `Gauge Table` | Generates a complete SVG speedometer gauge in DAX. Uses VAR-based SVG path construction with `<clipPath>` rotation for the fill arc and `<g transform='rotate(...)'>` for the needle. Maps 0-100% progress to 0-180 degrees of rotation. |
| `Table Proects` | Renders a full HTML table with embedded CSS styling. Uses conditional color variables (green/red) based on time, cost, and workload indicators. Includes `<head><style>` blocks and dynamic `<td>` content generation. |
| `Table Joao` | Constructs a milestone gate status display using 14 `UNICHAR()` variables mapping project phases (SPE incorporation, building permits, construction start, habite-se, etc.) to colored circle emojis -- green (128994), red (128992), yellow (128993), or gray (128998). |
| `% Conclusao` | Calculates portfolio-level completion rate as `DIVIDE([Etapas Concluidas], [Quantidade de Etapas], 0)`, providing a clean percentage of completed task stages across all projects. |
| `Project Percent Completition` | Aggregates project completion using `CALCULATE(SUM(Projects[ProjectPercentCompleted]))` from the Databricks-sourced Projects table, enabling drill-down by any dimension in the model. |

**Special Techniques**:
- SVG gauge generation entirely in DAX (no custom visuals)
- Full HTML page construction with CSS in DAX measures
- Unicode emoji-based status indicators via `UNICHAR()`
- Databricks SQL Warehouse as the data source (modern lakehouse pattern)

---

## 2. Portfolio Obra (Construction Projects) Fisico Financeiro

**Purpose**: Physical-financial progress tracking dashboard that overlays construction completion percentages from MS Project Online against budget consumption from the UAU ERP system. Enables portfolio managers to detect projects where financial spending is outpacing (or lagging behind) physical construction progress, using cumulative S-curve analysis and monthly variance tracking.

**Data Sources**:
- MS Project Online (PWA) via OData (`_api/ProjectData`) -- project completion, tasks, baselines
- SQL Server (ERP, `uau` database) -- disbursement control, accounts payable/paid, budget limits

**Complexity**: Medium (58 DAX measures)

**Top 5 Key DAX Measures**:

| Measure | Description |
|---------|-------------|
| `Variacao (Fisico Financeiro)` | Computes physical-financial variance: `DIVIDE((Physical% - Financial%), Physical%)`. Flags projects where budget consumption diverges from construction progress. |
| `Soma R$ Acumulado Realizado` | Cumulative realized expenditure using `CALCULATE` with `FILTER(ALL('dCalendario'))` to build running totals up to the reference date. Produces S-curve charts. |
| `% Desembolso Consumido` | Budget consumption percentage: `DIVIDE([Soma R$ Compromisso Pagar/Pago], [Soma R$ ControleDesembolso])`. Core metric for financial health. |
| `Desvio Financeiro` | Financial deviation: difference between consumed disbursement and projected disbursement, surfacing budget execution gaps. |
| `Saldo` | Budget balance: `[Soma R$ ControleDesembolso] - [Soma R$ Compromisso Pagar/Pago]`. Shows remaining budget per project. |

**Special Techniques**:
- Physical-financial variance analysis combining two independent data sources (PWA + ERP)
- Cumulative S-curve generation using `FILTER(ALL(...))` running totals
- Month-over-month comparison with `MONTH(TODAY())-1` logic including year boundary handling
- Dual-source data model joining OData project data with SQL Server financial data

---

## 3. Briefing One Page

**Purpose**: Executive-ready one-page project briefings that consolidate schedule indicators, contract values, budget execution, and risk counts into a concise format for each project. Designed for leadership meetings where portfolio managers present project-by-project status in a standardized, scannable layout. Integrates the deepest SQL Server queries in the suite, pulling contract details, approved budgets, and disbursement controls.

**Data Sources**:
- MS Project Online (PWA) via OData -- project status, tasks, baselines, risks
- SQL Server (ERP) -- contracts, approved budgets, disbursement control, budget limits
- SharePoint Lists -- DIM_Obras reference data, governance metadata

**Complexity**: Medium (51 DAX measures)

**Top 5 Key DAX Measures**:

| Measure | Description |
|---------|-------------|
| `Sum Acum Pgto` | Cumulative payment tracking: `CALCULATE([Sum Des Pg total mes], FILTER(ALL('dCalendario'), ...))`. Aggregates paid + payable + financial control into a running total. |
| `Sum Des Pg total mes` | Total monthly disbursement: sum of payable (`Sum Des Pagar`), paid (`Sum Des Pago`), and financial control (`Sum Des Cont Fin`). Core monthly cash flow metric. |
| `Saldo do Projeto` | Project balance: baseline cost minus (paid + payable from UAU ERP). Shows remaining budget from the original project baseline. |
| `Cont Riscos` | Risk count per project using `COUNTAX` with null-safe handling via `IF(ISBLANK(...), 0, ...)`. |
| `Qtd de Projeto` | Project count with robust null handling: uses `CALCULATE(COUNT(...), FILTER(ALLSELECTED(...), LEN(...) > 0))` to exclude blank project IDs. |

**Special Techniques**:
- Complex SQL Server queries with multiple subqueries for contract measurement values, discount amounts, and approval chains
- OData parameter-based connection (`PWAURL` parameter) enabling environment switching
- Schedule indicator mapping (Em dia/Atrasado/Concluido) to numeric codes for icon rendering
- SharePoint integration for reference dimension data

---

## 4. Governanca (Governance) e Sistema

**Purpose**: Governance and systems oversight dashboard tracking project status indicators, risk registers, and compliance data from MS Project Online and SharePoint. Provides the governance layer for the PPM suite, ensuring projects are publishing schedules on time, risks are being actively managed, and project indicators (on-time, at-risk, delayed) are visible to management. The simplest dashboard in the suite but essential for portfolio governance discipline.

**Data Sources**:
- MS Project Online (PWA) via OData -- projects, tasks, baselines, risks, issues
- SharePoint Lists -- DIM_Obras, governance checklists, project site data

**Complexity**: Low-Medium (43 DAX measures)

**Top 5 Key DAX Measures**:

| Measure | Description |
|---------|-------------|
| `Filtro por GP` | Row-Level Security filter using `USERNAME()` to restrict dashboard visibility to the logged-in project manager. Each manager sees only their assigned projects. |
| `Count % equal Projeto` | Calculates the proportion of tasks belonging to construction project categories, excluding non-project categories (Comercial, Legalizacao, Licenca, Rotina). Uses `COUNTA` with multi-condition filters. |
| `Count % <> Projeto` | Complement measure: proportion of tasks in non-project categories. Together with the above, provides a complete task distribution breakdown. |
| `Cont Riscos` | Risk count using `COUNTAX(Riscos, Riscos[ID do Risco])` with null-safe `IF(ISBLANK(...), 0, ...)` pattern. |
| `aux_percentual int` | Average task completion percentage used as an intermediate calculation for weighted progress indicators. |

**Special Techniques**:
- Row-Level Security (RLS) via `USERNAME()` for per-manager data isolation
- OData parameter pattern (`PWAURL`) for environment portability
- Schedule indicator to numeric code mapping for icon-based status rendering in Power Query
- Task categorization logic using the `ProjectPortfolio` column to classify tasks into project vs. non-project buckets
- Risk link generation combining SharePoint URLs with risk IDs for drill-through to PWA
