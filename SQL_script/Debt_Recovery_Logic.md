========================================================================================
PROJECT:    HMRC Debt Recovery Intelligence Hub

AUTHOR:     [Sagar Chaandwani]

OBJECTIVE:  ETL Script to transform raw instalment logs into a Strategic Risk Engine.
            
KEY LOGIC:
1. CTEs: Modularised logic for Cleaning, Behavioral Analysis, and Scoring.
2. WINDOW FUNCTIONS: 
   - LAG: To detect "Spiralling" debt (Consecutive missed payments).
   - SUM OVER: For rolling defaults without collapsing rows.
   - DENSE_RANK: To prioritise the top offenders within specific sectors.
3. HANDLING NULLS: Robust ISNULL and SAFE DIVIDE logic for financial accuracy.
========================================================================================
*/

-- CTE 1: CLEANING & PRE-PROCESSING
-- Objective: Standardise payment flags and handle NULLs in raw transactional data.
WITH Clean_Transactions AS (
    SELECT 
        i.Case_ID,
        i.Due_Date,
        i.Amount_Due,
        
        -- Handle Nulls: If no payment record exists, treat as 0
        ISNULL(i.Amount_Paid, 0) AS Amount_Paid, 
        
        -- Create Binary Flag: 1 if Missed/Defaulted, 0 if Paid in Full
        CASE 
            WHEN ISNULL(i.Amount_Paid, 0) < i.Amount_Due THEN 1 
            ELSE 0 
        END AS Is_Default_Flag
    FROM Fact_Installments i
),

-- CTE 2: BEHAVIORAL ANALYSIS (The "Window Function" Layer)
-- Objective: Look across time to find behavioral patterns, not just static totals.
Behavior_Metrics AS (
    SELECT 
        ct.Case_ID,
        ct.Amount_Due,
        ct.Amount_Paid,
        ct.Is_Default_Flag,
        
        -- LOGIC: "The Spiral" 
        -- Check if the PREVIOUS month was also a default. 
        -- If Current=1 AND Prev=1, they are "Spiralling".
        LAG(ct.Is_Default_Flag, 1, 0) OVER (
            PARTITION BY ct.Case_ID 
            ORDER BY ct.Due_Date
        ) AS Previous_Month_Status,

        -- LOGIC: "Cumulative Failures"
        -- Rolling count of defaults for this specific case over time
        SUM(ct.Is_Default_Flag) OVER (
            PARTITION BY ct.Case_ID 
            ORDER BY ct.Due_Date 
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS Rolling_Defaults
    FROM Clean_Transactions ct
),

-- CTE 3: AGGREGATION & SCORING (The "Group By" Layer)
-- Objective: Flatten the data to one row per Case for the Dashboard.
Case_Level_Aggregates AS (
    SELECT 
        d.Company_ID,
        d.Case_ID,
        d.Debt_Type, -- VAT, Corp Tax, etc.
        
        -- Financial Totals
        SUM(b.Amount_Due) AS Total_Case_Value,
        SUM(b.Amount_Paid) AS Total_Recovered,
        (SUM(b.Amount_Due) - SUM(b.Amount_Paid)) AS Current_Outstanding,
        
        -- Efficiency Metric (Protected against Divide by Zero)
        CAST(SUM(b.Amount_Paid) AS FLOAT) / NULLIF(SUM(b.Amount_Due), 0) AS Recovery_Rate_Pct,

        -- Risk Triggers
        MAX(b.Rolling_Defaults) AS Total_Missed_Payments,
        
        -- Did they miss 2 in a row at any point?
        MAX(CASE WHEN b.Is_Default_Flag = 1 AND b.Previous_Month_Status = 1 THEN 1 ELSE 0 END) AS Has_Spiraled_Flag

    FROM Behavior_Metrics b
    JOIN Dim_Debt_Cases d ON b.Case_ID = d.Case_ID
    GROUP BY d.Company_ID, d.Case_ID, d.Debt_Type
),

-- CTE 4: SECTOR RANKING & FINAL SEGMENTATION
-- Objective: Apply business rules to label the debtors.
Final_Risk_Engine AS (
    SELECT 
        agg.*,
        c.Company_Name,
        c.Sector,
        c.Region,

        -- SEGMENTATION LOGIC (The "Brain" of the Dashboard)
        CASE 
            WHEN agg.Has_Spiraled_Flag = 1 THEN 'CRITICAL - LEGAL ACTION'
            WHEN agg.Total_Missed_Payments >= 3 THEN 'HIGH - HABITUAL DEFAULTER'
            WHEN agg.Current_Outstanding > 0 AND agg.Total_Missed_Payments > 0 THEN 'MEDIUM - WATCHLIST'
            WHEN agg.Recovery_Rate_Pct >= 0.98 THEN 'LOW - COMPLIANT'
            ELSE 'LOW - MONITORING'
        END AS Risk_Category,

        -- RANKING LOGIC
        -- Rank debtors by outstanding amount within their Sector
        DENSE_RANK() OVER (
            PARTITION BY c.Sector 
            ORDER BY agg.Current_Outstanding DESC
        ) AS Sector_Risk_Rank

    FROM Case_Level_Aggregates agg
    JOIN Dim_Companies c ON agg.Company_ID = c.Company_ID
)

-- FINAL OUTPUT TO POWER BI,


SELECT 

    Risk_Category,
    Company_Name,
    Sector,
    Region,
    Debt_Type,
    Total_Case_Value,
    Current_Outstanding,
    Recovery_Rate_Pct,
    Total_Missed_Payments,
    Sector_Risk_Rank
FROM Final_Risk_Engine
WHERE Current_Outstanding > 0 -- Only show active debts
ORDER BY 
    CASE Risk_Category 
    WHEN 'CRITICAL - LEGAL ACTION' THEN 1
        WHEN 'HIGH - HABITUAL DEFAULTER' THEN 2
        ELSE 3  
        END,
