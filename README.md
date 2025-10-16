# Supply-Chain Metrics Dashboard

Supply Chain analytics project that builds an end-to-end KPI framework for a small/medium distribution business.  
The solution cleans and models order, inventory, shipment, product, supplier and target-KPI data in Excel and visualizes
operational performance in **Power BI** with clear target vs. actual tracking.

---

## 🧭 Business Case

**Problem.** Leadership lacked a single source of truth for service performance and inventory efficiency.  
Teams tracked Fill Rate, On-Time, Turns and Days of Supply in separate spreadsheets, which made it hard to spot trends,
prioritize suppliers, or quantify working-capital opportunities.

**Goal.** Deliver a live dashboard that:
- Measures **service quality** (Fill Rate %, On-Time Ship %) and **inventory efficiency** (Inventory Turns, Days of Supply),
- Benchmarks KPIs against **defined targets**,
- Highlights **exceptions** and **revenue concentration** by supplier,
- Enables drill-down from portfolio → category → SKU → orders without rebuilding spreadsheets.

**Users.** VP of Operations, Supply Chain Manager, Planning, and Procurement.

---

## ✅ Outcomes (what this dashboard enables)

- **Single KPI view** with **Targets vs Actuals** across months and year-to-date.
- **Weighted Days of Supply** to avoid inflated averages; **Turns** calculated consistently.
- **Top-supplier Pareto** reveals revenue concentration and guides supplier reviews.
- **Exception focus**: quickly identify where **DOS > target**, **Turns < target**, or **service < target**.
- Clear, repeatable process to update data and refresh visuals—no ad-hoc spreadsheet merges.

> In the sample dataset, service levels are strong (~100% Fill Rate, high On-Time),  
> while inventory shows targeted opportunities where DOS exceeds target despite good service.

---

## 📦 Data (sample)

- `Orders` – order header/line with `OrderDate`, `SKU`, `QtyOrdered`, `UnitPrice`, `Region`, `CustomerID`
- `Inventory` – `LocationID`, `SKU`, `OnHandQty`, `SafetyStock`, `AvgWeeklyDemand`
- `Shipments` – `OrderID`, `SKU`, `QtyShipped`, `ActualShipDate`, `ShipMode`
- `Products` – `SKU`, `ProductName`, `Category`, `SupplierID`, `UnitCost`
- `Suppliers` – `SupplierID`, `SupplierName`, `State`
- `KPI_Targets` – `Metric`, `TargetValue` (e.g., Fill Rate 0.97, On-Time 0.95, DOS 30 days, Turns 8)

> Relationships in Power BI (one-to-many):  
> `Products(SKU)` ↔ `Orders/Inventory/Shipments(SKU)`,  
> `Suppliers(SupplierID)` ↔ `Products(SupplierID)`,  
> `Orders(OrderID)` ↔ `Shipments(OrderID)`.

---

## 📐 KPI Definitions (business logic)

- **Fill Rate %** = `QtyShipped / QtyOrdered` (capped at 1.0 to avoid >100% when backorders ship)  
- **On-Time Ship %** = `OnTimeFlag / TotalOrders` (ActualShipDate ≤ RequestedShipDate)
- **Inventory Turns** (units-based) = `Annual Demand (units) / Average Inventory (units)`  
  - Annual Demand (units) = `SUM(AvgWeeklyDemand) × 52`  
- **Weighted Days of Supply** = `Total On-Hand / Total Daily Demand` (portfolio DOS)

**Targets** stored centrally in `KPI_Targets` and pulled via DAX measures  
(e.g., **Target Fill Rate % = 97%**, **Target On-Time % = 95%**, **Target DOS = 30**, **Target Turns = 8**).

---

## 🧰 Power BI Pages

1. **Executive Summary** – KPI cards (Actual vs Target), DOS by Category, Revenue by Supplier (Pareto), exceptions.
2. **KPI Actuals** – Month/Year tracking for **Fill Rate**, **On-Time**, **Turns vs Target**, **DOS vs Target**.
3. **Targets KPI’s** – Static target visuals for stakeholder alignment; single source for thresholds.
4. **Revenue / Supplier** – Concentration analysis and quick supplier ranking.

---

## 🧮 Key DAX (high level)

```DAX
-- Fill Rate (capped)
Fill Rate % :=
DIVIDE ( SUM ( Shipments[QtyShipped] ), SUM ( Orders[QtyOrdered] ), 0 )
    |> MINX ( { 1.0 }, [Value] )   -- conceptual cap at 100%


-- Annual Demand (units)
Annual Demand (Units) :=
SUMX ( Inventory, Inventory[AvgWeeklyDemand] ) * 52

-- Average Inventory (units)
Average Inventory (Units) :=
SUM ( Inventory[OnHandQty] )






