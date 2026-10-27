# DAX Measures — Retail Analytics Dashboard

This document records the key DAX measures used or discussed for the Power BI retail analytics project connected to Snowflake.

> **Important:** Verify table and column names against your actual Power BI model before using these measures. Because the model uses Snowflake DirectQuery, some complex measures may perform slowly or be restricted by the source/model configuration. Where possible, push heavy transformations and aggregations into Snowflake SQL.

## Model assumptions

The examples below assume these tables and columns exist:

- `SALES_TRANSACTIONS`: `TOTAL_VALUE`, `QUANTITY`, `RECEIPT_ID`, `PROMO_ID`, `STORE_ID`, `SKU_ID`, `TRANSACTION_TIMESTAMP`
- `SALES_TRANSACTIONS_CLEAN`: clean baseline table with equivalent sales columns
- `DimSKU`: `SKU_ID`, `COST_PRICE`
- `STORE_MASTER`: `STORE_ID`, store attributes
- `CUSTOMER_MASTER`: customer identifier
- `INVENTORY_SNAPSHOT`: `STOCK_ON_HAND`, `SAFETY_STOCK`, `REORDER_POINT`, `STORE_ID`, `SKU_ID`
- `SKU_INVENTORY_FLAGS`: anomaly flag fields such as `FLAG_TYPE`, if present
- `PROMOTIONS`: promotion attributes such as `PROMO_ID` and `PROMO_TYPE`
- `DimDate`: `Date`

Ensure relationships are configured correctly. In particular, `DimSKU[SKU_ID]` should have a valid relationship to the SKU key in the sales and inventory tables. If your actual field names differ, update the formulas.

---

## 1. Executive Overview

### Total Sales

```DAX
Total Sales =
SUM ( SALES_TRANSACTIONS[TOTAL_VALUE] )
```

**Purpose:** Sums the sales value in the currently selected filter context.

### Total Units Sold

```DAX
Total Units Sold =
SUM ( SALES_TRANSACTIONS[QUANTITY] )
```

**Purpose:** Total quantity sold.

### Total Transactions

```DAX
Total Transactions =
DISTINCTCOUNT ( SALES_TRANSACTIONS[RECEIPT_ID] )
```

**Purpose:** Counts distinct receipts/orders. Use this only if `RECEIPT_ID` identifies a transaction consistently across the fact table.

### Total Customers

Use the actual customer key in your model. For example:

```DAX
Total Customers =
DISTINCTCOUNT ( SALES_TRANSACTIONS[CUSTOMER_ID] )
```

**Note:** If the sales fact table does not contain `CUSTOMER_ID`, use the appropriate customer key/table and relationship from your model. Do not use a guessed column name without checking the schema.

### Cost of Goods Sold

```DAX
Cost of Goods Sold =
SUMX (
    SALES_TRANSACTIONS,
    SALES_TRANSACTIONS[QUANTITY] * RELATED ( DimSKU[COST_PRICE] )
)
```

**Purpose:** Estimates the cost of sold units using the product cost in `DimSKU`. Requires a working many-to-one relationship from sales to `DimSKU`.

### Gross Profit

```DAX
Gross Profit =
[Total Sales] - [Cost of Goods Sold]
```

### Gross Margin %

```DAX
Gross Margin % =
DIVIDE ( [Gross Profit], [Total Sales], 0 )
```

Format this measure as Percentage in Power BI.

### Average Order Value

```DAX
Average Order Value =
DIVIDE ( [Total Sales], [Total Transactions], 0 )
```

**Purpose:** Average sales value per distinct receipt.

### Store Revenue Rank

```DAX
Store Revenue Rank =
RANKX (
    ALL ( STORE_MASTER ),
    [Total Sales],
    ,
    DESC,
    DENSE
)
```

**Why `ALL(STORE_MASTER)`?** It removes filters from the store dimension while ranking stores, so each store is compared against all stores rather than only itself. Other report filters, such as date or category, can still affect the measure depending on the model and visual context. If you want ranking within selected stores only, use `ALLSELECTED(STORE_MASTER)` instead.

---

## 2. Inventory Intelligence

### Inventory Value

```DAX
Inventory Value =
SUMX (
    INVENTORY_SNAPSHOT,
    INVENTORY_SNAPSHOT[STOCK_ON_HAND] * RELATED ( DimSKU[COST_PRICE] )
)
```

**Purpose:** Values the snapshot inventory at product cost. Requires a valid relationship between `INVENTORY_SNAPSHOT` and `DimSKU`.

### Stores with Zero Stock

```DAX
Stores with Zero Stock =
CALCULATE (
    DISTINCTCOUNT ( INVENTORY_SNAPSHOT[STORE_ID] ),
    INVENTORY_SNAPSHOT[STOCK_ON_HAND] = 0
)
```

**Interpretation:** Counts stores represented by inventory rows with zero on-hand stock. This is not necessarily a count of all stores with any stockout unless the visual/filter context is at the relevant SKU grain.

### SKUs at or Below Safety Stock

```DAX
SKUs at or Below Safety Stock =
CALCULATE (
    DISTINCTCOUNT ( INVENTORY_SNAPSHOT[SKU_ID] ),
    FILTER (
        INVENTORY_SNAPSHOT,
        INVENTORY_SNAPSHOT[STOCK_ON_HAND]
            <= INVENTORY_SNAPSHOT[SAFETY_STOCK]
    )
)
```

**Purpose:** Counts distinct SKUs whose snapshot stock is at or below safety stock in the current context. In a store/SKU matrix this can be evaluated at the relevant context; for total risk counts, be explicit about whether the intended grain is SKU or store-SKU combination.

### Inventory Status (calculated column)

```DAX
Inventory Status =
SWITCH (
    TRUE (),
    INVENTORY_SNAPSHOT[STOCK_ON_HAND] = 0, "Out of Stock",
    INVENTORY_SNAPSHOT[STOCK_ON_HAND]
        < INVENTORY_SNAPSHOT[SAFETY_STOCK], "Low Stock",
    INVENTORY_SNAPSHOT[STOCK_ON_HAND]
        < INVENTORY_SNAPSHOT[REORDER_POINT], "Reorder Required",
    "Healthy"
)
```

**Important:** The order of conditions matters. This classifies each inventory snapshot row. If DirectQuery restrictions prevent a calculated column, create the classification in Snowflake instead. If equality with the safety stock should count as low stock, change `<` to `<=` in the relevant condition.

### Average Flagged Window Duration (days)

If `SKU_INVENTORY_FLAGS` contains `WINDOW_START` and `WINDOW_END` dates:

```DAX
Average Flagged Window Duration (Days) =
AVERAGEX (
    FILTER (
        SKU_INVENTORY_FLAGS,
        NOT ISBLANK ( SKU_INVENTORY_FLAGS[WINDOW_START] )
            && NOT ISBLANK ( SKU_INVENTORY_FLAGS[WINDOW_END] )
            && SKU_INVENTORY_FLAGS[WINDOW_END]
                >= SKU_INVENTORY_FLAGS[WINDOW_START]
    ),
    DATEDIFF (
        SKU_INVENTORY_FLAGS[WINDOW_START],
        SKU_INVENTORY_FLAGS[WINDOW_END],
        DAY
    ) + 1
)
```

**Interpretation:** This is the average duration of the recorded flag window, inclusive of both endpoints. It is **not proof of the number of days a product was actually out of stock** unless the source data explicitly defines the window that way.

---

## 3. Lost Sales Analysis

The project compares a clean baseline table with an anomaly-version table. These calculations should be described as **estimated gaps based on synthetic scenarios**, not confirmed real-world lost sales.

### Baseline Revenue

```DAX
Baseline Revenue =
SUM ( SALES_TRANSACTIONS_CLEAN[TOTAL_VALUE] )
```

### Anomaly-Version Revenue

```DAX
Anomaly Revenue =
SUM ( SALES_TRANSACTIONS[TOTAL_VALUE] )
```

### Estimated Revenue Gap

```DAX
Estimated Revenue Gap =
[Baseline Revenue] - [Anomaly Revenue]
```

### Estimated Revenue Gap %

```DAX
Estimated Revenue Gap % =
DIVIDE ( [Estimated Revenue Gap], [Baseline Revenue], 0 )
```

### Baseline Units

```DAX
Baseline Units =
SUM ( SALES_TRANSACTIONS_CLEAN[QUANTITY] )
```

### Anomaly-Version Units

```DAX
Anomaly Units =
SUM ( SALES_TRANSACTIONS[QUANTITY] )
```

### Estimated Unit Gap

```DAX
Estimated Unit Gap =
[Baseline Units] - [Anomaly Units]
```

**Modeling caveat:** For these comparisons to work by city, store, category, product, and date, the clean and anomaly fact tables must be filtered consistently by shared dimensions. Confirm that the relevant dimension relationships and keys align across both fact tables. If the two versions are not row-for-row comparable or do not share equivalent filtering logic, the difference may not represent a meaningful gap.

---

## 4. Product & Promotion Performance

### Orders with a Promotion

```DAX
Orders with a Promotion =
CALCULATE (
    DISTINCTCOUNT ( SALES_TRANSACTIONS[RECEIPT_ID] ),
    FILTER (
        SALES_TRANSACTIONS,
        NOT ISBLANK ( SALES_TRANSACTIONS[PROMO_ID] )
            && TRIM ( SALES_TRANSACTIONS[PROMO_ID] ) <> ""
            && UPPER ( TRIM ( SALES_TRANSACTIONS[PROMO_ID] ) ) <> "NULL"
    )
)
```

**Purpose:** Counts distinct receipts with a nonblank promotion ID. This treats both blank values and the literal text `"NULL"` as no promotion. Confirm how missing values are represented in Snowflake.

### Promotional Revenue

```DAX
Promotional Revenue =
CALCULATE (
    [Total Sales],
    FILTER (
        SALES_TRANSACTIONS,
        NOT ISBLANK ( SALES_TRANSACTIONS[PROMO_ID] )
            && TRIM ( SALES_TRANSACTIONS[PROMO_ID] ) <> ""
            && UPPER ( TRIM ( SALES_TRANSACTIONS[PROMO_ID] ) ) <> "NULL"
    )
)
```

### Promotional Revenue Share

```DAX
Promotional Revenue Share % =
DIVIDE ( [Promotional Revenue], [Total Sales], 0 )
```

Format as Percentage. This measures the share of revenue associated with transactions carrying a promotion ID; it does not by itself establish the incremental revenue caused by a promotion.

---

## Validation checklist

Before publishing the report or presenting these measures:

- [ ] Verify every table and column name against the actual Power BI model.
- [ ] Confirm `RELATED(DimSKU[COST_PRICE])` works through the intended relationship.
- [ ] Confirm whether `RECEIPT_ID` is the correct transaction identifier.
- [ ] Check that clean and anomaly sales tables are both filtered by the intended shared dimensions.
- [ ] Validate that the inventory snapshot grain is understood (for example, one row per store-SKU snapshot).
- [ ] Format currency and percentage measures correctly.
- [ ] Test measures with slicers and at total-row level.
- [ ] For DirectQuery performance issues, consider moving expensive row-level calculations into Snowflake SQL.
- [ ] Document any assumptions and label synthetic lost-sales results as estimates.

## DirectQuery performance note

Measures using `SUMX`, `FILTER`, `RELATED`, or complex row-level expressions may generate expensive source queries. If performance is poor, calculate reusable fields or pre-aggregate data in Snowflake views and keep Power BI measures simpler.
