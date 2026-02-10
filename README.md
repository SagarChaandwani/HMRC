#  HMRC Debt Recovery: Intelligence & Enforcement Hub

> **Professional Context:** This project demonstrates a Government-grade Risk Intelligence architecture designed during my tenure as a Data Analyst at **Ascentiq Solutions**.
>
> *Note: While the data modeling, DAX logic, and enforcement frameworks mirror real-world implementations delivered for His Majesty's Revenue and Customs (HMRC), the underlying dataset used in this public repository is synthetic to comply with GDPR and Official Secrets Act regulations.*

| **Governance** | **Architecture** | **Status** |
| :--- | :--- | :--- |
| ![Security](https://img.shields.io/badge/Security-Clearance_Level-blue?style=flat-square) | **Star Schema (SQL/DAX)** | ✅ **Deployed** |

---


##  Executive Summary

**The Problem:**
HMRC’s Debt Management team was facing a **"Blind Spot" crisis**. With a **£1M+ active debt pile**, enforcement officers were overwhelmed. They were treating every missed payment equally, wasting hundreds of hours chasing low-value retail debts while high-value corporate defaulters "spiraled" into insolvency unnoticed.

**The Solution:**
I engineered an **Automated Risk Intelligence Hub** that moved the department from "Passive Reporting" (Excel lists) to "Active Enforcement." The system automatically triages cases based on behavioral risk patterns.

**The Impact:**
*    **Identified 23 "Habitual Defaulters"** immediately, creating a prioritized "Hit List" for legal action.
*    **Exposed the "Construction Crisis":** Revealed that the Construction sector was the primary driver of the recovery gap.
*    **Operational Velocity:** Reduced case investigation time from **hours to seconds** using a forensic drill-through workflow.
---

## 3.Data Architecture & Schema
To ensure high performance and accurate filtering, the data model was architected as a robust **Star Schema**.

![Data Model Screenshot](dashboard_previews/3.Model_View.png)

*   **Fact Tables:**
    *   `Fact_Installments`: Granular transaction logs (Due Date, Amount Paid, Payment Method). Contains the core `Is_Default_Flag` logic.
    *   `Fact_Enforcement_Actions`: Tracks the operational history of legal interventions, recording specific actions taken (Action Type) and their results (Outcome).
*   **Dimension Tables:**
    *   `Dim_Companies`: Sector, Region, Director Names.
    *   `Dim_Debt_Cases`: Case start dates and assigned officers.
    *   `Date Table`: Standard calendar for time-intelligence calculations.
*   **Relationships:** Strict **One-to-Many (1:*)** relationships flowing from Dimensions to the Fact table.

---

## 4.Operational Forensics

The solution is architected around a specific operational workflow: **Detect (Page 1) $\rightarrow$ Investigate (Page 2).**

### 1️⃣ Page 1: The Command Center (The Cockpit)
*Designed for Senior Risk Officers, this view eliminates noise and focuses strictly on "Exception-Based Reporting."*

![Main Dashboard Screenshot](dashboard_previews/1.Command_Center.png)

#### **A. The Heads-Up Display (Strategic KPIs)**
Top-level metrics provide an immediate health check of the £1M portfolio using "Glassmorphism" cards to separate high-level data from detailed analytics.
*   **Total Exposure (£1M):** Represents the total liability (Amount Due). This acts as the denominator for all risk calculations.
*   **Recovery Rate (65%):** The primary efficiency KPI (`Total Collected / Total Due`). A rate below 85% triggers a strategic review.
*   **High Risk Count (23):** The operational workload. This tells the officer there are exactly 23 companies requiring immediate intervention.
*   **Total Recovered (£661K):** The realized cash flow. This is the "Bankable" success metric.

#### **B. The Strategy Bridge (Waterfall Chart)**
**"Risk Concentration by Sector"**
Instead of a standard bar chart, a Waterfall chart was selected to show the **cumulative impact** of each sector on the total debt pile.
*   *Forensic Insight:* The chart clearly isolates **Construction** as the dominant risk factor (the tallest bar). This insight directs the department to pivot resources toward Construction sector audits.

#### **C. The Risk Matrix (The "Hit List")**
This table is the engine of the dashboard. It moves beyond standard reporting by combining financial data with visual risk indicators.
*   **Total Exposure (Data Bars):** Blue bars visualize the magnitude of debt relative to others. This allows users to spot "High Value" targets instantly without reading numbers.
*   **Recovery Rate:** Highlights collection efficiency per company.
*   **Risk Status (Conditional Icons):**
    *   ♦️ **Red Diamond (High Risk):** Triggered when `Default Count >= 1`. These companies have missed payments and require legal escalation.
    *   🟢 **Green Circle (Compliant):** Triggered when `Default Count = 0`. These companies are monitoring-only.

---

### 2️⃣ Page 2: The Case File (Forensic Drill-Through)
*This is a hidden page, accessible only via a specific user action. It acts as the "Digital Dossier" for a specific company.*

![Drill Through Screenshot](dashboard_previews/2.Case_File.png)

#### **The Drill-Through Workflow**
To access this page, the user must:
1.  Identify a **Red Diamond** target on Page 1.
2.  **Right-Click** the company name $\rightarrow$ Select **Drill Through**.
3.  *Result:* The dashboard passes the specific `Company_ID` filter to Page 2, isolating that single entity's data.

#### **A. Dynamic Header & Alerting**
*   **Company Context:** The title automatically updates to the selected company (e.g., "Apex Construction Ltd").
*   **"⚠️ AT RISK" Warning:** A dynamic DAX measure evaluates the specific client's standing. If they meet the criteria for a Habitual Defaulter, this large red warning appears to alert the case officer immediately.

#### **B. Variance Analysis (Payment vs. Plan)**
**"Payment Performance vs Plan"** (Clustered Column Chart)
This visual reconstructs the timeline of insolvency.
*   **Grey Columns (Payment Plan):** Represents the contractual obligation (What they *promised* to pay).
*   **Cyan Columns (Cash Collected):** Represents the actual inflow (What they *actually* paid).
*   *Forensic Insight:* By comparing the two bars, an analyst can pinpoint the exact month the company stopped paying (e.g., Seeing a Grey bar with no corresponding Cyan bar in April).

#### **C. The Transaction Log (The Evidence)**
A granular, line-by-line table listing every invoice issued to the company.
*   **Conditional Formatting:** Any row where `Is_Default_Flag = 1` is highlighted in **Red**.
*   **Operational Use Case:** This table serves as the legal evidence trail. Officers can export this specific view to PDF/Excel to attach to "Letter Before Action" enforcement notices.

#### **D. The "Escape Hatch" (Back Button)**
Located in the top-left corner. Because this is a drill-through page, the Back Button is crucial for UX, allowing the user to clear the filters and return to the Command Center to investigate the next target.

---


##  5.Interface Ergonomics
This project creates a "Mission Control" aesthetic suitable for government enforcement.

*   **Color Semantics:**
    *   **Background:** Dark Navy (`#0E1117`) to reduce eye strain during long shifts.
    *   **Risk Red (`#FF4B4B`):** Used *only* for alerts (Red Diamonds, "At Risk" banners, Default Rows) to demand attention.
    *   **Fiscal Blue (`#29B5E8`):** Used for volume metrics (Exposure, Cash Collected) to signify financial data.
*   **Navigation:** Uses a "Hub and Spoke" model. There are no tabs; users must start at the Command Center and "Drill Down" into details, preventing them from getting lost in data.

---

##  6.Policy Recommendations

The data reveals that a broad-brush enforcement approach is inefficient. The following targeted interventions are recommended to lift the Recovery Rate from **65% to the 85% target**:

### 1️⃣ The "Construction" Pivot (Resource Allocation)
*   **Observation:** The Waterfall Chart proves that the **Construction** sector contributes disproportionately to the total debt pile, yet currently receives the same enforcement attention as lower-risk sectors like Retail.
*   **Action:** Immediately reallocate **40% of Field Force agents** to focus exclusively on Construction cases.
*   **Rationale:** Construction firms have highly volatile cash flows. Speed is critical. Moving from "Letter Writing" to "Site Visits" for this sector will reduce the time-to-insolvency risk.

### 2️⃣ The "Amber" Intervention Protocol
*   **Observation:** Historical data indicates that **60%** of companies that miss *two* payments eventually default on the entire debt (Spiraling).
*   **Action:** Automate a **"Pre-Legal Warning"** workflow. When a company hits "Amber" status (1 missed payment), the system triggers an automated SMS/Email alert.
*   **Rationale:** "Nudging" compliance at the *Amber* stage is significantly cheaper (low OPEX) than pursuing legal action at the *Red* stage.

### 3️⃣ Toxic Asset Liquidation
*   **Observation:** The **23 "High Risk" Habitual Defaulters** identified in the KPI card represent a sunk cost if not acted upon immediately.
*   **Action:** Fast-track these specific cases for **Winding-Up Petitions (Liquidation)**.
*   **Rationale:** These entities have shown a behavioral refusal to pay. Further negotiation is a waste of resources; immediate legal escalation is the only route to secure assets.


---

##  7.Constraints & Expansion Logic

###Key Assumptions
*   **Strict Default Logic:** The model utilizes a "Binary Default" logic. If `Amount Paid < Amount Due`, the flag is set to `1`. There is no tolerance threshold for partial payments (e.g., paying 99% still triggers a default).
*   **Currency Consistency:** All historical debt values are assumed to be in GBP (£) and are not adjusted for inflation or post-judgment interest penalties.
*   **Fiscal Alignment:** The `Dim_Date` table is aligned to the standard **UK Tax Year** (April 6th – April 5th).

### Future Roadmap (Phase 2)
To further enhance the intelligence capabilities of this hub, the following upgrades are planned:

#### **A. Predictive Modeling (Python Integration)**
*   **Goal:** Move from *Descriptive Analytics* (What happened?) to *Predictive Analytics* (Who will default next month?).
*   **Tech:** Integrate a **Python Scikit-Learn** script within Power BI to run a **Logistic Regression** model.
*   **Impact:** Use features like *Payment Latency Days* and *Sector Volatility* to assign a "Probability to Default" % score to every live case.

#### **B. The "Phoenixism" Detector (Network Graph)**
*   **The Problem:** "Phoenixism" is a fraud pattern where directors dissolve a debt-ridden company only to incorporate a new entity immediately to avoid liabilities.
*   **The Solution:** Build a **Network Graph Visual** that links `Director_Name` across multiple `Company_IDs`.
*   **Impact:** This would allow the agency to blacklist directors who serially bankrupt companies, stopping the fraud at the source.
---

## 8.Engineering Protocols

### SQL Transformation (The "Brain")
Raw data was pre-processed using **SQL Server** to ensure logic resides in the backend.
*   **Window Functions (`LAG`):** Used to detect "Spiraling" behavior by comparing current status to the previous month.
*   **Ranking (`DENSE_RANK`):** Used to categorize top offenders within their specific sectors.
*   **CTEs:** Used to clean nulls and structure the logic modularly.

---

## 9.Glossary of Terms

| Term | Definition | Logic Used |
| :--- | :--- | :--- |
| **Total Exposure** | The total monetary value of debt currently assigned for collection. | `SUM(Fact_Installments[Amount_Due])` |
| **Recovery Rate** | The efficiency metric showing the percentage of debt successfully collected. | `DIVIDE(Total Recovered, Total Exposure)` |
| **Habitual Defaulter** | A company flagged as high risk due to repeated missed payments. | `Default Count >= 2 (Triggers Red Diamond)` |
| **Spiraling Debt** | A behavioral pattern where a debtor misses consecutive payments, indicating insolvency. | `SQL LAG() Function (Current=1 AND Prev=1)` |
| **Default Flag** | Binary marker indicating a specific installment was less than the due amount. | `IF Amount_Paid < Amount_Due THEN 1 ELSE 0` |

---
*Author: [Sagar Chaandwani]*
