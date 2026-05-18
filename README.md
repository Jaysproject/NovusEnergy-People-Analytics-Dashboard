A multi-page Power BI dashboard that tracks workforce metrics for a fictional energy company. It covers headcount, turnover, talent acquisition, pay equity, engagement, leave liability, and timesheet compliance — built on 10 source tables and 41 DAX measures.
The raw dataset ships with 12 categories of data quality issues (mixed date formats, duplicate records, inconsistent casing, text in numeric columns, etc.) that are resolved entirely in Power Query. The project documents every cleaning step so it doubles as a reference for common ETL patterns in Power BI.
Some DAX measures were developed using Power BI's MCP server connected to Claude via the Model Context Protocol. This allowed Claude to interact directly with the Power BI semantic model — inspecting tables, columns, and relationships — to write and validate DAX measures against the live data model rather than working from descriptions alone.

Screenshot placeholder — drop your dashboard screenshot in screenshots/ and uncomment the image tag below.
[<!--->](https://github.com/Jaysproject/NovusEnergy-People-Analytics-Dashboard/blob/main/NovusEnergy%20Dashboard.png)


What's in the data
The source file NovusEnergy_HR_Data_Raw.xlsx contains 10 sheets:
TableRecordsRoleEmployee_Master~1,750Core employee records — the hub tableSalary_Data~1,700Base salary, bonus, total comp by gender and levelLeave_Balances~1,550Annual, personal, and long service leave hoursTimesheets~4,400Weekly hours and submission complianceJob_Requisitions150Recruitment pipeline and days-to-fillApplicant_Demographics500Applicant gender breakdown by requisitionHeadcount_History16Monthly headcount snapshots for trend linesPromotions_Secondments80Internal mobility actionsEngagement_Survey50Pulse survey scores by business unitAnnual_Spend50Monthly reward spend by category

Data cleaning techniques
All cleaning happens in Power Query. The raw data includes these issues by design:
IssueExampleFixMixed date formatsDD/MM/YYYY, YYYY-MM-DD, MM-DD-YYYY in one columnChange Type Using Locale + Replace ErrorsInconsistent casingMale, MALE, male, MLowercase → Replace ValuesLeading/trailing spaces" Sarah", "Johnson "Text.Trim + Text.CleanDuplicate rows~25 duplicate employeesRemove Duplicates on key columnDepartment name variantsIT, Information Technology, ITTrim → Replace Values chainText in numeric columns"N/A", "TBD" in Bonus fieldData Type conversion → Replace ErrorsFuture datesTermination dates past reporting periodCustom Filter: is null OR ≤ cutoffInconsistent Yes/NoYes, Y, YES, No, NLowercase → Replace ValuesNegative/outlier valuesSalary of $500 or $1,000,000Conditional filteringBlank keysEmpty Employee_IDsFilter Rows via column dropdownOrphan foreign keysLeave records with no matching employeeInner Join via Merge QueriesMonth format variants2026-01 vs 01/2026Custom Column with format detection

AI-assisted development
Several DAX measures were built with Claude connected to Power BI's semantic model via the Model Context Protocol (MCP). The MCP server exposes the live data model to Claude, so it can:

Read table schemas, column names, and data types directly
Inspect existing relationships and cardinalities
Write DAX measures that reference actual columns rather than assumed ones
Validate measure logic against the real model structure

This was particularly useful for the more complex measures like Size Adj Pay Gap (which uses AVERAGEX across job levels), the turnover MoM calculations with VAR blocks, and the Involuntary Terminations L12M measure that needed to account for multiple termination types (Involuntary, Redundancy, End Of Contract) discovered by inspecting distinct values in the live model.

DAX measures — 41 total
Measures live in a dedicated _Measures table. Here's the breakdown by section:
Demographics (6) — Headcount, FTE, Male/Female Count, Male/Female %
New Starters & Leavers (7) — Monthly hires and exits, gender splits, MoM change. Measures reference each other: New Starters Male = CALCULATE([New Starters This Month], Gender = "Male") rather than repeating filter logic.
Turnover (8) — Rolling 12-month terminations, voluntary/involuntary split (includes Redundancy and End Of Contract as involuntary), rates against average headcount, gender-specific voluntary rates.
Talent Acquisition (6) — Open reqs, avg days to fill, acceptance rate, applicants per role, roles filled, internal hire %. Internal Hire % filters to Status = "Filled" to avoid exceeding 100%.
Pay Equity (4) — Average salary by gender, overall gender pay gap, size-adjusted pay gap using AVERAGEX across job levels.
Reward & Mobility (7) — Annual spend, promotions and secondments YTD with gender splits.
Leave & Compliance (3) — Employees exceeding 8-week (304 hrs) and 10-week (380 hrs) leave thresholds, timesheet submission compliance rate.
Engagement (1) — Average engagement score from pulse survey.

Data model
                     ┌──────────────────┐
                     │  Employee_Master │
                     │     (hub)        │
                     └────────┬─────────┘
                              │ Employee_ID
        ┌───────────┬─────────┼──────────┬──────────┐
        ▼           ▼         ▼          ▼          ▼
  Salary_Data  Leave_Bal  Timesheets  Promotions  Headcount
                                      Secondments  History

  Job_Requisitions ◄─── Applicant_Demographics
                   Req_ID

  Engagement_Survey    Annual_Spend
  (standalone)         (standalone)
All relationships are Many-to-One with single-direction cross-filtering.

Project structure
NovusEnergy-People-Analytics-Dashboard/
├── Dashboard/
├── Data/
├── Screenshots/
├── Documentation/
└── README.md

Dashboard/ — the .pbix file
Data/ — raw Excel source with 10 sheets; optionally a cleaned version
Screenshots/ — dashboard page captures for this readme
Documentation/ — data dictionary, DAX reference, cleaning steps


Tools

Power BI Desktop
Power Query / M language
DAX (Data Analysis Expressions)
Claude AI + Model Context Protocol (DAX development)
Power BI MCP Server (semantic model access)
Microsoft Excel (source data)

