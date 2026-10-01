# boAt Sales Analytics Dashboard - Power BI Step-by-Step Guide

> **Complete guide to build the exact same dashboard as shown in the reference images**
> Using: SQL Analysis + Python Data Cleaning + Power BI + DAX

---

## TABLE OF CONTENTS

1. [Project Overview](#1-project-overview)
2. [Data Import](#2-data-import)
3. [Data Modeling](#3-data-modeling)
4. [DAX Measures](#4-dax-measures)
5. [Dashboard Layout & Design](#5-dashboard-layout--design)
6. [Visual #1 — KPI Cards](#6-visual-1--kpi-cards)
7. [Visual #2 — Slicers (State, Channel, Category)](#7-visual-2--slicers)
8. [Visual #3 — State Wise Performance Map](#8-visual-3--state-wise-performance-map)
9. [Visual #4 — Sales by Age Group (Donut Chart)](#9-visual-4--sales-by-age-group-donut-chart)
10. [Visual #5 — Payment Modes (Donut Chart)](#10-visual-5--payment-modes-donut-chart)
11. [Visual #6 — Revenue by Sales Channels (Bar Chart)](#11-visual-6--revenue-by-sales-channels-bar-chart)
12. [Visual #7 — Revenue Trend by Month (Line Chart)](#12-visual-7--revenue-trend-by-month-line-chart)
13. [Visual #8 — Top 5 Products by Sales (Bar Chart)](#13-visual-8--top-5-products-by-sales-bar-chart)
14. [Visual #9 — Top 5 Cities by Sales (Bar Chart)](#14-visual-9--top-5-cities-by-sales-bar-chart)
15. [Final Polish & Formatting](#15-final-polish--formatting)
16. [Color Reference](#16-color-reference)

---

## 1. PROJECT OVERVIEW

### Files Created

| File | Purpose |
|------|---------|
| `boat_sales_data.csv` | Raw generated dataset (10,000 rows) |
| `generate_boat_data.py` | Python script that generated the data |
| `sql_analysis.py` | SQL analysis queries (SQLite) |
| `data_cleaning.py` | Data cleaning & preparation |
| `boat_sales_cleaned.csv` | **Cleaned dataset for Power BI import** |
| `boat_sales.db` | SQLite database for SQL queries |

### Dataset Columns (27 columns)

| Column | Type | Description |
|--------|------|-------------|
| Order_ID | Text | Unique order identifier (BOAT-000001) |
| Order_Date | Date | DD-MM-YYYY format |
| Month | Text | Month name (January - December) |
| Month_Num | Number | Month number (1-12) |
| Year | Number | 2024 |
| State | Text | 35 Indian states/UTs |
| Region | Text | North, South, East, West, Northeast, Central, Islands |
| City | Text | 139 Indian cities |
| Category | Text | Earbuds, Headphones, Neckband, Smartwatch, Speaker, Wired Earphones |
| Product_Name | Text | 37 real boAt products |
| Unit_Price | Number | MRP in Rs (399-4499) |
| Discount_Pct | Number | Discount % (0-50) |
| Selling_Price | Number | Price after discount |
| Quantity | Number | Units per order (1-5) |
| Revenue | Number | Selling Price x Quantity |
| Cost | Number | Cost price x Quantity |
| Profit | Number | Revenue - Cost |
| Sales_Channel | Text | Amazon, Flipkart, boAt Website, Reliance Digital |
| Payment_Mode | Text | UPI, Card, COD, Net Banking |
| Age_Group | Text | 18-25, 26-35, 36-45, 45+ |
| Rating | Decimal | Customer rating (3.0-5.0) |
| Gender | Text | Male, Female |
| Profit_Margin | Decimal | Profit % |
| Quarter | Number | 1-4 |
| Quarter_Label | Text | Q1-Q4 |
| Day_of_Week | Text | Monday-Sunday |
| Price_Segment | Text | Budget, Value, Mid, Premium, Ultra |

---

## 2. DATA IMPORT

### Step 1: Open Power BI Desktop
1. Open **Power BI Desktop**
2. Click **Get Data** > **Text/CSV**
3. Navigate to `d:\DA_PROJECTS\boat\boat_sales_cleaned.csv`
4. Click **Load**

### Step 2: Verify in Power Query Editor
1. Click **Transform Data** to open Power Query Editor
2. Verify all 27 columns are loaded correctly
3. Check these data types (change if needed):

| Column | Required Type |
|--------|--------------|
| Order_Date | **Date** (DD/MM/YYYY) |
| Month_Num | Whole Number |
| Unit_Price | Whole Number |
| Selling_Price | Whole Number |
| Revenue | Whole Number |
| Cost | Whole Number |
| Profit | Whole Number |
| Quantity | Whole Number |
| Rating | Decimal Number |
| Discount_Pct | Whole Number |
| Profit_Margin | Decimal Number |

### Step 3: Fix Date Column (IMPORTANT)
If `Order_Date` shows as text:
1. Select the `Order_Date` column
2. Go to **Transform** tab > **Data Type** > **Date**
3. If it asks about locale, select **Using Locale** > **English (India)** > format **DD/MM/YYYY**
4. Click **Close & Apply**

---

## 3. DATA MODELING

### Create a Date Table (for time intelligence)
Go to **Modeling** tab > **New Table** and paste:

```dax
DateTable =
ADDCOLUMNS(
    CALENDARAUTO(),
    "Year", YEAR([Date]),
    "Month Number", MONTH([Date]),
    "Month Name", FORMAT([Date], "MMMM"),
    "Month Short", FORMAT([Date], "MMM"),
    "Quarter", "Q" & FORMAT([Date], "Q"),
    "Day of Week", FORMAT([Date], "dddd")
)
```

### Create Relationship
1. Go to **Model View** (left sidebar icon)
2. Drag `Order_Date` from `boat_sales_cleaned` to `Date` in `DateTable`
3. Relationship: **Many-to-One**, Single direction

> **Note:** The DateTable is optional since our data has Month columns already, but it enables time intelligence DAX functions.

---

## 4. DAX MEASURES

Create a **New Measure** for each of these. Go to **Modeling** > **New Measure**.

> **TIP:** Create a "Measures" folder — right-click on the table > **New Measure**, then drag measures into a Display Folder.

### KPI Card Measures

```dax
Total Revenue = SUM(boat_sales_cleaned[Revenue])
```

```dax
Total Profit = SUM(boat_sales_cleaned[Profit])
```

```dax
Total Orders = COUNTROWS(boat_sales_cleaned)
```

```dax
Avg Rating = AVERAGE(boat_sales_cleaned[Rating])
```

```dax
Avg Sales = AVERAGE(boat_sales_cleaned[Quantity])
```

### Formatted Display Measures (for KPI cards with K/M suffix)

```dax
Revenue Display =
VAR Rev = [Total Revenue]
RETURN
    IF(Rev >= 1000000,
        FORMAT(Rev / 1000000, "0.00") & "M",
        IF(Rev >= 1000,
            FORMAT(Rev / 1000, "0.00") & "K",
            FORMAT(Rev, "0")
        )
    )
```

```dax
Profit Display =
VAR Prof = [Total Profit]
RETURN
    IF(Prof >= 1000000,
        FORMAT(Prof / 1000000, "0.00") & "M",
        IF(Prof >= 1000,
            FORMAT(Prof / 1000, "0.00") & "K",
            FORMAT(Prof, "0")
        )
    )
```

```dax
Orders Display =
VAR Ord = [Total Orders]
RETURN
    IF(Ord >= 1000,
        FORMAT(Ord / 1000, "0.000") & "K",
        FORMAT(Ord, "0")
    )
```

### Additional Useful Measures

```dax
Profit Margin % = DIVIDE([Total Profit], [Total Revenue], 0) * 100
```

```dax
Total Cost = SUM(boat_sales_cleaned[Cost])
```

```dax
Avg Selling Price = AVERAGE(boat_sales_cleaned[Selling_Price])
```

```dax
Avg Discount = AVERAGE(boat_sales_cleaned[Discount_Pct])
```

---

## 5. DASHBOARD LAYOUT & DESIGN

### Page Setup
1. Go to **Format** pane (paint roller icon)
2. **Canvas Settings:**
   - Type: **Custom**
   - Width: **1280**
   - Height: **720**

### Background Color
1. **Page background:**
   - Color: **#0B1120** (very dark navy blue)
   - Transparency: **0%**

### Dashboard Layout Grid (approximate positions)

```
+------------------------------------------------------------------+
| [LOGO]  SALES ANALYTICS DASHBOARD    [State▼] [Channel▼] [Cat▼] |
|         Overview of boAt Performance                              |
+--------+----------+-----------+-----------+----------+------------+
| STATE  | TOTAL    | TOTAL     | TOTAL     | AVG      | AVG        |
| WISE   | REVENUE  | PROFIT    | ORDERS    | RATING   | SALES      |
| MAP    | 7.89M    | 2.16M     | 4.514K    | 4.28     | 3.01       |
|        +----------+-----------+-----+-----+----------+            |
|        | SALES BY AGE GROUP   | PAYMENT   | REVENUE BY SALES     |
|        | [Donut Chart]        | MODES     | CHANNELS             |
|        |                      | [Donut]   | [Bar Chart]          |
+--------+----------------------+-----------+----------------------+
| Revenue Trend by Month | TOP 5 PRODUCT   | TOP 5 CITIES         |
| [Line Chart]           | BY SALES        | BY SALES             |
|                        | [H-Bar Chart]   | [H-Bar Chart]        |
+------------------------+-----------------+----------------------+
```

---

## 6. VISUAL #1 — KPI CARDS (Top Row)

You need **5 KPI Cards** across the top row.

### Card 1: TOTAL REVENUE

1. Insert > **Card** visual (or use **New Card** visual in newer Power BI)
2. **Field:** `Total Revenue` measure
3. **Format settings:**
   - **Callout value:**
     - Font: **DIN** or **Segoe UI Bold**
     - Font size: **28-32**
     - Color: **White (#FFFFFF)**
     - Display units: **Auto** (shows K/M automatically)
   - **Category label:** 
     - Text: "TOTAL REVENUE"
     - Font size: **10**
     - Color: **#888888** (gray)
   - **Card background:**
     - Color: **#131B2E** (dark navy card)
     - Border: **Rounded**, Color **#1E2A40**, Width **1**
4. Position: Top row, left of center
5. Size: ~180w x 80h pixels

### Card 2: TOTAL PROFIT
- Same as above but field = `Total Profit` measure
- Label = "TOTAL PROFIT"

### Card 3: TOTAL ORDERS
- Field = `Total Orders` measure
- Label = "TOTAL ORDERS"

### Card 4: AVG RATING
- Field = `Avg Rating` measure
- Display units = **None** (show actual value like 4.28)
- Label = "AVG RATING"

### Card 5: AVG SALES
- Field = `Avg Sales` measure
- Display units = **None**
- Label = "AVG SALES"

### KPI Card Styling (apply to ALL 5 cards):
```
Background:  #131B2E (dark card)
Border:      #1E2A40, rounded corners, 1px
Title color: #888888 (muted gray)
Value color: #FFFFFF (white, bold)
Font:        Segoe UI Semibold
```

---

## 7. VISUAL #2 — SLICERS

### Slicer 1: State
1. Insert > **Slicer**
2. Field: `State`
3. **Slicer Settings:**
   - Style: **Dropdown** (click the dropdown arrow on the slicer header)
   - Selection: **Multi-select** with checkboxes
4. **Format:**
   - Header text: "State"
   - Header background: **#131B2E**
   - Header font color: **White**
   - Items background: **#131B2E**
   - Items font color: **White**
   - Font size: **10**
5. Position: Top-right area, first slicer
6. Size: ~150w x 35h

### Slicer 2: Sales Channel
- Same settings but field = `Sales_Channel`
- Header text: "Sales Channel"

### Slicer 3: Category
- Same settings but field = `Category`
- Header text: "Category"

### Slicer Styling (ALL 3):
```
Background:     #131B2E
Border:         #1E2A40
Header color:   White
Dropdown BG:    #1A2332
Font:           White, 10pt
Selection type: Dropdown + Multi-select
```

---

## 8. VISUAL #3 — STATE WISE PERFORMANCE MAP

### Option A: Bing Map (as shown in dashboard)
1. Insert > **Map** visual (the filled/bubble map)
2. **Fields:**
   - Location: `State`
   - Size: `Total Revenue` (measure)
   - Color saturation: `Total Revenue` (measure)
3. **Format:**
   - Map style: **Dark** (this gives the dark background as shown)
   - Bubble color: **Red (#FF0000)** or gradient
   - Data colors: Min color **#333333**, Max color **#FF0000** (red gradient)
   - Background: **#0B1120**
   - Title: "STATE WISE PERFORMANCE"
   - Title color: **White**
   - Title font size: **12**

### Option B: Filled Map (better for heatmap effect)
1. Insert > **Filled Map**
2. **Fields:**
   - Location: `State`
   - Color saturation: `Total Revenue`
3. Format same as above

### Option C: Shape Map (India specific)
1. Insert > **Shape Map**
2. In format, set Map > **Map type** to a custom India TopoJSON
3. Location: `State`
4. Color saturation: `Total Revenue`

> **Recommended:** Use the regular **Map** visual with Dark theme — it matches the dashboard screenshots the best.

### Box around the map:
1. Add a **Rectangle shape** behind the map
2. Fill: **#131B2E**
3. Border: **#1E2A40**, 1px
4. Send to back

---

## 9. VISUAL #4 — SALES BY AGE GROUP (Donut Chart)

1. Insert > **Donut Chart**
2. **Fields:**
   - Legend: `Age_Group`
   - Values: `Total Orders` (measure) — or just Count of Order_ID

3. **Format:**
   - **Data colors** (must match the dashboard exactly):
     - 18-25: **#2196F3** (Blue)
     - 26-35: **#FF0000** (Red)
     - 36-45: **#FFC107** (Amber/Gold)
     - 45+: **#4CAF50** (Green)
   - **Detail Labels:**
     - Label contents: **Percent of total**
     - Display units: **None**
     - Font color: **White**
     - Font size: **10**
     - Decimal places: **0**
   - **Legend:**
     - Position: **Right**
     - Font color: **White**
     - Font size: **9**
   - **Inner radius:** ~50-60%
   - **Title:**
     - Text: "SALES BY AGE GROUP"
     - Color: **White**
     - Font size: **12**, Bold
   - **Background:** Transparent
4. Position: Middle-left area
5. Size: ~280w x 200h

---

## 10. VISUAL #5 — PAYMENT MODES (Donut Chart)

1. Insert > **Donut Chart**
2. **Fields:**
   - Legend: `Payment_Mode`
   - Values: `Total Orders` (measure)

3. **Format:**
   - **Data colors:**
     - Card: **#FF0000** (Red)
     - COD: **#2196F3** (Blue)
     - Net Banking: **#FFC107** (Gold/Amber)
     - UPI: **#4CAF50** (Green)
   - **Detail Labels:**
     - Label contents: **Percent of total**
     - Font color: **White**
     - Decimal places: **2**
   - **Legend:**
     - Position: **Bottom**
     - Font color: **White**
     - Show colored icons
   - **Inner radius:** ~50%
   - **Title:**
     - Text: "Payment Modes"
     - Color: **White**
     - Font size: **12**, Bold
   - **Background:** Transparent
4. Position: Middle-center
5. Size: ~250w x 200h

---

## 11. VISUAL #6 — REVENUE BY SALES CHANNELS (Bar Chart)

1. Insert > **Clustered Column Chart**
2. **Fields:**
   - X-axis: `Sales_Channel`
   - Y-axis: `Total Revenue` (measure)

3. **Format:**
   - **Data colors** (each bar a different color):
     - Amazon: **#4CAF50** (Green)
     - Flipkart: **#FF0000** (Red)
     - boAt Website: **#2196F3** (Blue)
     - Reliance Digital: **#FFC107** (Gold/Amber)
   - **X-axis:**
     - Font color: **White**
     - Font size: **9**
   - **Y-axis:**
     - Font color: **White**
     - Font size: **9**
     - Display units: **Thousands** (K)
   - **Data labels:**
     - Show: **On**
     - Font color: **White**
     - Display units: **Thousands** (K)
     - Font size: **9**
   - **Title:**
     - Text: "REVENUE BY SALES CHANNELS"
     - Color: **White**
     - Font size: **12**, Bold
   - **Background:** Transparent
   - **Gridlines:** Off or very subtle (#1E2A40)
4. Position: Middle-right
5. Size: ~280w x 200h

---

## 12. VISUAL #7 — REVENUE TREND BY MONTH (Line Chart)

1. Insert > **Line Chart**
2. **Fields:**
   - X-axis: `Month` (sort by `Month_Num` — see below)
   - Y-axis: `Total Revenue` (measure)

### IMPORTANT: Sort Months Correctly
1. Click on the line chart
2. Click the **three dots (...)** in the top-right of the visual
3. **Sort axis** > `Month_Num`
4. **Sort axis** > `Sort ascending`

**Alternative: Sort by Column**
1. Go to **Data View** (table icon in left sidebar)
2. Select the `Month` column
3. **Column tools** tab > **Sort by Column** > select `Month_Num`
4. Now months will always sort Jan → Dec

3. **Format:**
   - **Line color:** **#FF0000** (Red) — matches the dashboard
   - **Line width:** 2-3px
   - **Markers:**
     - Show: **On**
     - Shape: **Circle**
     - Size: **5**
     - Color: **#FF0000**
   - **X-axis:**
     - Font color: **White**
     - Font size: **9**
     - Rotate labels if needed
   - **Y-axis:**
     - Font color: **White**
     - Display units: **Thousands** (K) or **Auto**
     - Start from: **0**
   - **Gridlines:**
     - Horizontal: subtle (#1E2A40) or off
     - Vertical: Off
   - **Title:**
     - Text: "Revenue Trend by Month"
     - Color: **White**
     - Font size: **12**, Bold
   - **Background:** Transparent
4. Position: Bottom-left
5. Size: ~380w x 180h

---

## 13. VISUAL #8 — TOP 5 PRODUCTS BY SALES (Horizontal Bar)

### Create a TopN DAX Measure (Optional — or use visual-level filter)

**Method 1: Visual-Level Filter (Recommended)**
1. Insert > **Clustered Bar Chart** (horizontal)
2. **Fields:**
   - Y-axis: `Product_Name`
   - X-axis: `Total Orders` measure (or Count of Order_ID)
3. **Add Visual-Level Filter:**
   - In the **Filters** pane (right side), under **Filters on this visual**
   - Drag `Product_Name` to filter
   - Filter type: **Top N**
   - Show items: **Top 5**
   - By value: `Total Orders`
   - Click **Apply Filter**

4. **Format:**
   - **Bar color:** **#FF0000** (Red) — all bars same red color
   - **Data labels:**
     - Show: **On**
     - Font color: **White**
     - Position: **Outside End**
     - Font size: **10**
   - **Y-axis (Product names):**
     - Font color: **White**
     - Font size: **9**
   - **X-axis:**
     - Turn **Off** (hide the axis)
   - **Gridlines:** Off
   - **Title:**
     - Text: "TOP 5 PRODUCT BY SALES"
     - Color: **White**
     - Font size: **12**, Bold
   - **Background:** Transparent
   - **Sort:** Descending by value
5. Position: Bottom-center
6. Size: ~280w x 180h

---

## 14. VISUAL #9 — TOP 5 CITIES BY SALES (Horizontal Bar)

1. Insert > **Clustered Bar Chart** (horizontal)
2. **Fields:**
   - Y-axis: `City`
   - X-axis: `Total Orders` measure
3. **Visual-Level Filter:**
   - Filter type: **Top N**
   - Show: **Top 5**
   - By value: `Total Orders`
   - Apply filter

4. **Format:**
   - **Bar color:** **#FF0000** (Red)
   - **Data labels:**
     - Show: **On**
     - Font color: **White**
     - Position: **Outside End**
   - **Y-axis (City names):**
     - Font color: **White**
   - **X-axis:** Off
   - **Gridlines:** Off
   - **Title:**
     - Text: "TOP 5 CITIES BY SALES"
     - Color: **White**
     - Font size: **12**, Bold
5. Position: Bottom-right
6. Size: ~280w x 180h

---

## 15. FINAL POLISH & FORMATTING

### Add boAt Logo
1. Insert > **Image**
2. Use the boAt logo (download from boAt's website or use a text box)
3. Position: Top-left corner
4. Size: ~120w x 50h

**Alternative — Text Box as Logo:**
1. Insert > **Text Box**
2. Type "boAt" in bold, red (#FF0000), size 28-32
3. Position at top-left

### Add Dashboard Title
1. Insert > **Text Box**
2. Line 1: **"SALES ANALYTICS DASHBOARD"** — White, Bold, size 18
3. Line 2: *"Overview of boAt Performance"* — Gray (#888888), Italic, size 11
4. Position: Top-center, next to logo

### Add Section Boxes (Card backgrounds)
For each visual group, add background rectangles:
1. Insert > **Shapes** > **Rectangle**
2. Fill: **#131B2E** (dark card color)
3. Border: **#1E2A40**, 1px
4. Corner radius: **5-8px**
5. **Send to Back** (right-click > Send to back)
6. Position behind each visual group

### Visual Borders
For each visual:
1. **General** > **Effects**
2. **Background:** Off (or #131B2E with 0% transparency)
3. **Border:** Off (the rectangle behind handles this)
4. **Shadow:** Off

### Interactions
By default, clicking on one visual filters all others. This is the desired behavior for the dashboard. To adjust:
1. Click on a visual
2. Go to **Format** tab in ribbon > **Edit Interactions**
3. Choose **Filter** (funnel icon) or **None** for each other visual

---

## 16. COLOR REFERENCE

### Theme Colors

| Element | Hex Code | Usage |
|---------|----------|-------|
| Page Background | `#0B1120` | Darkest navy — main canvas |
| Card Background | `#131B2E` | Slightly lighter — card surfaces |
| Card Border | `#1E2A40` | Subtle border outline |
| Gridlines | `#1E2A40` | Very subtle grid (if used) |
| Title Text | `#FFFFFF` | White — all visual titles |
| Subtitle Text | `#888888` | Gray — labels, subtitles |
| KPI Value | `#FFFFFF` | White bold — big numbers |
| boAt Logo/Accent | `#FF0000` | Red — brand color |

### Chart Colors

| Visual | Color | Hex |
|--------|-------|-----|
| Revenue line | Red | `#FF0000` |
| Top Products bars | Red | `#FF0000` |
| Top Cities bars | Red | `#FF0000` |
| Amazon bar | Green | `#4CAF50` |
| Flipkart bar | Red | `#FF0000` |
| boAt Website bar | Blue | `#2196F3` |
| Reliance Digital bar | Gold | `#FFC107` |

### Age Group Donut Colors

| Segment | Color | Hex |
|---------|-------|-----|
| 18-25 | Blue | `#2196F3` |
| 26-35 | Red | `#FF0000` |
| 36-45 | Amber | `#FFC107` |
| 45+ | Green | `#4CAF50` |

### Payment Mode Donut Colors

| Segment | Color | Hex |
|---------|-------|-----|
| Card | Red | `#FF0000` |
| COD | Blue | `#2196F3` |
| Net Banking | Amber | `#FFC107` |
| UPI | Green | `#4CAF50` |

---

## QUICK CHECKLIST

- [ ] Import `boat_sales_cleaned.csv` into Power BI
- [ ] Fix `Order_Date` column to Date type (DD/MM/YYYY)
- [ ] Sort `Month` column by `Month_Num`
- [ ] Create all DAX measures (Total Revenue, Profit, Orders, Avg Rating, Avg Sales)
- [ ] Set canvas to 1280x720, background #0B1120
- [ ] Add 5 KPI cards (top row)
- [ ] Add 3 dropdown slicers (State, Sales Channel, Category)
- [ ] Add India Map (State-wise performance)
- [ ] Add Age Group donut chart
- [ ] Add Payment Modes donut chart
- [ ] Add Revenue by Sales Channels clustered column chart
- [ ] Add Revenue Trend by Month line chart (red, sorted by Month_Num)
- [ ] Add Top 5 Products horizontal bar chart (Top N filter)
- [ ] Add Top 5 Cities horizontal bar chart (Top N filter)
- [ ] Add boAt logo and dashboard title text
- [ ] Add dark card rectangles behind each visual
- [ ] Apply all formatting (white text, dark backgrounds, correct colors)
- [ ] Test all slicer interactions
- [ ] Save as `boAt_Sales_Dashboard.pbix`

---

> **Total time estimate:** 2-3 hours for a polished dashboard
> 
> **Pro tip:** Save a Power BI theme JSON with all these colors to quickly apply consistent formatting.

---

*Generated for boAt Sales Analytics Dashboard Project*
*Dataset: 10,000 orders | 37 products | 35 states | 139 cities*
