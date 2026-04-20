# PrayaanCapital_Assignment

Dashboard Link: https://adb-7405618761087876.16.azuredatabricks.net/dashboardsv3/01f13cf22739161ab772f0543505ba85/published?o=7405618761087876


**First Round Assignment for Prayaan Capital**

I have started this assignment thinking of it as mid range assignment for me as a 3 yrs experienced data engineer but When I look into the data shared by interviewer, I got to know that why they gave 3 days to complete the assignment. As I don't have experience with financial data in data Engineering field it took so much time understanding the task while doing. 

First I worked with given data and I got to know that there is alot of relational inconsistency between data and end result was a disaster. So I generated my own csv data through Gemini. I tried multiple time to get desired data as given in task. It took me 4 prompts. 

Instead revamping/rewriting all the existing code I just modified download csv file name as per code and then run it through to save a lot of time. 

I started with check negative values in amount due and future dates completeing the data quality check with de duplication. 

After cleaning the table I wrote them to silver from bronze containers. 

Here starts the real task. Initiall I did a big mistake of joining all tables and pulling aggregated data spark.sql with CTEs, Window Functions. While working on the task I slept thinking of it and woke up suddenly with a blink of data modelling. As I don't have practice in data modelling I did not model the data aggregated tables. After waking up I compleleted revamped code in Silver folder with creating aggregated dataset with below columns and I kept repayments table as it is.

**fact table**
-loan_id:string
-borrower_id:string
-badge:string
-urs_score:integer
-branch_id:string
-principal:integer
-outstanding_amount:double
**repayments_table**
-repayment_id:string
-loan_id:string
-installment_no:integer
-due_date:date
-paid_date:date
-amount_due:double
-amount_paid:double
-days_past_due:integer
-repayment_status:string

Now I have come to conscious with clean and aggregated data for gold. 

 I started with fact_table to generate daily snapshot

  Portfolio Snapshot (daily grain):
- total_active_loans
- total_outstanding_amount
- average_urs_score
- percentage_overdue_loans
- percentage_severely_overdue (DPD > 30)
I purely used CTE and simple spark.sql for this task. I have generated individual tables for individual asks.

Then I have used the same fact_table to generate second ask of risk_segmentation which is very cruical.

Risk Segmentation (date x urs_badge):
- loan_count
- outstanding_amount
- average_dpd
- default_rate_proxy (DPD > 60)

I have used Multiple CTE and Window functions to generate this ask. I have used Badge column as Anchor for this query. 

The Final part and Critical. 
I had to use the joined table of Repayments and fact table to generate this ask. 
. Early Warning Indicators (date x branch_id):
- repayment_slippage_rate
- avg_dpd_change_vs_7_day_avg
- risk_flag (Low / Medium / High)

Honestly, I did not understand this ask I have used Gemini and Chatgpt to interpret the ask I have used Gemini code to get help for this. I probable revamped this complete ask 4 times and then went to Gemini to help me which ultimately worked. 
Here I have used CTE, Joins, Subqueries, Window Functions and what not of SQL. 

I started with creating joined table then creating a daily branch aggregates with grouping of due_date and branch_id then I have branch aggregates to create rolling aggregates with the help of window functions. 


Architecture:
- I have used Azure and Data Bricks for this task and used GIT for version Controlling DB Notebooks. 
- I have implemented Medallion Architecture for storing tables. I have stored Gold table in Delta format. 
- I have implemented Start schema with one fact table and one dimension table.
- I have usde Power BI to visualise the Gold Tables. 

