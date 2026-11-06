# Same Size, Different Damage

**Why do two earthquakes of the same magnitude cause completely different damage?**

In December 2024, a magnitude 6.8 earthquake hit the Kuril Islands. Three people reported feeling it, and the USGS gave it a green alert. In September 2023, an earthquake of the exact same size hit Morocco. It was much shallower, hit a populated area, got a red alert, and thousands of people died.

This project uses real earthquake and country data to find out what actually makes an earthquake dangerous, and to build toward a model that predicts which earthquakes will get a USGS warning alert.

![Magnitude vs shaking](images/magnitude_vs_shaking.png)

## Questions

1. **How much do magnitude and depth tell us about how strong the shaking will be?** (regression)
2. **Can we predict which earthquakes get a yellow, orange, or red alert instead of a green one?** (classification)

## Key findings so far

* **Depth matters more than size for shaking.** Depth has a correlation of **-0.50** with shaking intensity (MMI), compared to **0.36** for magnitude.
* **Deep earthquakes never got a warning.** None of the 53 deep earthquakes (300 km or more) got a warning alert. For shallow earthquakes of magnitude 7.5 and up, **55%** did.
* **Where it hits matters too.** Earthquakes that got a warning hit countries with about **twice the population density** (median 80.7 vs 37.2 people per sq km). A two sample t-test confirms this difference is statistically significant (**p = 0.0018**).
* **Accuracy alone is misleading here.** 85.6% of earthquakes get a green alert, so a model that always says "green" is 85.6% accurate but catches **zero** real warnings. This is why the project focuses on recall.

## Tech stack

| Area | Tools and skills |
|---|---|
| Data collection | Python, `requests`, REST APIs, parsing nested JSON (GeoJSON) |
| Data cleaning | `pandas`: missing values, invalid values, duplicates, type conversion, feature engineering |
| Data merging | Text parsing to extract countries, name matching, left join across two sources |
| Visualization | `matplotlib`: scatter plot, heatmap, map, box plot, residual plots |
| Statistics | Correlation, Welch's two sample t-test (`scipy`), log transforms for skewed data |
| Model evaluation | MAE, RMSE, R², confusion matrix, precision, recall, F1, all written from scratch in `numpy` |

## Data

| Source | What it gives | Rows |
|---|---|---|
| [USGS Earthquake Catalog API](https://earthquake.usgs.gov/fdsnws/event/1/) | Every magnitude 6+ earthquake from 2020 to 2025: magnitude, depth, shaking (MMI), felt reports, PAGER alert | 795 pulled, 701 after cleaning |
| [World Bank API](https://datahelpdesk.worldbank.org/knowledgebase/topics/125589) | Population density and GDP per capita for each country | 217 countries |

Both APIs are free and need no key.

## Pipeline

1. **Collect:** call the USGS API and flatten each GeoJSON feature into one row.
2. **Clean:** remove duplicates, treat MMI values of 0 as missing (the scale starts at 1), drop rows with no shaking estimate or alert, and fix date formats.
3. **Engineer features:** add depth groups (shallow, intermediate, deep, using USGS definitions) and a binary `warning` label.
4. **Match countries:** pull the country out of the USGS place text and fix names so they match the World Bank (for example "Turkey" to "Turkiye").
5. **Merge:** left join earthquakes with World Bank country data. Ocean earthquakes keep their row but have no country values. 616 of 701 earthquakes matched a country.
6. **Explore and test:** build charts, run correlation and a t-test, and test the evaluation functions on baseline models.

## Results

| Warning rate by size and depth | Where the earthquakes hit |
|---|---|
| ![Heatmap](images/warning_rate_heatmap.png) | ![Map](images/earthquake_map.png) |

| Population density, green vs warning | Baseline model check |
|---|---|
| ![Box plot](images/population_density_boxplot.png) | ![Residuals](images/residuals_baseline.png) |

### Baselines

Any real model has to beat these:

| Baseline | Result |
|---|---|
| Always guess average shaking | R² = 0.00 |
| Straight line on magnitude only | R² = 0.13, MAE = 1.23 MMI points |
| Always say "green" | Accuracy 85.6%, recall 0.00 |
| Say "warning" if MMI is 6.5 or higher | Accuracy 89.3%, recall 0.91, precision 0.58 |

## Next steps

* Train regression models (linear regression, k nearest neighbors) using magnitude **and** depth to predict shaking.
* Train classifiers (decision tree, k nearest neighbors) to predict warning alerts using both earthquake and country features.
* Compare every model against the baselines above, with a focus on recall.

## Ethical considerations

* **Uneven data:** felt reports come from people online, so wealthier and more connected regions are overrepresented.
* **Predicting a prediction:** USGS alerts come from their own model (PAGER), so this project explains alert patterns, not real damage.
* **Country averages hide local risk:** a country's average density or income can make a poor rural area look safer than it is.
* **Missed warnings are the worst error:** this project is for learning and research, not real emergency decisions.

## Project structure

```
earthquake-alert-prediction/
├── notebooks/
│   ├── 01_project_proposal.ipynb     # problem, questions, first data pull
│   └── 02_proof_of_concept.ipynb     # cleaning, merging, charts, evaluation functions
├── images/                           # charts used in this README
├── requirements.txt
└── README.md
```

## How to run

```bash
git clone https://github.com/nadim123189/earthquake-alert-prediction.git
cd earthquake-alert-prediction
pip install -r requirements.txt
jupyter notebook notebooks/02_proof_of_concept.ipynb
```

Run all cells. The notebook calls both APIs and saves the cleaned data as `earthquakes_clean.csv`.

## Team

Nadim-Marc El Achkar and Jaden (Minh) Pham, DS 3000, Northeastern University.
