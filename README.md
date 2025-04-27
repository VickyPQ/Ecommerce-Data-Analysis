# E‑Commerce SQL Analytics Portfolio
> **Dataset:** *Maven Fuzzy Factory* (MySQL)\
> **Role Demonstrated:** Data & Insights Analyst\
> **Author:** Vicky Peng\


## Introduction
Maven Fuzzy Factory is an 8‑month‑old DTC e‑commerce start‑up that sells irresistibly cute *plush creatures*.\
<br>
The founding team is lean — a CEO, a Marketing Director, and a Website Manager — so the analyst (that's ME) sits at the intersection of data and strategy, turning raw click‑stream data into commercial moves that grow revenue and margin. As an aspiring analyst I rebuilt six of the questions I was routinely asked by C‑level and marketing leadership into a single, reproducible SQL portfolio. \
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

<br>

## Key Skills Highlighted
1. **Complex Analytical Queries**: Built layered, multi-step queries for real-world business cases, including traffic analysis, conversion funnel drop-off, and cohort behavior tracking.
2. **Temporary Tables & Subqueries**: Extensively used CTEs (Common Table Expressions) and temporary tables to manage complex interim results, improving query clarity and modularity.
3. **Session and Funnel Reconstruction**: Engineered user session paths, entry points, multi-step product funnels, and retention curves directly from raw event-level clickstream data.
4. **A/B Test Analysis in SQL**: Designed and executed landing page, billing page, and cross-sell A/B tests purely in SQL, from experiment framing to uplift quantification.
5. **Time Series and Seasonality Decomposition**: Built weekly/monthly trend analyses to extract patterns, seasonality effects, and event-driven anomalies.
6. **Cross-Channel Portfolio Optimization**: Modeled device-level and channel-level performance for budget reallocation using SQL, integrating marketing ROI principles into database queries.
7. **Dynamic Cohort Creation**: Dynamically grouped users based on landing pages, devices, and marketing attribution using case-based cohorting inside queries.

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
## 1️⃣ Traffic Resource Analysis
| #   | Assignment                            | Business Lens                 |
| --- | ------------------------------------- | ----------------------------- |
| 1.1 | Finding Top Traffic Sources      | Baseline channel mix          |
| 1.2 | Traffic‑Source Conversion Rates  | Economic viability            |
| 1.3 | Traffic‑Source Trending           | Seasonality & cannibalisation |
| 1.4 | Bid Optimisation for Paid Traffic | Device‑level ROI              |
| 1.5 | Trending with Granular Segments   | Intersectional insights       |


### 1.1 Finding Top Traffic Sources
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


> **Insight →** The gsearch nonbrand campaign is the top-performing traffic source with 3613 sessions, accounting for the vast majority of incoming traffic.

> **Next move →** We should deep‑dive gsearch nonbrand.
---
### 1.2 Gsearch Nonbrand Conversion Rate
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
---
### 1.3 Gsearch Nonbrand Volume Trends (Traffic‑Source Trending)

```sql
USE mavenfuzzyfactory;

SELECT
    MIN(DATE(created_at)) AS week_start_date,
    COUNT(DISTINCT website_sessions.website_session_id) AS sessions
FROM website_sessions
WHERE website_sessions.created_at < '2012-05-10'
  AND website_sessions.utm_source = 'gsearch'
  AND website_sessions.utm_campaign = 'nonbrand'
GROUP BY YEARWEEK(website_sessions.created_at);
```



**Output**

| week_start_date | sessions |
|------------------|----------|
| 2012-03-19       | 896      |
| 2012-03-25       | 956      |
| 2012-04-01       | 1152     |
| 2012-04-08       | 983      |
| 2012-04-15       | 621      |
| 2012-04-22       | 594      |
| 2012-04-29       | 681      |
| 2012-05-06       | 399      |



> **Insight →** After the bid reduction on April 15, gsearch nonbrand session volume noticeably declined, with weekly sessions dropping from 983 to 621 and continuing downward to 399 by early May.

> **Next move →** We should continue tracking weekly volume and explore strategies to make campaigns more efficient, so that we can increase volume again without increasing bids.
---
### 1.4 Gsearch Device-Level Performance (Bid Optimisation for Paid Traffic)

```sql
SELECT
    website_sessions.device_type,
    COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
    COUNT(DISTINCT orders.order_id) AS orders,
    COUNT(DISTINCT orders.order_id) / COUNT(DISTINCT website_sessions.website_session_id) AS conv_rt
FROM website_sessions
LEFT JOIN orders
    ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-05-11'
  AND utm_source = 'gsearch'
  AND utm_campaign = 'nonbrand'
GROUP BY 1;
```



**Output**

| device_type | sessions | orders | conv_rt |
|-------------|----------|--------|---------|
| desktop     | 3911     | 146    | 0.0373  |
| mobile      | 2492     | 24     | 0.0096  |



> **Insight →** Conversion rates on desktop (3.73%) significantly outperform mobile (0.96%), indicating that desktop users are far more likely to complete a purchase.

> **Next move →** We should increase bids for desktop traffic to drive more volume from this high-converting segment, and monitor performance by device type to assess the impact of bid adjustments.

---

### 1.5. Gsearch Device-Level Weekly Trends (Trending with Granular Segments)

```sql
SELECT
    MIN(DATE(created_at)) AS week_start_date,
    COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN website_sessions.website_session_id ELSE NULL END) AS dtop_sessions,
    COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_sessions.website_session_id ELSE NULL END) AS mob_sessions
FROM website_sessions
WHERE website_sessions.created_at < '2012-06-09'
  AND website_sessions.created_at >= '2012-04-15'
  AND website_sessions.utm_source = 'gsearch'
  AND website_sessions.utm_campaign = 'nonbrand'
GROUP BY YEARWEEK(website_sessions.created_at);
```



**Output**

| week_start_date | dtop_sessions | mob_sessions |
|------------------|---------------|--------------|
| 2012-04-15       | 383           | 238          |
| 2012-04-22       | 360           | 234          |
| 2012-04-29       | 425           | 256          |
| 2012-05-06       | 430           | 282          |
| 2012-05-13       | 403           | 214          |
| 2012-05-20       | 661           | 190          |
| 2012-05-27       | 585           | 183          |
| 2012-06-03       | 582           | 157          |


> **Insight →** After the desktop bid increase on May 19, desktop session volume surged from 403 to 661 the following week, while mobile traffic remained relatively flat or declined, confirming a positive response to the bid strategy.

> **Next move →** We should continue monitoring device-level traffic and conversion performance to ensure bid levels are driving the right results and optimizing budget efficiency.

---
<br>

## 2️⃣ Website Performance Analysis

| #   | Assignment                              | Business Lens                        |
|-----|-----------------------------------------|--------------------------------------|
| 2.1 | Finding Top Website Pages               | Page value hierarchy                 |
| 2.2 | Finding Top Entry Pages                 | First-touch engagement               |
| 2.3 | Calculating Bounce Rates                | Visitor retention & experience       |
| 2.4 | Analyzing Landing Page Tests            | Conversion optimization              |
| 2.5 | Landing Page Trend Analysis             | Longitudinal performance shifts      |
| 2.6 | Building Conversion Funnels             | Funnel design & drop-off detection   |
| 2.7 | Analyzing Conversion Funnel Tests       | Funnel-level experimentation impact  |

### 2.1 Top Website Pages 
```sql
USE mavenfuzzyfactory;

SELECT
    pageview_url,
    COUNT(DISTINCT website_session_id) AS sessions
FROM website_pageviews
WHERE created_at < '2012-06-09'
GROUP BY pageview_url
ORDER BY sessions DESC;
```


**Output**

| pageview_url                | sessions |
|----------------------------|----------|
| /home                      | 10,398   |
| /products                  | 4,238    |
| /the-original-mr-fuzzy     | 3,036    |
| /cart                      | 1,305    |
| /shipping                  | 869      |
| /billing                   | 716      |
| /thank-you-for-your-order | 306      |



> **Insight →** The homepage, products page, and the Mr. Fuzzy product page account for the majority of total session volume, highlighting their importance as primary user touchpoints.

> **Next move →** Investigate whether these top-viewed pages are also leading entry pages, and assess each page’s performance to identify any areas for UX or conversion improvement.

---
### 2.2. Top Entry Pages 

```sql
USE mavenfuzzyfactory;

CREATE TEMPORARY TABLE first_pageviews AS
SELECT
    website_session_id,
    MIN(website_pageview_id) AS min_pageview_id
FROM website_pageviews
WHERE created_at < '2012-06-12'
GROUP BY website_session_id;

SELECT
    website_pageviews.pageview_url AS landing_page_url,
    COUNT(first_pageviews.website_session_id) AS sessions_hitting_page
FROM first_pageviews
LEFT JOIN website_pageviews
    ON website_pageviews.website_pageview_id = first_pageviews.min_pageview_id
GROUP BY website_pageviews.pageview_url;
```


**Output**

| landing_page_url | sessions_hitting_page |
|------------------|------------------------|
| /home            | 10,711                 |


> **Insight →** The homepage is currently the dominant entry point for nearly all user sessions, which suggests it's the first impression for most visitors and a crucial focus area for optimization.

> **Next move →** Analyze homepage performance in detail to ensure it delivers a strong first experience, and consider whether it’s truly the best entry point for all user journeys.

---
### 2.3 Bounce Rate Analysis

```sql
USE mavenfuzzyfactory;

-- Step 1: Identify first pageview per session
CREATE TEMPORARY TABLE first_pageviews AS
SELECT
    website_session_id,
    MIN(website_pageview_id) AS min_pageview_id
FROM website_pageviews
WHERE created_at < '2012-06-14'
GROUP BY website_session_id;

-- Step 2: Filter sessions landing on the homepage
CREATE TEMPORARY TABLE sessions_w_home_landing_page AS
SELECT
    first_pageviews.website_session_id,
    website_pageviews.pageview_url AS landing_page
FROM first_pageviews
LEFT JOIN website_pageviews
    ON website_pageviews.website_pageview_id = first_pageviews.min_pageview_id
WHERE website_pageviews.pageview_url = '/home';

-- Step 3: Identify sessions that only viewed one page (bounced)
CREATE TEMPORARY TABLE bounced_sessions AS
SELECT
    sessions_w_home_landing_page.website_session_id,
    sessions_w_home_landing_page.landing_page,
    COUNT(website_pageviews.website_pageview_id) AS count_of_pages_viewed
FROM sessions_w_home_landing_page
LEFT JOIN website_pageviews
    ON website_pageviews.website_session_id = sessions_w_home_landing_page.website_session_id
GROUP BY
    sessions_w_home_landing_page.website_session_id,
    sessions_w_home_landing_page.landing_page
HAVING COUNT(website_pageviews.website_pageview_id) = 1;

-- Step 4: Count sessions and bounced sessions to compute bounce rate
SELECT
    COUNT(DISTINCT sessions_w_home_landing_page.website_session_id) AS sessions,
    COUNT(DISTINCT bounced_sessions.website_session_id) AS bounced_sessions,
    COUNT(DISTINCT bounced_sessions.website_session_id) * 1.0 /
    COUNT(DISTINCT sessions_w_home_landing_page.website_session_id) AS bounce_rate
FROM sessions_w_home_landing_page
LEFT JOIN bounced_sessions
    ON sessions_w_home_landing_page.website_session_id = bounced_sessions.website_session_id;
```

**Output**

| sessions | bounced_sessions | bounce_rate |
|----------|------------------|-------------|
| 11044    | 6536             | 0.5918      |



> **Insight →** Nearly 60% of homepage entry sessions result in a bounce, indicating users are leaving the site without exploring further—this is unusually high for paid search traffic and suggests room for UX improvements.

> **Next move →** Build and test a dedicated landing page tailored for search visitors, then track whether it lowers bounce rate compared to the homepage.

--- 

### 2.4. Landing Page A/B Test Analysis

```sql
USE mavenfuzzyfactory;
select 
	min(created_at) as first_created_at,
	min(website_pageview_id) as first_pageview_id
from website_pageviews
where pageview_url='/lander-1' and created_at is not null;
	

-- STEP 1: Capture first pageview per session after the new lander launched
CREATE TEMPORARY TABLE first_test_pageviews AS
SELECT
    website_pageviews.website_session_id,
    MIN(website_pageviews.website_pageview_id) AS min_pageview_id
FROM website_pageviews
INNER JOIN website_sessions
    ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE 
    website_sessions.created_at < '2012-07-28'
    AND website_pageviews.website_pageview_id > 23504
    AND website_sessions.utm_source = 'gsearch'
    AND website_sessions.utm_campaign = 'nonbrand'
GROUP BY website_pageviews.website_session_id;

-- STEP 2: Capture landing pages of interest (/home vs /lander-1)
CREATE TEMPORARY TABLE nonbrand_test_sessions_w_landing_page AS
SELECT
    first_test_pageviews.website_session_id,
    website_pageviews.pageview_url AS landing_page
FROM first_test_pageviews
LEFT JOIN website_pageviews
    ON website_pageviews.website_pageview_id = first_test_pageviews.min_pageview_id
WHERE website_pageviews.pageview_url IN ('/home','/lander-1');

-- STEP 3: Count how many pages viewed per session to identify bounces
CREATE TEMPORARY TABLE nonbrand_test_bounced_sessions AS
SELECT
    nonbrand_test_sessions_w_landing_page.website_session_id,
    nonbrand_test_sessions_w_landing_page.landing_page,
    COUNT(website_pageviews.website_pageview_id) AS count_of_pages_viewed
FROM nonbrand_test_sessions_w_landing_page
LEFT JOIN website_pageviews
    ON website_pageviews.website_session_id = nonbrand_test_sessions_w_landing_page.website_session_id
GROUP BY
    nonbrand_test_sessions_w_landing_page.website_session_id,
    nonbrand_test_sessions_w_landing_page.landing_page
HAVING COUNT(website_pageviews.website_pageview_id) = 1;

-- STEP 4: Join and summarize
SELECT
    nonbrand_test_sessions_w_landing_page.landing_page,
    COUNT(DISTINCT nonbrand_test_sessions_w_landing_page.website_session_id) AS total_sessions,
    COUNT(DISTINCT nonbrand_test_bounced_sessions.website_session_id) AS bounced_sessions,
    COUNT(DISTINCT nonbrand_test_bounced_sessions.website_session_id) * 1.0 /
    COUNT(DISTINCT nonbrand_test_sessions_w_landing_page.website_session_id) AS bounce_rate
FROM nonbrand_test_sessions_w_landing_page
LEFT JOIN nonbrand_test_bounced_sessions
    ON nonbrand_test_sessions_w_landing_page.website_session_id = nonbrand_test_bounced_sessions.website_session_id
GROUP BY nonbrand_test_sessions_w_landing_page.landing_page;

```

**Output**\
The first instance of /lander-1 to set analysis timeframe
| first_created_at       | first_pageview_id |
|------------------------|-------------------|
| 2012-06-19 00:35:54    | 23504             |

Final analysis output
| landing_page | total_sessions | bounced_sessions | bounce_rate |
|--------------|----------------|------------------|-------------|
| /home        | 2261           | 1319             | 0.58337     |
| /lander-1    | 2315           | 1232             | 0.53218     |


> **Insight →** The custom landing page **/lander-1** outperformed the homepage **/home** with a lower bounce rate, suggesting a better user experience and stronger engagement for nonbrand paid search traffic.

> **Next move →** Redirect all nonbrand paid traffic to **/lander-1** and monitor ongoing performance. Consider rolling out similar optimization tests for other key segments.

---
### 2.5 Landing Page Trend Analysis

```sql
USE mavenfuzzyfactory;

-- STEP 1: Get each session’s first pageview ID and count of pageviews
CREATE TEMPORARY TABLE sessions_w_min_pv_id_and_view_count AS
SELECT
    website_sessions.website_session_id,
    MIN(website_pageviews.website_pageview_id) AS first_pageview_id,
    COUNT(website_pageviews.website_pageview_id) AS count_pageviews
FROM website_sessions
LEFT JOIN website_pageviews
    ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE website_sessions.created_at > '2012-06-01'
  AND website_sessions.created_at < '2012-08-31'
  AND website_sessions.utm_source = 'gsearch'
  AND website_sessions.utm_campaign = 'nonbrand'
GROUP BY website_sessions.website_session_id;

-- STEP 2: Attach landing page and session created date
CREATE TEMPORARY TABLE sessions_w_counts_lander_and_created_at AS
SELECT
    s.website_session_id,
    s.first_pageview_id,
    s.count_pageviews,
    p.pageview_url AS landing_page,
    p.created_at AS session_created_at
FROM sessions_w_min_pv_id_and_view_count s
LEFT JOIN website_pageviews p
    ON s.first_pageview_id = p.website_pageview_id;

-- STEP 3: Aggregate by week to get bounce rate and lander sessions
SELECT
    MIN(DATE(session_created_at)) AS week_start_date,
    COUNT(DISTINCT CASE WHEN count_pageviews = 1 THEN website_session_id ELSE NULL END) * 1.0 /
    COUNT(DISTINCT website_session_id) AS bounce_rate,
    COUNT(DISTINCT CASE WHEN landing_page = '/home' THEN website_session_id ELSE NULL END) AS home_sessions,
    COUNT(DISTINCT CASE WHEN landing_page = '/lander-1' THEN website_session_id ELSE NULL END) AS lander_sessions
FROM sessions_w_counts_lander_and_created_at
GROUP BY YEARWEEK(session_created_at);
```

**Output**

| week_start_date | bounce_rate | home_sessions | lander_sessions |
|------------------|--------------|----------------|------------------|
| 2012-06-01       | 0.60674      | 178            | 0                |
| 2012-06-03       | 0.58660      | 791            | 0                |
| 2012-06-10       | 0.61758      | 876            | 0                |
| 2012-06-17       | 0.55767      | 492            | 349              |
| 2012-06-24       | 0.58333      | 370            | 386              |
| 2012-07-01       | 0.58131      | 393            | 388              |
| 2012-07-08       | 0.56625      | 390            | 410              |
| 2012-07-15       | 0.54235      | 427            | 423              |
| 2012-07-22       | 0.51321      | 403            | 392              |
| 2012-07-29       | 0.49805      | 34             | 994              |
| 2012-08-05       | 0.53860      | 0              | 1088             |
| 2012-08-12       | 0.51205      | 0              | 996              |
| 2012-08-19       | 0.50148      | 0              | 1013             |
| 2012-08-26       | 0.53976      | 0              | 830              |



> **Insight →** Traffic was correctly split during the test, and eventually shifted entirely to `/lander-1`. Importantly, the **overall bounce rate trended downward**, validating the success of the custom landing page.

> **Next move →** Maintain close monitoring to ensure performance stays strong and look for other optimization opportunities now that this one is validated.

---
### **2.6. Conversion Funnel Analysis**

```sql
USE mavenfuzzyfactory;

-- STEP 1: Tag relevant sessions and their pageviews since Aug 5
SELECT
    website_sessions.website_session_id,
    website_pageviews.pageview_url,
    CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
    CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page,
    CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
    CASE WHEN pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
    CASE WHEN pageview_url = '/billing' THEN 1 ELSE 0 END AS billing_page,
    CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
FROM website_sessions
LEFT JOIN website_pageviews
    ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE website_sessions.utm_source = 'gsearch'
  AND website_sessions.utm_campaign = 'nonbrand'
  AND website_sessions.created_at > '2012-08-05'
  AND website_sessions.created_at < '2012-09-05'
ORDER BY website_sessions.website_session_id, website_pageviews.created_at;

-- STEP 2: Create session-level flags for each funnel step
CREATE TEMPORARY TABLE session_level_made_it_flags AS
SELECT
    website_session_id,
    MAX(products_page) AS product_made_it,
    MAX(mrfuzzy_page) AS mrfuzzy_made_it,
    MAX(cart_page) AS cart_made_it,
    MAX(shipping_page) AS shipping_made_it,
    MAX(billing_page) AS billing_made_it,
    MAX(thankyou_page) AS thankyou_made_it
FROM (
    SELECT
        website_sessions.website_session_id,
        CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
        CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page,
        CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
        CASE WHEN pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
        CASE WHEN pageview_url = '/billing' THEN 1 ELSE 0 END AS billing_page,
        CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
    FROM website_sessions
    LEFT JOIN website_pageviews
        ON website_sessions.website_session_id = website_pageviews.website_session_id
    WHERE website_sessions.utm_source = 'gsearch'
      AND website_sessions.utm_campaign = 'nonbrand'
      AND website_sessions.created_at > '2012-08-05'
      AND website_sessions.created_at < '2012-09-05'
) AS pageview_level
GROUP BY website_session_id;

-- STEP 3: Count number of sessions that made it to each step
SELECT
    COUNT(DISTINCT website_session_id) AS sessions,
    COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS to_products,
    COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END) AS to_mrfuzzy,
    COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) AS to_cart,
    COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END) AS to_shipping,
    COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END) AS to_billing,
    COUNT(DISTINCT CASE WHEN thankyou_made_it = 1 THEN website_session_id ELSE NULL END) AS to_thankyou
FROM session_level_made_it_flags;

-- STEP 4: Calculate click-through rates between steps
SELECT
    COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) * 1.0 /
    COUNT(DISTINCT website_session_id) AS lander_click_rt,

    COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END) * 1.0 /
    COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS products_click_rt,

    COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) * 1.0 /
    COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END) AS mrfuzzy_click_rt,

    COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END) * 1.0 /
    COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) AS cart_click_rt,

    COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END) * 1.0 /
    COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END) AS shipping_click_rt,

    COUNT(DISTINCT CASE WHEN thankyou_made_it = 1 THEN website_session_id ELSE NULL END) * 1.0 /
    COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END) AS billing_click_rt
FROM session_level_made_it_flags;
```



**Output**

Session counts at each funnel step:

| sessions | to_products | to_mrfuzzy | to_cart | to_shipping | to_billing | to_thankyou |
|----------|-------------|------------|---------|-------------|-------------|---------------|
| 4494     | 2116        | 1567       | 682     | 454         | 360         | 157           |

Click-through rates between steps:

| lander_click_rt | products_click_rt | mrfuzzy_click_rt | cart_click_rt | shipping_click_rt | billing_click_rt |
|------------------|--------------------|-------------------|----------------|---------------------|-------------------|
| 0.4709           | 0.7405             | 0.4352            | 0.6657         | 0.7930              | 0.4361            |


> **Insight →** The largest drop-offs occur at the **landing**, **Mr. Fuzzy**, and **billing** pages. These are priority targets for optimization.

> **Next move →** Evaluate billing page performance, and continue identifying opportunities for optimization deeper in the funnel.

---


### 2.7. Conversion Funnel Test Results

```sql
USE mavenfuzzyfactory;

-- STEP 0: Find the first time /billing-2 was seen to determine test start point
SELECT
    MIN(website_pageviews.created_at) AS first_created_at,
    MIN(website_pageviews.website_pageview_id) AS first_pv_id
FROM website_pageviews
WHERE pageview_url = '/billing-2';

-- STEP 1: Get sessions hitting either /billing or /billing-2 after test began
SELECT
    billing_version_seen,
    COUNT(DISTINCT website_session_id) AS sessions,
    COUNT(DISTINCT order_id) AS orders,
    COUNT(DISTINCT order_id) * 1.0 / COUNT(DISTINCT website_session_id) AS billing_to_order_rt
FROM (
    SELECT
        website_pageviews.website_session_id,
        website_pageviews.pageview_url AS billing_version_seen,
        orders.order_id
    FROM website_pageviews
    LEFT JOIN orders
        ON website_pageviews.website_session_id = orders.website_session_id
    WHERE website_pageviews.website_pageview_id >= 53550         -- test start pageview_id
      AND website_pageviews.created_at >= '2012-11-10'           -- assignment date
      AND website_pageviews.pageview_url IN ('/billing', '/billing-2')
) AS billing_sessions_w_orders
GROUP BY billing_version_seen;
```

**Output**

| billing_version_seen | sessions | orders | billing_to_order_rt |
|-----------------------|----------|--------|----------------------|
| /billing              | 657      | 300    | 0.4566               |
| /billing-2            | 654      | 410    | 0.6269               |

> **Insight →** The new billing page **/billing-2** clearly performs better, increasing the conversion rate from **45.66% to 62.69%** — a significant improvement.

> **Next move →** Now that the new billing page proves more effective, work with Engineering to roll it out fully and continue monitoring performance metrics.

<br>

## 3️⃣ Channel Portfolio Management

| #   | Assignment                                   | Business Lens                     |
|-----|----------------------------------------------|-----------------------------------|
| 3.1 | Analyzing Channel Portfolios                 | Contribution by channel           |
| 3.2 | Comparing Channel Characteristics            | Visitor quality & engagement      |
| 3.3 | Cross-Channel Bid Optimization               | ROI-based budget reallocation     |
| 3.4 | Analyzing Channel Portfolio Trends           | Strategic shifts over time        |
| 3.5 | Analyzing Direct, Brand-Driven Traffic       | Brand equity & campaign leakage   |

### 3.1. Analyzing Channel Portfolio

```sql
USE mavenfuzzyfactory;

-- STEP 1: Weekly session volume for gsearch and bsearch since bsearch launch
SELECT
    YEARWEEK(created_at) AS year_week,
    MIN(DATE(created_at)) AS week_start_date,
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS gsearch_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' THEN website_session_id ELSE NULL END) AS bsearch_sessions
FROM website_sessions
WHERE created_at > '2012-08-22'    -- date of bsearch launch
  AND created_at < '2012-11-29'    -- assignment due date
  AND utm_campaign = 'nonbrand'    -- limiting to paid search nonbrand
GROUP BY YEARWEEK(created_at);
```

**Output**

| week_start_date | gsearch_sessions | bsearch_sessions |
|------------------|------------------|-------------------|
| 2012-08-22       | 590              | 197               |
| 2012-08-26       | 1056             | 343               |
| 2012-09-02       | 925              | 290               |
| 2012-09-09       | 951              | 329               |
| 2012-09-16       | 1151             | 365               |
| 2012-09-23       | 1050             | 321               |
| 2012-09-30       | 999              | 316               |
| 2012-10-07       | 1002             | 330               |
| 2012-10-14       | 1257             | 420               |
| 2012-10-21       | 1302             | 431               |
| 2012-10-28       | 1211             | 384               |
| 2012-11-04       | 1350             | 429               |
| 2012-11-11       | 1246             | 438               |
| 2012-11-18       | 3508             | 1093              |
| 2012-11-25       | 2286             | 774               |

> **Insight →** While gsearch remains the dominant channel, bsearch consistently brings in about a third of gsearch’s traffic, making it a significant and scalable new acquisition source.

> **Next move →** Follow up with analyses on channel characteristics and conversion performance to inform future bidding, investment, and optimization strategies.

---
### 3.2. Comparing Channel Characteristics

```sql
USE mavenfuzzyfactory;

SELECT
    utm_source,
    COUNT(DISTINCT website_session_id) AS sessions,
    COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END) AS mobile_sessions,
    COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END) * 1.0 /
    COUNT(DISTINCT website_session_id) AS pct_mobile
FROM website_sessions
WHERE created_at > '2012-08-22'
  AND created_at < '2012-11-30'
  AND utm_campaign = 'nonbrand'
GROUP BY utm_source;
```

**Output**

| utm_source | sessions | mobile_sessions | pct_mobile |
|------------|----------|------------------|-------------|
| bsearch    | 6522     | 562              | 0.0862      |
| gsearch    | 20073    | 4921             | 0.2452      |

> **Insight →** bsearch traffic is much more desktop-heavy, while gsearch sees significantly more mobile usage. These device trends signal fundamental channel differences.

> **Next move →** Expect future questions on optimization based on device mix. Start thinking about how performance varies by device, especially as you prepare bid strategies.

---
### 3.3. Cross Channel Bid Optimisation

```sql
USE mavenfuzzyfactory;

SELECT
    website_sessions.device_type,
    website_sessions.utm_source,
    COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
    COUNT(DISTINCT orders.order_id) AS orders,
    COUNT(DISTINCT orders.order_id) * 1.0 / 
    COUNT(DISTINCT website_sessions.website_session_id) AS conv_rate
FROM website_sessions
LEFT JOIN orders
    ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at > '2012-08-22'   -- specified in the request
  AND website_sessions.created_at < '2012-09-19'   -- exclude gsearch holiday campaign
  AND website_sessions.utm_campaign = 'nonbrand'   -- only nonbrand traffic
GROUP BY
    website_sessions.device_type,
    website_sessions.utm_source;
```

**Output**

| device_type | utm_source | sessions | orders | conv_rate |
|-------------|------------|----------|--------|-----------|
| desktop     | bsearch    | 1162     | 44     | 0.0379    |
| desktop     | gsearch    | 3011     | 136    | 0.0452    |
| mobile      | bsearch    | 130      | 1      | 0.0077    |
| mobile      | gsearch    | 1015     | 13     | 0.0128    |

> **Insight →** Conversion rates differ by both channel** and device type. gsearch consistently outperforms bsearch on both desktop and mobile, reinforcing that uniform bidding may not be optimal.

> **Next move →** Use this data to implement channel- and device-specific bid strategies, ensuring the marketing budget is allocated based on performance. Keep monitoring and adjusting.


---

### 3.4. Cross Channel Portfolio Trend

```sql
USE mavenfuzzyfactory;

-- Weekly session volume by device and channel, post-bsearch bid change
SELECT
    YEARWEEK(created_at) AS year_week,
    MIN(DATE(created_at)) AS week_start_date,

    -- Desktop sessions
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' AND device_type = 'desktop' THEN website_session_id ELSE NULL END) AS g_dtop_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' AND device_type = 'desktop' THEN website_session_id ELSE NULL END) AS b_dtop_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' AND device_type = 'desktop' THEN website_session_id ELSE NULL END) * 1.0 /
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' AND device_type = 'desktop' THEN website_session_id ELSE NULL END) AS b_pct_of_g_dtop,

    -- Mobile sessions
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' AND device_type = 'mobile' THEN website_session_id ELSE NULL END) AS g_mob_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' AND device_type = 'mobile' THEN website_session_id ELSE NULL END) AS b_mob_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' AND device_type = 'mobile' THEN website_session_id ELSE NULL END) * 1.0 /
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' AND device_type = 'mobile' THEN website_session_id ELSE NULL END) AS b_pct_of_g_mob

FROM website_sessions
WHERE created_at > '2012-11-04'
  AND created_at < '2012-12-22'
  AND utm_campaign = 'nonbrand'
GROUP BY YEARWEEK(created_at);
```

**Output**

| week_start_date | g_dtop_sessions | b_dtop_sessions | b_pct_of_g_dtop | g_mob_sessions | b_mob_sessions | b_pct_of_g_mob |
|------------------|------------------|------------------|------------------|----------------|----------------|----------------|
| 2012-11-04       | 1027             | 400              | 0.3895           | 323            | 29             | 0.0898         |
| 2012-11-11       | 956              | 401              | 0.4195           | 290            | 37             | 0.1276         |
| 2012-11-18       | 2655             | 1008             | 0.3797           | 853            | 85             | 0.0996         |
| 2012-11-25       | 2058             | 843              | 0.4096           | 692            | 62             | 0.0896         |
| 2012-12-02       | 1326             | 517              | 0.3899           | 396            | 31             | 0.0783         |
| 2012-12-09       | 1277             | 293              | 0.2294           | 424            | 46             | 0.1085         |
| 2012-12-16       | 1270             | 348              | 0.2740           | 376            | 41             | 0.1090         |

> **Insight →** After the bid down on December 2nd, bsearch traffic volume dropped notably, especially on desktop, where its share of gsearch fell from ~39% to ~27%. This drop is aligned with Tom’s expectation due to bsearch’s lower conversion performance.

> **Next move →** Use this traffic shift data to isolate and evaluate campaign efficiency post-bid change. This will help validate whether lower traffic to bsearch aligns with the expected revenue tradeoffs.

--- 


### 3.5. Analyzing Direct Traffic

```sql
SELECT
    YEAR(created_at) AS yr,
    MONTH(created_at) AS mo,
    COUNT(DISTINCT CASE WHEN channel_group = 'paid_nonbrand' THEN website_session_id ELSE NULL END) AS nonbrand,
    COUNT(DISTINCT CASE WHEN channel_group = 'paid_brand' THEN website_session_id ELSE NULL END) AS brand,
    COUNT(DISTINCT CASE WHEN channel_group = 'paid_brand' THEN website_session_id ELSE NULL END) /
    COUNT(DISTINCT CASE WHEN channel_group = 'paid_nonbrand' THEN website_session_id ELSE NULL END) AS brand_pct_of_nonbrand,
    COUNT(DISTINCT CASE WHEN channel_group = 'direct_type_in' THEN website_session_id ELSE NULL END) AS direct,
    COUNT(DISTINCT CASE WHEN channel_group = 'direct_type_in' THEN website_session_id ELSE NULL END) /
    COUNT(DISTINCT CASE WHEN channel_group = 'paid_nonbrand' THEN website_session_id ELSE NULL END) AS direct_pct_of_nonbrand,
    COUNT(DISTINCT CASE WHEN channel_group = 'organic_search' THEN website_session_id ELSE NULL END) AS organic,
    COUNT(DISTINCT CASE WHEN channel_group = 'organic_search' THEN website_session_id ELSE NULL END) /
    COUNT(DISTINCT CASE WHEN channel_group = 'paid_nonbrand' THEN website_session_id ELSE NULL END) AS organic_pct_of_nonbrand
FROM (
    SELECT
        website_session_id,
        created_at,
        CASE
            WHEN utm_source IS NULL AND http_referer IN ('https://www.gsearch.com', 'https://www.bsearch.com') THEN 'organic_search'
            WHEN utm_campaign = 'nonbrand' THEN 'paid_nonbrand'
            WHEN utm_campaign = 'brand' THEN 'paid_brand'
            WHEN utm_source IS NULL AND http_referer IS NULL THEN 'direct_type_in'
        END AS channel_group
    FROM website_sessions
    WHERE created_at < '2012-12-23'
) AS sessions_w_channel_group
GROUP BY
    YEAR(created_at),
    MONTH(created_at);
```

**Output**

| yr   | mo  | nonbrand | brand | brand_pct_of_nonbrand | direct | direct_pct_of_nonbrand | organic | organic_pct_of_nonbrand |
|------|-----|----------|-------|------------------------|--------|--------------------------|---------|----------------------------|
| 2012 | 3   | 1852     | 10    | 0.0054                 | 9      | 0.0049                   | 8       | 0.0043                     |
| 2012 | 4   | 3509     | 76    | 0.0217                 | 71     | 0.0202                   | 78      | 0.0222                     |
| 2012 | 5   | 3295     | 140   | 0.0425                 | 151    | 0.0458                   | 150     | 0.0455                     |
| 2012 | 6   | 3439     | 164   | 0.0477                 | 170    | 0.0494                   | 190     | 0.0552                     |
| 2012 | 7   | 3660     | 195   | 0.0533                 | 187    | 0.0511                   | 207     | 0.0566                     |
| 2012 | 8   | 5318     | 264   | 0.0496                 | 250    | 0.0470                   | 265     | 0.0498                     |
| 2012 | 9   | 5591     | 339   | 0.0606                 | 285    | 0.0510                   | 331     | 0.0592                     |
| 2012 | 10  | 6883     | 432   | 0.0628                 | 440    | 0.0639                   | 428     | 0.0622                     |
| 2012 | 11  | 12260    | 556   | 0.0454                 | 571    | 0.0466                   | 624     | 0.0509                     |
| 2012 | 12  | 6643     | 464   | 0.0698                 | 482    | 0.0726                   | 492     | 0.0741                     |

> **Insight →** Brand, direct, and organic traffic not only **grew in absolute volume**, but also **grew as a percentage of nonbrand paid traffic**. This is a great sign of increasing brand strength and diversified user acquisition.

> **Next move →** Share this story with stakeholders and investors. Continue monitoring branded/organic growth to evaluate reliance on paid traffic over time.
---


<br>


## 4️⃣ Analyzing Business Patterns and Seasonality

| #   | Assignment                         | Business Lens                         |
|-----|------------------------------------|----------------------------------------|
| 4.1 | Analyzing Seasonality              | Operational planning & forecasting     |
| 4.2 | Analyzing Business Patterns        | Behavioral insights & customer cycles  |


### 4.1. Analyzing Seasonality

```sql
USE mavenfuzzyfactory;

-- Pulling session and order volume by month for 2012
SELECT
    YEAR(website_sessions.created_at) AS yr,
    MONTH(website_sessions.created_at) AS mo,
    COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
    COUNT(DISTINCT orders.order_id) AS orders
FROM website_sessions
LEFT JOIN orders
    ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2013-01-01'
GROUP BY 1, 2;
```

**Output**

| yr   | mo  | sessions | orders |
|------|-----|----------|--------|
| 2012 | 3   | 1879     | 60     |
| 2012 | 4   | 3734     | 99     |
| 2012 | 5   | 3736     | 108    |
| 2012 | 6   | 3963     | 140    |
| 2012 | 7   | 4249     | 169    |
| 2012 | 8   | 6097     | 228    |
| 2012 | 9   | 6546     | 287    |
| 2012 |10   | 8183     | 371    |
| 2012 |11   |14011     | 618    |
| 2012 |12   |10072     | 506    |


> **Insight →** The business showed steady growth throughout the year, with a clear spike in both sessions and orders during Q4, especially around November and December—highlighting the impact of the holiday season (Black Friday and Cyber Monday).

> **Next move →** Use these seasonal insights to plan for inventory, staffing, and marketing campaigns in Q4 of future years. Consider launching promotional efforts ahead of the seasonal peak to capture early holiday traffic.

---


### 4.2.Analyzing Business Patterns

```sql
-- STEP 1: Count sessions by hour, weekday, and date
SELECT
  DATE(created_at) AS created_date,
  WEEKDAY(created_at) AS wkday,
  HOUR(created_at) AS hr,
  COUNT(DISTINCT website_session_id) AS website_sessions
FROM website_sessions
WHERE created_at BETWEEN '2012-09-15' AND '2012-11-15' -- before holiday surge
GROUP BY 1, 2, 3;

-- STEP 2: Average session volume by hour and weekday
SELECT
  hr,
  ROUND(AVG(CASE WHEN wkday = 0 THEN website_sessions ELSE NULL END), 1) AS mon,
  ROUND(AVG(CASE WHEN wkday = 1 THEN website_sessions ELSE NULL END), 1) AS tue,
  ROUND(AVG(CASE WHEN wkday = 2 THEN website_sessions ELSE NULL END), 1) AS wed,
  ROUND(AVG(CASE WHEN wkday = 3 THEN website_sessions ELSE NULL END), 1) AS thu,
  ROUND(AVG(CASE WHEN wkday = 4 THEN website_sessions ELSE NULL END), 1) AS fri,
  ROUND(AVG(CASE WHEN wkday = 5 THEN website_sessions ELSE NULL END), 1) AS sat,
  ROUND(AVG(CASE WHEN wkday = 6 THEN website_sessions ELSE NULL END), 1) AS sun
FROM (
  SELECT
    DATE(created_at) AS created_date,
    WEEKDAY(created_at) AS wkday,
    HOUR(created_at) AS hr,
    COUNT(DISTINCT website_session_id) AS website_sessions
  FROM website_sessions
  WHERE created_at BETWEEN '2012-09-15' AND '2012-11-15'
  GROUP BY 1, 2, 3
) daily_hourly_sessions
GROUP BY hr;
```

**Output**

| hr  | mon  | tue  | wed  | thu  | fri  | sat  | sun  |
|----|------|------|------|------|------|------|------|
| 0  | 8.7  | 7.7  | 6.3  | 7.4  | 6.8  | 5.0  | 5.0  |
| 1  | 6.6  | 6.7  | 5.3  | 4.9  | 7.1  | 5.0  | 3.0  |
| 2  | 6.1  | 4.4  | 4.4  | 6.1  | 4.6  | 3.7  | 3.0  |
| 3  | 5.7  | 4.0  | 4.7  | 4.6  | 4.0  | 2.8  | 2.4  |
| 4  | 5.9  | 6.3  | 6.0  | 4.0  | 6.1  | 2.8  | 2.4  |
| 5  | 5.0  | 5.4  | 5.1  | 5.4  | 4.6  | 4.3  | 3.9  |
| 6  | 5.4  | 5.6  | 4.8  | 6.0  | 6.8  | 4.0  | 2.6  |
| 7  | 7.3  | 7.8  | 7.4  | 10.6 | 7.0  | 5.7  | 4.8  |
| 8  | 12.3 | 12.2 | 13.0 | 16.5 | 10.5 | 4.3  | 4.1  |
| 9  | 17.6 | 15.7 | 19.6 | 19.3 | 17.5 | 7.6  | 6.0  |
| 10 | 18.4 | 17.7 | 21.0 | 18.4 | 19.0 | 8.3  | 6.3  |
| 11 | 18.0 | 19.1 | 24.9 | 21.6 | 20.9 | 7.2  | 7.7  |
| 12 | 21.1 | 23.3 | 22.8 | 24.1 | 19.0 | 8.6  | 6.1  |
| 13 | 17.8 | 23.0 | 20.8 | 20.6 | 21.6 | 8.1  | 8.4  |
| 14 | 17.9 | 21.6 | 22.3 | 18.5 | 19.5 | 8.7  | 6.7  |
| 15 | 21.6 | 17.1 | 25.3 | 23.5 | 21.3 | 6.9  | 7.1  |
| 16 | 21.1 | 23.7 | 23.7 | 19.6 | 20.9 | 7.6  | 6.6  |
| 17 | 19.4 | 15.9 | 20.2 | 19.8 | 12.9 | 6.4  | 7.6  |
| 18 | 12.7 | 15.0 | 14.8 | 15.3 | 10.9 | 5.3  | 6.8  |
| 19 | 12.4 | 14.1 | 13.3 | 11.6 | 14.3 | 7.1  | 6.4  |
| 20 | 12.1 | 12.4 | 14.2 | 10.6 | 10.3 | 5.7  | 8.4  |
| 21 | 9.1  | 12.6 | 11.4 | 9.4  | 7.3  | 5.7  | 10.2 |
| 22 | 9.1  | 10.0 | 9.8  | 12.1 | 6.0  | 5.7  | 10.2 |
| 23 | 8.8  | 8.6  | 9.6  | 10.6 | 7.6  | 5.3  | 8.3  |



> **Insight →** Weekdays, especially between 8am–5pm, show consistently high session volume, with Wednesday and Thursday peaking around midday. This pattern aligns with standard working hours and suggests when customer service coverage is most needed.

> **Next move →** Staff customer support with at least two agents from 8am–5pm on weekdays, with one agent available around the clock to handle off-hours activity.

---
<br>

## 5️⃣ Product Analysis

| #   | Assignment                                | Business Lens                         |
|-----|-------------------------------------------|---------------------------------------|
| 5.1 | Product-Level Sales Analysis              | Core revenue drivers                  |
| 5.2 | Analyzing Product Launches                | Launch impact assessment              |
| 5.3 | Product-Level Website Pathing             | Customer journey insights             |
| 5.4 | Building Product-Level Conversion Funnels | Funnel optimization by product        |
| 5.5 | Cross-Sell Analysis                       | Product affinity and upsell strategy  |
| 5.6 | Product Portfolio Expansion               | Strategic portfolio development       |
| 5.7 | Analyzing Product Refund Rates            | Quality control and customer behavior |



### 5.1 Product-Level Sales Analysis

```sql
-- Pull monthly trends: number of sales, total revenue, and total margin
SELECT
  YEAR(created_at) AS yr,
  MONTH(created_at) AS mo,
  COUNT(DISTINCT order_id) AS number_of_sales,
  SUM(price_usd) AS total_revenue,
  SUM(price_usd - cogs_usd) AS total_margin
FROM orders
WHERE created_at < '2013-01-04' -- pulling data up to the request date
GROUP BY
  YEAR(created_at),
  MONTH(created_at);
```

**Output**

| yr   | mo  | number_of_sales | total_revenue | total_margin |
|-----|-----|-----------------|---------------|--------------|
| 2012 | 3   | 60              | 2999.40       | 1830.00      |
| 2012 | 4   | 99              | 4949.01       | 3019.50      |
| 2012 | 5   | 108             | 5398.92       | 3294.00      |
| 2012 | 6   | 140             | 6998.60       | 4270.00      |
| 2012 | 7   | 169             | 8448.31       | 5154.50      |
| 2012 | 8   | 228             | 11397.72      | 6954.00      |
| 2012 | 9   | 287             | 14347.13      | 8753.50      |
| 2012 | 10  | 371             | 18546.29      | 11315.50     |
| 2012 | 11  | 618             | 30893.82      | 18849.00     |
| 2012 | 12  | 506             | 25294.94      | 15433.00     |
| 2013 | 1   | 42              | 2099.58       | 1281.00      |


> **Insight →**  
The business experienced strong, consistent growth in 2012, both in sales volume and margins. November (2012-11) saw a significant revenue and margin peak, likely due to Black Friday/Cyber Monday sales periods. This trend offers a robust baseline to assess the upcoming product launch’s success.

> **Next move →**  
Use this dataset as the baseline to benchmark the new product’s launch performance. Monitor sales, revenue, and margin evolution monthly post-launch, and prepare to adjust strategies based on early trend indicators.



---

### 5.2. Conversion Funnel by Product Category

```sql
SELECT
  YEAR(website_sessions.created_at) AS yr,
  MONTH(website_sessions.created_at) AS mo,
  COUNT(DISTINCT orders.order_id) AS orders,
  COUNT(DISTINCT orders.order_id) / COUNT(DISTINCT website_sessions.website_session_id) AS conv_rate,
  SUM(orders.price_usd) / COUNT(DISTINCT website_sessions.website_session_id) AS revenue_per_session,
  COUNT(DISTINCT CASE WHEN primary_product_id = 1 THEN order_id ELSE NULL END) AS product_one_orders,
  COUNT(DISTINCT CASE WHEN primary_product_id = 2 THEN order_id ELSE NULL END) AS product_two_orders
FROM website_sessions
LEFT JOIN orders
  ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2013-04-05' -- the date of the request
  AND website_sessions.created_at > '2012-04-01' -- specified in the request
GROUP BY 1, 2;
```

**Output**

| yr   | mo | orders | conv_rate | revenue_per_session | product_one_orders | product_two_orders |
|-----|----|--------|-----------|---------------------|--------------------|--------------------|
| 2012 | 4 | 99     | 0.0265    | 1.325391            | 99                 | 0                  |
| 2012 | 5 | 108    | 0.0289    | 1.445107            | 108                | 0                  |
| 2012 | 6 | 140    | 0.0353    | 1.765985            | 140                | 0                  |
| 2012 | 7 | 169    | 0.0398    | 1.988305            | 169                | 0                  |
| 2012 | 8 | 228    | 0.0374    | 1.869398            | 228                | 0                  |
| 2012 | 9 | 287    | 0.0438    | 2.191740            | 287                | 0                  |
| 2012 | 10| 371    | 0.0453    | 2.266441            | 371                | 0                  |
| 2012 | 11| 618    | 0.0441    | 2.204969            | 618                | 0                  |
| 2012 | 12| 506    | 0.0502    | 2.511412            | 506                | 0                  |
| 2013 | 1 | 391    | 0.0611    | 3.127025            | 344                | 47                 |
| 2013 | 2 | 497    | 0.0693    | 3.692108            | 335                | 162                |
| 2013 | 3 | 385    | 0.0615    | 3.176269            | 320                | 65                 |
| 2013 | 4 | 96     | 0.0794    | 4.085227            | 82                 | 14                 |


> **Insight →**  
> Following the launch of the second product in January 2013, there is a noticeable improvement in conversion rates and revenue per session. Product 2 contributed meaningfully to order volume, with February showing a particularly strong uptake.



> **Next move →**  
> - Monitor how product mix evolves in future months to ensure both products continue to contribute positively.  
> - Plan a deeper analysis to isolate the incremental impact of Product 2 from overall business improvements.  
> - Stay ready for additional launch-specific analysis requests from Cindy and Tom.


---

### 5.3. Product Level Website Pathing

```sql
-- Step 1: Finding the /products pageviews we care about
CREATE TEMPORARY TABLE products_pageviews
SELECT
    website_session_id,
    website_pageview_id,
    created_at,
    CASE
        WHEN created_at < '2013-01-06' THEN 'A. Pre_Product_2'
        WHEN created_at >= '2013-01-06' THEN 'B. Post_Product_2'
        ELSE 'uh oh...check logic'
    END AS time_period
FROM website_pageviews
WHERE created_at < '2013-04-06' -- date of request
  AND created_at > '2012-10-06' -- 3 months before product 2 launch
  AND pageview_url = '/products';

-- Step 2: Finding the next pageview ID that occurs AFTER the product pageview
CREATE TEMPORARY TABLE sessions_w_next_pageview_id
SELECT
    products_pageviews.time_period,
    products_pageviews.website_session_id,
    MIN(website_pageviews.website_pageview_id) AS min_next_pageview_id
FROM products_pageviews
LEFT JOIN website_pageviews
  ON website_pageviews.website_session_id = products_pageviews.website_session_id
  AND website_pageviews.website_pageview_id > products_pageviews.website_pageview_id
GROUP BY 1,2;

-- Step 3: Find the pageview URL associated with any applicable next pageview ID
CREATE TEMPORARY TABLE sessions_w_next_pageview_url
SELECT
    sessions_w_next_pageview_id.time_period,
    sessions_w_next_pageview_id.website_session_id,
    website_pageviews.pageview_url AS next_pageview_url
FROM sessions_w_next_pageview_id
LEFT JOIN website_pageviews
  ON website_pageviews.website_pageview_id = sessions_w_next_pageview_id.min_next_pageview_id;

-- Step 4: Summarize the data and analyze the pre vs post periods
SELECT
    time_period,
    COUNT(DISTINCT website_session_id) AS sessions,
    COUNT(DISTINCT CASE WHEN next_pageview_url IS NOT NULL THEN website_session_id ELSE NULL END) AS w_next_pg,
    COUNT(DISTINCT CASE WHEN next_pageview_url IS NOT NULL THEN website_session_id ELSE NULL END) / COUNT(DISTINCT website_session_id) AS pct_w_next_pg,
    COUNT(DISTINCT CASE WHEN next_pageview_url = '/the-original-mr-fuzzy' THEN website_session_id ELSE NULL END) AS to_mrfuzzy,
    COUNT(DISTINCT CASE WHEN next_pageview_url = '/the-original-mr-fuzzy' THEN website_session_id ELSE NULL END) / COUNT(DISTINCT website_session_id) AS pct_to_mrfuzzy,
    COUNT(DISTINCT CASE WHEN next_pageview_url = '/the-forever-love-bear' THEN website_session_id ELSE NULL END) AS to_lovebear,
    COUNT(DISTINCT CASE WHEN next_pageview_url = '/the-forever-love-bear' THEN website_session_id ELSE NULL END) / COUNT(DISTINCT website_session_id) AS pct_to_lovebear
FROM sessions_w_next_pageview_url
GROUP BY time_period;
```


**Output**

| time_period       | sessions | w_next_pg | pct_w_next_pg | to_mrfuzzy | pct_to_mrfuzzy | to_lovebear | pct_to_lovebear |
|-------------------|----------|-----------|---------------|------------|---------------|-------------|-----------------|
| A. Pre_Product_2  | 15,696   | 11,347    | 0.7229        | 11,347     | 0.7229        | 0           | 0.0000          |
| B. Post_Product_2 | 10,709   | 8,200     | 0.7657        | 6,654      | 0.6213        | 1,546       | 0.1444          |


> **Insight →**  
> After the new product ("Love Bear") launched, the overall clickthrough rate from the `/products` page increased (from 72.29% to 76.57%).  
> However, clickthroughs to "Mr. Fuzzy" dropped slightly, while "Love Bear" captured a new 14.44% of traffic, indicating that the new product generated additional product interest and slightly redistributed attention across offerings.


> **Next move →**  
> - Follow up with a detailed conversion funnel for each product to better understand where customers may be dropping off.  
> - Proactively monitor if "Love Bear" customers also engage with other products, suggesting cross-sell opportunities.

---



### 5.4 Building Product-Level Conversion Funnels

```sql
-- STEP 1: Identify sessions seeing product pages
CREATE TEMPORARY TABLE sessions_seeing_product_pages AS
SELECT
  website_session_id,
  website_pageview_id,
  pageview_url AS product_page_seen
FROM website_pageviews
WHERE created_at BETWEEN '2013-01-06' AND '2013-04-10'
  AND pageview_url IN ('/the-original-mr-fuzzy', '/the-forever-love-bear');

-- STEP 2: Find next pageviews after product pages
SELECT DISTINCT
  website_pageviews.pageview_url
FROM sessions_seeing_product_pages
LEFT JOIN website_pageviews
  ON website_pageviews.website_session_id = sessions_seeing_product_pages.website_session_id
  AND website_pageviews.website_pageview_id > sessions_seeing_product_pages.website_pageview_id;

-- STEP 3: Create a session-level flag for funnel steps
SELECT
  sessions_seeing_product_pages.website_session_id,
  sessions_seeing_product_pages.product_page_seen,
  CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
  CASE WHEN pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
  CASE WHEN pageview_url = '/billing-2' THEN 1 ELSE 0 END AS billing_page,
  CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
FROM sessions_seeing_product_pages
LEFT JOIN website_pageviews
  ON website_pageviews.website_session_id = sessions_seeing_product_pages.website_session_id
  AND website_pageviews.website_pageview_id > sessions_seeing_product_pages.website_pageview_id
ORDER BY
  sessions_seeing_product_pages.website_session_id,
  website_pageviews.created_at;

-- STEP 4: Aggregate session-level results
CREATE TEMPORARY TABLE session_product_level_made_it_flags AS
SELECT
  website_session_id,
  CASE
    WHEN product_page_seen = '/the-original-mr-fuzzy' THEN 'mrfuzzy'
    WHEN product_page_seen = '/the-forever-love-bear' THEN 'lovebear'
    ELSE 'check logic'
  END AS product_seen,
  MAX(cart_page) AS cart_made_it,
  MAX(shipping_page) AS shipping_made_it,
  MAX(billing_page) AS billing_made_it,
  MAX(thankyou_page) AS thankyou_made_it
FROM (
  -- previous session flags
)
GROUP BY website_session_id, product_seen;

-- STEP 5a: Funnel raw counts
SELECT
  product_seen,
  COUNT(DISTINCT website_session_id) AS sessions,
  COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) AS to_cart,
  COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END) AS to_shipping,
  COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END) AS to_billing,
  COUNT(DISTINCT CASE WHEN thankyou_made_it = 1 THEN website_session_id ELSE NULL END) AS to_thankyou
FROM session_product_level_made_it_flags
GROUP BY product_seen;

-- STEP 5b: Funnel conversion rates
SELECT
  product_seen,
  ROUND(COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) / COUNT(DISTINCT website_session_id), 4) AS product_page_click_rt,
  ROUND(COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END) / COUNT(DISTINCT website_session_id), 4) AS cart_click_rt,
  ROUND(COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END) / COUNT(DISTINCT website_session_id), 4) AS shipping_click_rt,
  ROUND(COUNT(DISTINCT CASE WHEN thankyou_made_it = 1 THEN website_session_id ELSE NULL END) / COUNT(DISTINCT website_session_id), 4) AS billing_click_rt
FROM session_product_level_made_it_flags
GROUP BY product_seen;
```


**Output** \
Conversion Raw Counts

| product_seen | sessions | to_cart | to_shipping | to_billing | to_thankyou |
|--------------|----------|---------|-------------|------------|-------------|
| lovebear     | 1599     | 877     | 603         | 488        | 301         |
| mrfuzzy      | 6985     | 3038    | 2084        | 1710       | 1088        |

Conversion Rates

| product_seen | product_page_click_rt | cart_click_rt | shipping_click_rt | billing_click_rt |
|--------------|-----------------------|---------------|-------------------|------------------|
| lovebear     | 0.5485                 | 0.6876        | 0.8093            | 0.6168           |
| mrfuzzy      | 0.4349                 | 0.6860        | 0.8205            | 0.6363           |



> **Insight →**  
"Lovebear" has a higher product page clickthrough rate (0.5485) compared to "Mr. Fuzzy" (0.4349), suggesting stronger initial attraction.  
Further into the funnel (cart, shipping, billing), both products perform similarly, but Lovebear slightly outperforms in early-stage engagement.



> **Next move →**  
Invest more marketing efforts around Lovebear, perhaps testing different promotions or bundles.  
Monitor if the higher clickthrough translates into sustained higher final purchases as product familiarity increases.

---


### 5.5. Cross-Selling Analysis

```sql
-- Step 1: Identify /cart page views and sessions
CREATE TEMPORARY TABLE sessions_seeing_cart AS
SELECT
  CASE
    WHEN created_at < '2013-09-25' THEN 'A. Pre_Cross_Sell'
    WHEN created_at >= '2013-09-25' THEN 'B. Post_Cross_Sell'
    ELSE 'uh oh...check logic'
  END AS time_period,
  website_session_id AS cart_session_id,
  website_pageview_id AS cart_pageview_id
FROM website_pageviews
WHERE created_at BETWEEN '2013-08-25' AND '2013-10-25'
  AND pageview_url = '/cart';

-- Step 2: Identify sessions that continued after the cart page
CREATE TEMPORARY TABLE cart_sessions_seeing_another_page AS
SELECT
  sessions_seeing_cart.time_period,
  sessions_seeing_cart.cart_session_id,
  MIN(website_pageviews.website_pageview_id) AS pv_id_after_cart
FROM sessions_seeing_cart
LEFT JOIN website_pageviews
  ON website_pageviews.website_session_id = sessions_seeing_cart.cart_session_id
  AND website_pageviews.website_pageview_id > sessions_seeing_cart.cart_pageview_id
GROUP BY 1,2
HAVING MIN(website_pageviews.website_pageview_id) IS NOT NULL;

-- Step 3: Match cart sessions to orders
CREATE TEMPORARY TABLE pre_post_sessions_orders AS
SELECT
  time_period,
  cart_session_id,
  order_id,
  items_purchased,
  price_usd
FROM sessions_seeing_cart
INNER JOIN orders
  ON sessions_seeing_cart.cart_session_id = orders.website_session_id;

-- Step 4: Aggregate and summarize findings
SELECT
  time_period,
  COUNT(DISTINCT cart_session_id) AS cart_sessions,
  SUM(clicked_to_another_page) AS clickthroughs,
  SUM(clicked_to_another_page) / COUNT(DISTINCT cart_session_id) AS cart_ctr,
  SUM(items_purchased) / SUM(placed_order) AS products_per_order,
  SUM(price_usd) / SUM(placed_order) AS aov,
  SUM(price_usd) / COUNT(DISTINCT cart_session_id) AS rev_per_cart_session
FROM (
  SELECT
    sessions_seeing_cart.time_period,
    sessions_seeing_cart.cart_session_id,
    CASE WHEN cart_sessions_seeing_another_page.cart_session_id IS NULL THEN 0 ELSE 1 END AS clicked_to_another_page,
    CASE WHEN pre_post_sessions_orders.order_id IS NULL THEN 0 ELSE 1 END AS placed_order,
    pre_post_sessions_orders.items_purchased,
    pre_post_sessions_orders.price_usd
  FROM sessions_seeing_cart
  LEFT JOIN cart_sessions_seeing_another_page
    ON sessions_seeing_cart.cart_session_id = cart_sessions_seeing_another_page.cart_session_id
  LEFT JOIN pre_post_sessions_orders
    ON sessions_seeing_cart.cart_session_id = pre_post_sessions_orders.cart_session_id
) AS full_data
GROUP BY time_period;
```


**Output**

| time_period        | cart_sessions | clickthroughs | cart_ctr | products_per_order | aov        | rev_per_cart_session |
|--------------------|----------------|---------------|----------|--------------------|------------|----------------------|
| A. Pre_Cross_Sell  | 1830           | 1229          | 0.6716   | 1.0000              | 51.416380  | 18.318842            |
| B. Post_Cross_Sell | 1975           | 1351          | 0.6841   | 1.0447              | 54.251848  | 18.431894            |


> **Insight →**  
> After introducing the cross-sell option on September 25th, the **cart CTR increased slightly** (from 67.16% to 68.41%), and both **products per order** and **average order value (AOV)** rose modestly. **Revenue per cart session** also edged up, suggesting that cross-sell positively influenced customer purchase behavior, though the overall shift was moderate.

> **Next move →**  
> - Monitor long-term performance to ensure the cross-sell effect remains stable or improves.  
> - Explore testing additional cross-sell opportunities or promotions to further lift AOV.  
> - Evaluate whether different product pairings could increase uptake even more.

---


### 5.6. Portfolio Expansion Analysis: Impact of Birthday Bear Launch

```sql
SELECT
  CASE
    WHEN website_sessions.created_at < '2013-12-12' THEN 'A. Pre_Birthday_Bear'
    WHEN website_sessions.created_at >= '2013-12-12' THEN 'B. Post_Birthday_Bear'
    ELSE 'uh oh...check logic'
  END AS time_period,
  COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
  COUNT(DISTINCT orders.order_id) AS orders,
  COUNT(DISTINCT orders.order_id) / COUNT(DISTINCT website_sessions.website_session_id) AS conv_rate,
  SUM(orders.price_usd) AS total_revenue,
  SUM(orders.items_purchased) AS total_products_sold,
  SUM(orders.price_usd) / COUNT(DISTINCT orders.order_id) AS average_order_value,
  SUM(orders.items_purchased) / COUNT(DISTINCT orders.order_id) AS products_per_order,
  SUM(orders.price_usd) / COUNT(DISTINCT website_sessions.website_session_id) AS revenue_per_session
FROM website_sessions
LEFT JOIN orders
  ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at BETWEEN '2013-11-12' AND '2014-01-12'
GROUP BY 1;
```



**Output**

| time_period         | sessions | orders | conv_rate | total_revenue | total_products_sold | average_order_value | products_per_order | revenue_per_session |
|---------------------|----------|--------|-----------|---------------|---------------------|---------------------|--------------------|---------------------|
| A. Pre_Birthday_Bear | 10998    | 804    | 0.0731    | 41608.69      | 841                 | 51.7595             | 1.0460             | 3.7826              |
| B. Post_Birthday_Bear| 11234    | 900    | 0.0801    | 47532.86      | 962                 | 52.8143             | 1.0689             | 4.2305              |


> **Insight →**  
> After the launch of the Birthday Bear on December 12th, there was a **healthy increase in conversion rate** (7.31% → 8.01%) and a boost in both **revenue per session** and **average order value**. The **products per order** metric also climbed slightly, indicating the new product likely contributed to broader basket sizes.

> **Next move →**  
> - Track Birthday Bear's individual performance to ensure it maintains strong interest.  
> - Consider additional product launches targeting other gift-giving occasions (e.g., holidays, anniversaries).  
> - Monitor how the expanded portfolio affects long-term session value and repeat purchase behavior.

---


### 5.7 Product Refund Rate

```sql
SELECT
  YEAR(order_items.created_at) AS yr,
  MONTH(order_items.created_at) AS mo,
  COUNT(DISTINCT CASE WHEN product_id = 1 THEN order_item_id ELSE NULL END) AS p1_orders,
  COUNT(DISTINCT CASE WHEN product_id = 1 THEN order_item_refunds.order_item_id ELSE NULL END) / 
    COUNT(DISTINCT CASE WHEN product_id = 1 THEN order_item_id ELSE NULL END) AS p1_refund_rt,
  COUNT(DISTINCT CASE WHEN product_id = 2 THEN order_item_id ELSE NULL END) AS p2_orders,
  COUNT(DISTINCT CASE WHEN product_id = 2 THEN order_item_refunds.order_item_id ELSE NULL END) / 
    COUNT(DISTINCT CASE WHEN product_id = 2 THEN order_item_id ELSE NULL END) AS p2_refund_rt,
  COUNT(DISTINCT CASE WHEN product_id = 3 THEN order_item_id ELSE NULL END) AS p3_orders,
  COUNT(DISTINCT CASE WHEN product_id = 3 THEN order_item_refunds.order_item_id ELSE NULL END) / 
    COUNT(DISTINCT CASE WHEN product_id = 3 THEN order_item_id ELSE NULL END) AS p3_refund_rt,
  COUNT(DISTINCT CASE WHEN product_id = 4 THEN order_item_id ELSE NULL END) AS p4_orders,
  COUNT(DISTINCT CASE WHEN product_id = 4 THEN order_item_refunds.order_item_id ELSE NULL END) / 
    COUNT(DISTINCT CASE WHEN product_id = 4 THEN order_item_id ELSE NULL END) AS p4_refund_rt
FROM order_items
LEFT JOIN order_item_refunds
  ON order_items.order_item_id = order_item_refunds.order_item_id
WHERE order_items.created_at < '2014-10-15'
GROUP BY 1, 2;
```

**Output**

| yr   | mo | p1_orders | p1_refund_rt | p2_orders | p2_refund_rt | p3_orders | p3_refund_rt | p4_orders | p4_refund_rt |
|-----|----|-----------|--------------|-----------|--------------|-----------|--------------|-----------|--------------|
| 2012 | 4 | 60        | 0.0167       | NULL      | NULL         | NULL      | NULL         | NULL      | NULL         |
| 2012 | 5 | 99        | 0.0505       | NULL      | NULL         | NULL      | NULL         | NULL      | NULL         |
| 2012 | 6 | 108       | 0.0370       | NULL      | NULL         | NULL      | NULL         | NULL      | NULL         |
| 2012 | 7 | 140       | 0.0571       | NULL      | NULL         | NULL      | NULL         | NULL      | NULL         |
| 2012 | 8 | 169       | 0.0828       | NULL      | NULL         | NULL      | NULL         | NULL      | NULL         |
| 2012 | 9 | 228       | 0.0746       | NULL      | NULL         | NULL      | NULL         | NULL      | NULL         |
| 2012 | 10| 287       | 0.0906       | NULL      | NULL         | NULL      | NULL         | NULL      | NULL         |
| 2012 | 11| 371       | 0.0728       | NULL      | NULL         | NULL      | NULL         | NULL      | NULL         |
| 2012 | 12| 618       | 0.0744       | NULL      | NULL         | NULL      | NULL         | NULL      | NULL         |
| 2013 | 1 | 506       | 0.0593       | 47        | 0.0213       | 0         | NULL         | 0         | NULL         |
| 2013 | 2 | 336       | 0.0496       | 162       | 0.0123       | 0         | NULL         | 0         | NULL         |
| 2013 | 3 | 340       | 0.0714       | 94        | 0.0462       | 0         | NULL         | 0         | NULL         |
| 2013 | 4 | 489       | 0.0643       | 82        | 0.0244       | 0         | NULL         | 0         | NULL         |
| 2013 | 5 | 459       | 0.0614       | 84        | 0.0556       | 0         | NULL         | 0         | NULL         |
| 2013 | 6 | 403       | 0.0758       | 90        | 0.0102       | 0         | NULL         | 0         | NULL         |
| 2013 | 7 | 509       | 0.0727       | 95        | 0.0316       | 0         | NULL         | 0         | NULL         |
| 2013 | 8 | 537       | 0.0549       | 98        | 0.0102       | 0         | NULL         | 0         | NULL         |
| 2013 | 9 | 603       | 0.0282       | 135       | 0.0148       | 0         | NULL         | 0         | NULL         |
| 2013 | 10| 724       | 0.0325       | 174       | 0.0230       | 0         | NULL         | 0         | NULL         |
| 2013 | 11| 728       | 0.0326       | 183       | 0.0219       | 139       | 0.0719       | 0         | NULL         |
| 2013 | 12| 818       | 0.0235       | 200       | 0.0650       | 200       | 0.0650       | 0         | NULL         |
| 2014 | 1 | 784       | 0.0394       | 351       | 0.0171       | 211       | 0.0664       | 202       | 0.0099       |
| 2014 | 2 | 785       | 0.0396       | 193       | 0.0155       | 244       | 0.0697       | 205       | 0.0049       |
| 2014 | 3 | 917       | 0.0501       | 214       | 0.0187       | 267       | 0.0674       | 259       | 0.0154       |
| 2014 | 4 | 1030      | 0.0291       | 246       | 0.0163       | 299       | 0.0569       | 298       | 0.0067       |
| 2014 | 5 | 893       | 0.0571       | 245       | 0.0367       | 288       | 0.0556       | 249       | 0.0241       |
| 2014 | 6 | 961       | 0.0477       | 244       | 0.0369       | 276       | 0.0399       | 264       | 0.0156       |
| 2014 | 7 | 958       | 0.1378       | 237       | 0.0169       | 294       | 0.0680       | 303       | 0.0066       |
| 2014 | 8 | 1065      | 0.1326       | 251       | 0.0319       | 317       | 0.0662       | 327       | 0.0122       |
| 2014 | 9 | 513       | 0.0273       | 135       | 0.0074       | 165       | 0.0485       | 155       | 0.0323       |


> **Insight →**  
> After supplier changes in September 2013, the refund rate for Mr. Fuzzy (product 1) improved steadily, but a major quality problem surfaced again in August–September 2014, with refund rates spiking to ~13–14%.  
> Post supplier replacement in mid-September 2014, refund rates quickly dropped back to acceptable levels.

> **Next move →**  
> Continue monitoring refund rates monthly, especially for any unexpected upticks.  


---

<br>

## 6️⃣ User Analysis

| #    | Assignment                                   | Business Lens                         |
|------|----------------------------------------------|---------------------------------------|
| 6.1 | Identifying Repeat Visitors                  | User retention and loyalty measurement |
| 6.2 | Analyzing Time to Repeat                     | Behavioral cycle insights             |
| 6.3 | Analyzing Repeat Channel Behavior            | Channel efficiency for retention      |
| 6.4 | Analyzing New vs. Repeat Conversion Rates    | Value segmentation of users           |



### 6.1 Identifying Repeat Customers

```sql
CREATE TEMPORARY TABLE sessions_w_repeats AS
SELECT
  new_sessions.user_id,
  new_sessions.website_session_id AS new_session_id,
  website_sessions.website_session_id AS repeat_session_id
FROM (
  SELECT
    user_id,
    website_session_id
  FROM website_sessions
  WHERE created_at >= '2014-01-01'
    AND created_at < '2014-11-01'
    AND is_repeat_session = 0
) AS new_sessions
LEFT JOIN website_sessions
  ON website_sessions.user_id = new_sessions.user_id
  AND website_sessions.is_repeat_session = 1
  AND website_sessions.website_session_id > new_sessions.website_session_id
  AND website_sessions.created_at >= '2014-01-01'
  AND website_sessions.created_at < '2014-11-01';

SELECT
  repeat_sessions,
  COUNT(DISTINCT user_id) AS users
FROM (
  SELECT
    user_id,
    COUNT(DISTINCT new_session_id) AS new_sessions,
    COUNT(DISTINCT repeat_session_id) AS repeat_sessions
  FROM sessions_w_repeats
  GROUP BY user_id
  ORDER BY repeat_sessions DESC
) AS user_level
GROUP BY repeat_sessions;
```


**Output**

| repeat_sessions | users  |
|-----------------|--------|
| 0               | 126,813|
| 1               | 14,086 |
| 2               | 315    |
| 3               | 4,686  |


> **Insight →**  
> A significant number of users returned for at least one repeat session after their first visit. While most users did not return, a considerable chunk (~15%) did, highlighting an opportunity for deeper customer engagement analysis.

> **Next move →**  
> - Explore behavior differences between one-time and repeat visitors.    
> - Strategize ways to encourage first-time users to return.

---


### 6.2 Analyzing Repeat Behavior

```sql
-- Step 1: Create a temporary table identifying new and repeat sessions with their created_at timestamps
CREATE TEMPORARY TABLE sessions_w_repeats_for_time_diff
SELECT
  new_sessions.user_id,
  new_sessions.website_session_id AS new_session_id,
  new_sessions.created_at AS new_session_created_at,
  website_sessions.website_session_id AS repeat_session_id,
  website_sessions.created_at AS repeat_session_created_at
FROM (
  SELECT
    user_id,
    website_session_id,
    created_at
  FROM website_sessions
  WHERE created_at BETWEEN '2014-01-01' AND '2014-11-03'
    AND is_repeat_session = 0
) AS new_sessions
LEFT JOIN website_sessions
  ON website_sessions.user_id = new_sessions.user_id
  AND website_sessions.is_repeat_session = 1
  AND website_sessions.website_session_id > new_sessions.website_session_id
  AND website_sessions.created_at BETWEEN '2014-01-01' AND '2014-11-03';

-- Step 2: Get the first repeat session for each user
SELECT
  user_id,
  new_session_id,
  new_session_created_at,
  MIN(repeat_session_id) AS second_session_id,
  MIN(repeat_session_created_at) AS second_session_created_at
FROM sessions_w_repeats_for_time_diff
WHERE repeat_session_id IS NOT NULL
GROUP BY 1,2,3;

-- Step 3: Create a temporary table with the time difference between first and second sessions
CREATE TEMPORARY TABLE users_first_to_second
SELECT
  user_id,
  DATEDIFF(second_session_created_at, new_session_created_at) AS days_first_to_second_session
FROM (
  SELECT
    user_id,
    new_session_id,
    new_session_created_at,
    MIN(repeat_session_id) AS second_session_id,
    MIN(repeat_session_created_at) AS second_session_created_at
  FROM sessions_w_repeats_for_time_diff
  WHERE repeat_session_id IS NOT NULL
  GROUP BY 1,2,3
) AS first_second;

-- Step 4: Aggregate to find average, minimum, and maximum days between first and second sessions
SELECT
  AVG(days_first_to_second_session) AS avg_days_first_to_second,
  MIN(days_first_to_second_session) AS min_days_first_to_second,
  MAX(days_first_to_second_session) AS max_days_first_to_second
FROM users_first_to_second;
```


**Output**

| avg_days_first_to_second | min_days_first_to_second | max_days_first_to_second |
|---------------------------|---------------------------|---------------------------|
| 33.2622                   | 1                         | 69                        |



> **Insight →**  
> On average, repeat visitors return **around 33 days** after their initial session. Some users come back the very next day, while others take up to 69 days to return.

> **Next move →**  
> - Investigate which channels users are using to come back for the second session.  
> - Understand if certain acquisition channels drive faster repeat behavior.  

---


### 6.3 New vs Repeat Channel Patterns

```sql
SELECT
  CASE
    WHEN utm_source IS NULL AND http_referer IN ('https://www.gsearch.com', 'https://www.bsearch.com') THEN 'organic_search'
    WHEN utm_campaign = 'nonbrand' THEN 'paid_nonbrand'
    WHEN utm_campaign = 'brand' THEN 'paid_brand'
    WHEN utm_source IS NULL AND http_referer IS NULL THEN 'direct_type_in'
    WHEN utm_source = 'socialbook' THEN 'paid_social'
  END AS channel_group,
  COUNT(CASE WHEN is_repeat_session = 0 THEN website_session_id ELSE NULL END) AS new_sessions,
  COUNT(CASE WHEN is_repeat_session = 1 THEN website_session_id ELSE NULL END) AS repeat_sessions
FROM website_sessions
WHERE created_at < '2014-11-05' -- the date of the assignment
  AND created_at >= '2014-01-01' -- prescribed date range in assignment
GROUP BY 1
ORDER BY 3 DESC;
```

**Output**

| channel_group   | new_sessions | repeat_sessions |
|-----------------|--------------|-----------------|
| organic_search  | 7139         | 11507           |
| paid_brand      | 6432         | 11027           |
| direct_type_in  | 6591         | 10564           |
| paid_nonbrand   | 119950       | 0               |
| paid_social     | 7652         | 0               |

> **Insight →**  
> Customers returning for repeat sessions mostly come via organic search, direct type-in, and paid brand channels. Paid nonbrand and paid social hardly see repeat users.  
> Importantly, only about one-third of repeat visits involve paid clicks, and even those are cheaper brand clicks, suggesting low ongoing acquisition costs.

> **Next move →**  
> - Watch for an order conversion analysis to determine how well these repeat visits actually monetize.  
> - Start thinking about the different channel strategies: Should we invest more in brand awareness? Should we optimize direct traffic initiatives?

---



### 6.4. New vs Repeat Performance

```sql
SELECT
  is_repeat_session,
  COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
  COUNT(DISTINCT orders.order_id) / COUNT(DISTINCT website_sessions.website_session_id) AS conv_rate,
  SUM(price_usd) / COUNT(DISTINCT website_sessions.website_session_id) AS rev_per_session
FROM website_sessions
LEFT JOIN orders
  ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2014-11-08' -- the date of the assignment
  AND website_sessions.created_at >= '2014-01-01' -- prescribed date range in assignment
GROUP BY 1;
```

**Output**

| is_repeat_session | sessions | conv_rate | rev_per_session |
|-------------------|----------|-----------|-----------------|
| 0                 | 149,787  | 0.0680    | 4.343754        |
| 1                 | 33,577   | 0.0811    | 5.168828        |


> **Insight →**  
> Repeat sessions perform significantly better than new sessions. They not only have a higher conversion rate(8.1% vs. 6.8%), but also generate more revenue per session($5.17 vs. $4.34).  
> This suggests that customers who return are more valuable than new visitors.



> **Next move →**  
> - Factor the high-value repeat users into your paid acquisition strategy — perhaps spend more to acquire users with high potential for repeat behavior.  



---








