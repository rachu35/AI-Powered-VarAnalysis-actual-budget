### 🪴 Semantic Model

This describes the shape of the Power BI data model. What tables exist, how they relate, and the design decisions behind the GA cost allocation. For the actual DAX formulas, please see `dax-measures.md`.
<br>
<br>

**🌱 Tables**  
| Table | Source | Purpose |
|---|---|---|
| `sql_var_detail_v1` | Imported from MySQL (`actuals` + `budget` + `chart_of_accounts`, joined) | Core fact table: every account line, by department, actual vs. budget |
| `dept_id` | Imported from MySQL | Department code → department name lookup |
| `sql_aitext` | Imported from MySQL (`ai_summary`) | AI-generated variance summaries, one row per department per month |
| `Income Statement Line Items` | Built manually in Power BI via `DATATABLE()` | Controls P&L row order, display labels, and waterfall direction |
- Creating `Income Statement Line Items`
```DAX
Income Statement Line Items = 
DATATABLE(
    "line_item", STRING,
    "display_name", STRING,
    "sort_order", INTEGER,
    "row_type", STRING,
    "account_category_sub", STRING,
    "waterfall_sign", INTEGER,
    {
        {"Revenue", "Revenue", 1, "Detail", "Revenue", 1},
        {"COGS", "　COGS", 2, "Detail", "COGS", -1},
        {"Gross Profit", "Gross Profit", 3, "Subtotal", BLANK(), BLANK()},
        {"SG&A Expenses", "SG&A Expenses", 4, "Subtotal", BLANK(), BLANK()},
        {"Personnel Expense", "　　Personnel Expense", 5, "Detail", "Personnel Expense", -1},
        {"Administrative expense", "　　Administrative expense", 6, "Detail", "Administrative expense", -1},
        {"Freight expense", "　　Freight expense", 7, "Detail", "Freight expense", -1},
        {"Marketing expense", "　　Marketing expense", 8, "Detail", "Marketing expense", -1},
        {"R&D", "　　R&D", 9, "Detail", "R&D", -1},
        {"Operating profit", "Operating profit", 10, "Subtotal", BLANK(), BLANK()}
    }
)
```

<br>

**🌱 Relationships**  
| Table | Relationships | Nodes |
|---|---|---|
| `sql_var_detail_v1[dept_code]` | `dept_id[Dept_Code]` | many-to-one |
| `sql_aitext[dept_code]` | `dept_id[Dept_Code]` | many-to-one |
| `Income Statement Line Items` | n/a (intentionally disconnected — see "Income Statement Line Items" section below) | n/a |
<img width="841" height="575" alt="image" src="https://github.com/user-attachments/assets/1f3997ae-b617-4f02-a4f4-c9c93c2505ff" />  
<br>
<br>

**🌱 The "Income Statement Line Items" table**  
- A manually-built table that exists purely to control how the P&L is displayed, from row order (`sort_order`), the label shown to the user (`display_name` vs. the internal `line_item` key), whether a row is a subtotal (`row_type`), which account category it maps to (`account_category_sub`), and which direction it moves in the waterfall chart (`waterfall_sign`).
- I intentionally made them disconnected from `sql_var_detail_v1` table. I think the measures that need this table's context read it explicitly via `SELECTEDVALUE()` inside the measure, rather than relying on automatic filter propagation.
- The source data only contains transaction level account categories ( Revenue, COGS, Personnel Expense, and so on) which is the same way a general ledger or trial balance works. Subtotal lines like "Gross Profit" and "Operating profit" aren't accounts that exist in the ledger because they're calculated results, derived by summing and subtracting other accounting accounts.
- However, there is a small bug here: Some measures read filter context from `display_name`, others from `line_item`. Since these are two different columns on the same disconnected table, a mismatch meant `SELECTEDVALUE()` returned blank for the mismatched ones, the several `% of Rev` rows showed up empty. I fixed by standardizing on `line_item` and rebuilding context explicitly with `REMOVEFILTERS`. More detail in `dax-measures.md` and the project README.
<br>

**🌱 GA cost allocation logic**  
G&A (Personnel and Administrative only) is a shared cost sitting in its own department (`GA`), not tied to any single operating department. It needs to be distributed across Beauty, Health & Wellness, Household Products, and Food & Beverage before those departments' P&Ls reflect their full cost.

**🌱 Allocation basis: each department's share of total company gross profit**  
The reason is gross profit reflects actual contribution to profitability before overhead, so a higher margin department carries a fairer share of shared costs than a revenue-based split would give it.
