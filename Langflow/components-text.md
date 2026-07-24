## 🪴 Overall

### 🌱 SQL Database (read)
- Database URL: `MySQL connection to [project_var] (local)`
- SQL Query:  
```sql
SELECT
    c.account_category_sub,
    SUM(a.actual_amount) AS actual_amount,
    SUM(b.budget_amount) AS budget_amount,
    GROUP_CONCAT(DISTINCT a.description SEPARATOR ' | ') AS description,
    NULL AS metric_label, NULL AS metric_actual, NULL AS metric_budget
FROM actuals a
JOIN budget b ON a.account_code=b.account_code AND a.dept_code=b.dept_code AND a.month=b.month
JOIN chart_of_accounts c ON a.account_code=c.account_code
WHERE a.month='2025-12-01'
GROUP BY c.account_category_sub

UNION ALL

SELECT NULL,NULL,NULL,NULL,'Operating Profit',953510,428818
```


### 🌱 Parser
- Template
```
Account: {account_category_sub}, Actual: {actual_amount}, Budget: {budget_amount}, Description: {description}, Metric: {metric_label} Actual: {metric_actual} Budget: {metric_budget}
```


### 🌱 Prompt Template (1)
- Template
```
You are an FP&A analyst writing a monthly variance summary for management.

Write a concise 3-4 sentence summary in English that:
- Starts with a single clear sentence stating whether the department exceeded or missed its operating profit budget. ALWAYS state the dollar amount from the "Operating Profit (Incl GA)" row.
- If the percentage is below 500%, you MUST include it, e.g., "exceeded its operating profit budget by $358,122 (163.5%)".
- Only if the percentage would exceed 500% (rare, happens when the budget base is very small), omit the percentage and instead write "significantly exceeded" — do not use this phrasing for normal percentages under 500%.
- If the budget variance percentage is unusually large due to a small budget base, describe the magnitude qualitatively (e.g., "far exceeded") rather than quoting the exact percentage.
- Highlights the overall performance trend
- Explains the key drivers using the reasons provided, not just repeating numbers
- Uses a professional, business-appropriate tone suitable for a management report
- Does not simply list each line item; instead, weave the causes into a coherent narrative

Below is the December 2025 financial data for the 	company overall (all departments combined), including the specific reasons behind notable variances:

{data}
```
- Note: Rules 2–4 above overlap slightly (an earlier, vaguer version of the "large percentage" rule was kept alongside the more precise 500% threshold that replaced it). Functionally this doesn't change model output, but a cleaner version would consolidate these into a single rule.


### 🌱 Extended OpenAI Chat
- Model Name: `deepseek-ai/DeepSeek-V3`
- API Base URL: `https://api.deepinfra.com/v1/openai`
- API Key: `(Yours)`


### 🌱 Prompt Template (2)
- Template: `CALL upsert_ai_summary('ALL', '2025-12-01', '{summary}');`


### 🌱 SQL Database (write)
- Database URL: `MySQL connection to [project_var] (local)`


### 💧 Change monthly parts
- `WHERE a.month='2025-12-01'`
- `'Operating Profit',953510,428818`
- Prompt Template text: "December 2025" → update to the new month

<br>
<br>
<br>

## 🪴 Each department

### 🌱 SQL Database (read)
- Database URL: `MySQL connection to [project_var] (local)`
- SQL Query:  
```sql
SELECT
    a.account_code, c.account_category_sub, a.actual_amount, b.budget_amount, a.description,
    NULL AS metric_label, NULL AS metric_actual, NULL AS metric_budget
FROM actuals a
JOIN budget b ON a.account_code=b.account_code AND a.dept_code=b.dept_code AND a.month=b.month
JOIN chart_of_accounts c ON a.account_code=c.account_code
WHERE a.dept_code='<DEPT_CODE>' AND a.month='2025-12-01'

UNION ALL

SELECT NULL,NULL,NULL,NULL,NULL,'Operating Profit (Incl GA)',<OP_ACTUAL>,<OP_BUDGET>
```


### 🌱 Parser
- Template
```
Account: {account_category_sub}, Actual: {actual_amount}, Budget: {budget_amount}, Description: {description}, Metric: {metric_label} Actual: {metric_actual} Budget: {metric_budget}
```


### 🌱 Prompt Template (1)
- Template
```
You are an FP&A analyst writing a monthly variance summary for management.

Write a concise 3-4 sentence summary in English that:

Starts with a single clear sentence stating whether the department exceeded or missed its operating profit budget. ALWAYS state the dollar amount from the "Operating Profit (Incl GA)" row.
If the percentage is below 500%, you MUST include it, e.g., "exceeded its operating profit budget by $358,122 (163.5%)".
Only if the percentage would exceed 500% (rare, happens when the budget base is very small), omit the percentage and instead write "significantly exceeded" — do not use this phrasing for normal percentages under 500%.
If the budget variance percentage is unusually large due to a small budget base, describe the magnitude qualitatively (e.g., "far exceeded") rather than quoting the exact percentage.
Highlights the overall performance trend
Explains the key drivers using the reasons provided, not just repeating numbers
Uses a professional, business-appropriate tone suitable for a management report
Does not simply list each line item; instead, weave the causes into a coherent narrative

Below is the December 2025 financial data for the <DEPT_FULL_NAME> department, including the specific reasons behind notable variances:

{data}
```
- Note: Rules 2–4 above overlap slightly (an earlier, vaguer version of the "large percentage" rule was kept alongside the more precise 500% threshold that replaced it). Functionally this doesn't change model output, but a cleaner version would consolidate these into a single rule.

### 🌱 Extended OpenAI Chat
- Model Name: `deepseek-ai/DeepSeek-V3`
- API Base URL: `https://api.deepinfra.com/v1/openai`
- API Key: `(Yours)`


### 🌱 Prompt Template (2)
- Template: `CALL upsert_ai_summary('<DEPT_CODE>', '2025-12-01', '{summary}');`


### 🌱 SQL Database (write)
- Database URL: `MySQL connection to [project_var] (local)`


### 💧 Change monthly parts
- `WHERE a.dept_code='<DEPT_CODE>' AND a.month='2025-12-01'`
- `'Operating Profit (Incl GA)',<OP_ACTUAL>,<OP_BUDGET>`
- Prompt Template text: "December 2025" → update to the new month
- financial data for the `Health & Wellness (HW)` department


### 💧 Dept_code reference
| Dept | `<DEPT_CODE>` | `<DEPT_FULL_NAME>` | `<OP_ACTUAL>` | `<OP_BUDGET>` |
|---|---|---|---|---|
| Beauty | BT | Beauty (BT) | 577167 | 219045 |
| Health & Wellness | HW | Health & Wellness (HW) | 252859 | 146912 |
| Household Products | HP | Household Products (HP) | 91577 | 62256 |
| Food & Beverage | FB | Food & Beverage (FB) | 31906 | 604 |


