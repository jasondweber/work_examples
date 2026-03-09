Project: Supply Chain Management & Inventory Burn-Rate Engine

📌 Business Problem
The e-commerce team struggled with stockouts and under-ordering due to a lack of visibility into inventory total picture and misunderstanding "burn rates." Static 30-day averages failed to account for seasonality or aggressive sales targets, leading to lost revenue during peak periods and high holding costs for slow-movers.

🚀 The Solution
I developed a Daily Burn-Rate Simulation that merges real-time inventory levels (Warehouse, Amazon FBA, and In-Transit to Main Warehouse and Amazon Warehouse) with a 5-year seasonal sales forecast based on the past year sales behavior.

Key Features:
Multi-Source Inventory Aggregation: Centralized data from FFE (warehouse), Amazon, and Purchase Orders to create a "True Global Stock" view.  Data is sourced from importing CSV files and Google Sheets into Big Query and Google Cloud Storage.

Predictive Stockout Logic: Instead of simple math, the model runs a daily simulation to find the exact date inventory hits zero based on forecasted daily units.

Scenario Modeling: Created two distinct projections—Business as Usual (BAU) and Sales Goal (using a target_sales_goal multiplier)—allowing procurement to plan for "best-case" growth.

Financial Impact: Integrated unit costs and shipping estimates to provide the Finance team with a 6-month and 12-month capital requirement forecast.

🏗 Data Architecture & Logic
The model is built using a Modular CTE approach to ensure high performance and readability:

Inventory Snapshots: Uses QUALIFY RANK() to grab the most recent month-end inventory levels across all channels.

Sales Velocity: Calculates trailing 30/90-day averages across Amazon and Shopify to provide baseline context.

Burn-Rate Engine: Joins current stock against a t_forecast_5_years table to identify the first date where units_running_sum >= total_inventory.

Demand Projection: Calculates the specific number of units required to maintain stock for 6 and 12 months starting from the predicted stockout date.

🛠 Technical Implementation Details
Data Integrity: Implemented assertions for uniqueKey on product_sku to prevent fan-outs.

Schema: Materialized as a table in the marts schema for high-speed downstream reporting in BI tools (Tableau/Looker) based on Intermediate models (I call them Transformations for my end users)
