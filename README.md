# Superstore-sales-analysis
Power BI Sales and Profitability Analysis Dashboard
# Sales Performance Dashboard (Power BI)

An interactive three-page Power BI dashboard that turns four years of retail order data into clear business insights: revenue trends, top products and categories, regional performance and year-over-year growth.

> **Preview:** add your screenshots to `/screenshots` and link them here.
>
> `![Overview](screenshots/overview.png)`
> `![Product & Growth Analysis](screenshots/product_growth.png)`
> `![Regional Performance](screenshots/regional.png)`

---

## Business Questions

1. How is revenue trending month by month and quarter by quarter?
2. Which products and categories drive the most sales?
3. Which regions and states perform best, and which lag behind?
4. Is the business growing year over year, and where is that growth coming from?

## Dataset

| Item | Detail |
|---|---|
| Source | Superstore sales dataset (`train.csv`) |
| Period | 3 Jan 2015 to 30 Dec 2018 |
| Size | 9,800 order lines, 4,922 orders, 793 customers, 1,861 products |
| Geography | United States, 4 regions, 49 states |
| Hierarchy | 3 categories, 17 sub-categories, 3 customer segments |

**Limitation:** this version of the dataset has no `Profit`, `Quantity` or `Discount` column, so the analysis is about revenue and order volume. The "Profit vs Sales" view from the original brief was replaced by **Sales vs Orders by Sub-Category**. No profit values were estimated.

## Tools

- **Python (pandas):** data profiling and cleaning
- **Power BI Desktop:** data model, DAX and report
- **DAX:** measures and time intelligence

## Data Cleaning

Cleaning was done before loading into Power BI, producing `superstore_clean.csv`.

| Issue | Fix |
|---|---|
| Dates stored as `dd/mm/yyyy` text | Parsed into real dates (`yyyy-mm-dd`) |
| 11 missing postal codes (all Burlington, VT) | Filled with `05401` |
| Postal codes read as numbers | Stored as 5-digit text |
| 32 Product IDs with more than one name | Kept the most frequent name |
| `Country` had a single value | Dropped |
| 8 repeated Order ID + Product ID rows | Kept as separate line items (sales values mostly differ) |
| No time-grouping fields | Added Year, Quarter, Year-Quarter, Month, Month Number, Year-Month, Ship Days |

Checks passed: no duplicate rows, no zero or negative sales, no ship date earlier than order date.

## Data Model

One fact table and one date table.

- **Orders:** cleaned order lines (fact table)
- **Calendar:** Date, Year, Quarter, Month, Month Number, marked as the date table
- **Relationship:** `Calendar[Date]` to `Orders[Order Date]`, one-to-many
- **_Measures:** a table holding all measures

## Key Measures

```DAX
Total Sales      = SUM(Orders[Sales])
Orders Count     = DISTINCTCOUNT(Orders[Order ID])
Customers        = DISTINCTCOUNT(Orders[Customer ID])
Avg Order Value  = DIVIDE([Total Sales], [Orders Count])
Sales PY         = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Calendar'[Date]))
YoY Growth %     = DIVIDE([Total Sales] - [Sales PY], [Sales PY])
% of Total Sales = DIVIDE([Total Sales], CALCULATE([Total Sales], ALLSELECTED(Orders)))
```

The full list (Calendar table, Sales YTD, Avg Ship Days, an improved YoY measure, sanity-check values and troubleshooting) is in [`docs/DAX_Reference.pdf`](docs/DAX_Reference.pdf).

## Dashboard Pages

### 1. Overview
- KPI cards: Total Sales, Orders, Customers, Average Order Value, YoY Growth
- Top 10 products by sales
- Monthly sales, one line per year
- Quarterly sales in chronological order
- Sales share by category
- Slicers: Year, Quarter, Segment, Category, Region

### 2. Product & Growth Analysis
- Category and sub-category matrix with share of total sales
- Scatter chart: Sales vs Orders by sub-category (bubble size = average order value)
- Total Sales vs prior-year Sales by year

### 3. Regional Performance
- State map shaded by total sales
- Sales by region (column and donut)
- Top 10 states by sales

### Interactivity
| Feature | What it does |
|---|---|
| Slicers | Filter every visual by year, quarter, segment, category and region |
| Reset button | Bookmark that clears slicer selections (on every page) |
| Page navigator | Buttons for all three pages with the current page highlighted |
| Hover tooltip | Hovering over a scatter bubble shows sales, orders, average order value and a mini yearly chart |
| Matrix drill | Expand category to sub-category |

## Key Insights

| Metric | Result |
|---|---|
| Total sales (2015 to 2018) | **$2.26M** |
| Orders / customers | 4,922 orders from 793 customers |
| Average order value | $459 |
| 2018 sales | **$722K**, up 20.3% on 2017 |

1. **Growth recovered after a dip.** Sales fell 4.3% in 2016, then grew 30.6% in 2017 and 20.3% in 2018.
2. **Sales are highly seasonal.** November and December make up about 30% of annual sales; January and February about 7%.
3. **Technology leads.** $827K (36.6%), ahead of Furniture ($729K) and Office Supplies ($705K). In 2018 Office Supplies grew fastest (+31.8%) while Furniture slowed (+8.4%).
4. **A few products carry the business.** Phones and Chairs are about 29% of sales together, and the Canon imageCLASS 2200 copier ($61.6K) earns more than double the next product.
5. **West and East dominate.** They generate 61% of revenue, and California alone contributes 19.7%.
6. **Growth is from order volume, not bigger baskets.** Orders grew 75% (947 to 1,661) while average order value fell from $507 to $435.
7. **Central is stalling.** After +42% in 2017, Central dipped 2.8% in 2018 while every other region grew.
8. **Customers are loyal and consumer-led.** Consumer is 50.8% of sales, 98% of customers ordered more than once, and the top 10% of customers account for about 31% of sales.

## Recommendations

1. Plan inventory, staffing and marketing around Q4, and test promotions in January and February.
2. Investigate Central's 2018 slowdown, since it is the only region that declined.
3. Raise order value with bundles, free-shipping thresholds and upsells.
4. Keep investing in Technology and Office Supplies, and find out why Furniture slowed.
5. Review low-selling sub-categories such as Fasteners and Labels.
6. Protect top customers with a loyalty or account-management programme.

## Known Limitations

- **No profit analysis.** The dataset has no profit column.
- **YoY Growth % card.** With no year selected, it shows 46.9%, which compares the whole four-year total with combined prior-year sales and is not a true growth rate. Select a year for the correct figure (2018 vs 2017 is +20.3%). An improved measure is in the DAX reference.
- **Formatting.** The matrix "% of Total Sales" column displays decimals (0.37), and the Orders card displays 4,922 as "5K".

## Repository Structure

```
├── data/
│   ├── train.csv
│   └── superstore_clean.csv
├── dashboard/
│   └── Superstore_Sales_Analysis.pbix
├── docs/
│   ├── Project_Documentation.pdf
│   └── DAX_Reference.pdf
├── screenshots/
│   ├── overview.png
│   ├── product_growth.png
│   └── regional.png
└── README.md
```
