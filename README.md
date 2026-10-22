# Meta Ad Performance Analysis | Power BI

Meta Ad Performance Analysis | Power BI - Interactive dashboard analyzing Facebook &amp; Instagram campaigns across Impressions, Clicks, Engagement, Purchases, CTR, Conversion Rate, and Budget to identify performance trends and optimization opportunities.

---

## 🔷 Data Transformation & Cleaning

The first step was to load the **raw data into Power Query** for data transformation, cleaning, and quality validation.

### 🔷 Data Quality Check

- Checked the data quality of the key columns used for relationships between tables.
- Verified that the **primary key** columns contain **100% distinct and unique values**.
- Checked **foreign key** columns to ensure that the values correctly match the corresponding primary keys.
- **Valid Values:** Verified that the data contains valid and correctly formatted values.
- **Error Values:** Identified and reviewed any errors present in the dataset.
- **Empty Values:** Checked for blank or missing values that may affect the analysis.

> **Important:** The **Date** column must contain **100% valid values**, as it is used for Time Intelligence calculations.

### 🔷 Data Cleaning

- Identified and reviewed errors and empty values.
- Used **Replace Values** to correct inconsistent or incorrect data.
- Ensured the dataset was clean and consistent.
- Verified that key columns contain appropriate values for establishing relationships.
- Prepared the cleaned data for **Data Modeling and DAX calculations**.

---

## 🔷 Data Modeling

After completing the data transformation and cleaning process, the next step was to create the **Data Model** and establish relationships between the tables.

The model consists of the following tables:

- **`campaigns`** — Campaign-level information and budget.
- **`ads`** — Advertisement-level information and targeting details.
- **`ad_events`** — Fact table containing advertisement interaction events.
- **`users`** — User demographic and geographic information.

### 🔷 Relationship Management

The relationships between the tables were created based on their respective **Primary Key and Foreign Key columns**.

#### 1. `campaigns` → `ads`

- **Primary Key:** `campaigns[campaign_id]`
- **Foreign Key:** `ads[campaign_id]`
- **Cardinality:** **One-to-Many (1:*)**
- One campaign can contain multiple advertisements.

#### 2. `ads` → `ad_events`

- **Primary Key:** `ads[ad_id]`
- **Foreign Key:** `ad_events[ad_id]`
- **Cardinality:** **One-to-Many (1:*)**
- One advertisement can generate multiple interaction events.

#### 3. `users` → `ad_events`

- **Primary Key:** `users[user_id]`
- **Foreign Key:** `ad_events[user_id]`
- **Cardinality:** **One-to-Many (1:*)**
- One user can generate multiple advertisement interaction events.
- The `ad_events` table is on the **Many (*) side**.
- The `users` table is on the **One (1) side**.

```text
users
  │
  │  user_id
  │
  │  1
  ▼
ad_events
  *
```
---

## 🔷 Date Table Creation

Whenever working with **date values and Time Intelligence calculations** in Power BI, a dedicated **Calendar Table** is required to support date-based analysis, time filtering, and consistent reporting.

### 🔶 Step 1: Create the Event Date Column

**Definition:** The `Event Date` column extracts the date portion from the `timestamp` column in the `ad_events` table, removing the time component.

```DAX
Event Date =
DATEVALUE(ad_events[timestamp])
```

### 🔶 Step 2: Create the Calendar Table

**Definition:** The `Calendar Table` contains a continuous range of dates, starting from the earliest event date and ending at the latest event date in the `ad_events` table.

```DAX
Calendar Table =
CALENDAR(
    MIN(ad_events[Event Date]),
    MAX(ad_events[Event Date])
)
```

### 🔶 Step 3: Create Additional Date Columns

#### 1. Month

**Definition:** Returns the abbreviated month name for each date, such as Jan, Feb, and Mar.

```DAX
Month =
FORMAT(
    'Calendar Table'[Date],
    "mmm"
)
```

#### 2. Day Name

**Definition:** Returns the abbreviated weekday name, such as Mon, Tue, and Wed.

```DAX
Day Name =
FORMAT(
    'Calendar Table'[Date],
    "ddd"
)
```

#### 3. Day Number

**Definition:** Returns the day of the month as a number, such as 1, 2, or 31.

```DAX
Day Number =
FORMAT(
    'Calendar Table'[Date],
    "d"
)
```

#### 4. Week Day

**Definition:** Returns the weekday number, with Monday = 1 and Sunday = 7.

```DAX
Week Day =
WEEKDAY(
    'Calendar Table'[Date],
    2
)
```

#### 5. Week Number

**Definition:** Returns the week number of the year, using Monday as the first day of the week.

```DAX
Week Number =
WEEKNUM(
    'Calendar Table'[Date],
    2
)
```

### 🔶 Step 4: Establish the Date Relationship

After creating the Calendar Table, establish the following relationship in **Model View**:

- **Primary Date:** `Calendar Table[Date]`
- **Related Date:** `ad_events[Event Date]`
- **Cardinality:** One-to-Many (1:*)
- **Cross-filter direction:** Single, from `Calendar Table` to `ad_events`

The Calendar Table should be on the **One (1) side**, while `ad_events` should be on the **Many (*) side**.

### 🔶 Step 5: Configure the Calendar Table

- Mark the `Calendar Table` as a **Date Table** using its `Date` column.
- Ensure the `Date` column has the **Date** data type.
- Sort `Month` by a numeric month column if the months need to appear in calendar order.
- Use the Calendar Table fields for date-based filtering and time analysis.

**Purpose:** The Calendar Table supports consistent **monthly, weekly, daily, and time-based analysis** across the dashboard, including calendar heat maps and weekly performance trends.

---

## 🔷 DAX Calculations

After completing the **Data Transformation, Cleaning, and Data Modeling** process, the next step was to create **DAX measures** for the required business KPIs.

Each measure is first defined in terms of its **business meaning and analytical purpose**, followed by the corresponding **DAX calculation**.

---

### 🔷 1. Impressions

**Definition:** Impressions represent the **total number of times an advertisement was displayed to users**.

**Business Insight:** This metric measures the overall **reach and visibility** generated by the advertising campaigns.

```DAX
Impressions =
COUNTROWS(
    FILTER(
        ad_events,
        ad_events[event_type] = "Impression"
    )
)
```
---

### 🔷 2. Clicks

**Definition:** Clicks represent the total number of times users clicked on an advertisement.

**Business Insight:** This metric measures the level of user interest and engagement intent generated by the advertisements.

```DAX
Clicks =
COUNTROWS(
    FILTER(
        ad_events,
        ad_events[event_type] = "Click"
    )
)
```

### 🔷 3. Shares

**Definition:** Shares represent the total number of times users shared an advertisement.

**Business Insight:** This metric helps measure viral engagement and the extent to which users are willing to share the advertising content.

```DAX
Shares =
COUNTROWS(
    FILTER(
        ad_events,
        ad_events[event_type] = "Share"
    )
)
```
---

### 🔷 4. Comments

**Definition:** Comments represent the total number of comments made by users on advertisements.

**Business Insight:** This metric helps evaluate user feedback, interaction, and audience response to the advertising content.

```DAX
Comments =
COUNTROWS(
    FILTER(
        ad_events,
        ad_events[event_type] = "Comment"
    )
)
```
---

### 🔷 5. Purchases

**Definition:** Purchases represent the total number of purchases generated after users interacted with the advertisements.

**Business Insight:** This metric represents the conversion outcome of the advertising campaigns.

```DAX
Purchases =
COUNTROWS(
    FILTER(
        ad_events,
        ad_events[event_type] = "Purchase"
    )
)
```
---

🔷 6. Engagements

**Definition:** Engagements represent the total number of user interactions with advertisements, including Clicks, Shares, and Comments.

**Business Insight:** This metric measures the overall interaction volume generated by the advertisements beyond impressions.

```DAX
Engagements = [Shares] + [Clicks] + [Comments]
```
---

###🔷 7. CTR - Click Through Rate

**Definition:** CTR represents the percentage of impressions that resulted in clicks.

**Business Insight:** CTR measures the effectiveness of advertisements in attracting users to click after seeing them.

```DAX
CTR =
DIVIDE(
    [Clicks],
    [Impressions],
    0
)
```
**Formatting:** Format the CTR measure as Percentage in Power BI.

---

### 🔷 8. Engagement Rate

**Definition:** Engagement Rate represents the percentage of impressions that resulted in user engagements.

**Business Insight:** This metric measures the overall appeal and interaction level of the advertisements.

```DAX
Engagement Rate =
DIVIDE(
    [Engagements],
    [Impressions],
    0
)
```
**Formatting:** Format the Engagement Rate measure as Percentage in Power BI.

---

### 🔷 9. Conversion Rate

**Definition:** Conversion Rate represents the percentage of clicks that resulted in purchases.

**Business Insight:** This metric measures the efficiency of converting interested users into customers after they click on an advertisement.

```DAX
Conversion Rate =
DIVIDE(
    [Purchases],
    [Clicks],
    0
)
```
**Formatting:** Format the Conversion Rate measure as Percentage in Power BI.

---

### 🔷 10. Purchase Rate

**Definition:** Purchase Rate represents the percentage of impressions that resulted in purchases.

**Business Insight:** This metric measures the overall conversion efficiency from advertisement exposure to purchase.

```DAX
Purchase Rate =
DIVIDE(
    [Purchases],
    [Impressions],
    0
)
```

**Formatting:** Format the Purchase Rate measure as Percentage in Power BI.

---

### 🔷 11. Total Budget

**Definition:** Total Budget represents the total budget allocated across all advertising campaigns.

**Business Insight:** This metric provides an overall view of the advertising budget available for campaign performance analysis.

```DAX
Total Budget =
SUM(
    campaigns[total_budget]
)
```
---

### 🔷 12. Average Budget per Campaign

**Definition:** Average Budget per Campaign represents the average budget allocated to each advertising campaign.

**Business Insight:** This metric helps understand the average budget allocation across campaigns and supports budget distribution analysis.

```DAX
Avg. Budget per Campaign =
AVERAGE(
    campaigns[total_budget]
)
```
---

### 🔷 DAX Measures Summary
Measure	Purpose
Impressions	Measures advertisement visibility
Clicks	Measures user click activity
Shares	Measures content-sharing activity
Comments	Measures user feedback and interaction
Purchases	Measures completed purchases
Engagements	Measures total Clicks + Shares + Comments
CTR	Measures clicks generated from impressions
Engagement Rate	Measures engagements generated from impressions
Conversion Rate	Measures purchases generated from clicks
Purchase Rate	Measures purchases generated from impressions
Total Budget	Measures total campaign budget
Avg. Budget per Campaign	Measures average budget allocated per campaign


---

### 🔷 KPI Calculation Flow

The DAX measures create a complete advertising performance funnel:

Impressions → Clicks → Engagements → Purchases

Along with supporting efficiency metrics:

CTR → Engagement Rate → Conversion Rate → Purchase Rate

And budget metrics:

Total Budget → Avg. Budget per Campaign

----

## 🔷 Dashboard Development

### 🔷 1. Target Gender — Dynamic Donut Chart

The **Target Gender Donut Chart** visualizes advertising performance across different gender segments using a dynamically selected KPI. The chart uses `target_gender` from the `ads` table and a **Field Parameter** to switch between Impressions, Engagements, Clicks, Shares, Comments, and Purchases.

#### 🔶 Step 1: Create a Field Parameter

Create a Field Parameter named **`Select Dynamic Measure`** containing the following measures:

- Impressions
- Engagements
- Clicks
- Shares
- Comments
- Purchases

This parameter allows users to dynamically change the metric displayed in the donut chart without creating separate visuals for each KPI.

#### 🔶 Step 2: Create a Dynamic Title

**Definition:** A Dynamic Title displays the name of the KPI selected through the Field Parameter. It updates automatically whenever the user selects a different measure.

```DAX
Dynamic Title =
IF(
    'Select Dynamic Measure'[Select Dynamic Measure Order] = 0,
    "Impressions",
    IF(
        'Select Dynamic Measure'[Select Dynamic Measure Order] = 1,
        "Engagements",
        IF(
            'Select Dynamic Measure'[Select Dynamic Measure Order] = 2,
            "Clicks",
            IF(
                'Select Dynamic Measure'[Select Dynamic Measure Order] = 3,
                "Shares",
                IF(
                    'Select Dynamic Measure'[Select Dynamic Measure Order] = 4,
                    "Comments",
                    IF(
                        'Select Dynamic Measure'[Select Dynamic Measure Order] = 5,
                        "Purchases",
                        "Others"
                    )
                )
            )
        )
    )
)
```

#### 🔶 Step 3: Create the Gender Title Measure

**Definition:** The `Gender Title` measure combines the selected KPI name with the text `"by Gender"` to generate a descriptive chart title.

```DAX
Gender Title =
SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title])
    & " by Gender"
```

**Examples of Dynamic Titles:**

- Impressions by Gender
- Engagements by Gender
- Clicks by Gender
- Shares by Gender
- Comments by Gender
- Purchases by Gender

#### 🔶 Step 4: Configure the Donut Chart

Configure the visual using the following fields:

- **Legend:** `ads[target_gender]`
- **Values:** `Select Dynamic Measure`
- **Title:** `Gender Title` measure, applied through conditional formatting using **Format visual → Title → Text → fx**.

#### 🔶 Step 5: Chart Insights

The donut chart helps identify which **target gender segment contributes the most to the selected advertising metric**.

By switching between different KPIs, the analysis can reveal:

- **Impressions:** Which gender segment receives the greatest advertising exposure.
- **Engagements:** Which segment generates the most interactions.
- **Clicks:** Which segment shows the strongest click activity.
- **Shares and Comments:** Which segment contributes most to these engagement actions.
- **Purchases:** Which segment contributes the most purchase events.

**Business Purpose:** This analysis helps evaluate the performance of gender-based targeting and identify opportunities to refine advertising campaigns based on the selected KPI.

The results should be interpreted using the actual chart values. A higher number of impressions or engagements does not necessarily mean a higher purchase rate or better return on advertising spend.

---

## 🔷 2. Target Age Group — Dynamic Bar Chart

The **Target Age Group Bar Chart** visualizes advertising performance across different age groups using a dynamically selected KPI. The chart uses `user_age` to analyze user activity and allows the selected metric to change through the **Select Dynamic Measure** Field Parameter.

### 🔶 Step 1: Configure the Bar Chart

The following fields were assigned to the visual:

- **X-axis:** `users[user_age]`
- **Y-axis:** `Select Dynamic Measure`
- **Legend:** `Select Dynamic Measure`

The Field Parameter allows users to switch dynamically between **Impressions, Engagements, Clicks, Shares, Comments, and Purchases**.

### 🔶 Step 2: Apply Age Filter

A visual-level filter was applied to the `user_age` column to display users aged **16 to 50 years**.

**Purpose:**
- Focus the analysis on the selected age range.
- Exclude users outside the specified range from this visual.
- Compare advertising performance across the included ages.

### 🔶 Step 3: Create a Dynamic Chart Title

**Definition:** The `Age Title` measure dynamically displays the selected KPI followed by `"by Age"`. The title updates when users change the selected metric.

```DAX
Age Title =
SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title])
    & " by Age"
```

**Examples of Dynamic Titles:**

- Impressions by Age
- Engagements by Age
- Clicks by Age
- Shares by Age
- Comments by Age
- Purchases by Age

Apply the `Age Title` measure through **Format visual → Title → Text → fx**.

### 🔶 Step 4: Chart Insights

The bar chart helps identify how advertising activity varies across individual ages within the selected range of **16–50 years**.

By switching between KPIs, the analysis can reveal:

- **Impressions:** Which ages receive the most advertising exposure.
- **Engagements:** Which ages generate the most interactions.
- **Clicks:** Which ages show the highest click activity.
- **Shares and Comments:** Which ages contribute most to these interactions.
- **Purchases:** Which ages generate the highest number of purchase events.

**Business Purpose:** This analysis helps identify responsive age segments and supports data-driven audience targeting and campaign optimization.

The actual chart results should be used to identify the highest-performing ages. A higher event count alone does not necessarily indicate a higher engagement rate or conversion rate.

---

## 🔷 3. Calendar Month — Calendar Heat Map

The **Calendar Heat Map** visualizes advertising activity across weeks and days of the week. It uses the `Calendar Table` created from the `ad_events[timestamp]` column to organize event activity into a calendar structure.

### 🔶 Step 1: Configure the Calendar Heat Map

The following fields were assigned to the visual:

- **Rows:** `Calendar Table[Week Number]`
- **Columns:** `Calendar Table[Day Name]`
- **Values:** `First Day Number`

The calendar structure organizes dates by week and weekday, making it easier to identify daily activity patterns across the reporting period.

### 🔶 Step 2: Display Performance Intensity

Apply conditional formatting to the calendar visual to represent activity levels through color intensity.

- **Darker shades:** Higher activity.
- **Lighter shades:** Lower activity.

This helps highlight periods of increased and decreased advertising activity.

### 🔶 Step 3: Analyze Calendar Trends

The calendar heat map helps identify:

- **Peak Activity Days:** Dates with comparatively higher advertising activity.
- **Low-Activity Days:** Dates with comparatively lower activity.
- **Weekly Patterns:** Differences in activity across weekdays.
- **Seasonal Trends:** Changes in activity across the reporting period.

### 🔶 Step 4: Business Insights

The calendar heat map provides a visual overview of when advertising interactions occur most frequently. It helps identify potential activity spikes associated with campaign launches, promotions, or other events.

**Business Purpose:** Detect high- and low-activity periods to support campaign scheduling, performance monitoring, and advertising optimization.

> **Note:** `First Day Number` is used as the calendar's date-positioning value in the current visual configuration. To make the heat map represent advertising performance intensity, use the appropriate event-count or dynamic KPI measure as the color intensity value, where supported by the selected visual.

---

## 🔷 4. Weekly Trend — Stacked Column Chart by Ad Type

The **Weekly Trend Stacked Column Chart** visualizes advertising performance across different weeks, with each column divided into individual ad types. A dynamic KPI parameter allows users to switch between different performance metrics and compare the contribution of each ad type over time.

### 🔶 Step 1: Configure the Stacked Column Chart

The following fields are assigned to the visual:

- **X-axis:** `Calendar Table[Week Number]`
- **Y-axis:** `Select Dynamic Measure`
- **Legend:** `ads[ad_type]`

The **Select Dynamic Measure** Field Parameter allows users to dynamically switch between:

- Impressions
- Engagements
- Clicks
- Shares
- Comments
- Purchases

Each column represents a week, while the individual stacked segments represent the contribution of different advertisement types to the selected KPI.

### 🔶 Step 2: Create a Dynamic Chart Title

**Definition:** The `Weekly Title` measure dynamically combines the word `"Weekly"`, the selected KPI name, and `"Trend"` to generate a descriptive chart title.

```DAX
Weekly Title =
"Weekly "
    & SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title])
    & " Trend"
```

Examples of Dynamic Titles:
- Weekly Impressions Trend
- Weekly Engagements Trend
- Weekly Clicks Trend
- Weekly Shares Trend
- Weekly Comments Trend
- Weekly Purchases Trend


Apply the Weekly Title measure through Format visual → Title → Text → fx.

### 🔶 Step 3: Analyze Weekly Performance
The stacked column chart helps identify:
- Weekly Trends: Changes in the selected KPI across different weeks.
- Ad Type Contributions: Which advertisement formats contribute most to the selected metric.
- Performance Fluctuations: Weeks with higher or lower advertising activity.
- Format Comparisons: How the contribution of Image, Video, Carousel, and Stories ads changes over time.

### 🔶 Step 4: Business Insights
The visual provides a week-by-week comparison of advertising performance across different ad formats.
By switching between KPIs, users can identify which formats generate the most impressions, clicks, engagements, or purchases during particular weeks.
Business Purpose: Understand weekly performance patterns, compare ad type contributions, and identify opportunities to optimize campaign strategy and budget allocation based on the selected metric.

---

### 🔷 Ad Type – Matrix

The **Ad Type Matrix** compares advertising performance across different ad formats using multiple engagement and conversion metrics. It helps identify which ad types generate the most impressions, clicks, and efficient engagement and purchase outcomes.

**Matrix Configuration:**
- **Rows:** `ads[ad_type]`
- **Columns:** None (as shown in the current configuration).
- **Values:**
  - `Impressions`
  - `Clicks`
  - `CTR (Click Through Rate)`
  - `Purchase Rate`
  - `Engagement Rate`
  - `Conversion Rate`
- **Dynamic Title Measure:**

```DAX
  Matrix Title = "Analysis by Ad Type"
```

Purpose:
- Compare Image, Video, Carousel, and Story ad formats side by side.
- Evaluate ad visibility, click-through performance, engagement, and purchase efficiency.
- Identify ad formats that perform well at attracting attention versus converting clicks into purchases.
Key Insights:
- Use Impressions and Clicks to identify which ad formats generate the most visibility and traffic.
- Compare CTR to assess how effectively each ad format turns impressions into clicks.
- Compare Engagement Rate, Purchase Rate, and Conversion Rate to evaluate audience interaction and conversion efficiency.
- Review the metrics together before deciding which ad formats deserve more budget; higher engagement or click volume alone does not necessarily mean higher purchase performance.

----

**Edit Interactions:** Configured **Edit Interactions** for all three charts so that selecting a specific data point in one chart filters the other charts accordingly.

**Importance:** This enables interactive cross-filtering, allowing users to explore relationships between gender, age, and weekly shares and identify patterns across different audience segments and time periods.

---
  
## 🔷 Platform-Specific Pages and Navigation

Created two separate report pages to analyze advertising performance independently for **Facebook** and **Instagram**.

### 🔷 Facebook Page
- Displays campaign performance metrics and visualizations filtered for Facebook ads.
- Enables focused analysis of audience engagement, clicks, impressions, and purchases on Facebook.

### 🔷 Instagram Page
- Displays the same key metrics and visualizations filtered for Instagram ads.
- Helps evaluate Instagram campaign performance and compare audience engagement and conversion outcomes.

### 🔷 Page Navigation Buttons
- Added **Facebook** and **Instagram** page navigation buttons to move easily between the two report pages.
- Configured the buttons using Power BI's **Page Navigator** or **Page Navigation** action.

**Purpose:** Separate platform-specific pages make it easier to analyze each platform independently, while navigation buttons provide a simple and user-friendly way to switch between reports.

---
