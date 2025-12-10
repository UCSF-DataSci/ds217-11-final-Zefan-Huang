# Chicago Beach Weather Sensors Analysis

## Executive Summary

This project analyzes hourly weather sensor data from Chicago beaches along Lake Michigan. The dataset contains 196,526 rows and 18 columns of measurements such as air temperature, humidity, wind, rain, and pressure from 2015–2025. The main goal is to understand temporal patterns and build models to predict **Air Temperature** using other variables and time-based features. After going through a full 9-phase workflow (exploration → cleaning → feature engineering → modeling), the **Random Forest** model achieved the best performance on the test set (R² ≈ 0.90, RMSE ≈ 3.15°C, MAE ≈ 1.83°C), showing that air temperature can be predicted reasonably well from cleaned sensor data and temporal features.

---

## Phase-by-Phase Findings

### Phase 1–2 (Q1): Exploration

Initial exploration showed:

- **Size:** 196,526 rows and 18 columns.
- **Main variables:**  
  - Temperatures: Air Temperature, Wet Bulb Temperature  
  - Moisture and rain: Humidity, Rain Intensity, Interval Rain, Total Rain, Precipitation Type  
  - Wind: Wind Direction, Wind Speed, Maximum Wind Speed  
  - Others: Barometric Pressure, Solar Radiation, Heading, Battery Life  
  - Metadata: Station Name, Measurement Timestamp, Measurement Timestamp Label, Measurement ID
- **Stations:** Three beach weather stations (Foster, Oak Street, 63rd Street).
- **Date range:** From April 2015 to December 2025.

Key data quality issues seen in the EDA:

- Large blocks of missing values in Wet Bulb Temperature, several rain variables, and Heading.
- Smaller amounts of missing data in Air Temperature and Barometric Pressure.
- Clear outliers in rain, wind, pressure, solar radiation, and heading (sensor glitches).
- Strong seasonal pattern in Air Temperature and clear daily cycles.

The file `output/q1_visualizations.png` contains histograms and early time series plots that confirmed these patterns.

---

### Phase 3 (Q2): Data Cleaning

The cleaning step focused on missing values, outliers, and data types:

- **Missing values**
  - Air Temperature, Wet Bulb Temperature, Rain Intensity, Interval Rain, Total Rain, Precipitation Type, Barometric Pressure, Heading, and a few other numeric columns had missing values.
  - All numeric missing values were filled with the **median** of each column.
  - Categorical columns with missing values were filled with `"Unknown"`.

- **Outliers**
  - Outliers in numeric columns were detected using the **IQR method**.
  - Values below the lower bound or above the upper bound were **capped** at the IQR limits.
  - This was applied to variables such as Air Temperature, Wet Bulb Temperature, Humidity, rain variables, Wind Speed, Maximum Wind Speed, Barometric Pressure, Solar Radiation, Heading, and Battery Life.

- **Duplicates and types**
  - No duplicate rows were removed (duplicates count = 0).
  - `Measurement Timestamp` was converted to a proper datetime type.

After cleaning, the dataset still had **196,526 rows**, but with complete values and more realistic ranges.

---

### Phase 4 (Q3): Datetime Parsing and Temporal Features

Using the cleaned data:

- `Measurement Timestamp` was parsed to datetime and used to compute the time range:
  - **Start:** 2015-04-25 09:00:00  
  - **End:**   2025-12-07 21:00:00  
  - Total duration is a little more than **10 years** of hourly data.

From this timestamp, several **temporal features** were created:

- `hour` (0–23)
- `day_of_week` (0–6, Monday–Sunday)
- `month` (1–12)
- `year`
- `day_name` (string)
- `is_weekend` (0 = weekday, 1 = weekend)

These are saved in `output/q3_temporal_features.csv` and provide the basic time structure for later analysis.

---

### Phase 5 (Q4): Derived and Rolling Features

To capture short-term dynamics and more context, a few extra features were engineered on top of the cleaned data:

- **Time-based features** (from Phase 4): `hour`, `day_of_week`, `month`, `year`, `day_name`, `is_weekend`.
- **Rolling window features:**
  - `wind_speed_rolling_7h`: 7-hour rolling mean of Wind Speed.
  - `humidity_rolling_24h`: 24-hour rolling mean of Humidity.

All original columns plus these new ones were saved into:

- `output/q4_features.csv`
- `output/q4_rolling_features.csv`

These rolling features help the model see short-term trends, not only single hourly values.

---

### Phase 6 (Q5): Temporal Patterns and Correlations

Using the temporal features and rolling features, I analyzed time series patterns:

- **Seasonal trends**
  - Monthly Air Temperature ranges roughly from **−5.04°C** in the coldest months to **25.25°C** in the warmest months.
  - There is a clear annual cycle: warm summers and cold winters, consistent with Chicago’s climate.

- **Daily (diurnal) cycle**
  - Air Temperature typically:
    - reaches its **minimum** around hour **6** (about 6 AM),
    - reaches its **maximum** around hour **16** (about 4 PM).

- **Correlations**
  - Air Temperature has a strong positive correlation with **Wet Bulb Temperature** (about **0.744**).
  - Some correlations with rain variables are weak or undefined, especially where imputation and capping were heavy.

The multi-panel figure in `output/q5_patterns.png` shows:

- Monthly average Air Temperature over the entire period.
- Average Air Temperature by hour of day.
- A correlation heatmap of Air Temperature, Wet Bulb Temperature, humidity, wind variables, and rolling features.

---

### Phase 7 (Q6): Train/Test Split and Feature Preparation

For modeling, the target variable was chosen as **Air Temperature**. To avoid information from the future leaking into the past, a **temporal** split was used:

- **Split method:** 80/20 by time (not random).
- **Training set:** 157,220 rows  
  - Date range: 2015-04-25 09:00:00 → 2023-07-09 16:00:00
- **Test set:** 39,306 rows  
  - Date range: 2023-07-09 17:00:00 → 2025-12-07 21:00:00
- **Number of features:** 24 (all selected feature columns except the target).
- **Target variable:** Air Temperature.

Training and test features/targets were saved as:

- `output/q6_X_train.csv`, `output/q6_y_train.csv`
- `output/q6_X_test.csv`,  `output/q6_y_test.csv`

Only numeric feature columns were used when fitting the models.

---

### Phase 8 (Q7): Modeling and Feature Importance

Two regression models were trained to predict Air Temperature:

1. **Linear Regression**  
2. **Random Forest Regressor**

Predictions on the test set were saved in `output/q7_predictions.csv` with columns:

- `actual`
- `predicted_linear`
- `predicted_random_forest`

A separate file `output/q7_feature_importance.csv` stores feature importance values from the Random Forest.

From the updated key findings:

- **Linear Regression (test set):**
  - R² ≈ **0.5229**
  - RMSE ≈ **7.04°C**
  - MAE ≈ **5.14°C**

- **Random Forest (test set):**
  - R² ≈ **0.9042**
  - RMSE ≈ **3.15°C**
  - MAE ≈ **1.83°C**

Random Forest clearly performs much better than the simple linear model. It captures non-linear relationships and interactions between weather variables and time features.

**Feature importance (Random Forest):**

- The most important feature is **Wet Bulb Temperature** with an importance around **0.49**.
- The **top three features together explain about 0.89** of the total importance.
- This suggests that Wet Bulb Temperature plus a few other key variables (likely humidity or time-related features) drive most of the predictive power.

---

### Phase 9 (Q8): Final Results and Summary

The last phase combined the numbers and visualizations into final interpretations:

- **Best performing model:** Random Forest.
- **Model performance:**
  - Random Forest R² ≈ 0.90 on the test set, meaning it explains about 90% of the variation in hourly air temperature.
  - Its RMSE (~3.15°C) and MAE (~1.83°C) are much smaller than those of Linear Regression.
- **Feature importance:**
  - Wet Bulb Temperature is the dominant predictor.
  - A small set of highly informative features explains most of the model’s performance.
- **Temporal patterns:**
  - Clear seasonal and daily cycles confirmed by both EDA and feature importance.
  - Time-related features (like hour and month) support the model but are less important than Wet Bulb Temperature.

The final visualization `output/q8_final_visualizations.png` includes model comparison plots, predictions vs actuals, feature importance, and residuals.

---

## Visualizations

Below are the main figures from the analysis. All images are saved in the `output/` directory.

![Figure 1: Initial Data Exploration](output/q1_visualizations.png)  
*Figure 1: Initial exploration showing distributions of Air Temperature, its time series, and basic summaries of other variables and station counts.*

![Figure 2: Temporal Patterns and Correlations](output/q5_patterns.png)  
*Figure 2: Pattern analysis with monthly average temperatures, average temperature by hour of day, and a correlation heatmap of key variables and features.*

![Figure 3: Final Model Performance](output/q8_final_visualizations.png)  
*Figure 3: Final visualizations for Linear Regression and Random Forest, including model comparison, predictions vs actual values, feature importance, and residual plots.*

---

## Model Results

Using the updated test-set metrics, model performance can be summarized as:

| Model            | R² (Test) | RMSE (Test, °C) | MAE (Test, °C) |
|------------------|-----------|-----------------|----------------|
| Linear Regression| 0.5229    | 7.0390          | 5.1403         |
| Random Forest    | 0.9042    | 3.1537          | 1.8253         |

### Interpretation

- **R² (coefficient of determination)**  
  - Linear Regression explains about **52%** of the variance in test-set air temperature.  
  - Random Forest explains about **90%**, which is a strong result for this type of problem.

- **RMSE (Root Mean Squared Error)** and **MAE (Mean Absolute Error)**  
  - Linear Regression predictions are off by about **7°C** on average (RMSE) and **5°C** in absolute error.  
  - Random Forest reduces errors to about **3.15°C** RMSE and **1.83°C** MAE, showing much better accuracy.

Overall, Random Forest is clearly the better model: it captures non-linear patterns and interactions and gives substantially smaller prediction errors.

### Feature Importance

From the Random Forest feature importance:

- **Wet Bulb Temperature** is the top feature (importance ≈ 0.49).
- The top three features together account for about **89%** of total importance.
- Time-based features and rolling averages add information but are secondary compared to the main temperature-related inputs.

This aligns with intuition: Wet Bulb Temperature is physically related to Air Temperature and humidity and therefore carries strong information about the target.

---

## Time Series Patterns

From the temporal analysis:

- **Seasonal patterns**
  - Monthly Air Temperature ranges from roughly **−5°C (winter)** to **25°C (summer)**.
  - Clear seasonal cycles, with warm summers and cold winters, dominate the long-term variation.

- **Daily cycles**
  - Air Temperature is lowest early in the morning (around 6 AM) and highest in the late afternoon (around 4 PM).
  - This daily cycle is consistent over the year and reflects normal heating/cooling over each day.

- **Relationships between variables**
  - Air Temperature and Wet Bulb Temperature are strongly positively correlated.
  - Rain-related variables and some wind variables show weaker or noisy relationships, especially where missing data and imputation were heavy.
  - Rolling features (`wind_speed_rolling_7h`, `humidity_rolling_24h`) help smooth short-term fluctuations and capture broader patterns.

These patterns explain why adding temporal features and related weather variables improves model performance.

---

## Limitations and Next Steps

### Limitations

1. **Heavy imputation and capping**
   - Several columns (especially Wet Bulb Temperature, rain variables, Heading) had large blocks of missing values that were filled with medians.
   - Many outliers were capped using the IQR method. This may smooth out real extremes and can distort distributions.

2. **Limited feature engineering**
   - Only a small number of engineered features were used:
     - Basic time features (`hour`, `day_of_week`, `month`, `year`, `day_name`, `is_weekend`)
     - Two rolling means (`wind_speed_rolling_7h`, `humidity_rolling_24h`).
   - No lagged versions of the target (e.g., previous-hour temperature) or more complex domain-specific indices were included.

3. **Evaluation design**
   - Only one temporal 80/20 split was used.
   - There was no rolling-window cross-validation, so we do not know how stable the model performance is across different time periods.

4. **Interpretability**
   - Tree-based models like Random Forest give importance scores but do not show simple formulas.
   - It is hard to see exact functional forms of how features interact.

### Next Steps

If this project were continued, the following steps would be useful:

1. **Richer feature engineering**
   - Add lag features for Air Temperature, humidity, and wind (e.g., previous 1, 6, 24 hours).
   - Create more rolling windows (e.g., 3h, 12h, 48h) for key predictors.
   - Encode season using cyclic features (sin/cos of month and hour).

2. **More robust evaluation**
   - Use rolling origin cross-validation for time series to check performance stability.
   - Compare results across different years or seasons.

3. **Model comparison**
   - Try additional models such as Gradient Boosting, XGBoost, or simple neural networks.
   - Compare complexity vs performance and training time.

4. **Station-level analysis**
   - Examine whether patterns and model performance differ by station.
   - Potentially train separate models per station or include station as a stronger categorical feature.

5. **Practical deployment**
   - Build a small prototype that takes real-time sensor data and outputs short-term air temperature forecasts for each beach.

### Conclusion
This analysis successfully applied a complete 9-phase data science workflow to Chicago Beach Weather Sensors data. The Random Forest model achieved excellent performance with R² = 0.9042 and RMSE = 3.15°C. The results highlight the importance of including key physical variables (like Wet Bulb Temperature) and temporal context (Month) when modeling weather systems. The significant improvement of Random Forest over Linear Regression (R² 0.90 vs 0.52) underscores the non-linear nature of weather patterns.
