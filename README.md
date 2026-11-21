# MSCS 634 Project Deliverable 2

## Regression Modeling and Performance Evaluation

**Student:** Rohail Shaikh , Phaneendra Vanapamula, Abhilash Marthala
**Course:** Advanced Big Data and Data Mining (MSCS-634-M50)

## Dataset

This project continues Deliverable 1 using the UCI Bike Sharing hourly dataset: **17,379
recorded hours and 17 cleaned columns**, covering 2011–2012 in Washington, D.C. The target is
`total_rentals`. Calendar and weather inputs are used to predict hourly demand.

`casual_rentals` and `registered_rentals` are excluded because they sum to the target. The
cleaned CSV preserves the observations from Deliverable 1. Missing hours are not filled.

## Feature engineering

The notebook adds **13 features**:

- Six sine/cosine features at 24-, 12-, and 8-hour periods to describe daily demand cycles.
- Six interactions between those features and `working_day` to allow different daily curves.
- `days_since_start` to represent growth over time.

The linear baseline uses 11 original input columns. The engineered versions add the new
features to the same inputs. Numeric values are standardized; month, season, and weather are
one-hot encoded inside each pipeline. The feature comparison measures the combined effect
of the new features.

## Models and validation

1. **Linear baseline:** Multiple Linear Regression using original inputs.
2. **Linear engineered:** Multiple Linear Regression with the new features.
3. **Ridge engineered:** L2 regularized regression with the new features.

The first **13,747 records**, through July 31, 2012, are used for training and selection.
The final **3,632 records**, August–December 2012, form the test period. Five expanding
validation folds use two-month calendar windows, always training on earlier dates. The data
is never shuffled, and fitted preprocessing stays inside each training fold.

Ridge alpha is selected from **0.1, 1, 10, 100** using mean validation RMSE. The selected
alpha is **100**. Validation scores guide selection; they are not independent estimates
after tuning. The model recommendation is fixed before test evaluation.

## Evaluation results

| Model | Test R² | Test MSE | Test RMSE | CV R² | CV RMSE |
|---|---:|---:|---:|---:|---:|
| Linear baseline | 0.3284 | 32,689.24 | 180.80 | 0.2612 | 145.24 |
| Linear engineered | 0.6835 | 15,403.35 | 124.11 | 0.5854 | 105.00 |
| Ridge engineered | 0.6774 | 15,701.18 | 125.30 | 0.6567 | 99.59 |

Higher R² and lower MSE/RMSE are better. RMSE is in rentals per recorded hour; MSE is in
squared rental counts. CV values are means across the five calendar folds. The notebook
also reports training R², CV MSE, fold RMSE, and its standard deviation.

## Findings and model recommendation

- **Feature engineering has the largest effect.** Linear test RMSE falls from
  **180.80 to 124.11**, a **31.4% reduction**. Mean validation RMSE falls by
  **27.7%**. The new inputs help the linear model represent daily curves and growth.
- **Linear engineered has the best final test result:** R² **0.6835** and RMSE **124.11**.
- **Ridge engineered is recommended from validation:** mean CV RMSE **99.59** compared
  with **105.00** for Linear engineered. Its test RMSE is **125.30**, a difference of
  **1.19 rentals/hour**. Test results do not change the validation-based selection.
- **Peak demand remains harder to predict.** Ridge's test RMSE is **219.78** for hours
  at or above **410 rentals** (the training 90th percentile), versus **82.63** for other hours.
  Busy-hour mean residual is **+184.87**, showing underprediction. There are
  **25 negative test predictions**; reported scores use unmodified predictions.

## Visualizations

The notebook contains labeled charts comparing validation/test RMSE and test R², plus
actual-versus-predicted and residual plots for the validation-selected Ridge model. Each
figure is followed by an explanation of what it shows.

## Challenges and solutions

| Challenge | Solution |
|---|---|
| Daily and working-day patterns are nonlinear. | Added cyclic features and working-day interactions, then compared them with the linear baseline. |
| Temperature and apparent temperature are correlated. | Compared Ridge regularization with ordinary regression; standardized numeric inputs inside pipelines. |
| Time ordering and absent hours complicate validation. | Used expanding two-month calendar windows and a later test period. |
| Rental component counts reveal the target. | Excluded both components from all predictors. |
| High-demand hours have larger errors. | Retained valid extremes and examined residuals and busy-hour errors. |

Recorded weather describes the same hour. Advance forecasting would need weather forecasts
available at prediction time. Deliverable 1 already explored the full dataset descriptively,
so this test is a later-period check rather than a completely unseen research dataset.
The questionable humidity readings identified in Deliverable 1 remain unchanged. These
historical, system-wide records support planning analysis, not current station-level estimates.

## Repository files

| File | Purpose |
|---|---|
| `ProjectDeliverable_2_Rohail_Shaikh.ipynb` | Executed notebook with code, metrics, charts, and explanations. |
| `bike_sharing_cleaned.csv` | Cleaned dataset from Deliverable 1; required to rerun the notebook. |
| `model_comparison.csv` | Model evaluation results generated by the notebook. |
| `README.md` | Dataset, methods, results, findings, and challenges. |
| `requirements.txt` | Required Python packages. |

## Run the notebook

Extract the ZIP first. Open Anaconda Prompt in the extracted folder and run:

```bash
python -m pip install -r requirements.txt
jupyter notebook ProjectDeliverable_2_Rohail_Shaikh.ipynb
```

Choose **Restart Kernel and Run All Cells**. Keep the CSV in the same folder. No data
download is needed.

## Submission

Upload the five files above to `MSCS_634_ProjectDeliverable_2` on GitHub. Ensure the repository
is public or the instructor has access. Submit the repository URL on Blackboard, as required
by the project instructions. Upload the extracted files to GitHub, rather than only the ZIP.

## References

Fanaee-T, H. (2013). *Bike sharing* [Data set]. UCI Machine Learning Repository.
https://doi.org/10.24432/C5W894  
License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Column names and calendar
features were prepared in Deliverables 1 and 2.

Scikit-learn developers. (n.d.). *Time-related feature engineering*.
https://scikit-learn.org/stable/auto_examples/applications/plot_cyclical_feature_engineering.html

Wu, D. (2024). *Data mining with Python: Theory, application, and case studies*. CRC Press.
