### 🪴 Explanation for this query

- `actuals` and `budget` are separate tables with different column names
(`actual_amount` vs. `budget_amount`).
- Power BI's DAX measures need a single,
consistent shape to filter and sum from — one row per account/department/month/
type, with a plain `amount` column and an `amount_type` flag ("Actual" or
"Budget") to distinguish the two.
- This query does that combining and aggregation once, in SQL, so the DAX layer doesn't have to.
- This is used as a custom SQL query when importing into Power BI (not a MySQL view), while Power BI runs it at refresh time and loads the result as the
`sql_var_detail_v1` table.
<br>

**🌱 Query**
```sql
WITH combined AS (
    SELECT account_code, dept_code, month, actual_amount AS amount, 'Actual' AS amount_type
    FROM project_var.actuals

    UNION ALL

    SELECT account_code, dept_code, month, budget_amount AS amount, 'Budget' AS amount_type
    FROM project_var.budget
)

SELECT
    combined.month,
    combined.dept_code,
    c.account_category,
    c.account_category_sub,
    combined.amount_type,
    SUM(combined.amount) AS amount
FROM combined
JOIN project_var.chart_of_accounts c ON combined.account_code = c.account_code
GROUP BY
    combined.month,
    combined.dept_code,
    c.account_category,
    c.account_category_sub,
    combined.amount_type
ORDER BY
    combined.month,
    combined.dept_code; 
```
<br>
