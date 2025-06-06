# natural-outages-impact
A data science project for DSC 80 at UC San Diego exploring how the cause of power outages—natural events vs. other factors—impacts the number of customers affected. Includes data cleaning, visualizations, hypothesis testing, and predictive modeling.
## Introduction

> **What characteristics are associated with more severe power outages in the United States?**

This project explores the factors that influence the severity of major power outages in the United States where severity is measured by percentage of customers affected, using a rich dataset that includes event-level outage records, demographic and economic statistics, and environmental context. Specifically, we highlight on how factors such as geographic location, time of year, climate conditions, and other regional economic indicators influence the severity of power outages and aim for figuring out how these factors could help energy companies predict the location and impact of future major outages.

Power outages have wide-reaching impacts on society — disrupting transportation, healthcare, communication, and safety. Understanding the underlying patterns in when and where outages happen, and how bad they get, can help utilities and governments mitigate future risks.

---

Our dataset contains **1,534 outage events** from across the United States, covering multiple years and geographic regions.

Each row in the dataset represents one **major outage event**. Relevant columns include:

- **CAUSE.CATEGORY**/**CAUSE.CATEGORY.DETAIL**: Type of outage (e.g., "severe weather", "intentional attack")
- **OUTAGE.DURATION**: How long the outage lasted (in hours)
- **CUSTOMERS.AFFECTED**: How many customers were impacted
- **PCT_CUSTOMERS_AFFECTED**: Percent of customers affected in the region(We will used it as the measure of outage severity)
- **DEMAND.LOSS.MW**: Amount of electricity demand lost (in megawatts)
- **TOTAL.PRICE**: Electricity price during the outage (proxy for economic impact)
- **POPULATION**: Population of the affected area
- **U.S._STATE**
- **YEAR**/**MONTH**: Location and timing of the outage
- **TOTAL.CUSTOMERS**: The total number of electricity customers in the affected area (provides local demand context)
- **CLIMATE.REGION**: Broad climate category (e.g. “West”, “South”, “Pacific”) to group outages by environmental conditions
- **PCT_WATER_INLAND**: Percent of area covered by inland water (proxy for geography)
- **ANOMALY.LEVEL**: Measure of unusual environmental conditions

## Data Cleaning and Exploratory Data Analysis
### 1. **Data Cleaning**
We began by inspecting the structure and content of the dataset. Our first step was to drop the first row, which contained unit descriptions instead of actual data. We also removed the first column (variables), which simply held labels like “Units” called "obs".

Next, we converted several columns into appropriate formats:
* The OUTAGE.DURATION column, which originally contained unit suffixes and potentially invalid strings, was coerced into a numeric type using pd.to_numeric().
* Dates such as OUTAGE.START.DATE were parsed into datetime objects.
* We created a new column, PCT_CUSTOMERS_AFFECTED, by dividing CUSTOMERS.AFFECTED by TOTAL.CUSTOMERS.

To handle missing data, we removed rows with null or non-informative values in `PCT_CUSTOMERS_AFFECTED` and use IQR to remove extreme values, including:
* CAUSE.CATEGORY
* OUTAGE.DURATION

We then selected only the relevant columns for our analysis, including demographic, economic, and outage-specific variables. These included:
* YEAR, MONTH, U.S._STATE
* CAUSE.CATEGORY, CUSTOMERS.AFFECTED, OUTAGE.DURATION
* POPULATION, TOTAL.PRICE, TOTAL.CUSTOMERS
* CLIMATE.REGION, PCT_CUSTOMERS_AFFECTED, PCT_WATER_INLAND, and ANOMALY.LEVEL

This cleaning process reduced our dataset to 1,534 rows × 57 columns initially, and later to a more manageable shape 1091 rows × 15 columns after column selection and missing data filtering.
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
* Whether the missingness varies by `YEAR` (i.e., does the chance that DEMAND.LOSS.MW is missing differ between years?)
* Whether it depends on `TOTAL.PRICE`
#### Test 1: Is Missingness of `DEMAND.LOSS.MW` Dependent on `YEAR`?
We assess the MAR by creating a binary indicator `DEMAND_LOSS_IS_MISSING` and computed the variance in missingness rate across years as statistics. Then, we shuffled the missingness labels and repeated this 1,000 times to build a null distribution of variance under the assumption of no relationship.

Distribution of `YEAR` by missingness of `DEMAND.LOSS.MW`:

<iframe src="assets/year_distribution_missingness.html" width="800" height="600" frameborder="0"></iframe>
<iframe src="assets/Permutation_Test_Variance_in_DEMAND.LOSS.MW_Missingness_by_YEAR.html" width="800" height="600" frameborder="0"></iframe>

After running a permutation test with 1,000 simulations:
* The observed variance in missingness across years was significantly larger than any variance seen in the permuted (randomized) datasets.
* The p-value was 0.002, indicating that such a variance is extremely unlikely to occur by chance.

This result provides strong evidence that the missingness of `DEMAND.LOSS.MW` depends on `YEAR` and implies the missingness is MAR depending on `YEAR` column.

#### Test 2: Is Missingness of `DEMAND.LOSS.MW` Dependent on `TOTAL.PRICE`?

We ran a similar permutation test using average `TOTAL.PRICE`(i.e., the price of electricity)：

Distribution of `TOTAL.PRICE` by missingness of `DEMAND.LOSS.MW`:
<iframe src="assets/total_price_distribution_missingness.html" width="800" height="600" frameborder="0"></iframe>
<iframe src="assets/Permutation_Test_Variance_in_DEMAND.LOSS.MW_Missingness_by_TOTAL.PRICE.html" width="800" height="600" frameborder="0"></iframe>

After running a similar permutation test:
* The p-value was 0.108, meaning there is no statistically significant difference in missingness across different price levels under our chosen significant level(0.05).

There is not enough evidence to conclude that the missingness of `DEMAND.LOSS.MW` depends on `TOTAL.PRICE`. This suggests that electricity pricing likely does not influence whether demand loss is reported, and thus `TOTAL.PRICE` may be independent of this missingness.

## Hypothesis Testing

We conducted a hypothesis test to determine whether **severe weather events** result in a significantly higher number of customers affected compared to **non-severe events**. We chose to compare severe vs. non-severe weather events because weather-related causes are one of the most frequent and impactful types of outages in the dataset.

* **Null hypothesis**: The average number of customers affected by power outages is the same for outages caused by **severe weather events**(e.g., heavy winds, storms, earthquakes) and those caused by other factors such as equipment failure, fuel supply emergencies, or vandalism.

* **Alternative hypothesis**: The average number of customers affected is higher for outages caused by **severe weather events** (e.g., heavy winds, storms, earthquakes) than for those caused by other reasons.

* **Method**: A permutation test with 5000 repetitions was used. Labels indicating severe vs. non-severe events were randomly shuffled to generate a null distribution of mean differences.
* **Test Statistic**: The mean number of customers affected by outages due to severe weather events minus the mean number of customers affected by outages due to non-severe weather causes.

<iframe src="assets/null_dist_severe_weather.html" width="800" height="500" frameborder="0"></iframe>
* Observed Statistic: 108,544.95 customers
* p-value = 0.0


* **Result**: The p-value for our permutation testing is 0.0. Thus, we reject the null hypothesis at the significance level 0.05. This provides strong statistical evidence that natural events tend to result in more widespread outages since the average number of customers affected is higher by natural events than by other risk factors. However, this result is based on statistical inference, not a controlled experiment, so we cannot claim causation.


## Framing a Prediction Problem

We aim to predict the severity of a power outage, as measured by the percentage of customers affected (`PCT_CUSTOMERS_AFFECTED`), using features that would be known at the start of the outage.

**Problem Type**: Regression.

**Response Variable(target)**:
`PCT_CUSTOMERS_AFFECTED` — the percentage of customers in the region who were affected by the outage. This variable is bounded between 0 and 1, and captures the relative impact rather than raw counts (which vary by region size).
* We selected this variable because it normalizes for population size and utility customer count, making it easier to compare outage severity across regions of different scales. 

**predictor Variables(feature)**:
* `SEVERE.WEATHER` == True: Binary indicator for whether the outage was caused by a severe weather event.
* `POPULATION`: Total population in the affected area.
* `MONTH`: Numeric month of the year — useful for capturing seasonal patterns in outages.

To make sure our model is realistic, we’re only using features that would be known right when the outage starts. These include things like the cause of the outage (CAUSE.CATEGORY, CAUSE.CATEGORY.DETAIL), the state it happened in (U.S._STATE), the climate region, percentage of inland water, anomaly level, population, and total number of customers. We’re leaving out columns `OUTAGE.DURATION`, `CUSTOMERS.AFFECTED`, and `DEMAND.LOSS.MW`, because those values only make sense after the outage has already happened. This way, our model is actually useful for making predictions in real-time, not just analyzing what already happened.

To evaluate our regression model, we use **Root Mean Squared Error (RMSE)** as our evaluation metric. RMSE is appropriate because our target is continuous, and we care more about penalizing larger errors (e.g., underestimating a major outage). We chose RMSE over alternatives like MAE because it puts more weight on large deviations and RMSE retains the same units as the target making it easier to interpret, while metrics like accuracy and F1-score do not apply to regression tasks.



## Baseline Model
We built a linear regression model as a baseline to predict the **percentage of customers affected** (`PCT_CUSTOMERS_AFFECTED`) during a power outage. The goal of this step is to establish a simple, interpretable benchmark that can later be improved upon using more complex features and models.


**Features Used**

| Feature Name              | Type         | Description                                                               |
|---------------------------|--------------|----------------------------------------------------------------------------|
| `SEVERE.WEATHER == True`    | Nominal      | Binary indicator of whether the outage was caused by severe weather       |
| `LOG_POPULATION`            | Quantitative | Log-transformed total population                                          |
| `TOTAL.PRICE`          | Quantitative | Electricity price during the outage (proxy for economic impact)                                    |
| `SEASON`                    | Nominal      | Season derived from MONTH (Winter, Spring, Summer, Fall)                  |



- Quantitaive(2): `LOG_POPULATION`, `TOTAL.PRICE`
- Nominal(2): `SEVERE.WEATHER == True`, `SEASON`
- Ordinal(0)
	- Note: `MONTH` was converted to a nominal `SEASON` value


**Target Variable**

`PCT_CUSTOMERS_AFFECTED`: percentage of total customers impacted by the outage event

**Preprocessing**
- Missing Values: Filled in `POPULATION`, `MONTH`, `TOTAL.PRICE`, and `DEMAND.LOSS.MW` using their column means
- Outlier Removal: Removed statistical outliers in `CUSTOMERS.AFFECTED` using the IQR method
- Categorical Encoding: Applied **one-hot encoding** to `SEASON` via `ColumnTransformer`
- Feature Engineering: 
	- Derived `SEVERE.WEATHER == True` from the `CAUSE.CATEGORY`
	- Created `CUSTOMER_DENSITY` using the `TOTAL.CUSTOMERS` and `POPULATION`
	- Log-transformed `POPULATION` to reduce skew

<iframe src="assets/final_cleaned_df_step_6.html" width="100%" height="400" frameborder="0"></iframe>


**Model Training and Evaluation**
- Model: LinearRegression from sklearn, inside a pipeline with preprocessing.
- Train-Test Split: 80% training, 20% testing, using random_state=42.

Overall, the baseline linear regression model performed well, achieving 
- **Train RMSE**: 0.017369169379285124
- **Test RMSE**: 0.017171398578865387

The close values indivate that the model and the small feature set, this low RMSE suggests that the selected features are meaningful predictors of outage impact. This provides a strong foundation for future model improvements through more advanced techniques or feature engineering in next step.


## Final Model
To improve the baseline model performance, we introduced several new features that are more reflective of outage impact:
- `DEMAND.LOSS.MW`: Represents the scale of infrastructure affected. Outages with higher demand loss typically affect more customers, making this a direct and meaningful predictor.

- `PCT_WATER_INLAND`: Captures geographic vulnerability. Areas with high inland water percentages may be more exposed to flood-related outages or harder to restore.

- `TOTAL.PRICE`: Reflects the economic cost of electricity in each region, possibly linked to infrastructure quality or consumer density.

- `SEASON` (derived from `MONTH`): Encodes cyclical weather patterns. For example, outages in summer may be driven by heatwaves, while winter outages may involve storms or ice.

- `POP_QUARTILE`: Allows the model to capture nonlinear effects in population size by binning into quartiles, which can generalize better than treating population as purely linear.

These engineered features go beyond surface-level indicators and embed real-world structural factors that influence the severity and spread of power outages.

**Model Trained**
We developed and compared three final models to predict the percentage of customers affected during a power outage:
1. Final Model 1 - Linear Regression with SEASON and raw population features
	- This model incorporated seasonal patterns through the `SEASON` feature (derived from `MONTH`) and the size of the service region via `POPULATION`.
	- `SEASON` helps account for cyclical environmental effects like winter storms or summer heatwaves, which often drive outages.
	- `POPULATION` serves as a proxy for regional scale — larger areas may experience more widespread outages or different recovery logistics.

2. Final Model 2 – Linear Regression with `MONTH`, `POPULATION`, and `TOTAL.PRICE`
	- This model used the raw `MONTH` feature instead of `SEASON`, providing more granular time information.
	- `TOTAL.PRICE` likely reflects infrastructure cost and energy market dynamics, which could correlate with regional reliability or demand.
	- Compared to Model 1, this model emphasized economic and temporal precision.



3. FFinal Model 3 (Best) – Random Forest with GridSearchCV
	- This model included `SEASON`, `POPULATION`, `PCT_WATER_INLAND`, and `SEVERE.WEATHER == True`.
	- `PCT_WATER_INLAND` captures geographic vulnerability — regions with more water may face flood-prone outages.
	- The non-linear structure of **Random Forest** allows it to model complex interactions (e.g., population × season × weather).
	- This model used one-hot encoding for `SEASON` and captured heterogeneous effects across groups without requiring assumptions of linearity.

**Model Selection and Hyperparameters**
- Model: Random Forest Regressor
- Pipeline: One-hot encoding for `SEASON` via `ColumnTransformer`

**Best hyperparameters selected**:
- n_estimators: 100
- max_depth: 10
- min_samples_split: 2
This tuning process allowed the model to balance complexity and generalization, improving performance without overfitting.


**Performance Comparision**

| Model             | Train RMSE | Test RMSE |
|-------------------|------------|-----------|
| Baseline Model    | 0.0174     | 0.0172    |
| Final Model 1     | 0.0173     | 0.0169    |
| Final Model 2     | 0.0172     | 0.0170    |
| Final Model 3     | **0.0145** | **0.0166** |

The **Final Model 3** outperforms all others in terms of **test RMSE**, confirming its ability to generalize better due to its non-linear structure and enhanced feature set.

The performance improvement in Final Model 3 was not solely due to the use of a more flexible model but also the inclusion of domain-relevant features that reflect real-world outage conditions — such as weather, seasonal patterns, geographic risk, and population size. These features better capture the data-generating process behind power outage impacts, leading to more accurate and reliable predictions.



## Fairness Analysis
**Group Definition**
To evaluate fairness, we compared model performance between:

- Group X (`anomaly_group` = 0): Outages with `ANOMALY.LEVEL` < 0

- Group Y (`anomaly_group` = 1): Outages with `ANOMALY.LEVEL` ≥ 0

This split reflects outages that are less anomalous versus more anomalous in nature — representing "normal" vs. "extreme" events.

**Evaluation Metric**
Metric: **Root Mean Squared Error (RMSE)** on the original scale

RMSE captures the magnitude of prediction error in a way that is interpretable and sensitive to large mistakes — making it appropriate for comparing performance across groups with potentially different variance.

**Hypotheses**
- Null Hypothesis (H₀):
The model is fair. The absolute RMSE difference between the two anomaly groups is due to chance.

- Alternative Hypothesis (H₁):
The model is unfair. The observed absolute RMSE difference is statistically significant and unlikely to have occurred by chance.


**Test Statistic and Significance Level**
- Test statistic: absolute RMSE difference between Group X and Group Y, that is `anomaly_group` = 0 and `anomaly_group` = 1

- Significant Level: 0.05

We performed a permutation test with N = 1000 shuffles of the group labels and recalculated the absolute RMSE difference each time to simulate the null distributio

**Results**

Observed absolute RMSE difference: 0.0017
Permutation test p-value: 0.0770
(Note: The permutation test involves randomness, so the p-value may vary slightly on each run.)

At a typical significance level of α = 0.05, we failed to reject the null hypothesis. While the model performs slightly differently between more and less anomalous outages, the difference in RMSE is not statistically significant. This suggests that the model is reasonably fair with respect to anomaly severity.

<iframe src="assets/permutation_test_RMSE.html" width="800" height="500" frameborder="0"></iframe>











