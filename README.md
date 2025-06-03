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
<iframe src="assets/df_preview.html" width="100%" height="250"></iframe>

---

### 2. **Univariate Plot + Description**

Embed **Plotly charts showing one column’s distribution**, e.g. `CAUSE.CATEGORY`,`CLIMATE.CATEGORY`：

<iframe src="assets/Histogram_of_Cause_of_Power_Outage.html" width="800" height="500" frameborder="0"></iframe>

This plot shows that most power outages are caused by **severe weather**, followed by **intentional attacks**. Less frequent causes include equipment failure, system disruption, and islanding. This skew suggests that natural factors are the dominant driver of major outages.

<iframe src="assets/Histogram_of_CLIMATE.CATEGORY.html" width="800" height="500" frameborder="0"></iframe>

This chart shows how power outages are distributed across different **climate regions**. Most outages occur in areas categorized as **“normal”**, followed by **cold** and **warm** zones. This may reflect both population distribution and climate-linked vulnerabilities.

---

### 3. **Bivariate Plot + Description**

Embed **Plotly charts  that displays the relationship between two columns**. e.g. `Outage Duration` vs. `Customers Affected`：

<iframe src="assets/Scatter_OUTAGE_DURATION_vs_CUSTOMERS_AFFECTED.html" width="800" height="600" frameborder="0"></iframe>

This scatter plot shows the relationship between **outage duration** and the **number of customers affected**. While there's no strong linear trend, we observe that even **short-duration outages can impact large populations**, indicating that population density and infrastructure vulnerability both play roles in outage severity.

---

### 4. **Interesting Aggregates**
A **grouped table** of Average Customers Affected by Outage Cause:
<iframe src="assets/agg_final_by_cause.html" width="100%" height="400" frameborder="0"></iframe>

This grouped table summarizes the **average number of customers affected** by different combinations of outage causes and subcategories. It also shows how often each type of event occurs and includes a column indicating whether the cause was related to **severe weather**.

While some equipment-related issues impact large numbers of customers, severe weather remains the most consistently high-impact cause overall, both in terms of frequency and average disruption. We also observe that certain technical causes (like **substation** or **HVSubstation interruption**) can have as large or even larger impacts than severe weather events, despite occurring less frequently. This suggests the importance of infrastructure resilience alongside climate response.

## Assessment of Missingness

I believe the column `CAUSE.CATEGORY.DETAIL` is **not missing at random(NMAR)**.
The `CAUSE.CATEGORY.DETAIL` provides subcategories of why a power outage occurred under larger cause categories, such as it was due to thuderstorm, heavy winds, vandalism or other reasons. In many cases, this information is left blank, suggesting that the detailed cause was either not determined or not recorded. I perceive that the the missingness may depend on the nature of outage itself since normally arge, complex, or high-impact outages are more likely to attract attention which results in a completed `CAUSE.CATEGORY.DETAIL` entry, while routine or minor outages might not warrant a detailed investigation thus leave blank on cause details. Therefore, this column might be NMAR. 

To potentially reclassify this missingness as Missing At Random (MAR), there are several additional data that could possibly explain the missingness. First, If we could get an **Investigation status** column which indicates whether a formal investigation was conducted, this could explain why some outages lack detailed causes. Furthermore, we could include a column of **Regulatory reporting requirements by region or utility** which might help us to attribute the missingness to those external rules.

---
### Missingness Dependency Analysis of `DEMAND.LOSS.MW`
We investigated whether the missingness of the `DEMAND.LOSS.MW` column depends on other columns using permutation testing. Specifically, we tested on:
	•	Whether the missingness varies by `YEAR` (i.e., does the chance that DEMAND.LOSS.MW is missing differ between years?)
	•	Whether it depends on `TOTAL.PRICE`
#### Test 1: Is Missingness of `DEMAND.LOSS.MW` Dependent on YEAR?
We assess the MAR by creating a binary indicator `DEMAND_LOSS_IS_MISSING` and computed the variance in missingness rate across years as statistics. Then, we shuffled the missingness labels and repeated this 1,000 times to build a null distribution of variance under the assumption of no relationship.
	•	Observed test statistic (variance): Very high
	•	p-value: 0.0
Distribution of year by missingness of `DEMAND.LOSS.MW`:
<iframe src="assets/year_distribution_missingness.html" width="800" height="600" frameborder="0"></iframe>


