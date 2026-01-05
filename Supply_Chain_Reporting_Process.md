# Inventory Forecasting & Automated Reorder Engine

## Project Overview
This SQL project demonstrates a dynamic inventory forecasting engine designed to prevent stockouts and automate purchasing decisions for a high-volume retail brand. 

Instead of relying on static averages, this script builds a **"Burn Down" simulation**. It combines current inventory snapshots with historical seasonal trends to project exactly when stock will run out (the "Stockout Date") and calculates the optimal reorder quantity based on lead times and growth targets.

## Business Problem
Managing inventory requires balancing two risks:
1.  **Stockouts:** Losing revenue because product isn't available.
2.  **Overstock:** Tying up capital in inventory that moves too slowly.

Static formulas (e.g., "sell-through rate") often fail because they don't account for seasonality (e.g., Q4 spikes). This engine solves that by using a 365-day historical lookback to model future demand shape.

## Key Features
* **Multi-Location Inventory:** Aggregates stock from 3PL warehouses (FFE), Amazon FBA, and "In Transit" purchase orders.
* **Seasonal Forecasting:** Uses `DATE_SUB` logic to map the previous year's daily sales curve to the current year, preserving seasonal spikes.
* **Scenario Modeling:** Runs two parallel forecasts:
    * *Business as Usual (BAU):* Standard historical velocity.
    * *Growth Goal:* Historical velocity multipliers to test "what if" growth scenarios.
* **Automated Reorder Logic:** Dynamically calculates:
    * `Recommended Reorder Date`: (Stockout Date) minus (Lead Time + Safety Buffer).
    * `Order Quantity`: Exact units needed to cover the next 6 or 12 months *starting* from the predicted stockout date.

## Technical Highlights
This script is written in **Standard SQL (BigQuery dialect)** and utilizes several advanced techniques:

* **Common Table Expressions (CTEs):** Used extensively to modularize logic into readable steps (Dimensions -> Inventory -> Forecasting -> Reporting).
* **Window Functions:** `SUM() OVER (PARTITION BY ... ORDER BY ...)` is used to calculate the running total (cumulative sum) of future sales to simulate the inventory "burn down" day by day.
* **Date Manipulation:** Complex date arithmetic to project historical trends into future timelines (The "Loop" logic).
* **Handling Nulls:** Robust use of `COALESCE` and `SAFE_DIVIDE` to ensure production stability.

## Logic Flow
1.  **Data Staging:** Ingests raw data from ERP, Amazon Seller Central, and 3PL feeds.
2.  **Velocity Calculation:** Calculates 30/90-day averages for context, while pulling full 365-day history for the core model.
3.  **Simulation:** Stacks historical data to create a 3-year future timeline and runs the cumulative sales calculation against current stock.
4.  **Stockout Identification:** Identifies the first date where `Cumulative Sales > Total On Hand`.
5.  **Reorder Calculation:** Back-calculates when the PO must be placed to arrive before that stockout date.
