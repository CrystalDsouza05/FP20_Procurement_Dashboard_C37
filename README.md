# 🏭 Crystal Procurement Dashboard
### FP20 Analytics × ZoomCharts — Challenge 37

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?style=flat&logo=powerbi&logoColor=black)
![ZoomCharts](https://img.shields.io/badge/ZoomCharts-Visuals-FFD700?style=flat)
![DAX](https://img.shields.io/badge/DAX-65%20Measures-blue?style=flat)
![Theme](https://img.shields.io/badge/Theme-Dark%20%7C%20Gold-1a1a1a?style=flat)

---

## 📌 Overview

Crystal Procurement Dashboard is a 2-page Power BI report built for the FP20 Analytics Challenge 37 in collaboration with ZoomCharts. The report is designed to give procurement teams and executives a comprehensive view of organizational spending, supplier performance, delivery reliability, contract compliance, and invoice health — all in one place.

The dashboard follows a dark theme with gold/yellow gradient accents, giving it a clean and professional look suited for executive-level reporting. All charts are built using ZoomCharts custom visuals, which support drill-down, zoom, and rich interactivity natively within Power BI.

---

## 📊 Pages

### Page 1 — Procurement Spend Overview

This page answers the core question every procurement leader asks first: **where is the organization's money going, and how does it compare to what was planned?**

#### KPI Cards (Interactive Selectors)
The top section features three clickable KPI cards — **Total Spend**, **Total Savings**, and **Total POs**. These cards are not just display elements; they act as field parameter selectors for the entire page. Clicking a card filters all charts below to reflect that specific KPI. The selected card highlights with a gold accent border, giving the user clear visual feedback on the active selection. Each card also shows two reference labels beneath the main value — for example, Total Spend shows year-over-year change and budget variance percentage, while Total Savings shows the savings rate and YoY savings movement.

#### Total Spend Trend (ZoomCharts Bar + Line Combo)
A ZoomCharts bar and line combination chart showing monthly spend across the selected year, with a secondary line representing the prior year's spend for comparison. An average reference line runs across the chart to give a sense of typical monthly spend. This chart helps users understand seasonality, identify peak spending months, and compare current performance against the same period last year. The chart title updates dynamically based on the active KPI card selection and the selected year.

#### Total Spend Analysis by Dimensions (ZoomCharts Bar Chart)
A horizontal bar chart that breaks down the selected KPI across a user-chosen dimension. The dimension is controlled by a separate field parameter slicer labeled **Select Dimensions**, which includes Supplier Risk, Supplier Region, Department, Supplier Name, Contract Type, Category, and Sub Category. Clicking a bar drills down into that dimension, allowing users to progressively explore spend concentration without leaving the chart. This visual gives the user maximum flexibility to explore the data from any angle they choose.

#### Supplier Snapshot (Matrix Table)
A detailed matrix table in the bottom right of the page showing supplier-level data across multiple metrics. Columns include Supplier Region, Risk, Department, Contract Type, Total Spend, Budget Variance %, OTD Rate, and Invoice Health Score. Conditional formatting is applied to OTD Rate and Invoice Health Score columns to immediately flag underperforming suppliers. The table is expandable by region and department, making it easy to find exactly where performance issues are concentrated.

#### Slicers
Year and Quarter slicers sit at the top right of the page as tile-style buttons, allowing the user to filter the entire page to a specific time period.

---

### Page 2 — Risk & Compliance

This page answers the operational question: **where are the risks hiding in our supply chain and procurement process?** It combines delivery performance, maverick spend, lead time analysis, and contract compliance into a single view.

#### KPI Cards
Four KPI cards sit across the top — **OTD Rate**, **Maverick Spend %**, **Invoice Health Score**, and **Avg Lead Time Days**. Each card shows a primary metric alongside two supporting reference labels. OTD Rate shows year-over-year change and average days late. Maverick Spend % shows the contract compliance rate alongside its YoY trend. Invoice Health Score is accompanied by the overdue invoice rate and the at-risk invoice spend expressed as both a dollar amount and a percentage. Avg Lead Time Days shows the split between on-time and late lead times side by side.

#### Delay Drivers Across Departments, Categories and Sub-Categories (ZoomCharts Bar Chart)
A bar chart showing the count of late deliveries broken down by department, with a line overlay representing average days late per department. The chart supports three levels of drill-down: Department → Category → Sub-Category. This lets users pinpoint exactly which part of the business is generating the most delivery failures, and whether those failures are concentrated in a few large problem areas or spread across many smaller ones. Hovering over bars reveals additional delivery context.

#### Lead Time by Contract Type & Risk (ZoomCharts Bar Chart)
A grouped bar chart showing average lead time days across the four contract types — Spot, Framework, Long-term, and Master Supply. Three data series are plotted per contract type representing Low, Medium, and High supplier risk levels, shown as dot markers overlaid on the bars. This visual helps answer whether certain contract structures are associated with longer or more unpredictable lead times, and whether high-risk suppliers consistently take longer to deliver regardless of contract type.

#### Maverick vs Contracted Spend (ZoomCharts Stacked Bar Chart)
A horizontal stacked bar chart comparing maverick spend against contracted spend across the selected dimension. Like the dimension chart on Page 1, this visual is controlled by a **Select Dimension** field parameter slicer, supporting Supplier Risk, Supplier Region, Department, Supplier Name, Contract Type, Category, and Sub Category. Maverick spend (off-contract purchases) is shown in a contrasting color against contracted spend, making it immediately clear which segments of the business have the highest compliance risk. The chart supports drill-down so users can follow the maverick spend from a broad dimension down to specific suppliers or categories.

#### Slicers
Year and Quarter slicers are synced with Page 1, so any time period selection carries across both pages automatically.

---

## 🗃️ Data Model

The report is built on a **star schema** with one central fact table and four dimension tables.

```
Dim_Date        ──(active: PO Date)──────────────► Fact_PurchaseOrders
                ──(inactive: Requested Delivery)──► Fact_PurchaseOrders
                ──(inactive: Actual Delivery)──────► Fact_PurchaseOrders

Dim_Supplier    ──(active: Supplier ID)────────────► Fact_PurchaseOrders
Dim_Item        ──(active: Item Code)──────────────► Fact_PurchaseOrders
Dim_CostCentre  ──(active: Cost Centre)────────────► Fact_PurchaseOrders
```

All active relationships are Many-to-One from Fact to Dim with single-direction cross-filtering. The two inactive date relationships (Requested Delivery and Actual Delivery) are available for use via `USERELATIONSHIP()` in DAX when delivery date analysis is needed.

### Tables

| Table | Role | Key Column |
|---|---|---|
| `Fact_PurchaseOrders` | Central fact table containing all PO transactions | PO Number |
| `Dim_Supplier` | Supplier attributes including region, tier, risk, and geo coordinates | Supplier ID |
| `Dim_Item` | Item and category hierarchy | Item Code |
| `Dim_CostCentre` | Department and cost centre mapping | Cost Centre |
| `Dim_Date` | Full date table with calendar and fiscal year columns | Date |
| `FP_Measure` | Field parameter table for KPI card selection on Page 1 | — |
| `FP_Dimension` | Field parameter table for dimension switching on both pages | — |

---

## 📐 DAX Measures

All measures are organized into display folders within a single `Measure` table. Click each section to expand.

<details>
<summary><strong>💰 Spend & Budget</strong> — 15 measures</summary>

| Measure | Expression |
|---|---|
| Total Spend | `SUM(Fact_PurchaseOrders[Line Total Inc Tax])` |
| Total Spend Net | `SUM(Fact_PurchaseOrders[Line Net])` |
| Total POs | `DISTINCTCOUNT(Fact_PurchaseOrders[PO Number])` |
| Avg PO Value | `DIVIDE([Total Spend], [Total POs])` |
| Total Budget | `SUM(Fact_PurchaseOrders[Budget Total])` |
| Budget Variance | `[Total Spend Net] - [Total Budget]` |
| Budget Variance % | `DIVIDE([Budget Variance], [Total Budget])` |
| Budget Utilization % | `DIVIDE([Total Spend Net], [Total Budget])` |
| Total Savings | `SUM(Fact_PurchaseOrders[Savings Amount])` |
| Total Spend YTD | `TOTALYTD([Total Spend], Dim_Date[Date])` |
| Total Spend PYTD | `CALCULATE([Total Spend YTD], SAMEPERIODLASTYEAR(Dim_Date[Date]))` |
| Total Spend YoY Change | `[Total Spend YTD] - [Total Spend PYTD]` |
| Total Spend YoY % | `DIVIDE([Total Spend YoY Change], [Total Spend PYTD])` |
| Total Spend PY | `CALCULATE([Total Spend], SAMEPERIODLASTYEAR(Dim_Date[Date]))` |
| Budget YTD | `TOTALYTD([Total Budget], Dim_Date[Date])` |

</details>

<details>
<summary><strong>💸 Savings</strong> — 6 measures</summary>

| Measure | Expression |
|---|---|
| Total Savings Amount | `[Total Savings]` |
| Savings Rate % | `DIVIDE([Total Savings Amount], SUM(Fact_PurchaseOrders[Budget Total]))` |
| Savings YTD | `TOTALYTD([Total Savings Amount], Dim_Date[Date])` |
| Savings PYTD | `CALCULATE([Savings YTD], SAMEPERIODLASTYEAR(Dim_Date[Date]))` |
| Savings YoY Change | `[Savings YTD] - [Savings PYTD]` |
| Savings YoY % | `DIVIDE([Savings YoY Change], [Savings PYTD])` |

</details>

<details>
<summary><strong>🚚 Delivery & Lead Time</strong> — 12 measures</summary>

| Measure | Expression |
|---|---|
| Total Deliveries | `COUNTROWS(FILTER(Fact_PurchaseOrders, NOT(ISBLANK(Fact_PurchaseOrders[Actual Delivery]))))` |
| On Time Deliveries | `CALCULATE(COUNTROWS(Fact_PurchaseOrders), Fact_PurchaseOrders[On Time Delivery] = "Yes")` |
| Late Deliveries | `CALCULATE(COUNTROWS(Fact_PurchaseOrders), Fact_PurchaseOrders[On Time Delivery] = "No")` |
| OTD Rate | `DIVIDE([On Time Deliveries], [Total Deliveries])` |
| Late Delivery Rate | `DIVIDE([Late Deliveries], [Total Deliveries])` |
| Avg Lead Time Days | `AVERAGE(Fact_PurchaseOrders[Lead Time Days])` |
| Avg Days Late | `CALCULATE(AVERAGE(Fact_PurchaseOrders[Days Late]), Fact_PurchaseOrders[Days Late] > 0)` |
| Avg Lead Time On Time | `CALCULATE(AVERAGE(Fact_PurchaseOrders[Lead Time Days]), Fact_PurchaseOrders[On Time Delivery] = "Yes")` |
| Avg Lead Time Late | `CALCULATE(AVERAGE(Fact_PurchaseOrders[Lead Time Days]), Fact_PurchaseOrders[On Time Delivery] = "No")` |
| Spend on Late Deliveries | `CALCULATE([Total Spend], Fact_PurchaseOrders[On Time Delivery] = "No")` |
| OTD Rate PYTD | `CALCULATE([OTD Rate], SAMEPERIODLASTYEAR(Dim_Date[Date]))` |
| OTD Rate YoY Change | `[OTD Rate] - [OTD Rate PYTD]` |

</details>

<details>
<summary><strong>⚠️ Maverick & Contract Compliance</strong> — 10 measures</summary>

| Measure | Expression |
|---|---|
| Total Maverick Spend | `CALCULATE([Total Spend], Fact_PurchaseOrders[Contract ID] = "NO-CONTRACT")` |
| Total Contracted Spend | `CALCULATE([Total Spend], Fact_PurchaseOrders[Contract ID] <> "NO-CONTRACT")` |
| Maverick Spend % | `DIVIDE([Total Maverick Spend], [Total Spend])` |
| Contract Compliance Rate | `DIVIDE([Total Contracted Spend], [Total Spend])` |
| Maverick PO Count | `CALCULATE([Total POs], Fact_PurchaseOrders[Contract ID] = "NO-CONTRACT")` |
| Contracted PO Count | `CALCULATE([Total POs], Fact_PurchaseOrders[Contract ID] <> "NO-CONTRACT")` |
| Avg Maverick PO Value | `DIVIDE([Total Maverick Spend], [Maverick PO Count])` |
| Maverick Spend YTD | `TOTALYTD([Total Maverick Spend], Dim_Date[Date])` |
| Maverick Spend PYTD | `CALCULATE([Maverick Spend YTD], SAMEPERIODLASTYEAR(Dim_Date[Date]))` |
| Maverick Spend YoY % | `DIVIDE([Maverick Spend YTD] - [Maverick Spend PYTD], [Maverick Spend PYTD])` |

</details>

<details>
<summary><strong>🧾 Invoice Health</strong> — 8 measures</summary>

| Measure | Expression |
|---|---|
| Total Invoices | `COUNTROWS(FILTER(Fact_PurchaseOrders, NOT(ISBLANK(Fact_PurchaseOrders[Invoice Status]))))` |
| Paid Invoices | `CALCULATE(COUNTROWS(Fact_PurchaseOrders), Fact_PurchaseOrders[Invoice Status] = "Paid")` |
| Overdue Invoices | `CALCULATE(COUNTROWS(Fact_PurchaseOrders), Fact_PurchaseOrders[Invoice Status] = "Overdue")` |
| Disputed Invoices | `CALCULATE(COUNTROWS(Fact_PurchaseOrders), Fact_PurchaseOrders[Invoice Status] = "Disputed")` |
| Overdue Invoice Rate | `DIVIDE([Overdue Invoices], [Total Invoices])` |
| At Risk Invoice Spend | `CALCULATE([Total Spend], Fact_PurchaseOrders[Invoice Status] IN {"Overdue", "Disputed"})` |
| At Risk Invoice Spend % | `DIVIDE([At Risk Invoice Spend], [Total Spend])` |
| Invoice Health Score | `DIVIDE([Paid Invoices], [Paid Invoices] + [Overdue Invoices] + [Disputed Invoices])` |

</details>

<details>
<summary><strong>🎨 Formatting & Dynamic Titles</strong> — 14 measures</summary>

| Measure | Purpose |
|---|---|
| Title_LineChart | Dynamic title for the spend trend chart — combines selected KPI and selected year |
| Title_BarChart | Dynamic title for the dimension breakdown chart — combines selected KPI and dimension |
| Title_LeadTimeChart | Dynamic title for the lead time chart — includes selected year |
| Sub-Title_MaverickChart | Dynamic subtitle for the maverick chart — shows selected dimension path |
| Spend YOY Format | Formats YoY spend change with directional arrow icons (▲▼) |
| Spend Budvar Format | Formats budget variance percentage with directional arrow icons |
| Savings YoY Format | Formats savings YoY change with directional arrow icons |
| Savings rate Format | Formats savings rate with directional arrow icons |
| OTD Rate YoY Format | Formats OTD rate YoY change — shows No Data when multiple years selected |
| Maverick Spend YoY Format | Formats maverick YoY change with directional arrow icons |
| At Risk Inv (Spend \| %) | Combines at-risk invoice spend and percentage into a single formatted string |
| OTD R | `1 - [OTD Rate]` — complement of OTD for gauge visuals |
| Avg Spend by Weekday | Average spend per day of week — used in KPI card sparklines |
| Avg Savings by Weekday | Average savings per day of week — used in KPI card sparklines |

</details>

---

## 🛠️ Tools & Technologies

| Tool | Usage |
|---|---|
| Power BI Desktop | Report development |
| ZoomCharts Drill Down Visuals | All chart visuals |
| Power Query (M) | Data transformation and star schema creation |
| DAX | All measures and calculated columns |
| Figma | Dashboard Background Design |
| Icons | https://www.svgrepo.com/ |
| Field Parameters | KPI card filtering and dimension switching |

---

## 🎨 Design Choices

- **Theme:** Dark background (`#161719`) with gold/yellow (`#EFCD15`) as the primary accent color throughout
- **KPI Cards as Filters:** Page 1 uses a hidden transparent tile slicer overlaid on the KPI cards — clicking a card updates the field parameter and highlights the selected card with a gold border
- **ZoomCharts Gradients:** All ZoomCharts bar visuals use a gold gradient fill from dark to bright, giving depth to the bars without needing multiple colors
- **Dynamic Titles:** Every chart has a DAX-powered title that updates based on the active KPI card, selected dimension, or selected year — the report always tells you exactly what you're looking at
- **Synced Slicers:** Year and Quarter slicers are synced across both pages so time period selections persist when navigating between pages

---

## 🏆 Challenge Details

| Detail | Value |
|---|---|
| Challenge | FP20 Analytics Challenge 37 |
| Topic | Procurement Performance |
| Partner | ZoomCharts |
| Data Period | 2022 — 2024 |
| Total PO Records | 5,200+ |
| Total Spend | ~$97.9M |

---

## 👤 Author

Built by **Crystal Andrea Dsouza**
Connect on [LinkedIn](https://www.linkedin.com/in/crystal-andrea-dsouza-641196286/) · View more projects on [GitHub](https://github.com/CrystalDsouza05?tab=repositories)

---

*Built with Power BI Desktop · ZoomCharts Drill Down Visuals · FP20 Analytics Challenge 37*
