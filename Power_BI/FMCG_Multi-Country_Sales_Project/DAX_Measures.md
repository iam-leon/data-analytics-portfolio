# DAX Measures Documentation

This file contains all DAX measures used in the **FMCG Multi-Country Sales Project**.  
Each measure includes its formula and a short explanation of its purpose.

---

## 📊 Sales Measures

### 1. Total Net Sales
- `Total Net Sales = SUM(Sales[net_sales])`
- Purpose: Calculates total net revenue across all transactions after any discounts applied.

### 2. Total Gross Sales
- `Total Gross Sales = SUM(Sales[gross_sales])`
- Purpose: Calculates total revenues accross all transactions before netting off any discounts.

### 3. Total Units sold
- `Total Units Sold = SUM(Sales[units_sold])`
- Purpose: Calculates the sum of units of the product sold accross all transactions.

### 4. Top Country by Sales
- ```Top Country by Sales = 
                    VAR MaxSales = MAXX(VALUES(RegionDim[country]), [Total Net Sales])
                    VAR TopCountry = MAXX(
                                        FILTER(VALUES(RegionDim[country]), [Total Net Sales] = MaxSales), RegionDim[country]
                    )
                    RETURN
                    TopCountry```
- Purpose: Returns the country where the most sales were registered.

### 5. Top Country Flag
- ```Top Country Flag = 
    VAR TopCountry = [Top Country by Sales]
    RETURN
    LOOKUPVALUE(RegionDim[Flag-URL], RegionDim[country], TopCountry)```
- Purpose: Shows the flag of the country with the most sales for clarity in visuals.

---

## 📈 Trend Analysis Measures

### 1. Year-over-Year Sales Growth
- ```YoY Sales Growth = 
    VAR PreviousYearSales = CALCULATE([Total Net Sales], SAMEPERIODLASTYEAR(DateDim[date]))
    RETURN
    IF(
        NOT ISBLANK(PreviousYearSales) && PreviousYearSales <> 0,
        DIVIDE([Total Net Sales] - PreviousYearSales, PreviousYearSales),
        BLANK()
    )```
-Purpose: Measures sales growth compared to the same period last year.
