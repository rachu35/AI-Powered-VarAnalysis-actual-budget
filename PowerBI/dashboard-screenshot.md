### 🪴 What's in the dashboard?
The dashboard has 5 pages: a company-wide Overview and 4 department pages (Beauty, Health & Wellness, Household Products, Food & Beverage). All pages share the same layout, having a P&L table, variance visuals, and an AI-generated summary card, with department pages including GA-allocated cost figures that the Overview page deliberately excludes (see semantic-model.md).  

**🌱 Overview**
Company-wide P&L (excl. GA allocation), an expense breakdown by category, a variance waterfall, and GA cost allocation by department. The AI summary card on the right pulls from the `ai_summary` table (`dept_code = 'ALL'`).
<img width="1194" height="562" alt="image" src="https://github.com/user-attachments/assets/c2251e5c-3098-4748-a596-b74dabc99a07" />
<br>
<br>
<br>
**🌱 Beauty (BT)**
Department-level P&L (incl. GA allocation), an Operating Profit KPI card, an Actual vs. Budget comparison, and an expense variance waterfall.
<img width="1196" height="516" alt="image" src="https://github.com/user-attachments/assets/1506ab48-6130-4b86-9917-3536222aadcb" />
<br>
<br>
<br>
**🌱 Health & Wellness (HW)**
Same layout as Beauty, filtered to `Dept_Code = HW`.
<img width="1196" height="553" alt="image" src="https://github.com/user-attachments/assets/d57ce2d9-4409-4be2-a1f4-38121824e147" />
<br>
<br>
<br>
**🌱 Health & Wellness (HW)**
Same layout as Beauty, filtered to `Dept_Code = HP`.
<img width="1197" height="543" alt="image" src="https://github.com/user-attachments/assets/13f970e6-7626-4208-9134-08a228ea5ed9" />
<br>
<br>
<br>
**🌱 Food & Beverage (FB)**
Same layout as Beauty, filtered to `Dept_Code = FB`.  
Note: this department's budget base for Operating Profit is small, so its variance percentage is unusually large — the AI summary describes this qualitatively rather than quoting an exact percentage (see `prompt_templates.md`).
<img width="1197" height="542" alt="image" src="https://github.com/user-attachments/assets/a852c51e-f47d-4ae1-93fa-a3df68d7abb4" />
