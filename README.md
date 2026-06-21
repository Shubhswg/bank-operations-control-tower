Bank Operations Control Tower: Transaction Reconciliation Engine
   
An end-to-end reconciliation engine simulating how retail banks detect, age, and resolve transactional breaks between internal Core Banking and external Payment Gateway systems. Built using Excel (Power Query) for ETL data pipeline engineering and Power BI for risk-exposure visualization.

Dashboard Preview
 <img width="1297" height="731" alt="Screenshot 2026-06-21 201403" src="https://github.com/user-attachments/assets/79950e08-69f2-4806-b114-538850df20d4" />

Interactive dashboard isolating risk KPIs, volume-at-risk trends, and exception aging.

Key Analytical Insights
•	Financial Float at Risk: First-pass reconciliation intercepted a 9.66% break rate (1,499 breaks out of 15,516), exposing ₹368.49M in un-reconciled capital requiring manual remediation.
•	Systemic Failure Root-Cause: Amount Mismatches (547 cases) dominate over fully Missing Transactions (460 cases). This directs attention toward fixing rounding/conversion logic rather than connectivity issues.
•	Predictable Operational Spikes: Exception volumes do not occur randomly; the trend line reveals distinct, recurring spikes concentrated at month-end, enabling managers to optimize operational staffing schedules around known peak-stress days.
•	The "Silent Duplicates" Loophole: The engine intercepted 516 double-posted transactions that bypassed standard 1:1 matching by sharing identical dates and amounts. True risk exceptions total 2,015 records, proving the need for a duplicate-frequency check independent of amount/date matching.
•	Geographic Risk Concentration: Branch-level exposure analysis isolates the Chennai Branch as the highest priority backlog, providing a data-driven starting point for risk reduction.

 Data Architecture & Workflow
1.	Ingestion: Processes two independent asynchronous ledgers - System A (15,000 clean records, the source of truth) and System B (15,057 records, after controlled month-end-weighted error injection: 460 dropped, 516 duplicated). A full outer join between both produces the analysis table of 15,516 rows.
2.	ETL Pipeline: Executes a Full Outer Join in Power Query. Custom M conditional logic programmatically classifies each transaction into a break category (Matched / Missing in B / Amount Mismatch / Timing Shift), with duplicates flagged via an independent row-count check on Transaction_id.
3.	Aging Lifecycle: Each break is bucketed into operational clearing horizons (0-1 Days, 2-3 Days, 4-7 Days, 7+ Days) and logged with a simulated owner, resolution action, and status - tracking the full lifecycle from detection to resolution.
4.	Reporting Layer: DAX measures (Total_Breaks, Break_Rate_Pct, Total_Exposure) power a live Power BI dashboard with branch, break-type, and date-range slicers for interactive monitoring.

Repository Structure
•	Bank_Operations_Control_Tower.xlsx - Raw transactional ledgers, reconciliation mapping engine, aging, and exception logs.
•	Bank_Operations_Control_Tower.pbix - Interactive Power BI dashboard with KPI cards, DAX measures, and slicers.
•	Insight_Report.md - Deep-dive operational write-up and remediation recommendations.

Project Parameters
•	Synthetic Data: Built entirely on simulated data for educational modeling; no real institution or client data is utilized.
•	Aging Reference: Uses a static snapshot date for calculations; aging buckets represent the dataset's date range rather than an actively updating backlog.

Author
Shubh Gupta MBA - Digital Business Management, Indian Institute of Management Bodh Gaya ,Email Id- shubh.g2026d@iimbg.ac.in | LinkedIn
