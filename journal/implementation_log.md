# Implementation Log — A2 Sydney House Price Estimation

Hangyu Hu — 14735221

This log records what was built, what went wrong, where an AI assistant was used, and what I do not yet fully understand. The notebook was drafted with Cursor. I am responsible for the task definition, the cleaning rules, and for being able to explain both in the presentation.

---

## Part 1 — Development entries

### 2026-09-30 — Chose the task and checked the file
- What I did: Option 2. The task is to estimate the sale price of one Greater Sydney residential property. Dataset: `domain_properties.csv`, 11,160 Domain.com.au sales from 13 Jan 2016 to 1 Jan 2022, published on Kaggle by alexlau203.
- Problem / question: Is this a real dataset, or a toy table the specification disallows?
- What I tried / how I verified: Printed shape, missing counts, price summary, type counts and year counts. 17 columns, 0 missing values, 637 suburbs, 16 property types, price from 225,000 to 60,000,000 AUD. It is scraped market data, not Iris or Boston Housing.
- Outcome / decision: Use this file. The modelling target is `log(price)` because the raw price histogram is heavily right-skewed (median 1,388,000; max 60,000,000).
- AI tool used?: Cursor proposed the task framing and drafted notebook sections 0–1. I checked the printed numbers against the CSV (11,160 rows, 5,328 sales in 2021, median 1,388,000).
- Open knowledge gap: I do not yet know the publication lag of `property_inflation_index`.

### 2026-09-30 — Put the notebook where Colab can run it
- What I did: Created a public repository https://github.com/AlexHu1210/uts-ml-a2-sydney-house-price and set `DATA_URL` to the raw CSV on that repository.
- Problem / question: Colab cannot read a file on my laptop. The first `git push` was rejected.
- What I tried / how I verified: The error was `Permission denied to AlexHu021210` because the Mac keychain stored a different GitHub account. I created a personal access token for AlexHu1210 and pushed again. Then I opened the notebook in Colab and used Runtime → Run all.
- Outcome / decision: Run all finished with no error and drew the raw-price and log-price histograms. The download URL returns the CSV (1,135,849 bytes).
- AI tool used?: Cursor gave the Git commands and the raw URL. I created the token and ran the push and the Colab run myself.
- Open knowledge gap: none.

### 2026-09-30 — Defined the task and removed rows that are not one dwelling
- What I did: Wrote the training and deployment inputs and outputs in Section 2. Removed non-residential types and physically impossible rows. 11,160 → 10,864 (97.3%).
- Problem / question: Should the 17,000,000 AUD Woollahra sale be deleted as an outlier?
- What I tried / how I verified: Printed the extreme rows. A house with 47 bedrooms and 46 bathrooms, a 7 m² house, and a 20,695 m² apartment are not usable dwellings. The Woollahra sale has 3 bedrooms and 325 m², so it is an expensive real sale, not a typo. Sequential removal counts from `clean()`: type 235, bedrooms 22, bathrooms 1, parking 28, size 10.
- Outcome / decision: Keep residential types only. Keep bedrooms in 0–10, bathrooms in 0–8, parking in 0–8, land size in 20–10,000 m². Do not cap price. Deal with expensive houses by training on `log(price)`.
- AI tool used?: Cursor drafted `clean()`. I am using the printed removal counts above as the check that the function does what the rules say.
- Open knowledge gap: 10,000 m² is a judgement. A genuine outer-suburb lot larger than that would be dropped. I have not found a published cutoff to cite.

### 2026-09-30 — Section 3: split by sale date
- What I did: Train = sales before 2021-01-01 (5,629 rows). Validation = 1 Jan 2021 to 30 Jun 2021 (1,328). Test = on or after 1 Jul 2021 (3,907).
- Problem / question: Why not `train_test_split` with a random shuffle?
- What I tried / how I verified: A random training set of the same size still has about 47% of its rows from 2021 or later. Its median price is about 1.38 M AUD. The time-split training median is about 1.11 M AUD, while validation and test medians are about 1.73 M and 1.65 M AUD.
- Outcome / decision: Use only the time split from here on. The random split is kept as a leakage demonstration.
- AI tool used?: Cursor drafted `split_by_time`. The row counts and medians above are the notebook output.
- Open knowledge gap: I have not yet measured how many percentage points a real model would look better under the random split. That comparison belongs with the first baseline.

### 2026-09-30 — Checked sections 0–3 against the A2 rubric and fixed the gaps
- What I did: Compared the notebook with the criterion A table (OK / Clear levels). Section 2.1 listed column names but not type, unit and range per column, which is the OK level, not Clear.
- Problem / question: How to give the Clear-level specification without typing numbers by hand that could drift from the data.
- What I tried / how I verified: Added Section 2.3, a table generated from the cleaned `sales` frame (dtype and min/max are computed; unit and note are written by me). Bathrooms now top out at 7, land size at 29–10,000 m², price at 272,500–17,000,000 AUD. Also expanded the motivation in 2.1 with how AVMs are used and tested (IAAO AVM standard; Zillow's within-10% reporting) and wrote the research question explicitly.
- Outcome / decision: Input specification is now at the Clear level. Colab link added to README and the notebook header.
- AI tool used?: Cursor did the rubric comparison and drafted the table code and the background paragraph. I checked the census note: the source does not state the census year, so the note says so rather than claiming 2016.
- Open knowledge gap: I have read about the IAAO standard's ratio-based tests only at summary level. I have not read the full standard.

### 2026-09-30 — Section 4: metrics and rule-based baselines
- What I did: Wrote `metrics()` (PPE10, MdAPE, median bias, RMSE in log and in AUD), a `record()` helper that collects every model's scores in one table, and three baselines fitted on the training period only: global median, suburb median, suburb × type median.
- Problem / question: The suburb median scored *worse* than the global median on PPE10 (10.7% vs 21.1%). That looked like a bug.
- What I tried / how I verified: Checked `metrics()` on a hand-made case (+5%, −20%, 0% → PPE10 66.7, MdAPE 5.0). Then looked at median bias: all three rules under-price the test set by about 31–33%. The suburb medians are accurate for 2016–2020 and every one of them is too low for 2021, so almost none land within 10%. The global median (1.11 M) happens to sit near many 2021 sales. Also 148 test rows are in suburbs with no training sales.
- Outcome / decision: Not a bug. It is the market shift measured. Primary metrics are PPE10 and MdAPE; RMSE in AUD is reported but not used for choosing models because the 17 M house dominates it. The leakage check is now quantitative: the same rule on a random split has bias −1.2% and MdAPE 24.1%.
- AI tool used?: Cursor drafted the metric and baseline code. I checked the metric with the hand-made case and reasoned through the suburb-median result above before accepting it.
- Open knowledge gap: I chose 10% for PPE10 because Zillow and the IAAO material use it. I have not checked what tolerance Australian lenders use.

### 2026-09-30 — Section 5: Ridge regression on log(price)
- What I did: Built the shared feature encoding (standardised numerics, log1p land size, one-hot type and suburb; 616 columns). Fitted Ridge on log(price), chose α = 1 on validation, scored test once.
- Problem / question: (1) `get_feature_names_out` failed with "Estimator log1p does not provide get_feature_names_out". (2) Is sklearn's Ridge really solving the loss I wrote down?
- What I tried / how I verified: (1) `FunctionTransformer` needs `feature_names_out="one-to-one"` to pass column names through; added it. (2) Implemented the closed form w = (XᵀX + αI)⁻¹Xᵀy on centred data in numpy and compared to `Ridge(alpha=1).coef_`: largest difference 7e-15.
- Outcome / decision: Test PPE10 33.6%, MdAPE 15.9%, median bias +8.7% (validation +1.7%). The 30% under-pricing of the baselines is gone; the linear extension of year / cash rate / price index into 2021 now overshoots in H2. α barely matters (validation PPE10 within 3 points from 0.01 to 100). Largest weights are rare-suburb indicators.
- AI tool used?: Cursor drafted the pipeline and the section text. The closed-form check is the evidence that the library call matches the formula; the α sweep and coefficient listing are notebook output.
- Open knowledge gap: I know sklearn centres X and y to handle the intercept without penalising it, which is why my centred closed form matches. I have not read how it handles the sparse one-hot matrix internally (it did not need `.toarray()`).

### 2026-09-30 — Section 5.2b: training the Ridge loss by iteration
- What I did: Added hand-written gradient descent and Adam loops on the same Ridge loss, with a loss-curve plot and a comparison to the closed-form weights.
- Problem / question: Gradient descent at the largest stable step (1/L) was still far from the minimum after 3,000 steps in my first test (max weight difference 0.70). I first thought the gradient was wrong.
- What I tried / how I verified: Checked the gradient formula against the closed form (setting it to zero gives the same equation). Then computed the eigenvalues of XᵀX: largest ≈ 15,246, smallest ≈ 0, so the condition number with α = 1 is about 15,000. One learning rate cannot serve both the steep directions (population, income) and the flat ones (rare-suburb indicators). Adam with lr 0.01 reached the closed-form weights (difference < 1e-4) in about 1,500 steps.
- Outcome / decision: Kept both loops in the notebook. GD is the honest picture of what a single step size does on this design matrix; Adam is the optimiser used for the MLP later, and now it is written out rather than imported.
- AI tool used?: Cursor drafted the loops. The convergence numbers and the conditioning explanation come from running them and from the eigenvalue check.
- Open knowledge gap: I have not derived why Adam's bias correction terms (1 − β^t) are needed; I know they undo the zero initialisation of m and v but have not worked through the expectation argument.

---

## Part 2 — Use of AI tools

| Where | What the AI produced | How I checked it / what I changed |
|-------|----------------------|-----------------------------------|
| Notebook sections 0–1 | Environment setup, download-with-local-fallback, summary tables, histograms | Ran in Colab (Run all, no error). Year counts and the price summary match a direct read of the CSV. |
| Notebook section 1, first draft | An unused variable `by_half_year` | Deleted. It was computed and never used. |
| Notebook section 2 | `clean()` and the training/deployment tables | Ran locally. Removal counts are 235, 22, 1, 28, 10. Kept the 17 M AUD house because its bedroom count and land size are plausible. |
| Notebook section 5 | Feature pipeline, Ridge fit, α sweep | Fixed the `feature_names_out` error myself after reading the traceback. Verified the solver against the closed form (7e-15). |
| This log and section 1.3 | First draft of the observation bullets and the log entries | Rewrote section 1.3 in the first person and tied every bullet to a printed number. The log states which parts Cursor wrote. |

---

## Part 3 — Knowledge gaps

| Component | Why it is needed | How I checked it | What I still do not understand |
|-----------|------------------|------------------|--------------------------------|
| `pd.to_datetime(..., format="%d/%m/%y")` | Sale dates are day/month/year (`13/1/16`). A wrong parse would put sales in the wrong year and break the time split. | The printed range is 2016-01-13 to 2022-01-01, which matches the first and last rows of the file. | I have not tried the same call without `format` to see the wrong parse myself. |
| `property_inflation_index` | It may let the model follow the 2020–2021 price jump. | I only checked that the column changes by year (about 166 in early 2020, 220.1 in late 2021). | I have not confirmed when each quarterly value was published, so I do not know if it is a legal input on the day of the sale. |
| Land-size cap of 10,000 m² | Drops the 7 m² house and the 20,695 m² apartment. | Those two rows are visible in the extremes and are removed by `property_size.between(20, 10000)`. | I do not have an external source for 10,000 rather than, say, 5,000. |
