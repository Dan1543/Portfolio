---
layout: post
title: "Model Design"
parent: "Weather Station"
nav_order: 3
project_url: https://github.com/Dan1543/Weather-Monitor
---

## Predictive Analytics Approach
While hardware collects the data, the core value of this project lies in its ability to anticipate weather conditions. The objective was to build a classification model capable of predicting the weather state (Clear, Cloudy, Rain, Storm) for the next 60 minutes based on the **BME280** sensor lectures.

---

## Data mining
Ideally, the model would be trained on local sensor data. However, collecting years of historical data for initial training is impractical. To solve this, I utilized **Meteostat**, an open database of historical weather data.
### Data Sources
The following cities were selected to represent diverse climate zones:

- **Tijuana** (Semi-arid)
- **London** (Oceanic)
- **Tokyo** (Humid Subtropical)
- **Sydney** (Southern Hemisphere)
- **Cairo** (Arid Desert)
- **Sao Paulo** (Subtropical)
- **Moscow** (Cold Continental)
- **Singapore** (Tropical)
- **Reykjavik** (Subpolar)
- **Vancouver** (Temperate Rainy)

---

## Feature selection
The initial dataset included several raw features. To optimize performance, I analyzed which variables actually contributed to accurate predictions.

| Feature | Description |
| :--- | :--- |
| **Temperature (temp)** | Air temperature in °C. |
| **Relative Humidity (rhum)** | Relative humidity percentage. |
| **Pressure (pres)** | Atmospheric pressure in hPa. |
| **Weather Condition (coco)** | Target variable: weather state classification. |
| **Year, Month, Day, Hour** | Temporal features for seasonal and diurnal patterns. |
| **City** | Location identifier for geographic context. |
| **Longitude & Latitude** | Geographic coordinates for regional weather patterns. |

### Correlation Analysis
To verify data quality and reduce dimensionality, a correlation heatmap was generated. This helped visualize linear relationships and identify redundant features.

![CorrelationMap]({{ site.baseurl }}/assets/images/correlation.png)
*Figure 1: Feature correlation heatmap*
{: .text-center .d-block .mt-2 .text-delta }

**Key Findings:**
* **Year and Day** were dropped as they showed no correlation on weather changes. While months capture seasonality effectively, specific days or years could introduce noise.
* **City Name** was excluded from training (used only for visualization).

### Feature Importance (Random Forest Test)
I ran a preliminary Random Forest model to determine feature importance. The results indicated that **latitude and longitude** were not significant predictors compared to direct atmospheric readings, so they were also removed to simplify the model.

![FeatureImportances]({{ site.baseurl }}/assets/images/importances.png){: .d-block .mx-auto }

*Figure 2: Feature importance ranking (Random Forest)*
{: .text-center .d-block .mt-2 .text-delta }

### Data Visualization (T-SNE)
For further analysis, I used **T-SNE** to project the high-dimensional data into 2D space. The resulting clusters (islands) suggest that distinct weather patterns are separable, although some overlap that indicates potential classification challenges specially with storms.

![T-SNE]({{ site.baseurl }}/assets/images/tsne.png)
*Figure 3: T-SNE 2D visualization of weather clusters*
{: .text-center .d-block .mt-2 .text-delta }


## Data Processing 
Raw sensor data is rarely enough for accurate predictions. Before feeding the model, the data goes through a preprocessing stage.

| Feature | Description | Relevance |
| :--- | :--- | :--- |
| **Pressure Delta (delta_p)** | Change in atmospheric pressure over time (hPa/hour). | Indicates rapid weather shifts. |
| **Humidity Delta (delta_rhum)** | Change in relative humidity over time (%/hour). | Detects moisture pattern changes. |
| **Temperature Delta (delta_temp)** | Change in air temperature over time (°C/hour). | Captures temperature trend momentum. |
| **Dew Point (dewpoint)** | Temperature at which air becomes saturated and water vapor condenses (°C).| High dew point signals humid; low dew point indicates dry air. Reflects atmospheric instability. |

---

## Model Selection: XGBoost
Several algorithms were evaluated for this task. The table below summarizes the trade-offs considered:

| Algorithm | Pros | Cons | Verdict |
| :--- | :--- | :--- | :--- |
| **Linear Regression** | Simple and fast. | Cannot capture non-linear relationships in weather. |  Discarded |
| **Random Forest** | High accuracy, robust. | Large memory footprint (many trees), harder to implement manually in pure Python. | Discarded |
| **XGBoost** | **High accuracy and efficiency.** | Sensitive to the hyperparameters. | **Selected** |

### Why XGBoost?
I selected **XGBoost** (Extreme Gradient Boosting) because it provides the best balance between performance and efficiency for this specific dataset.
1.  **Built-in Regularization:** XGBoost includes L1 and L2 regularization, which helps prevent the model from memorizing the "noise" in the weather data.
2.  **Portability to Edge:** Using the library **mc2gen** the model can be converted into standard Python code (nested if-else statements). This allows it to run within the strict memory limits of AWS Lambda Free Tier without heavy dependencies.

## Model training:
Two versions of the model were trained to explore accuracy and deployment size.

* **Model A (High Accuracy):** Achieved **78% accuracy** but required 134 MB, exceeding Lambda Free Tier limits.
* **Model B (Optimized):** Achieved **58% accuracy** but fit within 3 MB, making it viable for the serverless architecture.

**Hyperparameter Comparison:**

| Parameter | Value (Lambda Free Tier) | Value (Best Accuracy) |
| :--- | :--- | :--- |
| **n_estimators** | 50 | 75 |
| **max_depth** | 7 | 25 |
| **learning_rate** | 0.1 | 0.1 |
| **num_parallel_tree** | 1 | 1 |
| **n_jobs** | -1 | -1 |
| **Model Size** | **2.7 mB** | 134 MB |
| **Accuracy** | 58% | 78% |
| :--- | :--- |

---

## Conclusion & Future Work
While the larger model offers significantly better performance, the primary goal of this project was to demonstrate a functional **Cloud IoT architecture** within the constraints of the AWS Free Tier.

The deployed model (58% accuracy) serves as a proof of concept. For a production environment, upgrading to paid AWS instances (or using SageMaker) would allow deploying the larger model. 