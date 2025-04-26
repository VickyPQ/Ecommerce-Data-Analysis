# E‑Commerce SQL Analytics Portfolio

> **Dataset:** *Maven Fuzzy Factory* (MySQL)\
> **Role Demonstrated:** Data & Insights Analyst\
> **Author:** Vicky Peng\
> **Repo structure:**
>

## Introduction

Maven Fuzzy Factory is an 8‑month‑old DTC e‑commerce start‑up that sells irresistibly cute *plush creatures*.\
<br>
The founding team is lean — a CEO, a Marketing Director, and a Website Manager — so the analyst (that's ME) sits **at the intersection of data and strategy**, turning raw click‑stream data into commercial moves that grow revenue and margin.\
<br>
As an aspiring analyst I rebuilt six of the questions I was routinely asked by C‑level and marketing leadership into a single, reproducible SQL portfolio. \
<br>
This project answers that call, end‑to‑end—**from raw tables to board‑ready insights**—showcasing advanced SQL techniques such as *pivoting with `CASE` & aggregate counts, window functions, multi‑step analyses with temporary tables & sub‑queries.*

---
<br>

## Database Description
*Schema:* `mavenfuzzyfactory`  (6 tables)  

| Table | Grain | Key Fields  |
|-------|-------|----------------------|
| `website_sessions` | One row per visit | `website_session_id`, `created_at`, `utm_source`, `utm_campaign`, `device_type`, `is_repeat_session` |
| `website_pageviews` | One pageview | `website_session_id`, `website_pageview_id`, `created_at`, `url` |
| `orders` | One customer order | `order_id`, `website_session_id`, `primary_product_id`, `price_usd`, `cogs_usd`, `created_at` |
| `order_items` | One line‑item |`order_item_id`,`created_at`,`order_id`,`product_id`,`is_primary_item` |
| `products` | One Product | `product_id`, `product_name`, `price_usd` |
|order_item_refunds | One refunded order item | `order_item_refund_id`, `order_item_id`, `order_id`, `refund_amount_usd`, `created_at`|

---
<br>

## Project Objectives

1. **Map** where visitors come from and how they convert → optimise marketing spend.
2. **De‑risk** the website experience → lift CVR by systematic A/B tests.
3. **Balance** the marketing channel portfolio → diversify away from a single SEM engine.
4. **Forecast & prepare** for demand swings using pattern & seasonality analysis.
5. **Expand** the product catalogue intelligently → track launch success + cross‑sell.
6. **Deepen** knowledge of user life‑cycle behaviour → unlock LTV growth.

<br>

## Analytical Roadmap
The storyline deliberately mirrors how an exec meeting unfolds—top‑down & action oriented:
1. **Traffic Resources** → Where do the eyeballs come from?  
2. **Website Performance** → What do those eyeballs *do*?  
3. **Channel Portfolio Management** → Are we spending the next \$1 in the right place?  
4. **Business Pattern & Seasonality** → When should we expect surges or lulls?  
5. **Product Analysis** → Which SKUs create or destroy margin?  
6. **User Analysis** → Are we retaining valuable customers?  

<br>

---
### 1️⃣ Traffic Resource Analysis
| #   | Assignment                            | Business Lens                 |
| --- | ------------------------------------- | ----------------------------- |
| 1.1 | Finding Top Traffic Sources      | Baseline channel mix          |
| 1.2 | Traffic‑Source Conversion Rates  | Economic viability            |
| 1.3 | Bid Optimisation & Trend Analysis | Price‑volume curve            |
| 1.4 | Traffic‑Source Trending           | Seasonality & cannibalisation |
| 1.5 | Bid Optimisation for Paid Traffic | Device‑level ROI              |
| 1.6 | Trending with Granular Segments   | Intersectional insights       |


#### 1.1.Finding Top Traffic Sources
```sql
USE mavenfuzzyfactory;

SELECT
    utm_source,
    utm_campaign,
    http_referer,
    COUNT(website_session_id) AS sessions
FROM website_sessions
WHERE created_at < '2012-04-12'
GROUP BY 1,2,3
ORDER BY sessions DESC;
```
**Output** 
| utm_source | utm_campaign | http_referer              | sessions |
|------------|--------------|---------------------------|----------|
| gsearch    | nonbrand     | https://www.gsearch.com   | 3613     |
| (null)     | (null)       | https://www.gsearch.com   | 28       |
| (null)     | (null)       | https://gsearch.com       | 27       |
| gsearch    | brand        | https://www.gsearch.com   | 26       |
| bsearch    | brand        | https://www.bsearch.com   | 7        |
| (null)     | (null)       | https://www.bsearch.com   | 7        |


> **Insight →** *The gsearch nonbrand campaign is the top-performing traffic source with 3613 sessions, accounting for the vast majority of incoming traffic.

> **Next move →** deep‑dive gsearch nonbrand.
---
### 1.2. Gsearch Nonbrand Conversion Rate
```sql
SELECT
    COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
    COUNT(DISTINCT orders.order_id) AS orders
FROM website_sessions
LEFT JOIN orders
    ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-04-14'
  AND utm_source = 'gsearch'
  AND utm_campaign = 'nonbrand';
```
**Output**

| sessions | orders | session_to_order_conv_rt |
|----------|--------|---------------------------|
| 3895     | 112    | 0.0288                    |


> **Insight →** The conversion rate from session to order for the gsearch nonbrand campaign is **2.88%**, which is below the 4% threshold needed to justify current bid costs.

> **Next move →** We should monitor the impact of reduced bids and analyze device-level performance trends to inform future bid strategy adjustments.





```sql
CREATE TEMPORARY TABLE landings AS
SELECT ws.website_session_id,
       MIN(wp.created_at)                              AS landing_ts,
       MIN(wp.pageview_url)                            AS landing_url
FROM   website_sessions ws
JOIN   website_pageviews wp USING(website_session_id)
GROUP  BY 1;

SELECT  COUNT(*)                                            AS sessions,
        SUM(bounced_flag)                                   AS bounced_sessions,
        ROUND(100*AVG(bounced_flag),1) AS bounce_rate_pct
FROM (
    SELECT l.*,      
           CASE WHEN pv_cnt = 1 THEN 1 ELSE 0 END AS bounced_flag
    FROM   landings l
    JOIN  (SELECT website_session_id, COUNT(*) AS pv_cnt
            FROM website_pageviews
            GROUP BY 1) c USING(website_session_id)
    WHERE  landing_url = '/home'
) x;
```

> **Insight →** The generic home page bounces **58.7 %** of gsearch visitors — a glaring leak.

> **Next move →** design /lander‑1 A/B test (scripts 2.4‑2.7).

*New skills:* differential bidding logic, cross‑channel elasticity, %‑of‑control metrics.

Script catalogue:

1. Channel session trends
2. Comparing device splits
3. Cross‑channel bid optimisation
4. Portfolio trend with comparison metric …

*See **``** for full SQL.*

Focus: `MONTH()`, `WEEK()`, `WEEKDAY()`, `HOUR()` — staffing recommendation.

Covers product‑level funnels, launch impact, refund diagnostics, cross‑sell matrices (pivot via `CASE WHEN`).

Repeat‑buyer identification, time‑to‑repeat distribution, conversion deltas by cohort.

---

## 🔧 SQL Techniques Highlighted

| Technique                                          | Where used                             | Commercial Pay‑off                                 |
| -------------------------------------------------- | -------------------------------------- | -------------------------------------------------- |
| `CASE WHEN ... END` + `COUNT()` **pivot**          | 1.2, 2.10, 5.12                        | Excel‑like slices inside SQL, zero export overhead |
| **Temporary tables** (`CREATE TEMPORARY TABLE…`)   | 2.\* conversions, 5.\* launch analysis | Multi‑step logic without polluting prod schema     |
| **Date functions** (`MONTH()`, `WEEK()`, `HOUR()`) | 4.1‑4.5                                | Surface hidden seasonality & staffing needs        |
| **LEFT JOIN** orders ⇒ sessions                    | All performance funnels                | Turn traffic into money, fast                      |

---

## 🧑‍💼 Storytelling for Executives

*SQL is necessary but not sufficient.*  After each module, the README distils the finding into a 2‑sentence **Insight & Action** block, mirroring how I would brief the CEO or Marketing Director.

> "Dial bsearch bids ‑20 % — desktop CVR trails gsearch by 43 %, so we gain profit with negligible volume loss."



