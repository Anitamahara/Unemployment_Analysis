# CodeAlpha – Unemployment Analysis with Python

**Task 2 of the CodeAlpha Data Science Internship.**

This project analyses unemployment in India from May 2019 to October 2020. It measures the impact of the Covid-19 lockdown nationally, by state, by zone, and for rural vs urban areas. It also checks whether the recovery after the lockdown was real, and finishes with policy recommendations.

---

## Datasets

Both files come from the "Unemployment in India" dataset linked in the CodeAlpha task sheet.

| File | Period | Rows (after cleaning) | Extra information |
|---|---|---|---|
| `Unemployment in India.csv` | May 2019 – Jun 2020 | 740 | Rural / Urban split, 28 states |
| `Unemployment_Rate_upto_11_2020.csv` | Jan 2020 – Oct 2020 | 267 | Zone (North, South, East, West, Northeast) |

Main columns: state, date, estimated unemployment rate (%), estimated employed, estimated labour participation rate (%).

---

## Data cleaning

- **Hidden spaces in column names.** For example, `' Date'` instead of `'Date'`. These were removed with `str.strip()`, and long names were renamed (e.g. `Unemployment_Rate`).
- **Hidden spaces in values.** These would have split one category into two (`' Monthly'` vs `'Monthly'`), so text values were stripped too.
- **28 completely empty rows were removed** (768 → 740).
- **Dates were stored as text.** They were converted with `pd.to_datetime(dayfirst=True)`. The day-first setting matters because the dates are written day-month-year.
- **No duplicate rows** were found.
- A reusable `load_and_clean()` function applies the same cleaning to both files.

---

## Method: a weighted national rate

A simple average of state rates treats a small state the same as Uttar Pradesh, so it is **not** India's unemployment rate. Instead, the labour force and number of unemployed people were reconstructed for each row:

- Labour force = Employed ÷ (1 − rate / 100)
- Unemployed = Labour force − Employed
- **National rate = total unemployed ÷ total labour force**

The simple average was about 2 points **higher** than the weighted rate before Covid. That means smaller states had higher unemployment than the large ones.

---

## Findings

### 1. National impact of the lockdown
- Before Covid, the national rate was about **7–8%**.
- It peaked at about **24% in April 2020** and stayed at **23.6% in May**, roughly **3 times** the normal level.
- It recovered to **11.0% in June** once the phased Unlock began, but was still above the pre-Covid level.

### 2. Hardest-hit states (lockdown vs before Covid, percentage points)

| State | Before Covid | Lockdown | Increase |
|---|---|---|---|
| Puducherry* | 1.87% | 75.56% | +73.7 |
| Jharkhand | 11.82% | 53.18% | +41.4 |
| Tamil Nadu | 2.83% | 41.91% | +39.1 |
| Bihar | 11.46% | 46.40% | +34.9 |
| Karnataka | 3.28% | 25.07% | +21.8 |

\*Puducherry is a small territory with a small survey sample, so its figure may be exaggerated.

Two different patterns stand out:
- **Industrial and service states** (Tamil Nadu, Karnataka) had very low unemployment before Covid, but factories, construction and services shut down completely.
- **Migrant home states** (Jharkhand, Bihar) already had high unemployment, and then migrant workers returned from the cities with no jobs waiting.

At the other end, **Chhattisgarh, Tripura and Sikkim** barely changed, and **Jammu & Kashmir decreased**. J&K's "before" period was likely already disrupted by its own lockdown and communication shutdown from August 2019, so its baseline was not normal.

### 3. Recovery by zone (Jan – Oct 2020)

| Zone | Feb (before) | Apr (peak) | Oct (after) |
|---|---|---|---|
| East | 7.17% | **32.52%** | **8.94%** |
| South | 4.87% | 28.64% | 3.14% |
| North | **12.43%** | 20.82% | **11.46%** |
| Northeast | 8.08% | 15.47% | 4.10% |
| West | 5.43% | 16.51% | 4.11% |

- **The East was hit hardest, and is the only zone still above its pre-Covid level.** It includes Bihar, Jharkhand, West Bengal and Odisha, which matches the migrant-state finding above.
- **The South had the second-biggest spike, but recovered best.**
- **The North had the highest rate both before and after Covid**, which points to a long-term problem rather than a Covid one.

### 4. The recovery was only partly real
The labour participation rate (people working **or** looking for work) fell from **44.18% in February to 35.30% in April**. By October it had recovered only to **41.44%**, still about 2.7 points (about 6%) below February.

People who stop looking for work are not counted as unemployed. So part of the fall in unemployment after the lockdown came from people **leaving the workforce**, not from people finding jobs. The unemployment rate alone understates the real damage.

### 5. Long-term (structural) unemployment
Some states had high unemployment in **every month, even before Covid**: Tripura (about 23–33%), Haryana (about 18–29%), Himachal Pradesh (about 13–23%) and Jammu & Kashmir (about 11–21%).

### 6. Rural vs urban

| Period | Rural | Urban |
|---|---|---|
| Before Covid | _fill in_ | _fill in_ |
| March 2020 | _fill in_ | _fill in_ |
| Lockdown | _fill in_ | _fill in_ |
| Unlock (June) | _fill in_ | _fill in_ |

_Fill in from the Step 5b table in the notebook, and add one sentence on which area was hit harder and which recovered faster._

### 7. Seasonal patterns
With only about 14 months of data, each calendar month appears only once, or twice in months that were distorted by Covid. **True seasonal patterns cannot be confirmed from this dataset.** Confirming them would need several years of data.

The national rate does zigzag up and down every other month before Covid. The heatmap shows this clearly in Chandigarh, Himachal Pradesh and Rajasthan. A real economy does not swing like this every month, so the pattern most likely comes from the survey interviewing different groups of households in alternating months.

---

## Data quality notes
- **April 2020 data looks disrupted.** Some states show their lowest rate in the worst lockdown month: Himachal Pradesh 2% (May: 28%), Chhattisgarh 3%, Sikkim 2%, Punjab 3% (May: 34%). Several states also have missing months during the lockdown. The survey itself was likely affected by the lockdown it was measuring.
- **Small regions are less reliable**, because they have small survey samples.
- **The every-other-month zigzag** is most likely a survey artifact, as described above.

---

## Policy recommendations

| Finding | Recommendation |
|---|---|
| Unemployment tripled within weeks of the lockdown | Prepare emergency income support that can start immediately when a crisis begins |
| Migrant home states were hit hard and the East had still not recovered by October | Expand local employment programmes in these states, such as rural job schemes, so returning workers have work nearby |
| Industrial states spiked sharply but recovered fastest | Support businesses during shutdowns, for example with wage subsidies, so workers stay employed and return quickly |
| Labour participation stayed about 6% below pre-Covid | Track participation alongside unemployment, and include people who stopped searching in support programmes |
| Some states had very high unemployment even before Covid | Long-term programmes (skills training, industry investment) are needed, not only crisis relief |
| Crisis-period survey data was disrupted | Interpret crisis data with caution, and invest in more robust data collection |

---

## How to run
```bash
pip install pandas numpy matplotlib seaborn
```
Open `Unemployment_Analysis.ipynb` in Jupyter or VS Code and choose **Run All**. Both CSV files are included in this repository.

## Tools
Python · Pandas · NumPy · Matplotlib · Seaborn · Jupyter Notebook

## Author
_Your name_ — CodeAlpha Data Science Intern