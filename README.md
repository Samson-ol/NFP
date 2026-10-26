# Nigeria Food Price Dashboard

An interactive Power BI dashboard that analyses monthly food prices and inflation across 66 Nigerian markets from January 2007 to August 2026.


## Project overview

The project covers the full workflow: understanding a raw dataset, finding and fixing data quality problems, designing a data model, writing calculations, and presenting the results in a dashboard.

It answers four questions:

1. What is happening to food prices overall?
2. How much has each food cost over time?
3. How fast are prices rising, and for which foods?
4. Where are prices highest and lowest?

## Dataset

| Item | Detail |
|---|---|
| File | NFP DATA.xlsx |
| Size | 17,464 rows, 43 columns |
| Period | Jan 2007 to Aug 2026 (236 months) |
| Locations | 74 (66 real markets and 8 summary rows) |
| Foods | Rice, beans, yam, gari, fish, eggs, beef, milk |
| Measures | Closing, high and low price, year-on-year inflation, food price index |
| Source | [Add the original source and licence] |

Price units are not documented in the file and are still to be confirmed.

## Data cleaning

I flagged problems instead of deleting data, and worked only on copies of the raw file.

- **Summary rows:** 8 rows (zone rows, National Average, Market Average, Kano State (1)) are not real markets. I added a `location_type` column (Market, Zone, Benchmark, Unverified) and counted Market rows only.
- **Location columns:** the original state column mixed real states with summary labels. I built a clean `State` column and added a `Region` column from a lookup sheet.
- **Kano State (1):** an unexplained series with its own coordinates, marked Unverified and excluded from rankings until the source confirms what it is.
- **Missing values:** inflation is blank for 2007 (it needs a prior year) and was left blank on purpose.
- **Checks:** no duplicate market-months, no zero or negative prices, and the low is never above the close or the high.
- **geo_id:** its different lengths come from the coordinates it is built from. All IDs are unique, so it was kept as text and used as the key.

## Data model

A star schema built in Power BI:

| Table | Role |
|---|---|
| Location | Dimension: state, LGA, market, region, coordinates, location_type |
| Date | Dimension: one row per month, with year, quarter and a partial-year flag |
| Commodity | Dimension: food name and category |
| Prices | Fact (wide): used for the food price index |
| PricesLong | Fact (long): one row per market, month and food |

`PricesLong` was created in Power Query by unpivoting the food columns, so one measure works for every food.

## Measures (DAX)

14 measures, all filtered to Market rows only. Food measures return blank unless exactly one food is selected, because averaging foods with different units is meaningless. Example:

```
Avg Price =
IF(
    HASONEVALUE(Commodity[Name]),
    CALCULATE(AVERAGE(PricesLong[Close]), Location[location_type] = "Market")
)
```

## Dashboard pages

| Page | Shows |
|---|---|
| Overview | Food price index, latest inflation, index by state |
| Prices | Price of one selected food over time, and how many times higher it is than in Jan 2007 |
| Inflation | Median year-on-year inflation over time and by food |
| Geography | Average price by state and region, and by market |
| About the Data | Coverage, limitations and definitions |

## Key findings

- The average rice price rose from about ₦272 (Jan 2007) to about ₦4,724 (Aug 2026), roughly 17.4 times.
- The food price index (National Average row) rose from 1.82 in 2023 to 3.56 in 2024 and 4.06 in 2025.
- In 2025, average rice prices were about ₦6,002 in Abia and about ₦4,023 in Sokoto, a gap of about 49%.
- Fish, yam, beans and milk are far more volatile than rice, gari, eggs and beef, mainly in Borno and Yobe.

## Limitations

- 54 of the 66 markets are in the North East, so results are not representative of Nigeria as a whole.
- Prices are nominal (not adjusted for inflation), and units are unconfirmed.
- The Region column is my own classification and should be verified.
- 2026 contains only eight months.

## How to use

1. Download or clone this repository.
2. Open `[dashboard-file-name].pbix` in Power BI Desktop.
3. If prompted, point the data source to `[data-file-path]`.


## Tools

Microsoft Excel (cleaning and table preparation), Power Query, Power BI Desktop, DAX.

## Future work

- Market map (after verifying coordinates)
- Price index chart (Jan 2007 = 100) to compare all foods on one scale
- Inflation heatmap, volatility page and seasonality chart
- Inflation-adjusted prices or comparison with wages

## Author

[Samson Oloyede] | [oloyedes212@gmail.com]

## Acknowledgements

Data: [source]. [Add any AI assistance disclosure your institution requires.]
