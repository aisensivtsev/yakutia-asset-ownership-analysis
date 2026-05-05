# Tableau Dashboard Build Guide
## Yakutia Asset Ownership — 4-Dashboard Setup

**Data source:** `../data/cleaned/yakutia_assets.csv`  
**Key calculated field needed:** `Attributed Revenue = [estimated_revenue_usd_m] * [ownership_percentage] / 100`

---

## Setup: Connect Data

1. Open Tableau Desktop → Connect → Text File → `yakutia_assets.csv`
2. In Data Source tab: verify all columns loaded correctly
3. Change data types:
   - `ownership_percentage` → Number (Decimal)
   - `estimated_revenue_usd_m` → Number (Decimal)
4. Create **Calculated Field**: `Attributed Revenue (USD M)`
   ```
   [estimated_revenue_usd_m] * [ownership_percentage] / 100
   ```
5. Create **Color Parameter** for owner_type (or use Groups):
   - federal → #C0392B (Red)
   - regional → #2E86C1 (Blue)
   - municipal → #1ABC9C (Teal)
   - private → #7F8C8D (Gray)
   - foreign → #27AE60 (Green)

---

## Dashboard 1: Ownership Overview
**Title:** "Who Controls Yakutia's Economic Output?"

### Sheet 1A — Ownership Donut
- Chart type: **Pie Chart**
- Columns: (empty)
- Rows: (empty)
- Angle: `SUM(Attributed Revenue)` → Angle shelf
- Color: `owner_type`
- Detail: `owner_type`
- Size: make large
- Marks card: change to **Pie**
- Add Label: `ATTR(owner_type)` + `SUM(Attributed Revenue)` formatted as `$#,##0M`
- Hole: duplicate sheet, set inner circle white → overlay (donut trick)

### Sheet 1B — Owner Type Summary Table
- Rows: `owner_type`
- Text: `SUM(Attributed Revenue)`, `COUNTD(asset_name)`
- Sort: descending by revenue

**Layout:** Place 1A (large, center) + 1B (right side) in dashboard

---

## Dashboard 2: Asset Control Map
**Title:** "Which Sectors Are Controlled by Whom?"

### Sheet 2A — Sector × Owner Heatmap
- Chart type: **Text Table** with color
- Columns: `owner_type`
- Rows: `asset_type`
- Text: `SUM(Attributed Revenue)`
- Color: `SUM(Attributed Revenue)` → Continuous color (Orange-Red scale)
- Filter: show only records where Attributed Revenue > 0

### Sheet 2B — Stacked Bar by Asset Type
- Chart type: **Bar Chart**
- Columns: `SUM(Attributed Revenue)`
- Rows: `asset_type`
- Color: `owner_type`
- Marks: Bar, stacked
- Sort rows: descending by total revenue

**Layout:** 2A (top 60%) + 2B (bottom 40%)

---

## Dashboard 3: Power Concentration
**Title:** "Economic Dominance — Top Controlling Entities"

### Sheet 3A — Top 10 Entities Bar
- Chart type: **Horizontal Bar**
- Columns: `SUM(Attributed Revenue)`
- Rows: `owner_entity`
- Color: `owner_type`
- Filter: Top 10 by SUM(Attributed Revenue)
- Sort: descending
- Add reference line at average

### Sheet 3B — Concentration KPI Cards
Create 5 KPI text sheets:
- Total Revenue: `SUM(Attributed Revenue)` formatted
- Federal Share %: calculated field `IF [owner_type]='federal' THEN [Attributed Revenue] END / TOTAL(SUM([Attributed Revenue]))`
- Regional Share %: same logic for regional
- Top 1 Entity Revenue share
- Number of distinct entities

**Layout:** 3B KPIs at top (row of 5 tiles) + 3A large bar below

---

## Dashboard 4: Strategic Assets Analysis
**Title:** "Control vs Importance — The Structural Mismatch"

### Sheet 4A — Strategic Tier × Owner Stacked Bar (%)
- Chart type: **Bar Chart** (100% stacked)
- Columns: `strategic_importance`
- Rows: `SUM(Attributed Revenue)`
- Color: `owner_type`
- Mark: Bar, **Percent of Total** (Table calculation → Percent of Total, compute using Color)
- Sort columns: high → medium → low (manual sort)

### Sheet 4B — Strategic Asset Table
- Filter: `strategic_importance = 'high'`
- Rows: `asset_name`
- Columns: `owner_entity`, `owner_type`, `ownership_percentage`, `Attributed Revenue`
- Color rows by `owner_type`

### Sheet 4C — Scatter: Revenue vs Ownership %
- X-axis: `AVG(ownership_percentage)`
- Y-axis: `SUM(Attributed Revenue)`
- Color: `owner_type`
- Size: `estimated_revenue_usd_m`
- Label: `asset_name` (selected points only)
- Filter: strategic_importance = 'high'

**Layout:** 4A (left 40%) + 4C (right 60%) on top row; 4B full width below

---

## Final Dashboard Assembly Tips

1. **Consistent color legend** — pin the `owner_type` color legend to the dashboard, not individual sheets
2. **Global filter** — add `asset_type` as a dashboard-level filter affecting all sheets
3. **Tooltips** — customize each sheet tooltip:
   ```
   <Asset: asset_name>
   Sector: <asset_type>
   Owner: <owner_entity> (<owner_type>)
   Stake: <ownership_percentage>%
   Attributed Revenue: $<Attributed Revenue>M
   Strategic Importance: <strategic_importance>
   ```
4. **Title styling** — use dark charcoal (#2C3E50) for titles, subtitle in gray (#7F8C8D)
5. **Export** — File → Export as Packaged Workbook (.twbx) to include the CSV data

---

## Calculated Fields Reference

| Field Name | Formula |
|-----------|---------|
| Attributed Revenue | `[estimated_revenue_usd_m] * [ownership_percentage] / 100` |
| Is Federal | `IF [owner_type] = 'federal' THEN 1 ELSE 0 END` |
| Is Regional | `IF [owner_type] = 'regional' THEN 1 ELSE 0 END` |
| Federal Share % | `SUM(IF [owner_type]='federal' THEN [Attributed Revenue] END) / SUM([Attributed Revenue])` |
| Regional Share % | `SUM(IF [owner_type]='regional' THEN [Attributed Revenue] END) / SUM([Attributed Revenue])` |
| Owner Label | `IF [owner_type]='federal' THEN 'Federal (Russian Federation)' ELSEIF [owner_type]='regional' THEN 'Regional (Republic of Sakha)' ELSEIF [owner_type]='municipal' THEN 'Municipal Districts' ELSEIF [owner_type]='private' THEN 'Private / Corporate' ELSE 'Foreign Investors' END` |
