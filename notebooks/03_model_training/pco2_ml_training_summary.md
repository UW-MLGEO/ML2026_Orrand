# Predicting Ocean pCO₂ Using Machine Learning

## What Is This Project About?

The ocean absorbs a huge amount of CO₂ from the atmosphere, and scientists track this by measuring **pCO₂** — the partial pressure of carbon dioxide dissolved in surface seawater. Measuring pCO₂ directly is expensive and limited to specific buoy locations, so our goal was to **predict pCO₂ using satellite data** that covers the whole ocean.

We built machine learning models that take in satellite observations (sea surface temperature and chlorophyll-a concentration) and try to predict what the pCO₂ reading would be at a given location and time.

---

## The Data

Our training dataset came from **7 NOAA buoy sites** and included **479 buoy-days** of matched data. For each day at each site, we had:

- **Satellite sea surface temperature (SST)** — 7 summary statistics (mean, std, min, max, median, closest pixel value, number of pixels)
- **Satellite chlorophyll-a (chl-a)** — 8 features (same stats plus how many days offset the nearest satellite composite was)
- **In-situ SST** — the actual temperature measured by the buoy
- **Location** — latitude and longitude

The **target variable** (what we're predicting) was `pco2_mean` — the daily average pCO₂ in µatm (micro-atmospheres).

---

## Our Approach

### 1. Split the Data

We held out **20% of the data** as a test set that the models never saw during training. This lets us honestly evaluate how well each model generalizes to new, unseen data. We also made sure each buoy site was proportionally represented in both the training and test sets.

We **standardized** (z-score normalized) the features so each one has a mean of 0 and standard deviation of 1. This is especially important for Linear Regression.

### 2. Train Three Models

We tested three models of increasing complexity:

---

### Model 1: Linear Regression (The Simple Baseline)

**What it does:** Draws a single straight-line relationship between the input features and pCO₂. Think of it like: *pCO₂ = a×SST + b×chl-a + c×latitude + ...*

**Strengths:** Very fast, easy to understand, good starting point.

**Weakness:** Assumes the relationship between each feature and pCO₂ is a straight line — which it usually isn't in the real ocean.

---

### Model 2: Random Forest

**What it does:** Builds a "forest" of **200 decision trees**, each trained on a random subset of the data. Each tree makes its own prediction, and the final answer is the **average of all the trees**. Think of it like asking 200 slightly different experts and averaging their opinions.

**Strengths:** Handles non-linear (curvy) relationships naturally. Very robust and hard to break.

**Weakness:** Can be slow with very large datasets and doesn't extrapolate well outside the range of training data.

---

### Model 3: Gradient Boosted Trees (HistGradientBoosting)

**What it does:** Also builds decision trees, but **sequentially** — each new tree specifically focuses on correcting the mistakes of the previous trees. It's like a student who reviews their wrong answers after each practice test and focuses on improving those areas.

**Strengths:** Often the best performer for tabular (spreadsheet-like) data. Very efficient.

**Weakness:** More hyperparameters to tune and can overfit if not careful.

---

## Results

### How We Measured Performance

- **RMSE** (Root Mean Squared Error) — The average prediction error in µatm. **Lower is better.**
- **MAE** (Mean Absolute Error) — Similar to RMSE but less sensitive to big outliers. **Lower is better.**
- **R²** (R-squared) — The fraction of the variability in pCO₂ that the model explains. **1.0 = perfect, 0.0 = no skill.**

### Predicted vs. Actual pCO₂

The plot below shows each model's predictions (y-axis) against the actual measured pCO₂ values (x-axis). Points on the dashed 1:1 line are perfect predictions. The further a point is from the line, the worse that individual prediction was.

![Predicted vs Actual pCO2](../../plots/ml_results/01_predicted_vs_actual.png)

**What we see:** Linear Regression has the most scatter (points far from the line), while the tree-based models cluster much more tightly around the diagonal — meaning they made more accurate predictions.

---

### Residual Analysis (Error Check)

Residuals are simply **Actual − Predicted**. Ideally, residuals should be randomly scattered around zero with no patterns. If we see a trend, it means the model is systematically over- or under-predicting in certain ranges.

![Residual Analysis](../../plots/ml_results/02_residuals.png)

**What we see:** The residuals for the best model are roughly centered around zero, which is a good sign. The histogram on the right shows most errors are small, with only a few larger misses.

---

### Feature Importance

One powerful thing about tree-based models is they can tell us **which input features mattered most** for making predictions. This isn't just useful for building better models — it teaches us something about what drives pCO₂ in the ocean.

![Feature Importance](../../plots/ml_results/03_feature_importance.png)

**What we see:** The features related to **SST** (sea surface temperature) tend to be the most important predictors of pCO₂, followed by **location** (latitude/longitude) and **chlorophyll-a** features. This makes physical sense — temperature strongly controls how much CO₂ the ocean can hold.

---

### Performance by Buoy Site

Not all ocean locations are equally easy to predict. Some sites (like open-ocean buoys) have more predictable patterns, while coastal or estuary sites can have complex local dynamics that are harder to capture.

![Predictions by Site](../../plots/ml_results/04_predictions_by_site.png)

**What we see:** The model performs better at some sites than others. Sites where points hug the 1:1 line are well-predicted; sites with more scatter have higher error — these may need more training data or additional features to improve.

---

## Key Takeaways

1. **Tree-based models significantly outperform Linear Regression** for this problem — the relationships between satellite observations and ocean pCO₂ are non-linear.
2. **Sea surface temperature is the most important predictor** of pCO₂, which aligns with known ocean chemistry (CO₂ solubility decreases as water warms).
3. **Some buoy sites are harder to predict than others**, reflecting real physical differences in ocean carbon dynamics across regions.
4. **Machine learning can bridge the gap** between sparse buoy measurements and continuous satellite coverage, potentially enabling ocean-wide pCO₂ estimates.

---

## Next Steps

Here are the logical directions to take this project further:

1. **Hyperparameter Tuning** — Use cross-validation and grid search (or Bayesian optimization) to systematically find the best settings for each model, rather than using hand-picked values.

2. **Add More Features** — Incorporate additional satellite-derived variables like sea surface salinity, mixed layer depth, wind speed, or ocean color indices that are known to influence pCO₂.

3. **Temporal Features** — Add time-based features like day of year, month, or seasonal indicators. Ocean pCO₂ has strong seasonal cycles that the model could learn from.

4. **Cross-Validation Instead of a Single Split** — Use k-fold cross-validation to get a more robust estimate of model performance and reduce the risk of a "lucky" or "unlucky" train/test split.

5. **Spatial Generalization Test** — Try a "leave-one-site-out" experiment where you train on 6 sites and predict the 7th. This tests whether the model can predict at locations it has never seen — critical for making ocean-wide maps.

6. **Try Neural Networks** — Explore simple feedforward neural networks or more advanced architectures to see if they can capture even more complex patterns in the data.

7. **Deploy for Prediction** — Apply the trained model to full satellite image grids to produce **spatial maps of predicted pCO₂** across the ocean, filling in the gaps between buoy locations.
