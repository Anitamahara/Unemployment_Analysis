# CodeAlpha – Car Price Prediction with Machine Learning

**Task 3 of the CodeAlpha Data Science Internship.**

This project predicts the resale price of a used car from its showroom price, age, kilometres driven, fuel type, seller type, transmission, number of previous owners and brand. It compares four regression models and explains why the simplest one generalised best.

---

## Dataset

`car data.csv`: the CarDekho used-vehicle dataset linked in the CodeAlpha task sheet (301 rows, 9 columns).
./car data.csv

| Column | Meaning |
|---|---|
| `Car_Name` | Model name (used to extract the brand) |
| `Year` | Year of manufacture |
| `Selling_Price` | **Target**: resale price (lakhs) |
| `Present_Price` | Current showroom price of the same model (lakhs) |
| `Driven_kms` | Kilometres driven |
| `Fuel_Type` | Petrol / Diesel / CNG |
| `Selling_type` | Dealer / Individual |
| `Transmission` | Manual / Automatic |
| `Owner` | Number of previous owners |

---

## Project workflow

### 1. Data cleaning
- No missing values were found.
- **2 duplicate rows were removed** (301 → 299 rows), so the same car cannot appear in both the training and the test set.

### 2. Exploratory data analysis
- `Selling_Price` is **right-skewed**: the mean is about 4.6 lakhs, but the maximum is 35 lakhs.
- `Present_Price` has the strongest correlation with `Selling_Price` (about 0.88).
- Diesel cars sell for more than petrol cars, automatics for more than manuals, and dealer listings for more than individual sales.

### 3. Feature engineering
| New feature | How it was made | Why |
|---|---|---|
| `Car_Age` | 2026 − `Year` | Age is a more direct signal of depreciation than the raw year |
| `Brand` | First word of `Car_Name`; brands with fewer than 3 cars grouped as `other` | Captures brand goodwill without overfitting to rare names |
| `Log_Kms` | log(1 + `Driven_kms`) | Reduces skew in kilometres driven |
| `Log_Price` (target) | log(1 + `Selling_Price`) | Reduces skew, so expensive cars do not dominate training |

Text columns were **one-hot encoded** (`pd.get_dummies`, `drop_first=True`), giving **39 input features**.

### 4. Train / test split
- 80% training (239 cars), 20% test (60 cars), `random_state=42`.
- The test set was used only once per model, for the final evaluation.

### 5. Models and evaluation
Four models were trained. Each was scored with 5-fold cross-validation on the training set, and then on the test set. Predictions were converted back from log price to lakhs before calculating the test metrics.

| Model | CV R² (log scale) | Test R² | MAE (lakhs) | RMSE (lakhs) |
|---|---|---|---|---|
| **Linear Regression** | 0.877 | **0.929** | **0.765** | 1.353 |
| Lasso Regression | 0.855 | 0.930 | 0.802 | 1.342 |
| Gradient Boosting | 0.972 | 0.815 | 0.964 | 2.186 |
| Random Forest | 0.971 | 0.636 | 1.279 | 3.062 |

**Chosen model: Linear Regression.** It is tied with Lasso for the best test R², has the lowest MAE, and is the simplest to explain. On average, its predictions are off by about **0.77 lakhs**.

---

## Key findings

### Why the tree models failed on the test set
Random Forest and Gradient Boosting scored highest in cross-validation, but worst on the test set. Their RMSE was much larger than their MAE, which means a few cars got very wrong predictions.

Inspecting the largest errors showed the cause. All three of Random Forest's biggest mistakes were **old cars that were expensive when new** (16–20 years old, showroom price around 23 lakhs). They actually sold for 2.5–5 lakhs, but Random Forest predicted 14–17 lakhs.

Tree models predict by averaging similar training cars, and the training data contains very few cars that are both old and originally expensive. Linear Regression on log price learns one rule for every car ("each year reduces the price by about X%"), so it handled these cars correctly. **With a small dataset, a simpler model that learns general rules generalised better.**

### What drives the price (Linear Regression)
The model predicts log price, so each coefficient is converted into a % change in price (other features held constant):

| Feature | Effect on price |
|---|---|
| Each extra year of age | **−7.0%** |
| Each extra lakh of showroom price | **+3.1%** |
| Manual (vs automatic) | **−7.3%** |
| Sold by an individual (vs dealer) | **−25.4%** |
| Diesel (vs petrol) | about **+7%** |
| Doubling the kilometres driven | about **−1%** |

---

## Limitations
- **The data reflects about 2019 prices.** Car age was calculated from 2026, which adds the same offset to every car. This does not affect the model's accuracy, but its predictions reflect the market when the data was collected, not today.
- **Small categories are unreliable.** Only about 10 cars have a previous owner and only 2 use CNG, so the effects of `Owner` and CNG cannot be trusted. For example, the model gives `Owner` a small positive effect, which is not realistic.
- **Possible confounding.** The dataset also includes motorbikes, which are mostly sold by individuals. Part of the "individual seller" effect may really be a "bikes are cheaper" effect.
- **Small dataset.** With only 299 rows, the scores can change noticeably with a different train/test split.

---

## How to run
```bash
pip install pandas numpy scikit-learn matplotlib seaborn
```
1. Download `car data.csv` from the CodeAlpha task sheet and put it in this folder.
2. Open `Car_prediction.ipynb` in Jupyter or VS Code and choose **Run All**.

## Tools
Python · Pandas · NumPy · Scikit-learn · Matplotlib · Seaborn · Jupyter Notebook

## Author
Prabin Rokaya — CodeAlpha Data Science Intern