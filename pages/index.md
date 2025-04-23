---
title: Project Overview & Usage Guide
---

<Alert status="info">
This is the home page for the dashboard. To view the full analysis, click on "E-commerce Dashboard" from the menu on the left.<br><br>

💡 Tip: Before diving in, check out the **"How to Use This Dashboard"** section below to learn how to view the underlying SQL queries and get the most out of the interface.
</Alert>

## Business Context/Scenario

This dashboard was created for the **Analytics team at a Brazilian e-commerce marketplace** to help the Operations and Growth teams monitor performance, understand customer behavior, and guide decision-making around marketing, logistics, and promotions.

Key business questions this dashboard helps answer:

- How are revenue and orders trending over time?
- What’s the average order value and how does it shift?
- Which product categories drive the most sales volume?
- How do customers prefer to pay, and are promos overused?
- Where are our highest-value customers located?
- Which sellers are generating the most revenue and volume?
- Which regions have the highest customer concentration?

This dashboard uses realistic e-commerce transaction data (simulated) and is designed to be used by:
- The **Operations team** (delivery times, region analysis)
- The **Growth & Marketing team** (category performance, AOV)
- The **Finance team** (payment method breakdown)

It supports custom date ranges and is meant to be reviewed on a **weekly or monthly basis**.

--------------------------------------

## How to Use This Dashboard
- To hide/show SQL queries click the "..." menu on the top right and click either "**Show Queries**" or "**Hide Queries**" (They will not appear on this page, but on the E-Commerce Dashboard" page)
- You can then click on the individual query name or records to see either the **query** or the **query results** respectively
- You can also toggle Light/Dark mode from the "..."  menu as well
- Adjust the **Date Range** at the top of the dashboard to filter all metrics and visualizations dynamically
- You can navigate the page by scrolling, or you can click the individual sections at the top right under "**On this page**"
- Use **Alerts and Info Boxes** throughout the dashboard to interpret key insights and business implications.

<Alert status="info">
ℹ️ All SQL queries used in this project are also available at <a href="https://github.com/pbello22/E-CommerceProjectSQL/blob/main/queries.sql" target="_blank">https://github.com/pbello22/E-CommerceProjectSQL/blob/main/queries.sql</a>.
</Alert>

--------------------------------------
## SQL Queries
All data visualizations and metrics are powered by SQL queries written in BigQuery SQL syntax. Each query is:

- Fully parameterized using `${inputs.date_filter.start}` and `${inputs.date_filter.end}` for dynamic filtering
- Designed for **performance and clarity** (using joins, aggregations, and window functions)



--------------------------------------
## Tech Stack
- **Evidence.dev**: Open-source BI tool used for building markdown-powered dashboards with SQL + modern components
- **BigQuery**: Used for storing and querying data
