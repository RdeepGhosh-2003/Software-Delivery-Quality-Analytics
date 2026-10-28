# 📊 Software Delivery & Quality Analytics (2024–2025) — Power BI & ZoomCharts Challenge

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Power Query](https://img.shields.io/badge/Power%20Query-107C41?style=for-the-badge&logo=microsoft-excel&logoColor=white)
![ZoomCharts](https://img.shields.io/badge/ZoomCharts-Drill%20Down%20PRO-2563EB?style=for-the-badge)

## 📌 Project Overview
This project was developed for the **ZoomCharts Power BI Software Delivery & Quality Analytics Challenge**. It features an interactive, single-page Power BI report that evaluates how effectively a software company delivers product updates across **2024 and 2025**, maintains post-release quality, and mitigates production incidents and financial downtime.

Using **ZoomCharts Drill Down PRO visuals** and custom Power BI report page tooltips, engineering leaders and product managers can drill seamlessly from portfolio-level KPIs down to individual delivery teams, product modules, release types, and work items.

---

## 🖼️ Dashboard Preview

<!-- Replace the image paths below with your actual screenshot file paths in your repo -->
![Main Dashboard Overview](assets/dashboard_overview.png)
![Custom Drill-Down Tooltip](assets/tooltip_details.png)

---

## 🎯 Business Questions Answered
In alignment with the challenge brief, this dashboard answers six core operational questions:
1. **Release Performance:** Are software updates delivered on time, and where do the biggest schedule delays occur?
2. **Product Quality:** Which product groups, modules, teams, or release types generate the most post-release defects?
3. **Testing Effectiveness:** Does higher test automation ($\ge 75\%$) reduce production defects, incidents, and release delays?
4. **Incidents & Financial Impact:** Which releases drive the highest production downtime and business cost?
5. **Teams & Products:** Which delivery teams and product lines set the benchmark, and which require immediate intervention?
6. **Performance Over Time:** Is software delivery becoming faster, more automated, and more reliable from 2024 to 2025?

---

## 🗂️ Dataset & Star Schema Data Model

The underlying dataset (`Software_Delivery_Quality_challenge_dataset.xlsx`) tracks **643 software releases** and **6,226 individual release items** across **6 Product Groups**, **36 Modules**, and **15 Delivery Teams** from January 2024 through December 2025.

### Data Model Architecture
To optimize query performance and prevent ambiguous relationship paths, the raw sheets were transformed into a clean **Star Schema**:
* **`FactReleasePerformance` (643 rows):** Central fact table containing release dates, planned dates, development/testing days, test execution counts, automated test share, pre-release and production defects, incidents, downtime minutes, and calculated downtime cost.
* **`DimProduct` (36 rows):** Product hierarchy (`Product Group` -> `Product` -> `Module`), merged with `DowntimeCost` to store `Downtime Cost per Minute ($)` directly on the product dimension and disambiguated via a combined `Product - Module` key.
* **`DimTeam` (15 rows):** Lookup table for the 15 engineering delivery teams (`Falcon`, `Kestrel`, `Atlas`, `Beacon`, `Titan`, etc.).
* **`DimRelease` (3 rows):** Release classification (`Major`, `Minor`, `Patch`).
* **`DimDate` (731 rows):** Calendar dimension spanning `2024-01-01` to `2025-12-31` with chronological sort columns (`Year`, `Year-Quarter`, `Year-Month`, `Month No`).
* **`ReleaseItems` (6,226 rows):** Granular scope breakdown (`4,959 Features` and `1,267 Bug Fixes`) across 11 functional categories (`Security`, `Usability`, `Integration`, `Analytics`, `Workflow`, etc.) linked to `FactReleasePerformance` via `Release ID`.
* **`Benchmarks` (7 rows):** Disconnected reference table storing official SLA targets for dynamic DAX threshold evaluation.

---

## ⚙️ Data Cleaning & Transformation Pipeline (Power Query)

Key data engineering steps applied in Power Query (`M`) to ensure **Data Cleanliness** and **Data Accuracy**:
* **Snowflake Elimination:** Merged `DowntimeCost[Downtime Cost per Minute ($)]` into `DimProduct` and disabled load on the staging `DowntimeCost` table.
* **Module Disambiguation:** Created `Product - Module` (`[Product] & " - " & [Module]`) to prevent two separate products sharing the module name `Messaging` (`P01: Connect - Messaging` and `P21: Engage - Messaging`) from aggregating together.
* **Schedule Variance Engineering:** Calculated `Delay Days` (`Duration.Days([Release Date] - [Planned Release Date])`), `Is On Time Flag` (`Delay Days <= 0`), and `Delivery Status` (`Early`, `On Time`, `Late`).
* **Financial Impact Calculation:** Computed row-level `Downtime Cost ($)` (`[Downtime Minutes] * [Downtime Cost per Minute ($)]`) typed explicitly as a Decimal/Currency number.
* **Automation Tier Segmentation:** Grouped releases into three test automation tiers (`1. Low (<65%)`, `2. Moderate (65-75%)`, and `3. High (>=75% Target)`) to evaluate the direct impact of test automation on post-release stability.

---

## 📐 Core DAX Measures & Benchmark Evaluation

All measures are organized into a dedicated Measure Table with structured Display Folders (`00. Benchmarks`, `01. Release & Schedule Performance`, `02. Testing & Product Quality`, `03. Incidents, Downtime & Financial Impact`, and `04. KPI Status & Conditional Colors`).

| KPI / Metric | Overall Dataset Value | Official Benchmark | Performance Status |
| :--- | :--- | :--- | :--- |
| **Total Releases** | **643** *(320 in 2024, 323 in 2025)* | — | Baseline Volume |
| **On-time Release %** | **65.5%** *(421 On-Time, 222 Late)* | **>= 90.0%** | 🔴 Below Target (-24.5%) |
| **Avg Delay Days (Late Releases)** | **4.08 days** *(0.79 days across all)* | **<= 3.00 days** | 🔴 Above Target (+1.08 days) |
| **Automated Tests %** | **73.7%** | **>= 75.0%** | 🟡 Near Target (-1.3%) |
| **Test Failure Rate** | **4.10%** *(15,295 / 373,153 tests)* | **<= 5.0%** | 🟢 Meeting Target |
| **Production Defects per Release** | **1.15** *(738 Total, 79 Critical)* | **<= 2.00** | 🟢 Meeting Target |
| **Defect Detection Efficiency (DDE %)** | **69.6%** *(1,693 Pre-release defects)* | Maximize | 🟢 Strong Pre-Release Catch |
| **Production Incidents per Release** | **0.34** *(218 Total across 146 releases)* | **<= 0.30** | 🟡 Slightly Above (+0.04) |
| **Avg Incident Resolution Hours** | **3.17 hours** *(active incident releases)* | **<= 2.00 hours** | 🔴 Above Target (+1.17 hours) |
| **Total Downtime Cost ($)** | **$2,950,024.31** *(11,632 mins / 193.9 hrs)* | Minimize | **$4,588** Avg Cost / Release |

### Highlighted DAX Formulas
```dax
// On-Time Release Percentage (Benchmark: >= 90%)
On-time Release % = 
DIVIDE(
    CALCULATE(COUNTROWS(FactReleasePerformance), FactReleasePerformance[Delay Days] <= 0),
    COUNTROWS(FactReleasePerformance),
    0
)

// Average Delay Days on Late Releases Only (Benchmark: <= 3.0 days)
Avg Delay Days - Late Releases = 
CALCULATE(
    AVERAGE(FactReleasePerformance[Delay Days]),
    FactReleasePerformance[Delay Days] > 0
)

// Prevents 0-incident releases (497 rows with 0.0 hrs) from artificially diluting resolution time
Avg Incident Resolution Hours = 
CALCULATE(
    AVERAGE(FactReleasePerformance[Avg Incident Resolution Hours]),
    FactReleasePerformance[Production Incidents] > 0
)

// Measures the share of total defects caught during testing before reaching production
Defect Detection Efficiency (DDE %) = 
DIVIDE(
    [Pre-release Defects],
    [Pre-release Defects] + [Production Defects],
    0
)

// Automation Tier Calculated Column
Automation Tier = 
SWITCH(
    TRUE(),
    FactReleasePerformance[Automated Tests %] < 0.65, "1. Low (<65%)",
    FactReleasePerformance[Automated Tests %] < 0.75, "2. Moderate (65-75%)",
    "3. High (>=75% Target)"
)
```

---

## 🎨 Interactive Visuals & ZoomCharts Features

The single-page report integrates **5 ZoomCharts Drill Down PRO visuals** alongside an executive KPI banner and a custom **Report Page Tooltip (`Tooltip_Details`)**:

1. **Delivery Volume, On-Time % & Test Automation Over Time (`Drill Down Combo PRO`):**
   * *Drill Path:* `Year` -> `Year-Quarter` -> `Year-Month` -> `Release Date`
   * Compares release volume on the primary axis against `On-time Release %` and `Automated Tests %` on the secondary percentage axis.
2. **Downtime Cost ($) & Incident Rate by Product Hierarchy (`Drill Down Combo Bar PRO`):**
   * *Drill Path:* `Product Group` -> `Product` -> `Product - Module` -> `Release ID`
   * Plots `Total Downtime Cost ($)` bars against a secondary `Production Incidents per Release` line with a benchmark threshold at `0.30` incidents/release.
3. **Team Performance: On-Time Delivery vs. Production Defects (`Drill Down Combo Bar PRO`):**
   * *Drill Path:* `Team` -> `Release Type` -> `Release ID`
   * Ranks all 15 engineering teams by `On-time Release %` (`90%` target line) overlaid with `Production Defects per Release` (`2.00` max target line).
4. **Impact of Test Automation on Defects & Release Delays (`Drill Down Combo Bar PRO`):**
   * *Drill Path:* `Automation Tier` -> `Delivery Status`
   * Demonstrates how moving from Low (`<65%`) to High (`>=75%`) test automation reduces both production defects (`1.55` -> `0.89`) and late release delays (`5.03d` -> `2.72d`, beating the `3.0d` target).
5. **Release Items by Category & Type (`Drill Down Donut PRO`):**
   * *Drill Path:* `Item Category` -> `Item Type` (`Feature` vs. `Bug Fix`)
   * Breaks down all `6,226` delivered items across 11 functional categories (`Security 13.8%`, `Usability 13.7%`, `Integration 13.2%`, `Analytics 10.6%`, `Workflow 10.3%`, etc.).
6. **Contextual Drill-Down Tooltip (`Tooltip_Details`):**
   * Hovering over any bar or slice reveals an 8-card breakdown of `Total Releases`, `Features Delivered`, `Bug Fixes Delivered`, `Pre-release Defects`, `Critical Production Defects`, `Defect Detection Efficiency (DDE %)`, `Total Downtime Minutes`, and `Avg Incident Resolution Hours`.

---

## 💡 Key Executive Insights & Recommendations

1. **Test Automation Directly Solves Both Quality and Schedule Delays:**
   * Releases meeting the **>= 75% Automated Testing benchmark** achieve a **75.0% on-time rate**, cut **late release delays to 2.72 days** (beating the <= 3.0d SLA), and reduce **production defects to 0.89 per release**.
   * Conversely, releases with **Low Automation (<65%)** suffer a **5.03-day average late delay** and **1.55 production defects per release**.
2. **73% of Financial Downtime Is Concentrated in Two Product Groups:**
   * **Platform ($1.14M, 0.48 incidents/rel)** and **Commerce ($1.01M, 0.42 incidents/rel)** account for **$2.15M (73%)** of the company's **$2.95M** total downtime cost.
   * **Data & AI ($505.20K)** exhibits the highest incident frequency at **0.53 incidents per release** (77% above the `0.30` target), whereas **Customer Experience ($39.30K, 0.10 incidents/rel)** and **Collaboration ($92.07K, 0.18 incidents/rel)** consistently beat quality benchmarks.
3. **Engineering Team Benchmarks & Bottlenecks:**
   * **Top Performers:** **Kestrel** (`79.2%` on-time, `0.58` defects/rel, `84.6%` automated tests) and **Aurora** (`78.8%` on-time, `0.48` defects/rel) set the internal standard for fast, low-defect delivery.
   * **High-Cost Incident Hotspots:** **Beacon** achieves the highest on-time delivery (`84.8%`) but averages `1.45` production defects/release and **$608K** in downtime cost; **Atlas** (`73.1%` on-time, `1.22` defects/rel) drives **$826K** in downtime cost.
   * **Underperforming Teams Needing CI/CD Support:** **Pulse** (`30.8%` on-time, `2.15` defects/rel), **Vertex** (`43.8%` on-time, `58.5%` automated), **Orion** (`46.2%` on-time), and **Titan** (`46.9%` on-time, `59.1%` automated) require immediate investment in automated regression suites.
4. **Year-over-Year Progress (2024 vs. 2025):**
   * On-time delivery improved from **64.1% in 2024** to **66.9% in 2025**, and quarterly downtime costs dropped **59%** from **Q2 2025 ($511K)** to **Q4 2025 ($209K)**.

---

## 📂 Repository Structure

```text
├── assets/
│   ├── dashboard_overview.png                        # Full single-page dashboard screenshot
│   └── tooltip_details.png                           # Custom Report Page Tooltip screenshot
├── data/
│   └── Software_Delivery_Quality_challenge_dataset.xlsx  # Challenge dataset
├── Software Delivery & Quality Analytics.pbix        # Power BI report file
├── RULES.md                                          # Challenge brief & evaluation criteria
└── README.md                                         # Project documentation
```

---

## 🚀 How to Open & Explore the Report

1. Clone this repository:
   ```bash
   git clone [https://github.com/YOUR-USERNAME/software-delivery-quality-analytics.git](https://github.com/YOUR-USERNAME/software-delivery-quality-analytics.git)
   ```
2. Open **`Software Delivery & Quality Analytics.pbix`** in **Microsoft Power BI Desktop**.
3. *(Optional)* If refreshing the data source, update the file path in **Transform Data > Data Source Settings** to point to `Software_Delivery_Quality_challenge_dataset.xlsx`.
4. Click any bar, line node, or donut slice on the report canvas to drill down through hierarchies, and hover over data points to inspect granular metrics via the custom report tooltip.

---

## 📜 Acknowledgments
* **Competition:** ZoomCharts Power BI Software Delivery & Quality Analytics Challenge
* **Custom Visuals:** [ZoomCharts Drill Down Visuals for Power BI](https://zoomcharts.com/en/microsoft-power-bi-custom-visuals/)
