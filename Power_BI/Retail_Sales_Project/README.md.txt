# Retail Sales Performance Dashboard

## Business Problem

- Management wants to understand regional performance, Product category profitability and declining sales trends.

## Dataset

- Source: [Kaggle Retail Sales Dataset](https://www.kaggle.com/datasets/abbas829/retail-sales-dataset/)
- Containing 1825 rows of sales data
- Columns: Date, Category, Sales, Quantity, Profit, Region
- Note: Missing values in Category column cleaned using Power Query.
- Since the dataset covers only one year of entries, we shall heavily rely on monthly trends rather than yearly trends.

## Steps Taken

- **Data Cleaning**

- Removed duplicates
- Standardized category names and values formating
- Filled blank values, Null values and error values with Unknown.
- In the Quantity column (Now standardized as Quantity_Sold), rows with valid Sales and Profit but missing Quantity were retained.
- Those quantity values were marked as ‘Unknown’ to preserve transaction integrity while highlighting data quality issues.
- Rows with Profit showing blanks or errors were removed as they would not give clarity on profitability.
- Rows with Region = 0 or errors were removed, as they represented invalid or placeholder entries rather than real geographic regions

- **Data Modeling**

- Created surrogate keys (Transaction_ID) to ensure proper relationships in the star schema and maintain data integrity.
- From a duplicate sales table, surrogate keys were create for the unique roducts (Product_ID) and that table named the Products table.
- Merged Sales with Products using a left outer join to preserve all transactions while enriching them with ProductID from the dimension table
- Expanded the merged Products table to bring in Product_ID, ensuring each transaction in Sales is linked to the coresponding product key
- Linked the Products table to Sales via the Product_ID column.
- Created a Date Dimension table with Month, Quarter, Year, and additional time attributes. Linked it to Sales via Date_Sold column for time intelligence analysis.
- Created a Region Dimension table with four unique regions (East, West, North, South).
- Added surrogate key Region_ID and linked it to Sales via Region to enable regional analysis.
- Trimmed the Sales Fact table to contain only measures and foreign keys (Transaction_ID, Date_Sold, Product_ID, Region_ID, Sales_Value, Quantity_Sold, Profit).
- Moved descriptive attributes (Product_Category, Region) into their respective dimension tables.
- Designed a star schema with FactSales linked to DimDate, DimProduct, and DimRegion. Relationships set to many‑to‑one cardinality.
- Marked Date table for time intelligence. Added surrogate keys for clean modeling. 


- **DAX Measures**

- Created core measures for the base KPIs
- `Total Sales = SUM(Sales[Sales Value])`
- `Total Profit = SUM(Sales[Profit])`
- `Profit Margin = DIVIDE([Total Profit], [Total Sales])`
- `Quantity Sold = SUM(Sales[Quantity Sold])`

- Added Time inteligence by creating Year-Over-Year sales growth
- ```YOY Sales Growth = 
   	DIVIDE(
        	[Total Sales] - CALCULATE([Total Sales], SAMEPERIODLASTYEAR(Date[Date_Sold])),
        	CALCULATE([Total Sales], SAMEPERIODLASTYEAR(Date[Date_Sold])))```

- Created Month-To-Date measure to include sales from the start of the current month to current date.
- `MTD Sales = CALCULATE([Total Sales], DATESMTD(Date[Date_Sold]))`

- Created Quater-To-Date measure to include sales from the start of the current quater to date.
- `QTD Sales = CALCULATE([Total Sales], DATESQTD(Date[Date_Sold]))`

- Created Year-To-Date measure to include sales from the start of the current year to date.
- `YTD Sales = CALCULATE([Total Sales], DATESYTD(Date[Date_Sold]))`

- Created measure for a card that returns the top performing region
- ```Top Region by Sales =
	VAR MaxSales = MAXX(VALUES(Region[Region]), [Total Sales])
	VAR TopRegion = MAXX(
        	FILTER(VALUES(Region[Region]), [Total Sales] = MaxSales),
        	Region[Region])
	RETURN
	TopRegion```

- Created a measure for the total sales value of the top region.
- `Top Region Sales Value = MAXX(VALUES(Region[Region]), [Total Sales])`

- Created a measure to return the region with the highest profit margiin
- ```Top Region Profit Margin =
    VAR MaxMargin = MAXX(VALUES(Region[Region]), [Profit Margin])
    VAR TopRegion = MAXX(FILTER(VALUES(Region[Region]), [Profit Margin] = MaxMargin), Region[Region])
    RETURN
    TopRegion```

- Created a measure for the top region profit margin value
- `Top Region Profit Margin Value = MAXX(VALUES(Region[Region]), [Profit Margin])`

- Created a measure for the Highest Profitable Category
- ```Highest Profit Category = 
    VAR MaxProfit = MAXX(VALUES(Products[Product_Category]), [Total Profit])
    VAR MaxProfitCategory = MAXX(FILTER(VALUES(Products[Product_Category]), [Total Profit] = MaxProfit), Products       [Product_Category])
    RETURN
    MaxProfitCategory```

- Created a measure for the lowest profitable product category
- ```Lowest Profit Category = 
    VAR MinProfit = MINX(VALUES(Products[Product_Category]), [Total Profit])
    VAR LeastProfitableCategory = MAXX(FILTER(VALUES(Products[Product_Category]), [Total Profit] = MinProfit), Products[Product_Category])
    RETURN
    LeastProfitableCategory```

- Created a measure for the average profit margin accross all categories
- ```Average Profit Margin =
    	AVERAGEX(
       	 	VALUES(Products[Product_Category]),
       	 	DIVIDE([Total Profit], [Total Sales])
   	 )```

- Created MoM(Month over Month) Sales Change Measure to track the monthly change in the sales trend.
- ```MoM Sales Change = 
    	VAR CurrentMonthSales = [Total Sales]
    	VAR PreviousMonthSales = CALCULATE([Total Sales], DATEADD('Date'[Date_Sold], -1, MONTH))
    RETURN
    CurrentMonthSales - PreviousMonthSales```

- Created MoM(Month over Month) Sales Decline percentage Measure to track the monthly percentage change in the sales trend.
- ```MoM Decline % = 
    VAR CurrentMonthSales = [Total Sales]
    VAR PreviousMonthSales = CALCULATE([Total Sales], DATEADD('Date'[Date_Sold], -1, MONTH))
    RETURN
    IF(
        NOT ISBLANK(PreviousMonthSales) && PreviousMonthSales <> 0,
        DIVIDE(CurrentMonthSales - PreviousMonthSales, PreviousMonthSales),
        BLANK()
    )```

- Created measure to point out the worst monthly sales.
- ```Worst Month Sales = 
    	MINX(VALUES('Date'[Month name]), [Total Sales])```

- Created measure to point outh the month associated with the worst monthly sales
- ```Month With Lowest Sales = 
    	VAR LowestMonthlySales = MINX(VALUES('Date'[Month name]), [Total Sales])
    	VAR MonthWithLowestSales = 
       		 CALCULATE(SELECTEDVALUE('Date'[Month name]),
       		 FILTER(
            		VALUES('Date'[Month name]), [Total Sales] = LowestMonthlySales
        	))
    RETURN
    MonthWithLowestSales```

- Created a measure to point out the biggest month-over-month percentage drop.
- Since declines are negative percentages, we use  MINX to get the most negative (worst) decline.
- ```Largest Decline % =
    	MINX(
        	VALUES('Date'[Month name]), [MoM Decline %]
    	)```

- Created a measure to return the month with the largest percentage decline in sales.
- ```Largest Decline Month = 
    VAR LargestDeclinePercentage = MINX(VALUES('Date'[Month name]), [MoM Decline %])
    VAR LargestDeclineMonthName = CALCULATE(SELECTEDVALUE('Date'[Month name]),
                                    FILTER(
                                        VALUES('Date'[Month name]), [MoM Decline %] = LargestDeclinePercentage
                                    ))
	RETURN
	LargestDeclineMonthName```

- Created a measure for the average montly decline. This shows the net average trend **(An average of all the month-over-month percentage changes for the whole year)**.
- ```Average Monthly Decline = 
    	AVERAGEX(
        	VALUES('Date'[Month name]),
        	[MoM Decline %]
    	)```

- Created a subtitle text measeure for the percentage decline in sales over the months and bound it through the conditional formatting for line breaks.
	- `Sub-Title for % Decline Chart = "Worst decline in August!" & UNICHAR(10) & "Gradualy drop between September and November"`
	-  This was for the **Bar Chart: Month‑over‑Month percentage changes** in the **Declining Sales Trends Dashboard** To give more clarity to the chart.


- **Dashboard Design**

- **Page 1: Overview Key Performance Indicators (KPIs)**

	-**Purpose**
	- This page is designed to give a general overview of the business performance.
	- Focusing mostly on key metrics to provide an executive summary of volume, sales and profitability performance

	- **KPI Cards**
		- **Total Sales** – Overall revenue generated.
		- **Total Profit** – Net profitability.
		- **Profit Margin** – Profitability ratio formatted as a percentage.
		- **Quantity Sold** – Total units sold.
		- **MTD Sales** – Month-to-date performance.
		- **QTD Sales** – Quarter-to-date performance.
		- **YTD Sales** – Year-to-date performance.

- **Page 2: Regional Analysis**
	- **Purpose**
	- This dashboard provides a comparative analysis of regional performance across **Sales, Profit Margin, and Quantity Sold**.  
	- It is designed to give executives a clear view of which regions are driving business success and how they compare over time.

	- **Key Features**
		- **Headline KPI Cards**  
		- Highest Performing Region.
 		- Highest Region Sales.
 		- Highest Region Profit Margin. 
 	- These cards summarize the top performer at a glance, with background colors aligned to the leading region for immediate visual recognition.

	- **Interactive Slicers**  
 		- Quarter of the Year  
		- Month  
		- These filters allow users to drill down into specific time periods within the year, enhancing flexibility and exploration.

	- **Comparative Visuals**  
		- **Total Sales per Region (Bar Chart)** → Highlights West’s narrow lead in sales.  
		- **Profit Margin per Region (Bar Chart)** → Shows efficiency differences, with West leading and North trailing.  
		- **Quantity Sold by Region (Tree Map)** → Provides volume context.  
		- **Sales Over Time by Region (Line Chart)** → Displays monthly sales trends for each region, enabling time‑series analysis.

	- **Design Decisions**
		- **Color Consistency**: Each region is represented by a distinct color across all visuals (South = Blue, West = Pink, North = Green, East = Orange).  
		- **Narrative Titles**: Charts include explanatory captions (e.g., *“West Leads in Sales by Narrow Margin”*) to transform visuals into insights.  
		- **Visual Hierarchy**: Headline KPI cards are placed at the top to deliver the “answer first,” followed by detailed breakdowns.  
		- **Clean Layout**: Neutral background, aligned visuals, and consistent formatting ensure readability and professional presentation.

	- **Business Value**
	- This dashboard enables management to:
		- Identify the top‑performing region quickly.  
		- Compare regions across sales, margin, and quantity.  
		- Track monthly sales trends for performance monitoring.  
		- Drill down by quarter or month to answer specific business questions.  

- **Page 3: Product Category Profitability Analysis**

	- **Purpose**
		- This page is designed to:
			- Identify which categories drive profitability.  
			- Compare efficiency (margins) versus scale (sales).  
			- Support decision-making on product strategy and resource allocation.
			- It highlights both absolute profit performance and efficiency (profit margins), helping identify strengths and weaknesses in the product mix.

	- **Key Visuals**
		- **Total Profit Per Product Category (Bar Chart)**: Displays total profit generated by each category.  
  			- Books lead with the highest profit.  
  			- Home Goods generate the lowest profit.

		- **Profit Margin Per Category (Horizontal Bar Chart)**: Shows efficiency by comparing profit margins across categories.  
  			- Sports achieve the highest margin despite not being the most profitable overall.  
			- Electronics and Clothing trail with lower margins.

		- **Total Sales vs Profit Margin (Scatter Plot)**: Plots each category’s sales against its margin to reveal trade-offs.  
			- Useful for spotting categories with high sales but weaker margins.  
			- Highlights balance between revenue scale and profitability efficiency.

		- **KPI Cards**  
			- **Highest Profit Category**: Books  
			- **Lowest Profit Category**: Home Goods  
			- **Average Profit Margin**: 25.35%

	- **Insights**
		- Books are the strongest contributor to overall profit.  
		- Sports stand out for efficiency, leading in profit margin despite lower total profit.  
		- Home Goods underperform both in profit and margin, signaling potential for review.  
		- The average profit margin across categories is **25.35%**, providing a benchmark for comparison.

	- **Interactivity**
		- **Region Filter**: East, North, South, West  
		- **Month Filter**: February through August  
		- These slicers allow users to drill down into profitability by geography and time period.


- **Page 4: Declining Sales Trends Dashboard.**
	- **Purpose.**
		- This page highlights periods of declining performance within the sales dataset and provides clear insights into risk months.
		- It combines headline KPIs with interactive visuals to tell the story of sales decline across the year.
		- It enables stakeholders to identify when and where sales performance is weakening

	- **Key Features**

		- **Headline KPIs**: 
  			- Lowest Monthly Sales Value
  			- Month With Lowest Sales
 			- Largest % Decline
			- Month With Largest Decline
			- Average Monthly % Decline: With **Contextual Tooltip** to clarify how metrics are calculated.

		- **Interactive Filters**: Product Category and Region slicers allow drill‑down into specific segments.

		- **Trend Visuals**:
			- **Line Chart**: Monthly sales values with markers for the lowest sales and sharpest % decline months, visually emphasizing critical points.

			- **Bar Chart**: Month‑over‑Month percentage changes.
	
	
	- **Key Insights**
		- Sales were lowest in January, with the sharpest decline in August