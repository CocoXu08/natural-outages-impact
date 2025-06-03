# natural-outages-impact
A data science project for DSC 80 at UC San Diego exploring how the cause of power outages—natural events vs. other factors—impacts the number of customers affected. Includes data cleaning, visualizations, hypothesis testing, and predictive modeling.
## Introduction

> **What characteristics are associated with more severe power outages in the United States?**

This project explores the factors that influence the severity of major power outages in the United States where severity is measured by number of customers affected, using a rich dataset that includes event-level outage records, demographic and economic statistics, and environmental context. Specifically, we highlight on how factors such as geographic location, time of year, climate conditions, and other regional economic indicators influence the severity of power outages and aim for figuring out how these factors could help energy companies predict the location and impact of future major outages.

Power outages have wide-reaching impacts on society — disrupting transportation, healthcare, communication, and safety. Understanding the underlying patterns in when and where outages happen, and how bad they get, can help utilities and governments mitigate future risks.

---

Our dataset contains approximately **1,500 outage events** from across the United States, covering multiple years and geographic regions.

Each row in the dataset represents one **major outage event**. Relevant columns include:

- **CAUSE.CATEGORY**/**CAUSE.CATEGORY.DETAIL**: Type of outage (e.g., "severe weather", "intentional attack")
- **OUTAGE.DURATION**: How long the outage lasted (in hours)
- **CUSTOMERS.AFFECTED**: How many customers were impacted
- **DEMAND.LOSS.MW**: Amount of electricity demand lost (in megawatts)
- **TOTAL.PRICE**: Electricity price during the outage (proxy for economic impact)
- **POPULATION**: Population of the affected area
- **U.S._STATE**
- **YEAR**/**MONTH**: Location and timing of the outage
- **TOTAL.CUSTOMERS**: The total number of electricity customers in the affected area (provides local demand context)
- **CLIMATE.REGION**: Broad climate category (e.g. “West”, “South”, “Pacific”) to group outages by environmental conditions

## Data Cleaning and Exploratory Data Analysis
### 1. **Data Cleaning**
We began by identifying and standardizing column types (e.g. converting `OUTAGE.DURATION` to numeric). We dropped unit suffixes`OBS`, parsed dates, and created boolean indicators for missingness.

We also filtered out rows with null or uninformative values in key columns such as `CAUSE.CATEGORY`, `CUSTOMERS.AFFECTED`, and `OUTAGE.DURATION`.

Here is a preview of our cleaned dataset:

```python
df_cleaned.head()

---

### 2. **Univariate Plot + Description**
