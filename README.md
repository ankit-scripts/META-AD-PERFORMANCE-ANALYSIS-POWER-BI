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
