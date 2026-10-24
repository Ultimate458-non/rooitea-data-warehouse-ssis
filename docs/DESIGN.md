# Dimensional design (Parts A and B)

Design work for the RooiTea warehouse, done before the ETL: an information package (requirements) and a dimensional model.

## 1. Measures

| Measure | Type | Source | Formula |
|---|---|---|---|
| Sales Amount (ZAR) | Monetary | OrderLine.SalesAmount | Direct |
| Sales Amount (Local) | Monetary | OrderLine.SalesAmount, ExchangeRates.Rate | SalesAmount / Rate |
| Inventory Value (ZAR) | Monetary | Inventory.Quantity, UnitCost | Quantity × UnitCost |
| Backorder Value (ZAR) | Monetary | Inventory.BackOrderQty, UnitCost | BackOrderQty × UnitCost |
| Processing Cost | Monetary | ProcessingRun.ProcessingCost | Direct |
| Quantity Sold | Count | OrderLine.Quantity | SUM(Quantity) |
| Quantity Harvested (kg) | Count | HarvestBatch.QuantityKg | SUM(QuantityKg) |
| Quantity on Hand | Count | Inventory.Quantity | Direct |
| Backorder Quantity | Count | Inventory.BackorderQty | Direct |
| Number of Orders | Count | OrderHeader.OrderID | COUNT(DISTINCT OrderID) |
| Days Since Harvest | Time | HarvestDate, OrderDate | DATEDIFF(day, HarvestDate, OrderDate) |
| Order Lead Time | Time | OrderDate, ShipmentDate | DATEDIFF(day, OrderDate, ShipmentDate) |
| Transit Time (days) | Time | ShipmentDate, DeliveryDate | DATEDIFF(day, ShipmentDate, DeliveryDate) |

## 2. Dimensions and hierarchies

| Dimension | Roll-up hierarchy |
|---|---|
| Farm | Farm > Region > Province > Country |
| Product | Product > Product Type > Product Category |
| Batch / Quality | Batch > Quality Grade > Quality Class |
| Distribution | Distribution Centre > Destination > Market Type |
| Customer | Customer > Market Segment > Market Type |
| Time | Day > Month > Quarter > Year > Financial Year |
| Currency | Currency > Market Type |

## 3. Schema choice: fact constellation

Two fact tables share conformed dimensions:

- **Sales Fact** (production and sales): Time, Product, Customer, Farm, Currency, Batch, Distribution.
- **Inventory Fact**: Time, Product, Batch, Distribution.

Why two facts: inventory (stock levels, backorders) and sales transactions have different grains, so forcing them into one table would skew the measures. Shared dimensions (Time, Product, Batch, Distribution) let management compare production, sales and stock together. Each fact-to-dimension link is a denormalised star (one-to-many, no snowflaking) for simple, fast queries.

![Dimensional model](star-schema.png)

## Note on the ETL
The SSIS `LoadDimensions` package loads the dimensions at a finer level than the logical model above (for example Region, Province and Country as separate tables). This is a physical design choice, not a change to the business hierarchies.
