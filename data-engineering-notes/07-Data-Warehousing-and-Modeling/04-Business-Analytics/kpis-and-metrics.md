# KPIs and Metrics in Data Engineering and Analytics

## 1. Introduction

Businesses use data to measure performance, understand trends, and make decisions.

To do this, they define **metrics** and **Key Performance Indicators (KPIs)**.

- **Metric:** A measurable value used to track or analyze business activity.
- **KPI:** A metric selected as a key indicator of progress toward an important business objective.

For example, an e-commerce company may track:
- Total revenue.
- Number of orders.
- Average order value.
- Customer retention rate.
- Order delivery rate.

These metrics help the business understand its performance.

However, not every metric is a KPI. A KPI is directly connected to an important business goal.

**Key idea:** Metrics measure business activity; KPIs measure progress toward important business objectives.

---

## 2. What Is a Metric?

A metric is a quantitative measurement used to evaluate a process, activity, or business outcome.

### Examples

| Metric | What It Measures |
|---|---|
| Total revenue | Money generated from sales |
| Number of orders | Order volume |
| Average order value | Average revenue per order |
| Customer acquisition cost | Cost of acquiring a customer |
| Delivery time | Time taken to deliver an order |
| Website conversion rate | Percentage of visitors who complete a desired action |

### Example

Suppose an e-commerce company processes the following orders:

| order_id | sales_amount |
|---|---:|
| O001 | 5,000 |
| O002 | 3,000 |
| O003 | 2,000 |

Total revenue:

\[
5,000 + 3,000 + 2,000 = 10,000
\]

Number of orders:

\[
3
\]

Average order value:

\[
\frac{10,000}{3} = 3,333.33
\]

These are business metrics that help measure sales performance.

---

## 3. What Is a KPI?

A Key Performance Indicator is a measurable value used to evaluate progress toward a specific business objective.

A KPI should be connected to a business goal.

### Example

Business objective:

Increase customer satisfaction by improving delivery performance.

Possible KPI:

**On-time delivery rate**

\[
\text{On-Time Delivery Rate} =
\frac{\text{Orders Delivered On Time}}
{\text{Eligible Delivered Orders}} \times 100
\]

Suppose:
- Eligible delivered orders = 1,000
- Orders delivered on time = 920

\[
\text{On-Time Delivery Rate} =
\frac{920}{1000} \times 100 = 92\%
\]

If the business target is 95%, the KPI indicates that delivery performance needs improvement.

### Characteristics of a Good KPI

A useful KPI should be:

1. Relevant to a business objective.
2. Clearly defined.
3. Measurable using reliable data.
4. Understandable to stakeholders.
5. Associated with a target or expected performance.
6. Measured over a defined time period.
7. Actionable when performance changes.

---

## 4. Difference Between Metrics and KPIs

| Aspect | Metric | KPI |
|---|---|---|
| Meaning | Measures an activity or result | Measures progress toward a key objective |
| Purpose | Understand performance | Evaluate important business goals |
| Business importance | Can be operational or analytical | Directly tied to strategic priorities |
| Target | May not have a target | Usually has a target or threshold |
| Example | Number of website visits | Conversion rate target |
| Scope | Broad | Selective and goal-oriented |

### Example

An e-commerce company tracks:

- Website visitors.
- Page views.
- Number of orders.
- Conversion rate.
- Revenue.

If the company's main objective is to increase online sales conversion, conversion rate may be a KPI.

The remaining measurements can provide supporting context.

**Important:** A KPI is a type of metric, but not every metric is a KPI.

---

## 5. Common Business KPIs and Metrics

### 5.1 Sales KPIs

| KPI / Metric | Formula |
|---|---|
| Total Revenue | Sum of qualifying sales revenue |
| Average Order Value | Revenue / Number of Orders |
| Sales Growth | (Current Revenue - Previous Revenue) / Previous Revenue × 100 |
| Order Count | Count of qualifying orders |
| Revenue per Customer | Revenue / Number of Customers |

### 5.2 Customer KPIs

| KPI / Metric | Formula |
|---|---|
| Customer Acquisition Cost | Acquisition Spend / New Customers Acquired |
| Customer Retention Rate | Retained Customers / Eligible Starting Customers × 100 |
| Customer Churn Rate | Lost Customers / Eligible Starting Customers × 100 |
| Repeat Purchase Rate | Customers with Repeat Purchases / Eligible Customers × 100 |
| Customer Lifetime Value | Estimated value generated over the customer relationship |

The exact definitions of customer retention, churn, and lifetime value depend on the business model and measurement period.

### 5.3 Operational KPIs

| KPI / Metric | Formula |
|---|---|
| On-Time Delivery Rate | On-Time Deliveries / Eligible Deliveries × 100 |
| Average Delivery Time | Total Delivery Duration / Delivered Orders |
| Order Fulfillment Rate | Fulfilled Orders / Eligible Orders × 100 |
| Defect Rate | Defective Units / Inspected Units × 100 |
| Inventory Turnover | Cost of Goods Sold / Average Inventory |

### 5.4 Marketing KPIs

| KPI / Metric | Formula |
|---|---|
| Click-Through Rate | Clicks / Impressions × 100 |
| Conversion Rate | Conversions / Eligible Visitors or Sessions × 100 |
| Cost per Click | Advertising Spend / Clicks |
| Return on Ad Spend | Attributed Revenue / Advertising Spend |
| Lead Conversion Rate | Converted Leads / Eligible Leads × 100 |

The denominator must be clearly defined. For example, conversion rate can use visitors, sessions, or qualified leads depending on the business context.

---

## 6. KPI vs Target vs Threshold

These terms are related but different.

### KPI

The metric used to evaluate business performance.

Example: On-time delivery rate.

### Target

The desired value the business wants to achieve.

Example: At least 95%.

### Threshold

A boundary used to identify a performance condition or trigger an alert.

Example:
- Green: 95% or higher.
- Warning: 90% to below 95%.
- Critical: Below 90%.

### Example

| Component | Value |
|---|---|
| KPI | On-time delivery rate |
| Current value | 92% |
| Target | 95% |
| Warning threshold | Below 95% |
| Critical threshold | Below 90% |

A threshold should reflect business impact and operational requirements, not an arbitrary number.

---

## 7. Leading and Lagging Indicators

KPIs can also be classified based on when they provide information about performance.

### 7.1 Leading Indicators

Leading indicators provide signals that may influence future outcomes.

Examples:
- Number of qualified sales leads.
- Website engagement.
- Training completion rate.
- Number of orders awaiting dispatch.

### 7.2 Lagging Indicators

Lagging indicators measure outcomes that have already happened.

Examples:
- Total revenue.
- Customer churn.
- Completed sales.
- Quarterly profit.

### Comparison

| Leading Indicator | Lagging Indicator |
|---|---|
| Provides an early signal | Measures an achieved result |
| Can help predict outcomes | Confirms past performance |
| Often supports proactive action | Often supports performance evaluation |
| Example: Qualified leads | Example: Revenue |

A business usually needs both.

---

## 8. Designing KPIs from Business Requirements

A Data Engineer or Analytics Engineer should understand how a KPI is defined before building its data pipeline.

Consider this requirement:

"The business wants to increase monthly revenue by 15%."

### Step 1: Understand the Objective

Business objective: Increase monthly revenue.

### Step 2: Identify the KPI

Monthly revenue growth percentage.

### Step 3: Define the Formula

\[
\text{Revenue Growth} =
\frac{\text{Current Month Revenue} - \text{Previous Month Revenue}}
{\text{Previous Month Revenue}} \times 100
\]

### Step 4: Define the Business Rules

Clarify:
- Does revenue include taxes?
- Are canceled orders excluded?
- Are refunds deducted?
- Are discounts included?
- Which order statuses qualify?
- Which date determines the revenue month?

### Step 5: Identify Required Data

Possible source data:
- Orders.
- Order items.
- Payments.
- Refunds.
- Product details.
- Order dates.

### Step 6: Design the Data Model

Possible tables:
- `fact_sales`
- `dim_date`
- `dim_product`
- `dim_customer`

The fact table should have a clearly defined grain and contain the measures needed for the KPI.

### Step 7: Implement the Calculation

Calculate revenue using the agreed business definition.

### Step 8: Validate the Result

Reconcile the KPI with trusted source totals and confirm that the same business rules are applied consistently.

---

## 9. Implementing KPIs Using SQL

Consider an order table:

`orders`

| order_id | order_date | order_status | sales_amount |
|---|---|---|---:|
| O001 | 2026-10-01 | Delivered | 5,000 |
| O002 | 2026-10-02 | Delivered | 3,000 |
| O003 | 2026-10-03 | Cancelled | 2,000 |
| O004 | 2026-10-04 | Delivered | 4,000 |

For the following examples, assume the business defines revenue as the sales amount of delivered orders, with no additional tax, refund, or discount adjustments.

### 9.1 Total Revenue

```sql
SELECT
    SUM(sales_amount) AS total_revenue
FROM orders
WHERE order_status = 'Delivered';
```

Result:

\[
5,000 + 3,000 + 4,000 = 12,000
\]

### 9.2 Total Delivered Orders

```sql
SELECT
    COUNT(*) AS delivered_orders
FROM orders
WHERE order_status = 'Delivered';
```

Result: 3.

This assumes one row per order.

### 9.3 Average Order Value

```sql
SELECT
    SUM(sales_amount) / NULLIF(COUNT(*), 0) AS average_order_value
FROM orders
WHERE order_status = 'Delivered';
```

Result:

\[
12,000 / 3 = 4,000
\]

### 9.4 Monthly Revenue

```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS sales_year,
    EXTRACT(MONTH FROM order_date) AS sales_month,
    SUM(sales_amount) AS monthly_revenue
FROM orders
WHERE order_status = 'Delivered'
GROUP BY
    EXTRACT(YEAR FROM order_date),
    EXTRACT(MONTH FROM order_date)
ORDER BY
    sales_year,
    sales_month;
```

Date extraction syntax varies by SQL engine.

### 9.5 Revenue Growth

```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', order_date) AS sales_month,
        SUM(sales_amount) AS revenue
    FROM orders
    WHERE order_status = 'Delivered'
    GROUP BY DATE_TRUNC('month', order_date)
),
revenue_comparison AS (
    SELECT
        sales_month,
        revenue,
        LAG(revenue) OVER (
            ORDER BY sales_month
        ) AS previous_month_revenue
    FROM monthly_sales
)
SELECT
    sales_month,
    revenue,
    previous_month_revenue,
    CASE
        WHEN previous_month_revenue IS NULL
          OR previous_month_revenue = 0
        THEN NULL
        ELSE
            (revenue - previous_month_revenue)
            * 100.0 / previous_month_revenue
    END AS revenue_growth_percentage
FROM revenue_comparison
ORDER BY sales_month;
```

This calculates growth against the previous month present in the aggregated result. If months with zero sales must also appear, use a calendar/date dimension and fill missing months appropriately.

---

## 10. Implementing KPIs Using PySpark

The same business calculations can be implemented using PySpark.

### Example Dataset

```python
from pyspark.sql import functions as F

data = [
    ("O001", "Delivered", 5000),
    ("O002", "Delivered", 3000),
    ("O003", "Cancelled", 2000),
    ("O004", "Delivered", 4000)
]

columns = [
    "order_id",
    "order_status",
    "sales_amount"
]

df = spark.createDataFrame(data, columns)
```

### Total Revenue

```python
revenue_df = df.filter(
    F.col("order_status") == "Delivered"
).agg(
    F.sum("sales_amount").alias("total_revenue")
)

revenue_df.show()
```

### Delivered Order Count

```python
order_count_df = df.filter(
    F.col("order_status") == "Delivered"
).agg(
    F.countDistinct("order_id").alias("delivered_orders")
)

order_count_df.show()
```

### Average Order Value

```python
aov_df = df.filter(
    F.col("order_status") == "Delivered"
).agg(
    (
        F.sum("sales_amount") /
        F.countDistinct("order_id")
    ).alias("average_order_value")
)

aov_df.show()
```

These examples assume one row per order. If the input has one row per order item, aggregate order-level amounts carefully to avoid counting each order multiple times.

---

## 11. KPI Data Pipeline Architecture

A typical analytics pipeline can calculate and publish KPIs using the following flow:

```text
Source Systems
      |
      v
Bronze Layer
Raw Orders, Payments, Customers
      |
      v
Silver Layer
Cleaned and Validated Data
      |
      v
Gold Layer
Business Metrics and KPI Tables
      |
      v
BI Dashboard / Reports
      |
      v
Business Decisions
```

### Bronze Layer

Stores raw or minimally processed source data.

### Silver Layer

Cleans and standardizes data.

Examples:
- Remove or identify duplicates.
- Validate required fields.
- Standardize order statuses.
- Handle invalid records.
- Apply agreed transformation rules.

### Gold Layer

Contains business-ready data.

Examples:
- Monthly revenue.
- Average order value.
- Customer retention.
- On-time delivery rate.

### BI Layer

Tools such as Power BI, Tableau, or other reporting systems consume Gold data and display KPI dashboards.

**Important:** A Data Engineer is responsible for reliable data pipelines and correct metric inputs. Business owners and analysts should agree on KPI definitions and interpretation.

---

## 12. KPI Data Quality and Monitoring

A KPI is useful only when its underlying data is trustworthy.

A pipeline should monitor both technical execution and business data quality.

### Important Checks

| Check | Example |
|---|---|
| Completeness | Every qualifying order has an order date |
| Uniqueness | Order identifiers are unique at order grain |
| Validity | Sales amount is within expected business limits |
| Consistency | Order status follows standard definitions |
| Reconciliation | Warehouse revenue matches trusted source totals |
| Freshness | KPI data is updated within the agreed SLA |

### Example: Revenue Reconciliation

Suppose:
- Source revenue = ₹10,00,000.
- Warehouse revenue = ₹9,80,000.

Difference:

\[
10,00,000 - 9,80,000 = 20,000
\]

The pipeline should investigate the difference.

Possible causes:
- Missing orders.
- Duplicate removal.
- Incorrect status filters.
- Late-arriving records.
- Refund or discount logic differences.
- Incorrect joins.

Do not assume a difference is an error until the business rules and expected exclusions have been checked.

---

## 13. Common KPI Design Mistakes

### 13.1 Ambiguous Definitions

Different teams calculate revenue differently.

**Solution:** Document the formula, filters, date rules, and exclusions.

### 13.2 Wrong Denominator

A conversion rate may use visitors, sessions, or qualified leads.

**Solution:** Specify the exact denominator and measurement population.

### 13.3 Duplicate Records

Duplicate orders can inflate revenue and order counts.

**Solution:** Validate uniqueness at the correct grain.

### 13.4 Incorrect Aggregation

Averages or percentages are summed or averaged incorrectly.

**Solution:** Store the underlying numerator and denominator when possible and recalculate the ratio at the required level.

### 13.5 Ignoring Time Periods

A monthly KPI may use order date in one report and delivery date in another.

**Solution:** Define the business event date for every KPI.

### 13.6 Measuring Too Many KPIs

Too many KPIs can make it difficult to focus on important business outcomes.

**Solution:** Select a small set of meaningful KPIs aligned with business goals and retain supporting metrics for investigation.

### 13.7 Ignoring Data Freshness

A dashboard may show an old KPI value even when the pipeline is running successfully.

**Solution:** Monitor data freshness separately from job execution status.

---

## 14. Common Interview Questions and Answers

### Q1. What is a KPI?

A KPI is a measurable indicator used to evaluate progress toward an important business objective.

### Q2. What is the difference between a metric and a KPI?

A metric measures an activity or result. A KPI is a metric selected because it directly indicates progress toward an important business goal.

### Q3. Give examples of business KPIs.

Examples include revenue growth, customer retention rate, conversion rate, customer acquisition cost, and on-time delivery rate.

### Q4. What is the difference between a KPI and a target?

A KPI is what the business measures. A target is the desired value the business wants to achieve.

### Q5. What are leading and lagging indicators?

Leading indicators provide early signals about future outcomes. Lagging indicators measure results that have already occurred.

### Q6. How do you calculate average order value?

Divide qualifying revenue by the number of qualifying orders.

\[
AOV = \frac{Total\ Revenue}{Number\ of\ Orders}
\]

### Q7. How do you calculate revenue growth?

Subtract previous-period revenue from current-period revenue, divide by previous-period revenue, and multiply by 100.

### Q8. How do you ensure KPI accuracy?

Define the business formula, validate source data, maintain the correct grain, prevent duplicate counting, apply consistent filters, and reconcile the output.

### Q9. What is KPI data freshness?

It measures how recently the data used to calculate a KPI was updated.

### Q10. What is the role of a Data Engineer in KPI reporting?

A Data Engineer builds reliable ingestion and transformation pipelines, prepares business-ready datasets, implements agreed calculation logic where appropriate, and monitors data quality and freshness.

### Q11. Why should KPI definitions be documented?

Documentation ensures that different teams calculate and interpret the same metric consistently.

### Q12. Can every metric be a KPI?

No. A KPI should be connected to an important business objective. Many metrics are useful for supporting analysis but are not key indicators.

### Q13. How would you investigate an unexpected revenue drop?

Check data freshness, source completeness, duplicate handling, status filters, refunds, late-arriving data, transformation logic, and reconciliation results before concluding that business revenue actually declined.

### Q14. Why is the grain important when calculating KPIs?

The grain determines what each record represents. An incorrect grain can inflate counts, duplicate revenue, or produce incorrect averages.

### Q15. What is the difference between operational metrics and business KPIs?

Operational metrics monitor process execution, such as pipeline duration or failed tasks. Business KPIs evaluate outcomes, such as revenue growth or customer retention. Both are important but serve different purposes.

---

## 15. Quick Revision

| Concept | Meaning |
|---|---|
| Metric | Measurable business value |
| KPI | Metric tied to an important business objective |
| Target | Desired performance value |
| Threshold | Boundary for warning or action |
| Leading indicator | Early signal of future performance |
| Lagging indicator | Measurement of past outcome |
| Data freshness | How recently the data was updated |
| KPI validation | Checking whether the result is correct and reliable |

### Important Formulas

**Average Order Value**

\[
AOV = \frac{Total\ Revenue}{Number\ of\ Orders}
\]

**Revenue Growth**

\[
Growth\% =
\frac{Current - Previous}{Previous} \times 100
\]

**Conversion Rate**

\[
Conversion\% =
\frac{Conversions}{Eligible\ Population} \times 100
\]

**Customer Acquisition Cost**

\[
CAC = \frac{Acquisition\ Spend}{New\ Customers}
\]

**On-Time Delivery Rate**

\[
OTD\% =
\frac{On-Time\ Deliveries}{Eligible\ Deliveries} \times 100
\]

### Most Important Interview Points

1. A KPI is a metric connected to a business objective.
2. Not every metric is a KPI.
3. Every KPI needs a clear business definition.
4. Formula, filters, denominator, and time period must be defined.
5. Correct grain is essential for accurate KPI calculations.
6. Data quality and reconciliation protect KPI accuracy.
7. Data freshness is different from pipeline job success.
8. Gold-layer datasets commonly support business KPI reporting.

### One-Minute Interview Answer

A metric is a measurable value used to understand business performance, while a KPI is a metric directly connected to an important business objective.

For example, an e-commerce company may track total revenue, order count, and average order value. If its objective is to improve sales performance, monthly revenue growth may be selected as a KPI.

As a Data Engineer, I first understand the KPI definition, including its formula, filters, denominator, and reporting period. Then I identify the required source data, design the appropriate fact and dimension model, build the transformation pipeline, and validate the result using data quality checks and reconciliation.

This ensures that dashboards provide accurate and consistent business information.

---

## 16. Self-Assessment

- [ ] Define a metric.
- [ ] Define a KPI.
- [ ] Explain the difference between a metric and KPI.
- [ ] Explain KPI, target, and threshold.
- [ ] Differentiate leading and lagging indicators.
- [ ] Calculate average order value.
- [ ] Calculate revenue growth.
- [ ] Calculate conversion rate.
- [ ] Explain customer acquisition cost.
- [ ] Translate a business objective into a KPI.
- [ ] Identify the source data required for a KPI.
- [ ] Explain the role of Bronze, Silver, and Gold layers.
- [ ] Validate KPI calculations using SQL.
- [ ] Explain how grain affects KPI accuracy.
- [ ] Investigate an unexpected KPI change.
- [ ] Explain data freshness and reconciliation.

**Interview readiness:** Given a business objective, define a suitable KPI, explain its formula and business rules, identify the required data, and describe how you would calculate and validate it in a data pipeline.