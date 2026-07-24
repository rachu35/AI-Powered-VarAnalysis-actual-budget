### What's in the langflow page?

**🪴 Langflow's project- 5 Flow for overall and each dept seperately** 
<img width="831" height="470" alt="image" src="https://github.com/user-attachments/assets/1de9b147-87eb-44a0-a1d3-390b24da57c0" />
<br>
<br>
**🌱 All of pipelines are the same, but the SQL query and prompt templates are a bit different for each.**
<img width="1466" height="617" alt="image" src="https://github.com/user-attachments/assets/530e11f6-568e-4c2a-94bc-4efc11f4f896" />
<br>
<br>
Components used in each flow
- SQL Database (read) — queries `actuals`, `budget`, and `chart_of_accounts` for the department/company scope
- Parser — formats the SQL result into a plain-text block the LLM can read
- Prompt Template — combines instructions + parsed data into the final prompt
- Extended OpenAI Chat — sends the prompt to DeepSeek-V3 via DeepInfra, returns the generated summary
- Prompt Template (second one) — wraps the LLM's summary into a `CALL upsert_ai_summary(...)` SQL statement
- SQL Database (write) — executes that stored procedure call, writing the summary into `ai_summary`
