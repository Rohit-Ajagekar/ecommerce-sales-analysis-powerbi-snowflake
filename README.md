\# E-Commerce Sales Analysis Dashboard | Power BI \& Snowflake



An end-to-end business intelligence project built using \*\*Microsoft Power BI, Snowflake, SQL, and DAX\*\* to analyze sales performance, profitability, inventory risks, estimated lost sales, and promotional effectiveness.



\## Project Overview



Retail businesses need actionable insights into revenue, product performance, inventory availability, and promotional activity.



This project uses an interactive Power BI dashboard connected to Snowflake through DirectQuery to explore sales performance and support data-driven business decisions.



The project uses a synthetic retail dataset covering \*\*2022–2025\*\*, with approximately \*\*10 million sales transactions\*\*, 30 stores, 5,000 products (SKUs), and 10,000 customers.



\*\*Data disclaimer:\*\* The dataset is synthetic. Business findings are illustrative and should not be interpreted as verified results from a real retailer.



\## Dashboard Pages



\### 1. Executive Overview

\- Total sales and gross profit

\- Customer and transaction KPIs

\- Monthly sales trends

\- Revenue by city, store, and product category

\- Top-performing stores



\### 2. Inventory Intelligence

\- Inventory valuation

\- Stockout-risk and slow-moving products

\- Stock levels versus safety stock and reorder points

\- Inventory status analysis

\- Products requiring further investigation



\### 3. Lost Sales Analysis

\- Comparison of baseline and anomaly-version sales

\- Estimated revenue and unit gaps

\- Analysis by city, store, product, and category

\- Identification of potential inventory-related sales gaps



\*Lost-sales figures are estimates based on synthetic baseline and anomaly scenarios, not confirmed real-world losses.\*



\### 4. Product \& Promotion Performance

\- Gross profit and gross margin

\- Revenue associated with promotions

\- Promotion-type performance

\- Product and category profitability

\- Customer segments and preferred sales channels



\## Dataset Overview



| Attribute | Details |

|---|---|

| Dataset type | Synthetic multi-store retail dataset |

| Period | 2022–2025 |

| Sales transactions | Approximately 10 million |

| Stores | 30 |

| Products / SKUs | 5,000 |

| Customers | 10,000 |

| Data platform | Snowflake |

| Visualization | Microsoft Power BI |



\## Technology Stack



\- \*\*Snowflake:\*\* Cloud data storage and SQL querying

\- \*\*Power BI:\*\* Interactive dashboard development

\- \*\*SQL:\*\* Data querying and validation

\- \*\*DAX:\*\* KPI calculations and analytical measures

\- \*\*Power Query:\*\* Data preparation and transformation

\- \*\*DirectQuery:\*\* Connects Power BI to Snowflake for source-side queries

\- \*\*Data Modeling:\*\* Relationships between sales, product, store, customer, date, and inventory tables



\## Business Questions



1\. Which stores, cities, and product categories generate the most revenue?

2\. How do revenue and profitability change over time?

3\. Which products have potential stockout risks or slow-moving inventory?

4\. Where are the largest estimated sales gaps?

5\. Which products and promotion types perform best?

6\. How can interactive reporting help stakeholders investigate performance?



\## Repository Contents



\- `E-Commerce Dashboard.pbix` — Power BI report

\- `Snaps/` — Dashboard screenshots

\- `SQL \& DAX/` — SQL scripts and DAX documentation

\- `Video Walkthrough.mp4` — Project explanation and dashboard walkthrough



\## How to View the Project



1\. Browse the screenshots in the `Snaps` folder.

2\. Open `Video Walkthrough.mp4` to watch the project demonstration.

3\. Review the SQL scripts and DAX measures in `SQL \& DAX`.

4\. Open `E-Commerce Dashboard.pbix` using Power BI Desktop.



\*\*Connection requirement:\*\* The Power BI report uses Snowflake DirectQuery. Interacting with the report may require access to the relevant Snowflake database, the appropriate permissions, and a valid connection configuration.



\## Limitations



\- The dataset is synthetic and does not represent an actual retailer.

\- Lost-sales analysis estimates differences between baseline and anomaly scenarios.

\- Inventory snapshots do not necessarily represent continuous historical stock levels.

\- Reproducing the live report may require access to the underlying Snowflake tables.



\## Skills Demonstrated



\*\*Business Intelligence | Power BI | Snowflake | SQL | DAX | Data Modeling | KPI Development | Data Visualization | Retail Analytics\*\*



\---



Developed as a portfolio project demonstrating end-to-end data analysis and business intelligence dashboard development.

