### 🪴 upsert_ai_summary

- The Langflow pipeline runs once per department (BT/HW/HP/FB/ALL) per month, and gets re-run often during prompt tuning — the same department/month combination might be generated 5+ times while iterating on wording.
- Without an upsert, every re-run would insert a new row, leaving stale duplicate summaries in the table. This stored procedure makes re-running safe: if a row for that `dept_code` + `month` already exists, it's updated in place instead of duplicated.
- This depends on `ai_summary` having a unique constraint on (`dept_code`, `month`) — that's what `ON DUPLICATE KEY UPDATE` checks against.

**🌱 Definition**

```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `upsert_ai_summary`(
    IN p_dept_code VARCHAR(10),
    IN p_month DATE,
    IN p_summary_text TEXT
)
BEGIN
    INSERT INTO ai_summary (dept_code, month, summary_text)
    VALUES (p_dept_code, p_month, p_summary_text)
    ON DUPLICATE KEY UPDATE
        summary_text = p_summary_text,
        generated_at = NOW();
END
```
