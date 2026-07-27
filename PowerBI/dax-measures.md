### 🪴 Why this file exists

- The dashboard needs to answer two different questions with the same data: "how did the whole company do?" and "how did this one department do?", and department numbers include a share of shared G&A costs while the company overall view doesn't (otherwise those costs get counted twice).
- Most measures come in two versions: a plain version and an "Incl GA" version. There's also a general split between calculation measures (used inside other formulas) and display measures (used only in
charts and tables), which mixing the two caused a real bug in this project, explained at the bottom of this file.
- All measures live in the `sql_var_detail_v1` table unless noted otherwise.
<br>

### 🪴 DAX Measures

**🌱 Core P&L (before GA allocation)**

```dax
Revenue Actual = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Revenue",
    sql_var_detail_v1[amount_type] = "Actual"
)
```
```dax
Revenue Budget = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Revenue",
    sql_var_detail_v1[amount_type] = "Budget"
)
```
```dax
COGS Actual = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "COGS",
    sql_var_detail_v1[amount_type] = "Actual"
)
```
```dax
COGS Budget = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "COGS",
    sql_var_detail_v1[amount_type] = "Budget"
)
```
```dax
Gross Profit Actual = [Revenue Actual] - [COGS Actual]
```
```dax
Gross Profit Budget = [Revenue Budget] - [COGS Budget]
```
```dax
OI Actual = 
CALCULATE(
    [Amount Actual],
    REMOVEFILTERS ( 'Income Statement Line Items' ),
    'Income Statement Line Items'[line_item] = "Operating profit"
)
```
```dax
OI Budget = 
CALCULATE(
    [Amount Budget],
    REMOVEFILTERS ( 'Income Statement Line Items' ),
    'Income Statement Line Items'[line_item] = "Operating profit"
)
```
<br>

**🌱 SG&A expenses by category (department's own costs, before GA)**

```dax
Personnel Actual = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Personnel Expense",
    sql_var_detail_v1[amount_type] = "Actual"
)
```
```dax
Personnel Budget = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Personnel Expense",
    sql_var_detail_v1[amount_type] = "Budget"
)
```
```dax
Administrative Actual = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Administrative expense",
    sql_var_detail_v1[amount_type] = "Actual"
)
```
```dax
Administrative Budget = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Administrative expense",
    sql_var_detail_v1[amount_type] = "Budget"
)
```
```dax
Freight Actual = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Freight expense",
    sql_var_detail_v1[amount_type] = "Actual"
)
```
```dax
Freight Budget = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Freight expense",
    sql_var_detail_v1[amount_type] = "Budget"
)
```
```dax
Marketing Actual = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Marketing expense",
    sql_var_detail_v1[amount_type] = "Actual"
)
```
```dax
Marketing Budget = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Marketing expense",
    sql_var_detail_v1[amount_type] = "Budget"
)
```
```dax
RD Actual = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "R&D",
    sql_var_detail_v1[amount_type] = "Actual"
)
```
```dax
RD Budget = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "R&D",
    sql_var_detail_v1[amount_type] = "Budget"
)
```
<br>

**🌱 GA cost pool**  
The GA department's own raw Personnel/Administrative costs, before being distributed out to the four operating departments.

```dax
GA Personnel Actual = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Personnel Expense",
    sql_var_detail_v1[amount_type] = "Actual",
    sql_var_detail_v1[dept_code] = "GA",
    ALL(dept_id[Dept_Code])
)
```
```dax
GA Personnel Budget = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Personnel Expense",
    sql_var_detail_v1[amount_type] = "Budget",
    sql_var_detail_v1[dept_code] = "GA",
    ALL(dept_id[Dept_Code])
)
```
```dax
GA Administrative Actual = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Administrative expense",
    sql_var_detail_v1[amount_type] = "Actual",
    sql_var_detail_v1[dept_code] = "GA",
    ALL(dept_id[Dept_Code])
)
```
```dax
GA Administrative Budget = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[account_category_sub] = "Administrative expense",
    sql_var_detail_v1[amount_type] = "Budget",
    sql_var_detail_v1[dept_code] = "GA",
    ALL(dept_id[Dept_Code])
)
```
<br>

**🌱 GA allocation ratio**  
Each department's share of total company gross profit — the basis used to split GA costs. Full reasoning in `semantic-model.md`.

💧 A small bug this caused: an early version of this measure multiplied the decimal by 100 inside the same measure for display purposes, then that 
same measure got reused as a multiplier elsewhere that is silently corrupting downstream calculations. Fixed by keeping them permanently separate: 
- `Allocated Rate Actual`/`Allocated Rate Budget` return the plain decimal (used inside other measures like GA Allocated Personnel Actual)
- `Allocated % Actual`/`Allocated % Budget` multiply by 100 purely for display.

```dax
Allocated Rate Actual = 
VAR DeptGP = [Gross Profit Actual]
VAR TotalGP = 
    CALCULATE(
        [Gross Profit Actual],
        ALL(dept_id[Dept_Code]),
        dept_id[Dept_Code] <> "GA"
    )
RETURN
    DIVIDE(DeptGP, TotalGP)
```
```dax
Allocated Rate Budget = 
VAR DeptGP = [Gross Profit Budget]
VAR TotalGP = 
    CALCULATE(
        [Gross Profit Budget],
        ALL(dept_id[Dept_Code]),
        dept_id[Dept_Code] <> "GA"
    )
RETURN
    DIVIDE(DeptGP, TotalGP)
```
```dax
Allocated % Actual = [Allocated Rate Actual] * 100
```
```dax
Allocated % Budget = [Allocated Rate Budget] * 100
```
<br>

**🌱 GA allocated cost (distributed into each department)**

```dax
GA Allocated Personnel Actual = [GA Personnel Actual] * [Allocated Rate Actual]

```
```dax
GA Allocated Personnel Budget = [GA Personnel Budget] * [Allocated Rate Budget]
```
```dax
GA Allocated Administrative Actual = [GA Administrative Actual] * [Allocated Rate Actual]
```
```dax
GA Allocated Administrative Budget = [GA Administrative Budget] * [Allocated Rate Budget]
```
```dax
GA Allocated Actual = 
VAR CurrentLine = SELECTEDVALUE('Income Statement Line Items'[line_item])
RETURN
    SWITCH(
        TRUE(),
        CurrentLine = "Personnel Expense", [GA Allocated Personnel Actual],
        CurrentLine = "Administrative expense", [GA Allocated Administrative Actual],
        CurrentLine = "SG&A Expenses", [GA Allocated Personnel Actual] + [GA Allocated Administrative Actual],
        CurrentLine = "Operating profit", [GA Allocated Personnel Actual] + [GA Allocated Administrative Actual],
        BLANK()
    )
```
```dax
GA Allocated Budget = 
VAR CurrentLine = SELECTEDVALUE('Income Statement Line Items'[line_item])
RETURN
    SWITCH(
        TRUE(),
        CurrentLine = "Personnel Expense", [GA Allocated Personnel Budget],
        CurrentLine = "Administrative expense", [GA Allocated Administrative Budget],
        CurrentLine = "SG&A Expenses", [GA Allocated Personnel Budget] + [GA Allocated Administrative Budget],
        CurrentLine = "Operating profit", [GA Allocated Personnel Budget] + [GA Allocated Administrative Budget],
        BLANK()
    )
```
<br>

**🌱 Amount rollups: plain vs. "Incl GA"**  
Department pages use the "Incl GA" versions. The Overview page uses the plain versions — using "Incl GA" there would double-count GA costs.

```dax
Amount Actual = 
VAR CurrentLine = SELECTEDVALUE('Income Statement Line Items'[line_item])
VAR CurrentSub = SELECTEDVALUE('Income Statement Line Items'[account_category_sub])
RETURN
    SWITCH(
        TRUE(),
        CurrentSub = "Revenue", [Revenue Actual],
        CurrentSub = "COGS", [COGS Actual],
        CurrentSub = "Personnel Expense", [Personnel Actual],
        CurrentSub = "Administrative expense", [Administrative Actual],
        CurrentSub = "Freight expense", [Freight Actual],
        CurrentSub = "Marketing expense", [Marketing Actual],
        CurrentSub = "R&D", [RD Actual],
        CurrentLine = "Gross Profit", [Gross Profit Actual],
        CurrentLine = "SG&A Expenses", [Personnel Actual] + [Administrative Actual] + [Freight Actual] + [Marketing Actual] + [RD Actual],
        CurrentLine = "Operating profit", [Gross Profit Actual] - ([Personnel Actual] + [Administrative Actual] + [Freight Actual] + [Marketing Actual] + [RD Actual]),
        BLANK()
    )
```
```dax
Amount Budget = 
VAR CurrentLine = SELECTEDVALUE('Income Statement Line Items'[line_item])
VAR CurrentSub = SELECTEDVALUE('Income Statement Line Items'[account_category_sub])
RETURN
    SWITCH(
        TRUE(),
        CurrentSub = "Revenue", [Revenue Budget],
        CurrentSub = "COGS", [COGS Budget],
        CurrentSub = "Personnel Expense", [Personnel Budget],
        CurrentSub = "Administrative expense", [Administrative Budget],
        CurrentSub = "Freight expense", [Freight Budget],
        CurrentSub = "Marketing expense", [Marketing Budget],
        CurrentSub = "R&D", [RD Budget],
        CurrentLine = "Gross Profit", [Gross Profit Budget],
        CurrentLine = "SG&A Expenses", [Personnel Budget] + [Administrative Budget] + [Freight Budget] + [Marketing Budget] + [RD Budget],
        CurrentLine = "Operating profit", [Gross Profit Budget] - ([Personnel Budget] + [Administrative Budget] + [Freight Budget] + [Marketing Budget] + [RD Budget]),
        BLANK()
    )
```
```dax
Amount Actual (incl GA) = 
VAR CurrentLine = SELECTEDVALUE('Income Statement Line Items'[line_item])
RETURN
    IF(
        CurrentLine = "Operating profit",
        [Amount Actual] - [GA Allocated Actual],
        [Amount Actual] + [GA Allocated Actual]
    )
```
```dax
Amount Budget (incl GA) = 
VAR CurrentLine = SELECTEDVALUE('Income Statement Line Items'[line_item])
RETURN
    IF(
        CurrentLine = "Operating profit",
        [Amount Budget] - [GA Allocated Budget],
        [Amount Budget] + [GA Allocated Budget]
    )
```
<br>

**🌱 % of Revenue**

```dax
% of Rev Actual = 
VAR RevenueAmt = 
    CALCULATE(
        [Amount Actual],
        REMOVEFILTERS ( 'Income Statement Line Items' ),
        'Income Statement Line Items'[line_item] = "Revenue"
    )
RETURN
    DIVIDE ( [Amount Actual], RevenueAmt )*100
```
```dax
% of Rev Budget = 
VAR RevenueAmt = 
    CALCULATE(
        [Amount Budget],
        REMOVEFILTERS ( 'Income Statement Line Items' ),
        'Income Statement Line Items'[line_item] = "Revenue"
    )
RETURN
    DIVIDE ( [Amount Budget], RevenueAmt )*100
```
```dax
% of Rev Actual Incl GA = 
VAR RevenueAmt = 
    CALCULATE(
        [Amount Actual (Incl GA)],
        REMOVEFILTERS ( 'Income Statement Line Items' ),
        'Income Statement Line Items'[line_item] = "Revenue"
    )
RETURN
    DIVIDE ( [Amount Actual (Incl GA)], RevenueAmt )*100
```
```dax
% of Rev Budget Incl GA = 
VAR RevenueAmt = 
    CALCULATE(
        [Amount Budget (Incl GA)],
        REMOVEFILTERS ( 'Income Statement Line Items' ),
        'Income Statement Line Items'[line_item] = "Revenue"
    )
RETURN
    DIVIDE ( [Amount Budget (Incl GA)], RevenueAmt )*100
```
<br>

**🌱 Variance measures**

```dax
Var Amount = [Amount Actual (incl GA)] - [Amount Budget (incl GA)]
```
```dax
Var Amount (Company) = [Amount Actual] - [Amount Budget]
```
```dax
Var % = DIVIDE([Var Amount], [Amount Budget (incl GA)])*100
```
```dax
Var % (Company) = DIVIDE([Var Amount (Company)], [Amount Budget])*100
```
```dax
Waterfall Value = 
VAR CurrentSign = SELECTEDVALUE('Income Statement Line Items'[waterfall_sign])
RETURN
    IF(
        NOT ISBLANK(CurrentSign),
        [Var Amount] * CurrentSign
    )
```
<br>

**🌱 Expense breakdown by category**  
Powers the Overview page's pie chart and waterfall chart, showing which expense category is driving the variance — independent of department.

```dax
Expense Actual by Category = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[amount_type] = "Actual"
)
```
```dax
Expense Budget by Category = 
CALCULATE(
    SUM(sql_var_detail_v1[amount]),
    sql_var_detail_v1[amount_type] = "Budget"
)
```
```dax
Expense Var Abs by Category = 
ABS([Expense Actual by Category] - [Expense Budget by Category])
```
```dax
Expense Actual by Category (Incl GA) = 
VAR CurrentCat = SELECTEDVALUE(sql_var_detail_v1[account_category_sub])
RETURN
SWITCH(
    TRUE(),
    CurrentCat = "Personnel Expense", [Personnel Actual] + [GA Allocated Personnel Actual],
    CurrentCat = "Administrative expense", [Administrative Actual] + [GA Allocated Administrative Actual],
    CurrentCat = "Freight expense", [Freight Actual],
    CurrentCat = "Marketing expense", [Marketing Actual],
    CurrentCat = "R&D", [RD Actual],
    BLANK()
)
```
```dax
Expense Budget by Category (Incl GA) = 
VAR CurrentCat = SELECTEDVALUE(sql_var_detail_v1[account_category_sub])
RETURN
SWITCH(
    TRUE(),
    CurrentCat = "Personnel Expense", [Personnel Budget] + [GA Allocated Personnel Budget],
    CurrentCat = "Administrative expense", [Administrative Budget] + [GA Allocated Administrative Budget],
    CurrentCat = "Freight expense", [Freight Budget],
    CurrentCat = "Marketing expense", [Marketing Budget],
    CurrentCat = "R&D", [RD Budget],
    BLANK()
)
```
```dax
Expense Var Abs by Category (Incl GA) = 
ABS([Expense Actual by Category (Incl GA)] - [Expense Budget by Category (Incl GA)])
```
<br>

**🌱 AI summary text**  
Reads the matching row from the `ai_summary` table, filtered to the current page's department. The Overview page has no department filter, so it defaults to `"ALL"`.

```dax
AI Summary Text =
VAR CurrentDept = SELECTEDVALUE(dept_id[Dept_Code], "ALL")
RETURN
CALCULATE(
    SELECTEDVALUE(sql_aitext[summary_text]),
    sql_aitext[dept_code] = CurrentDept,
    sql_aitext[month] = DATE(2025,12,1)
)
```
