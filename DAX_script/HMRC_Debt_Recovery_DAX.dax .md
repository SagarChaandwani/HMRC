/*
=========================================================================
PROJECT: HMRC Debt Recovery Intelligence Hub
AUTHOR: [Sagar Chaandwani]
TECHNOLOGY: Power BI / DAX

DESCRIPTION: 
This file contains the DAX (Data Analysis Expressions) logic used for 
KPI calculations, Risk Segmentation, and Dynamic UI elements.
=========================================================================
*/



/* -----------------------------------------------------------------------
   SECTION 1: CORE FINANCIAL METRICS
   Aggregations based on the transaction fact table (Fact_Installments).
   ----------------------------------------------------------------------- */



-- Measure: Total Exposure
-- Description: Calculates the total debt value currently monitored in the system.
Total Exposure = 
SUM('Fact_Installments'[Amount_Due])





-- Measure: Total Recovered
-- Description: Calculates the total amount successfully collected from debtors.
Total Recovered = 
SUM('Fact_Installments'[Amount_Paid])





-- Measure: Recovery Rate %
-- Description: Efficiency KPI. Uses DIVIDE to handle potential divide-by-zero errors safely.
-- Format: Percentage (0.0%)
Recovery Rate = 
DIVIDE(
    [Total Recovered], 
    [Total Exposure], 
    0
)







/* -----------------------------------------------------------------------
   SECTION 2: RISK ENGINE & LOGIC
   Logic to identify and count high-risk entities.
   ----------------------------------------------------------------------- */



-- Measure: High Risk Count
-- Description: Counts distinct Case IDs associated with at least one defaulted payment.
High Risk Count = 
CALCULATE(
    DISTINCTCOUNT('Dim_Debt_Cases'[Case_ID]),
    'Fact_Installments'[Is_Default_Flag] = 1
)

/* 
   NOTE: 'Is_Default_Flag' is a Calculated Column (or SQL derived) in Fact_Installments.
   Logic: IF(Amount_Paid < Amount_Due, 1, 0)
*/






/* -----------------------------------------------------------------------
   SECTION 3: UX & CONDITIONAL FORMATTING
   Measures designed specifically for the User Interface (Icons and Dynamic Text).
   ----------------------------------------------------------------------- */




-- Measure: Case Status Text
-- Description: Dynamic text for the "Drill-Through" header. 
-- Changes based on the selected company's risk profile.
Case Status Text = 
IF(
    [High Risk Count] > 0, 
    "⚠️ AT RISK", 
    "✅ COMPLIANT"
)






-- Measure: Risk Score Icon
-- Description: Returns a numeric value used to drive Conditional Formatting icons in the matrix.
-- Rules: 3 = Red (Habitual), 2 = Yellow (Warning), 1 = Green (Clean).
Risk Score Icon = 
VAR MissedPayments = SUM('Fact_Installments'[Is_Default_Flag])
RETURN
    SWITCH(TRUE(),
        MissedPayments >= 3, 3, 
        MissedPayments >= 1, 2, 
        1 
    )