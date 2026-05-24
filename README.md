# 🖥️ SQL Project on Consumer Goods — AtliQ Hardware

<p align="center">
  <img src="https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white"/>
  <img src="https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black"/>
  <img src="https://img.shields.io/badge/Canva-00C4CC?style=for-the-badge&logo=canva&logoColor=white"/>
  <img src="https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge"/>
</p>

> **Codebasics SQL Resume Project Challenge** — Solving 10 real-world business requests for a leading computer hardware manufacturer using SQL, with actionable insights presented to simulated business stakeholders.

---

## 📌 Table of Contents
- [About the Project](#-about-the-project)
- [Company Background](#-company-background)
- [Problem Statement](#-problem-statement)
- [Database Schema](#-database-schema)
- [Ad Hoc Requests & SQL Solutions](#-ad-hoc-requests--sql-solutions)
- [Key Insights](#-key-insights)
- [Recommendations](#-recommendations)
- [Tools Used](#-tools-used)
- [Presentation](#-presentation)
- [Connect with Me](#-connect-with-me)

---

## 📖 About the Project

This project is part of the **Codebasics SQL Resume Project Challenge**. The goal was to answer 10 ad hoc business requests from AtliQ Hardware's management using SQL — demonstrating technical proficiency, analytical thinking, and the ability to communicate insights clearly to stakeholders.

---

## 🏢 Company Background

**AtliQ Hardware** is a leading computer hardware manufacturer in India with a strong domestic presence and expanding international footprint across the APAC region. The company operates on a **September–August fiscal year** and sells products across channels including Retailers, Direct, and Distributors.

Their product portfolio spans 6 segments:
| Segment | Description |
|---|---|
| Notebook | Personal, Gaming & Business Laptops |
| Accessories | Mouse, Keyboard, etc. |
| Peripherals | Graphic Cards, Processors, etc. |
| Desktop | Personal & Business Desktops |
| Storage | External SSDs, USB Flash Drives |
| Networking | Wi-Fi Extenders, N&S products |

---

## ❗ Problem Statement

Rapid regional expansion and increasing transaction volumes made it difficult for management to derive timely, data-driven insights. To address this, the company planned to build a dedicated data analytics team. A structured SQL challenge was introduced to identify candidates with both strong technical skills and effective business communication ability.

Project Link : [Codebasics Resume Project Challenge #7](https://codebasics.io/challenges/codebasics-resume-project-challenge/7)
Documentaions : [10 Adhoc request](ad-hoc-requests.pdf)
---

## 🗄️ Database Schema

The database follows a **Star Schema** with **2 Dimension tables** and **4 Fact tables**, all joined via `customer_code` or `product_code`.

```
dim_customer          fact_sales_monthly
dim_product     ───►  fact_gross_price
                      fact_manufacturing_cost
                      fact_pre_invoice_deductions
```

### Table Details

| Table | Type | Key Columns |
|---|---|---|
| `dim_customer` | Dimension | `customer_code` 🔑, customer, market, region, channel, platform |
| `dim_product` | Dimension | `product_code` 🔑, product, segment, category, division, variant |
| `fact_sales_monthly` | Fact | `customer_code` 🔗, `product_code` 🔗, fiscal_year, sold_quantity |
| `fact_gross_price` | Fact | `product_code` 🔗, fiscal_year, gross_price |
| `fact_manufacturing_cost` | Fact | `product_code` 🔗, cost_year, manufacturing_cost |
| `fact_pre_invoice_deductions` | Fact | `customer_code` 🔗, fiscal_year, pre_invoice_discount_pct |

---

## 📊 Ad Hoc Requests & SQL Solutions

---

### Request 1 — APAC Market Presence of Atliq Exclusive

**Business Question:**
> Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.

**Result:** Atliq Exclusive operates in **8 APAC markets** — India, Bangladesh, South Korea, Japan, Indonesia, Australia, New Zealand, and Philippines.

```sql
SELECT
    DISTINCT market
FROM dim_customer
WHERE customer = 'Atliq Exclusive'
      AND region = 'APAC'
ORDER BY market;
```

---

### Request 2 — Unique Product Growth: 2021 vs 2020

**Business Question:**
> What is the percentage of unique product increase in 2021 vs. 2020?

**Result:** Unique products grew by **36.33%** — from 245 in FY2020 to 334 in FY2021.

```sql
WITH product_count AS (
    SELECT
        fiscal_year,
        COUNT(DISTINCT product_code) AS unique_product
    FROM fact_sales_monthly
    WHERE fiscal_year IN (2020, 2021)
    GROUP BY fiscal_year
)
SELECT
    MAX(CASE WHEN fiscal_year = 2020 THEN unique_product END) AS unique_products_2020,
    MAX(CASE WHEN fiscal_year = 2021 THEN unique_product END) AS unique_products_2021,
    ROUND(
        (
            MAX(CASE WHEN fiscal_year = 2021 THEN unique_product END) -
            MAX(CASE WHEN fiscal_year = 2020 THEN unique_product END)
        ) * 100 / MAX(CASE WHEN fiscal_year = 2020 THEN unique_product END), 2
    ) AS percentage_chg
FROM product_count;
```

---

### Request 3 — Unique Products Per Segment

**Business Question:**
> Provide a report with all the unique product counts for each segment, sorted in descending order.

**Result:** Notebook (129), Accessories (116), Peripherals (84), Desktop (32), Storage (27), Networking (9). The top 3 segments account for **~83%** of the total portfolio.

```sql
SELECT
    segment,
    COUNT(product_code) AS product_count
FROM dim_product
GROUP BY segment
ORDER BY product_count DESC;
```

---

### Request 4 — Segment with Most New Products in 2021 vs 2020

**Business Question:**
> Which segment had the most increase in unique products in 2021 vs 2020?

**Result:** **Accessories** led with +34 new products. **Desktop** had the highest percentage growth at **214.29%** (7 → 22).

```sql
WITH product_count AS (
    SELECT
        p.segment,
        f.fiscal_year,
        COUNT(DISTINCT f.product_code) AS unique_product
    FROM fact_sales_monthly f
    JOIN dim_product p
    ON f.product_code = p.product_code
    WHERE fiscal_year IN (2020, 2021)
    GROUP BY p.segment, f.fiscal_year
)
SELECT
    segment,
    MAX(CASE WHEN fiscal_year = 2020 THEN unique_product END) AS product_count_2020,
    MAX(CASE WHEN fiscal_year = 2021 THEN unique_product END) AS product_count_2021,
    MAX(CASE WHEN fiscal_year = 2021 THEN unique_product END) -
    MAX(CASE WHEN fiscal_year = 2020 THEN unique_product END) AS difference
FROM product_count
GROUP BY segment
ORDER BY difference DESC;
```

---

### Request 5 — Highest & Lowest Manufacturing Cost Products

**Business Question:**
> Get the products that have the highest and lowest manufacturing costs.

**Result:**
- 🔺 Highest: **AQ HOME Allin1 Gen 2 Personal Desktop** — **$240.54**
- 🔻 Lowest: **AQ Master Wired x1 Ms Mouse** — **$0.89** *(270x cost gap)*

```sql
SELECT
    f.product_code,
    p.product,
    f.manufacturing_cost
FROM fact_manufacturing_cost f
JOIN dim_product p
ON f.product_code = p.product_code
WHERE manufacturing_cost IN (
    SELECT MAX(manufacturing_cost) FROM fact_manufacturing_cost
    UNION
    SELECT MIN(manufacturing_cost) FROM fact_manufacturing_cost
)
ORDER BY manufacturing_cost DESC;
```

---

### Request 6 — Top 5 Customers by Pre-Invoice Discount in India (FY2021)

**Business Question:**
> Generate a report of the top 5 customers who received the highest average pre-invoice discount percentage for FY2021 in the Indian market.

**Result:** Flipkart led with **30.83%**, followed by Viveks (30.38%), Ezone (30.28%), Croma (30.25%), Vijay Sales (27.53%).

```sql
SELECT
    c.customer_code,
    c.customer,
    ROUND(AVG(f.pre_invoice_discount_pct), 2) AS average_discount_pct
FROM dim_customer c
JOIN fact_pre_invoice_deductions f
ON c.customer_code = f.customer_code
WHERE fiscal_year = 2021
    AND market = 'India'
GROUP BY c.customer_code, c.customer
ORDER BY average_discount_pct DESC
LIMIT 5;
```

---

### Request 7 — Monthly Gross Sales for Atliq Exclusive

**Business Question:**
> Get the complete report of gross sales amount for the customer "Atliq Exclusive" for each month, to identify low and high-performing months.

**Result:**
- 📉 Lowest: **March 2020 — $0.77M** *(caused by COVID-19 lockdowns + global silicon chip shortage)*
- 📈 Highest: **November 2020 — $32.25M** *(holiday season demand surge)*
- Average Gross Sales: **$12.66M**

```sql
SELECT
    MONTHNAME(s.date) AS Month,
    YEAR(s.date) AS Year,
    ROUND((SUM(g.gross_price) * SUM(s.sold_quantity)) / 1000000, 2) AS Gross_sales_Amount_mln
FROM fact_sales_monthly s
JOIN fact_gross_price g
ON s.product_code = g.product_code
AND s.fiscal_year = g.fiscal_year
JOIN dim_customer c
ON s.customer_code = c.customer_code
WHERE c.customer = 'Atliq Exclusive'
GROUP BY Month, Year;
```

---

### Request 8 — Best Quarter by Total Sold Quantity in FY2020

**Business Question:**
> In which quarter of 2020 was the maximum total_sold_quantity recorded?

**Result:** **Q1 (Sep–Nov 2020)** — **7.01M units**, followed by Q2 (6.65M), Q4 (5.04M), Q3 (2.08M). December was the single highest month at 3.18M.

> ⚠️ Note: AtliQ follows a Sep–Aug fiscal year, so Q1 = September, October, November.

```sql
SELECT
    CASE
        WHEN MONTH(date) IN (9, 10, 11)  THEN 'Q1'
        WHEN MONTH(date) IN (12, 1, 2)   THEN 'Q2'
        WHEN MONTH(date) IN (3, 4, 5)    THEN 'Q3'
        WHEN MONTH(date) IN (6, 7, 8)    THEN 'Q4'
    END AS Quarter,
    ROUND(SUM(sold_quantity) / 1000000, 2) AS total_sold_quantity_mln
FROM fact_sales_monthly
WHERE fiscal_year = 2020
GROUP BY Quarter
ORDER BY total_sold_quantity_mln DESC;
```

---

### Request 9 — Channel Contribution to Gross Sales in FY2021

**Business Question:**
> Which channel brought the most gross sales in FY2021 and what was its percentage contribution?

**Result:** **Retailer** dominated with **$1,924M (73.22%)**, followed by Direct ($407M, 15.47%) and Distributor ($297M, 11.31%).

```sql
WITH gross_sales AS (
    SELECT
        c.channel,
        g.gross_price * s.sold_quantity AS Gross_sales
    FROM fact_sales_monthly s
    JOIN fact_gross_price g
    ON s.product_code = g.product_code
    AND s.fiscal_year = g.fiscal_year
    JOIN dim_customer c
    ON s.customer_code = c.customer_code
),
agg AS (
    SELECT
        channel,
        ROUND(SUM(Gross_sales) / 1000000, 2) AS Gross_sales_mln
    FROM gross_sales
    GROUP BY channel
)
SELECT
    channel,
    Gross_sales_mln,
    ROUND((Gross_sales_mln / SUM(Gross_sales_mln) OVER()) * 100, 2) AS percentage
FROM agg
ORDER BY Gross_sales_mln DESC;
```

---

### Request 10 — Top 3 Products per Division by Sold Quantity (FY2021)

**Business Question:**
> Get the top 3 products in each division with the highest total_sold_quantity in FY2021.

**Result:**
| Division | #1 Product | Units Sold |
|---|---|---|
| N & S | AQ Pen Drive 2 IN 1 | 701,373 |
| P & A | AQ Gamers Ms | 428,498 |
| PC | AQ Digit | 17,434 |

```sql
WITH CTE1 AS (
    SELECT
        p.division,
        p.product_code,
        p.product,
        SUM(f.sold_quantity) AS total_sold_quantity
    FROM dim_product p
    JOIN fact_sales_monthly f
    ON p.product_code = f.product_code
    WHERE fiscal_year = 2021
    GROUP BY p.division, p.product_code, p.product
    ORDER BY total_sold_quantity DESC
),
CTE2 AS (
    SELECT
        *,
        DENSE_RANK() OVER(PARTITION BY division
                          ORDER BY total_sold_quantity DESC) AS rank_order
    FROM CTE1
)
SELECT *
FROM CTE2
WHERE rank_order <= 3;
```

---

## 💡 Key Insights

| # | Finding |
|---|---|
| 1 | Atliq Exclusive operates in **8 APAC markets** — strong regional footprint |
| 2 | Unique products grew **36.33% YoY** (245 → 334), signaling portfolio expansion |
| 3 | Notebook, Accessories & Peripherals dominate **83%** of the product portfolio |
| 4 | Accessories added the most products (**+34**); Desktop grew fastest at **+214%** |
| 5 | Manufacturing cost spans **$0.89 to $240.54** — a 270x gap |
| 6 | Flipkart received the highest avg. discount of **30.83%** in India FY2021 |
| 7 | March 2020 hit just **$0.77M** (COVID-19 + chip shortage); recovered 40x to **$32.25M** by Nov 2020 |
| 8 | **Q1 (Sep–Nov)** is the strongest quarter at 7.01M units sold |
| 9 | **Retailer channel** drives 73.22% of gross sales — Direct & Distributor underutilised |
| 10 | Top products: AQ Pen Drive (N&S), AQ Gamers Ms (P&A), AQ Digit (PC) |

---

## 🎯 Recommendations

1. **Reduce Retailer dependence (73%)** — invest in Direct & Distributor channels to diversify revenue and reduce single-channel risk. Target a 60/20/20 split over 3 years.

2. **Double down on Accessories & Desktop** — both showed the strongest expansion momentum in FY2021. Bundle deals, loyalty programs, and student discounts can accelerate growth further.

3. **Audit Flipkart's 30.83% discount ROI** — assess if the volume justifies the margin erosion compared to Viveks (30.38%) and Ezone (30.28%). Pull a gross margin by customer analysis.

4. **Build a silicon chip buffer stock** — the March 2020 crash to $0.77M exposed supply chain fragility. A 60-day safety stock policy for key semiconductors protects against the next global disruption.

5. **Attack the Q3 slump proactively** — Q3 (Mar–May) is historically the weakest quarter. Pre-season campaigns, student discounts, and bundle offers launched before March can smooth the revenue curve.

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| **MySQL** | Writing and executing all SQL queries |
| **Power BI** | Data model visualization |
| **Canva** | Presentation design |

---

## 📽️ Presentation

🔗 [View Full Presentation](https://github.com/addityaa95/SQL-Project-on-Consumer-Goods/blob/main/Business%20Presentation.pdf)
🔗 [View Power BI Presentation](Resume_projects_10_Adhoc_request.pbix)

---

## 🤝 Connect with Me

**S K Adityanarayan**

<p align="left">
  <a href="https://www.linkedin.com/in/addityaa95" target="_blank">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/>
  </a>


  <a href="https://codebasics.io/portfolio/S-K-Adityanarayan" target="_blank">
    <img src="https://img.shields.io/badge/Portfolio-FF6B00?style=for-the-badge&logo=google-chrome&logoColor=white"/>
  </a>
</p>

---

<p align="center">
  ⭐ If you found this project helpful, please give it a star!
</p>
