# Denis-Sales Report and Dashboard

## Table of Contents :

- [Problem Statement]()
- [Datasource](https://github.com/yogeshkasar778/PowerBI-Project--ICCMen-s-T20-Cricket-World-Cup-2022-data-analysis/edit/main/README.md#datasource-)
- [Data Collection]()
- [Data Transformation]()
- [Data Modelling]()
- [Data Analysis Expression (DAX)]()
- [Dashboard]()
- [Tools, Software and Libraries]()
- [References]()

## Problem Statement :

In This project Created a Power BI Dashboard which helps to create a comprehensive sales report and insightful dashboard that provides a clear overview of sales performance, trends, and analysis. 
We have gathered data from various sources, including sales data, product information, geography, sales representatives, and categories. The task involves data gathering, data modeling, DAX calculations, 
and creating visuals for the dashboard.

## Data Gathering / Requirement:
         1. Sales (folder by year)
         2. Categories (Excel)
         3. Geography (Excel)
         4. Product (CSV)
         5. SalesRep (Excel)
         6. SubCategories (Excel)

## Data Gathering and Integration:
We need to create a mechanism to load all the files from the sales folder into a single Sales fact table. The mechanism should be resilient, meaning it should not cause errors when files are removed from the sales folder or when new yearly sales files are added. The fact table should be automatically updated when new data is added.

## Data Modelling:
  ### :black_small_square: Transformations and Data Type Setting:
Connected all the datasets with based on sales fact table and dimension table. In the Sales fact table, we need to split the "Location" field into separate fields for "Country" and "City" to allow for Geo maps. Additionally, we need to ensure that the "Date" field is correctly formatted to facilitate date-based analysis.

     #"Filtered Hidden Files1" = Table.SelectRows(Source, each [Attributes]?[Hidden]? <> true),
     #"Invoke Custom Function1" = Table.AddColumn(#"Filtered Hidden Files1", "Transform File", each #"Transform File"([Content])),
     #"Renamed Columns1" = Table.RenameColumns(#"Invoke Custom Function1", {"Name", "Source.Name"}),
     #"Removed Other Columns1" = Table.SelectColumns(#"Renamed Columns1", {"Source.Name", "Transform File"}), 
     #"Expanded Table Column1" = Table.ExpandTableColumn(#"Removed Other Columns1", "Transform File", Table.ColumnNames(#"Transform File"(#"Sample File"))),  
     #"Changed Type" = Table.TransformColumnTypes(#"Expanded Table Column1",{{"Source.Name", type text}, {"fSalesPrimaryKey", Int64.Type}, {"ProductID", Int64.Type}, {"SalesRepID", Int64.Type}, {"Location", type text}, {"Date", type date}, {"Units", Int64.Type}, {"PercentOfStandardCost", type number}, {"RevenueDiscount", type number}}),
     #"Removed Sourse deatils" = Table.RemoveColumns(#"Changed Type",{"Source.Name"}),
     #"Split location" = Table.SplitColumn(#"Removed Sourse deatils", "Location", Splitter.SplitTextByDelimiter(";", QuoteStyle.Csv), {"Location.1", "Location.2"}),
     #"Changed Type1" = Table.TransformColumnTypes(#"Split location",{{"Location.1", type text}, {"Location.2", type text}}),
     #"Renamed Columns location1 to Country and location 2 to Town" = Table.RenameColumns(#"Changed Type1",{{"Location.1", "Country"}, {"Location.2", "Town"}}),
     #"Merged Queries" = Table.NestedJoin(#"Renamed Columns location1 to Country and location 2 to Town", {"Country", "Town"}, Geography, {"Country", "Town"}, "Geography", JoinKind.LeftOuter),
     #"Expanded Geography" = Table.ExpandTableColumn(#"Merged Queries", "Geography", {"Geokey"}, {"Geography.Geokey"}),`
     #"Renamed Columns" = Table.RenameColumns(#"Expanded Geography",{{"Geography.Geokey", "Geokey"}}),
     #"Reordered Columns" = Table.ReorderColumns(#"Renamed Columns",{"fSalesPrimaryKey", "ProductID", "SalesRepID", "Geokey", "Country", "Town", "Date", "Units", "PercentOfStandardCost", "RevenueDiscount"}),
     #"Removed Columns" = Table.RemoveColumns(#"Reordered Columns",{"Country", "Town"})
     
  ### :black_small_square: Creating Unique Keys:
  We will create a unique key (GeoKey) in both the Sales and Geography tables to establish a relationship between them.
  
    Geography_Sheet = Source{[Item="Geography",Kind="Sheet"]}[Data],
    #"Changed Type" = Table.TransformColumnTypes(Geography_Sheet,{{"Column1", type text}, {"Column2", type text}, {"Column3", type text}}),
    #"Promoted Headers" = Table.PromoteHeaders(#"Changed Type", [PromoteAllScalars=true]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Promoted Headers",{{"Country", type text}, {"Town", type text}, {"Wikipedia", type text}}),
    #"Added Index" = Table.AddIndexColumn(#"Changed Type1", "Index", 1, 1, Int64.Type),
    #"Renamed Columns" = Table.RenameColumns(#"Added Index",{{"Index", "Geokey"}}) 
    
  ### :black_small_square: Cleaning ID Columns:
  For the SalesRep and SubCategory dimensions, some ID columns have a prefix "ID -" that needs to be removed. We will create a function to clean these IDs, making it reusable for both queries.
  #### SalesRep Table - 
  
    #"Sales rep_Sheet" = Source{[Item="Sales rep",Kind="Sheet"]}[Data],
    #"Changed Type" = Table.TransformColumnTypes(#"Sales rep_Sheet",{{"Column1", type text}, {"Column2", type text}}),
    #"Promoted Headers" = Table.PromoteHeaders(#"Changed Type", [PromoteAllScalars=true]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Promoted Headers",{{"SalesRepID", type text}, {"Sales Rep Name", type text}}),
    #"Replaced Value" = Table.ReplaceValue(#"Changed Type1","ID - ","",Replacer.ReplaceText,{"SalesRepID"})
    
  #### SalesRep Table -
  
    #"Changed Type" = Table.TransformColumnTypes(SubCategory_Table,{{"SubCategoryKey", Int64.Type}, {"CategoryKey", type text}, {"SubCategory Name", type text}}),
    #"Replaced Value" = Table.ReplaceValue(#"Changed Type","ID -","",Replacer.ReplaceText,{"CategoryKey"}),
    #"Replaced Value1" = Table.ReplaceValue(#"Replaced Value"," ","",Replacer.ReplaceText,{"CategoryKey"})
    
  ### :black_small_square: Data Model Creation:
We will create the data model connecting all tables and utilize the Calendar table that has already been set up.

![image](https://github.com/yogeshkasar778/Denis-Retail_sales_report_and_dashboard/assets/118357991/f8eee6be-c099-4abb-a182-ff7996d511a5)


## Data Analysis Expression (DAX) Calculation :
Measures used in visualization are:

- `Total Revenue = sales[Units]*RELATED('Product'[RetailPrice])`
  
- `Total Cost = sales[Units] * RELATED('Product'[StandardCost])`
  
- `Gross Profit = sales[Total Revenue] - sales[Total Cost]`
  
- `MoM growth = ([Total Profit]-[Prev Month profit])/[Prev Month profit]`
  
- `Prev Month profit = CALCULATE([Total Profit],PREVIOUSMONTH(DateMaster[Date]))`
  
- `Prev Qtr = CALCULATE([Tot Revenue],PREVIOUSQUARTER(DateMaster[Date]))`
  
- `QoQ growth = ([Tot Revenue]-[Prev Qtr])/[Prev Qtr]`
  
- `Total Profit = SUM(sales[Gross Profit])`
    
Date Calculation:

- `Year = YEAR(DateMaster[Date]) `
  
- `Querter = QUARTER(DateMaster[Date])`
  
- `Month = MONTH(DateMaster[Date])`
  
- `Month Name = FORMAT(DateMaster[Date],"MMM")`
  
- `Month Order = DateMaster[Date].[MonthNo]`
  
- `Week Day = WEEKDAY(DateMaster[Date])`
  
- `Week Day Name = FORMAT(DateMaster[Date],"DDD")`
  
- `Week Num = WEEKNUM(DateMaster[Date])`

## Report:
Data visualization for the dataset was done using Microsoft Power BI Desktop:

View Report Link - [Sales Report](https://app.powerbi.com/links/v8u0gUhEvS?ctid=b9cd496c-35ed-4f56-9942-e91f9a3d8d48&pbi_source=linkShare)

## Dashboard:
Using the measures and calculations, we will design a one-page sales dashboard with various visuals, including charts, graphs, and geo maps. The dashboard will represent sales insights and trends, making it easy to comprehend and facilitate data-driven decision-making.

View Dashboard Link - [Dashboard](https://app.powerbi.com/Redirect?action=OpenDashboard&appId=9faf2422-baa8-412d-9935-e4ced5c261c5&dashboardObjectId=8aed4c0b-0a8d-45b7-9a6f-b1b042c25d0e&ctid=b9cd496c-35ed-4f56-9942-e91f9a3d8d48&pbi_source=appShareLink&portalSessionId=fd2521d3-9594-4e1c-b58d-aa8ba8cf8761)

## Tools, Software and Libraries :

1.Jupyter Notebook

2.Python

3.Pandas

4.Webscraping

5.Beautifual Soup

6.Power Query Editor

7.Power BI

8.Anaconda Envirement

## References
