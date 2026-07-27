### 🪴 Database Schema

The database (`project_var`) has 4 source tables which were created via DBeaver's import.  
<br>

**🌱 actuals**
| Column | Type | Description |
|---|---|---|
| transaction_id | VARCHAR | |
| account_code | VARCHAR | Links to chart_of_accounts |
| dept_code | VARCHAR | Links to dept_id |
| month | VARCHAR | |
| actual_amount | DECIMAL | |
| description | TEXT | Variance explanation (nullable) |
<br>

**🌱 budget**
| Column | Type | Description |
|---|---|---|
| transaction_id | VARCHAR | |
| account_code | VARCHAR | Links to chart_of_accounts |
| dept_code | VARCHAR | Links to dept_id |
| month | VARCHAR | |
| budget_amount | DECIMAL | |
<br>

**🌱 chart_of_accounts**
| Column | Type | Description |
|---|---|---|
| account_code | VARCHAR | Primary key |
| account_name | VARCHAR | |
| account_category | VARCHAR | |
| account_category_sub | VARCHAR | e.g. "Personnel Expense" |
<br>

**🌱 dept_id**
| Column | Type | Description |
|---|---|---|
| Department | VARCHAR | Full department name |
| Dept_Code | VARCHAR | |
| Description | TEXT | |

<br>

**🌱 Data cleanup: fixing the `month` column type**  
When first imported, `month` came in as `VARCHAR` (e.g., `"12/1/25"`) instead of `DATE` — this would have broken date-based filtering in DAX
downstream (Power BI would need string comparisons instead of proper `DATE()` comparisons). Fixed by adding a new typed column, converting the values, then swapping it in:
```sql
-- Add a new properly-typed column
ALTER TABLE project_var.actuals ADD COLUMN month_date DATE;

-- Convert the string format (m/d/yy) into a real date
UPDATE project_var.actuals
SET month_date = STR_TO_DATE(month, '%c/%e/%y');

-- Sanity check before proceeding
SELECT month, month_date FROM project_var.actuals LIMIT 15;

-- Drop the old column and rename the new one into its place
ALTER TABLE project_var.actuals DROP COLUMN month;
ALTER TABLE project_var.actuals CHANGE month_date month DATE;

-- Confirm the final structure
DESCRIBE project_var.actuals;
```

Also, Same fix applied to `budget.month`.

