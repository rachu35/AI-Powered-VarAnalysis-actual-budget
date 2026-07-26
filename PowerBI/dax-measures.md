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

The GA department's own raw Personnel/Administrative costs, before being
distributed out to the four operating departments.

```dax
GA Personnel Actual = ...
GA Personnel Budget = ...
GA Administrative Actual = ...
GA Administrative Budget = ...
GA Allocated Actual = ...    -- total GA pool (Personnel + Administrative)
GA Allocated Budget = ...
```
<br>

**🌱 GA allocation ratio**

Each department's share of total company gross profit — the basis used to
split GA costs. Full reasoning in `semantic-model.md`.

```dax
Allocated % Actual = ...
Allocated % Budget = ...
```

**⚠️ A bug this caused:** `Allocated Rate Actual` originally multiplied a
decimal by 100 for display ("55.17" instead of "0.5517"), but was reused as
a multiplier elsewhere, silently corrupting downstream calculations. Fixed
by splitting it into two:

```dax
Allocated Rate Actual (calc) = ...      -- plain decimal, used inside other measures
Allocated Rate Actual (display) = ...   -- calc * 100, used only in visuals
Allocated Rate Budget (calc) = ...
Allocated Rate Budget (display) = ...
```

**🌱 GA allocated cost (distributed into each department)**

```dax
GA Allocated Personnel Actual = ...
GA Allocated Personnel Budget = ...
GA Allocated Administrative Actual = ...
GA Allocated Administrative Budget = ...
```

**🌱 Amount rollups: plain vs. "Incl GA"**

Department pages use the "Incl GA" versions. The Overview page uses the
plain versions — using "Incl GA" there would double-count GA costs.

```dax
Amount Actual = ...
Amount Budget = ...
Amount Actual (incl GA) = ...
Amount Budget (incl GA) = ...
```

**🌱 % of Revenue**

```dax
% of Rev Actual = ...
% of Rev Budget = ...
% of Rev Actual Incl GA = ...
% of Rev Budget Incl GA = ...
```

**🌱 Variance measures**

```dax
Var Amount = ...
Var Amount (Company) = ...
Var % = ...
Var % (Company) = ...
Waterfall Value = ...
```

**🌱 Expense breakdown by category**

Powers the Overview page's pie chart and waterfall chart, showing which
expense category is driving the variance — independent of department.

```dax
Expense Actual by Category = ...
Expense Budget by Category = ...
Expense Var Abs by Category = ...
Expense Actual by Category (Incl GA) = ...
Expense Budget by Category (Incl GA) = ...
Expense Var Abs by Category (Incl GA) = ...
```


**🌱 AI summary text**

Reads the matching row from the `ai_summary` table, filtered to the current
page's department. The Overview page has no department filter, so it
defaults to `"ALL"`.

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
