# 🏠 Housing Market Analysis — DAX Measures

> **Power BI Project:** Housing Market Analysis  
> **Data Source:** Google BigQuery  
> **Primary Table:** `Housing`

This document contains the DAX measures used in the **Housing Market Analysis** Power BI project. The measures are organized by analytical purpose and can be referenced alongside the Power BI report.

> **Important:** If the column names in your Power BI model differ from those shown below, replace them with the corresponding names from your actual `Housing` table.

---

## 📊 1. Total Sales

```DAX
Total Sales =
SUM(Housing[purchase_price])
```

**Purpose:** Calculates the total purchase/sales value.

---

## 📈 2. YOY Sales Growth

```DAX
YOY Sales Growth =
VAR CurrentYearSales =
    CALCULATE(
        [Total Sales],
        YEAR(Housing[date]) = YEAR(MAX(Housing[date]))
    )
VAR PrevYearSales =
    CALCULATE(
        [Total Sales],
        YEAR(Housing[date]) = YEAR(MAX(Housing[date])) - 1
    )
RETURN
    IF(
        PrevYearSales <> 0,
        DIVIDE(
            CurrentYearSales - PrevYearSales,
            PrevYearSales
        ),
        BLANK()
    )
```

**Purpose:** Measures the percentage change in sales compared with the previous year.

**Format:** Percentage

---

## 🏘️ 3. Units Sold in Latest Year & Quarter

```DAX
Units Sold Latest Year & Quarter =
CALCULATE(
    COUNTROWS(Housing),
    YEAR(Housing[date]) = YEAR(MAX(Housing[date])),
    QUARTER(Housing[date]) = QUARTER(MAX(Housing[date]))
)
```

**Purpose:** Counts housing transactions/units sold in the latest available year and quarter.

---

## 📅 4. Last 12 Months Sales

```DAX
Last 12 Months Sales =
CALCULATE(
    [Total Sales],
    DATESINPERIOD(
        Housing[date],
        MAX(Housing[date]),
        -12,
        MONTH
    )
)
```

**Purpose:** Calculates sales for the latest rolling 12-month period.

> **Note:** For more robust time-intelligence calculations, a dedicated Calendar/Date table related to `Housing[date]` is recommended.

---

## 🌍 5. Sales by Region

```DAX
Sales by Region =
CALCULATE(
    [Total Sales],
    ALLEXCEPT(
        Housing,
        Housing[region]
    )
)
```

**Purpose:** Calculates sales while keeping the regional filter context.

**Suggested Visual:** Bar chart — Region vs Sales

---

## 📆 6. Total YTD Sales

```DAX
Total YTD Sales =
TOTALYTD(
    [Total Sales],
    Housing[date]
)
```

**Purpose:** Calculates cumulative sales from the beginning of the year up to the current date.

---

## 📐 7. Average Price per Sqm

### If the dataset contains a direct price-per-square-meter column:

```DAX
Average Price per Sqm =
AVERAGE(Housing[price_per_sqm])
```

### If the dataset contains total purchase price and property area (`sqm`):

```DAX
Average Price per Sqm =
DIVIDE(
    SUM(Housing[purchase_price]),
    SUM(Housing[sqm])
)
```

**Purpose:** Measures housing price relative to property area.

> **Note:** Use the version that matches the structure of your dataset.

---

## 🌍 8. Average Price per Sqm by Region

```DAX
Average Price per Sqm by Region =
DIVIDE(
    SUM(Housing[purchase_price]),
    SUM(Housing[sqm])
)
```

**Purpose:** Used with `region` on the visual axis to compare average price per square meter across regions.

---

## 💰 9. Offer Price

### If `offer` represents a discount percentage:

```DAX
Offer Price =
DIVIDE(
    100 * SUM(Housing[purchase_price]),
    100 - AVERAGE(Housing[offer])
)
```

### Row-level calculated column version:

```DAX
Offer Price =
DIVIDE(
    100 * Housing[purchase_price],
    100 - Housing[offer]
)
```

**Purpose:** Estimates the price before applying the offer/discount percentage.

> **Note:** Use the version that matches how `offer` is stored in your dataset.

---

## 📏 10. Offer to Sqm Ratio

```DAX
Offer to Sqm Ratio =
DIVIDE(
    SUMX(
        Housing,
        DIVIDE(
            100 * Housing[purchase_price],
            100 - Housing[offer]
        )
    ),
    SUM(Housing[sqm])
)
```

**Purpose:** Compares the calculated offer price with property area.

---

## 🏷️ 11. Offer to Sqm Ratio by Sales Type

```DAX
Offer to Sqm Ratio by Sales Type =
DIVIDE(
    SUMX(
        Housing,
        DIVIDE(
            100 * Housing[purchase_price],
            100 - Housing[offer]
        )
    ),
    SUM(Housing[sqm])
)
```

**Purpose:** Use `sales_type` as the visual category to compare the offer-to-area ratio across different sales types.

---

## 📊 12. Previous Median Price

```DAX
Previous Median Price =
CALCULATE(
    MEDIAN(Housing[purchase_price]),
    YEAR(Housing[date]) = YEAR(MAX(Housing[date])) - 1
)
```

**Purpose:** Calculates the median purchase price for the previous year.

---

## 📊 13. Current Median Price

```DAX
Current Median Price =
CALCULATE(
    MEDIAN(Housing[purchase_price]),
    YEAR(Housing[date]) = YEAR(MAX(Housing[date]))
)
```

**Purpose:** Calculates the median purchase price for the current/latest year.

---

## 📈 14. Median Sales Price Change

```DAX
Median Sales Price Change =
VAR CurrentMedianPrice =
    [Current Median Price]
VAR PrevMedianPrice =
    [Previous Median Price]
RETURN
    IF(
        PrevMedianPrice <> 0,
        DIVIDE(
            CurrentMedianPrice - PrevMedianPrice,
            PrevMedianPrice
        ),
        BLANK()
    )
```

**Purpose:** Measures the percentage change in median sales price between the current and previous year.

**Format:** Percentage

---

## 📐 15. Average Sqm

```DAX
Average Sqm =
AVERAGE(Housing[sqm])
```

**Purpose:** Calculates the average property size.

---

## 🏦 16. Average Mortgage Credit

If the dataset contains a mortgage-credit column:

```DAX
Average Mortgage Credit =
AVERAGE(Housing[mortgage_credit])
```

**Purpose:** Calculates the average mortgage credit value.

---

## 📉 17. Average Non-Interest Rate

If the dataset contains a non-interest-rate column:

```DAX
Average Non-Interest Rate =
AVERAGE(Housing[non_interest_rate])
```

**Purpose:** Calculates the average non-interest-rate value.

---

# 📊 Recommended Visuals

| Analysis | Suggested Fields |
|---|---|
| **Sales by Region** | `region` + `[Sales by Region]` |
| **YTD Sales Performance** | `date` + `[Total YTD Sales]` |
| **Latest Year & Quarter Units** | `[Units Sold Latest Year & Quarter]` |
| **Last 12 Months Sales** | `date` + `[Last 12 Months Sales]` |
| **Price vs Age** | `age` + `[Average Price per Sqm]` or purchase price |
| **Average Price per Sqm by Region** | `region` + `[Average Price per Sqm by Region]` |
| **Offer Price vs Purchase Price** | Purchase Price + Offer Price |
| **Offer to Sqm Ratio by Sales Type** | `sales_type` + `[Offer to Sqm Ratio by Sales Type]` |
| **Median Sales Price Change** | `[Median Sales Price Change]` |
| **House Type Analysis** | `house_type` + relevant average measures |

---

# 🧠 DAX Concepts Covered

### Aggregation Functions
- `SUM()`
- `AVERAGE()`
- `MEDIAN()`
- `COUNTROWS()`

### Filter & Context Functions
- `CALCULATE()`
- `ALLEXCEPT()`

### Mathematical & Logical Functions
- `DIVIDE()`
- `IF()`
- `BLANK()`
- `SUMX()`

### Date & Time Functions
- `YEAR()`
- `QUARTER()`
- `MAX()`
- `DATESINPERIOD()`
- `TOTALYTD()`

### Advanced DAX Concepts
- Variables using `VAR`
- Filter context
- Time intelligence
- Year-over-year analysis
- Rolling 12-month analysis
- Regional analysis
- Percentage-change calculations
- Iterator functions

---

# 📚 Learning Outcomes

Through these measures, the project demonstrates practical experience in:

- Creating reusable DAX measures
- Performing sales and price analysis
- Applying filter context
- Performing time-based analysis
- Calculating year-over-year growth
- Calculating rolling 12-month metrics
- Comparing regional performance
- Creating price-per-square-meter metrics
- Building measures for interactive Power BI dashboards

---

# 📝 Notes

- These measures are documented for the **Housing Market Analysis Power BI project**.
- The formulas should be validated against the final Power BI data model before reuse.
- Column names may need to be adjusted if the source schema differs.
- For production-quality time-intelligence calculations, a dedicated **Calendar/Date table** related to `Housing[date]` is recommended.
- The main `README.md` contains the project overview, while this file documents the DAX calculations separately to keep the repository organized.

---

## 🔗 Project

**Housing Market Analysis — Power BI & Google BigQuery**

**Technologies:** Google BigQuery | SQL | Power BI | Power Query | DAX
