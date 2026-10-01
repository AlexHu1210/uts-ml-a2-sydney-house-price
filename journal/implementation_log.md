# Implementation Log — A2 Sydney House Price Estimation

Hangyu Hu — 14735221

This log records what was built, what went wrong, where an AI assistant was used, and what I do not yet fully understand. ChatGPT was used as an implementation tool. I made the decisions about the task, the data, the cleaning rules, the split, the metrics and the experiments; the AI turned those decisions into code and first-draft text, which I ran and checked. I am responsible for every design decision and for being able to explain it in the presentation.

---

## Part 1 — Development entries

### 2026-09-30 — Chose the task and checked the file
- What I did: Option 2. The task is to estimate the sale price of one Greater Sydney residential property. Dataset: `domain_properties.csv`, 11,160 Domain.com.au sales from 13 Jan 2016 to 1 Jan 2022, published on Kaggle by alexlau203.
- Problem / question: Is this a real dataset, or a toy table the specification disallows?
- What I tried / how I verified: Printed shape, missing counts, price summary, type counts and year counts. 17 columns, 0 missing values, 637 suburbs, 16 property types, price from 225,000 to 60,000,000 AUD. It is scraped market data, not Iris or Boston Housing.
- Outcome / decision: Use this file. The modelling target is `log(price)` because the raw price histogram is heavily right-skewed (median 1,388,000; max 60,000,000).
- AI tool used?: ChatGPT proposed the task framing and drafted notebook sections 0–1. I checked the printed numbers against the CSV (11,160 rows, 5,328 sales in 2021, median 1,388,000).
- Open knowledge gap: I do not yet know the publication lag of `property_inflation_index`.

### 2026-09-30 — Put the notebook where Colab can run it
- What I did: Created a public repository https://github.com/AlexHu1210/uts-ml-a2-sydney-house-price and set `DATA_URL` to the raw CSV on that repository.
- Problem / question: Colab cannot read a file on my laptop. The first `git push` was rejected.
- What I tried / how I verified: The error was `Permission denied to AlexHu021210` because the Mac keychain stored a different GitHub account. I created a personal access token for AlexHu1210 and pushed again. Then I opened the notebook in Colab and used Runtime → Run all.
- Outcome / decision: Run all finished with no error and drew the raw-price and log-price histograms. The download URL returns the CSV (1,135,849 bytes).
- AI tool used?: ChatGPT gave the Git commands and the raw URL. I created the token and ran the push and the Colab run myself.
- Open knowledge gap: none.

### 2026-09-30 — Defined the task and removed rows that are not one dwelling
- What I did: Wrote the training and deployment inputs and outputs in Section 2. Removed non-residential types and physically impossible rows. 11,160 → 10,864 (97.3%).
- Problem / question: Should the 17,000,000 AUD Woollahra sale be deleted as an outlier?
- What I tried / how I verified: Printed the extreme rows. A house with 47 bedrooms and 46 bathrooms, a 7 m² house, and a 20,695 m² apartment are not usable dwellings. The Woollahra sale has 3 bedrooms and 325 m², so it is an expensive real sale, not a typo. Sequential removal counts from `clean()`: type 235, bedrooms 22, bathrooms 1, parking 28, size 10.
- Outcome / decision: Keep residential types only. Keep bedrooms in 0–10, bathrooms in 0–8, parking in 0–8, land size in 20–10,000 m². Do not cap price. Deal with expensive houses by training on `log(price)`.
- AI tool used?: ChatGPT drafted `clean()`. I am using the printed removal counts above as the check that the function does what the rules say.
- Open knowledge gap: 10,000 m² is a judgement. A genuine outer-suburb lot larger than that would be dropped. I have not found a published cutoff to cite.

### 2026-09-30 — Section 3: split by sale date
- What I did: Train = sales before 2021-01-01 (5,629 rows). Validation = 1 Jan 2021 to 30 Jun 2021 (1,328). Test = on or after 1 Jul 2021 (3,907).
- Problem / question: Why not `train_test_split` with a random shuffle?
- What I tried / how I verified: A random training set of the same size still has about 47% of its rows from 2021 or later. Its median price is about 1.38 M AUD. The time-split training median is about 1.11 M AUD, while validation and test medians are about 1.73 M and 1.65 M AUD.
- Outcome / decision: Use only the time split from here on. The random split is kept as a leakage demonstration.
- AI tool used?: ChatGPT drafted `split_by_time`. The row counts and medians above are the notebook output.
- Open knowledge gap: I have not yet measured how many percentage points a real model would look better under the random split. That comparison belongs with the first baseline.

### 2026-09-30 — Checked sections 0–3 against the A2 rubric and fixed the gaps
- What I did: Compared the notebook with the criterion A table (OK / Clear levels). Section 2.1 listed column names but not type, unit and range per column, which is the OK level, not Clear.
- Problem / question: How to give the Clear-level specification without typing numbers by hand that could drift from the data.
- What I tried / how I verified: Added Section 2.3, a table generated from the cleaned `sales` frame (dtype and min/max are computed; unit and note are written by me). Bathrooms now top out at 7, land size at 29–10,000 m², price at 272,500–17,000,000 AUD. Also expanded the motivation in 2.1 with how AVMs are used and tested (IAAO AVM standard; Zillow's within-10% reporting) and wrote the research question explicitly.
- Outcome / decision: Input specification is now at the Clear level. Colab link added to README and the notebook header.
- AI tool used?: ChatGPT did the rubric comparison and drafted the table code and the background paragraph. I checked the census note: the source does not state the census year, so the note says so rather than claiming 2016.
- Open knowledge gap: I have read about the IAAO standard's ratio-based tests only at summary level. I have not read the full standard.

### 2026-09-30 — Section 4: metrics and rule-based baselines
- What I did: Wrote `metrics()` (PPE10, MdAPE, median bias, RMSE in log and in AUD), a `record()` helper that collects every model's scores in one table, and three baselines fitted on the training period only: global median, suburb median, suburb × type median.
- Problem / question: The suburb median scored *worse* than the global median on PPE10 (10.7% vs 21.1%). That looked like a bug.
- What I tried / how I verified: Checked `metrics()` on a hand-made case (+5%, −20%, 0% → PPE10 66.7, MdAPE 5.0). Then looked at median bias: all three rules under-price the test set by about 31–33%. The suburb medians are accurate for 2016–2020 and every one of them is too low for 2021, so almost none land within 10%. The global median (1.11 M) happens to sit near many 2021 sales. Also 148 test rows are in suburbs with no training sales.
- Outcome / decision: Not a bug. It is the market shift measured. Primary metrics are PPE10 and MdAPE; RMSE in AUD is reported but not used for choosing models because the 17 M house dominates it. The leakage check is now quantitative: the same rule on a random split has bias −1.2% and MdAPE 24.1%.
- AI tool used?: ChatGPT drafted the metric and baseline code. I checked the metric with the hand-made case and reasoned through the suburb-median result above before accepting it.
- Open knowledge gap: I chose 10% for PPE10 because Zillow and the IAAO material use it. I have not checked what tolerance Australian lenders use.

### 2026-09-30 — Section 5: Ridge regression on log(price)
- What I did: Built the shared feature encoding (standardised numerics, log1p land size, one-hot type and suburb; 616 columns). Fitted Ridge on log(price), chose α = 1 on validation, scored test once.
- Problem / question: (1) `get_feature_names_out` failed with "Estimator log1p does not provide get_feature_names_out". (2) Is sklearn's Ridge really solving the loss I wrote down?
- What I tried / how I verified: (1) `FunctionTransformer` needs `feature_names_out="one-to-one"` to pass column names through; added it. (2) Implemented the closed form w = (XᵀX + αI)⁻¹Xᵀy on centred data in numpy and compared to `Ridge(alpha=1).coef_`: largest difference 7e-15.
- Outcome / decision: Test PPE10 33.6%, MdAPE 15.9%, median bias +8.7% (validation +1.7%). The 30% under-pricing of the baselines is gone; the linear extension of year / cash rate / price index into 2021 now overshoots in H2. α barely matters (validation PPE10 within 3 points from 0.01 to 100). Largest weights are rare-suburb indicators.
- AI tool used?: ChatGPT drafted the pipeline and the section text. The closed-form check is the evidence that the library call matches the formula; the α sweep and coefficient listing are notebook output.
- Open knowledge gap: I know sklearn centres X and y to handle the intercept without penalising it, which is why my centred closed form matches. I have not read how it handles the sparse one-hot matrix internally (it did not need `.toarray()`).

### 2026-09-30 — Section 5.2b: training the Ridge loss by iteration
- What I did: Added hand-written gradient descent and Adam loops on the same Ridge loss, with a loss-curve plot and a comparison to the closed-form weights.
- Problem / question: Gradient descent at the largest stable step (1/L) was still far from the minimum after 3,000 steps in my first test (max weight difference 0.70). I first thought the gradient was wrong.
- What I tried / how I verified: Checked the gradient formula against the closed form (setting it to zero gives the same equation). Then computed the eigenvalues of XᵀX: largest ≈ 15,246, smallest ≈ 0, so the condition number with α = 1 is about 15,000. One learning rate cannot serve both the steep directions (population, income) and the flat ones (rare-suburb indicators). Adam with lr 0.01 reached the closed-form weights (difference < 1e-4) in about 1,500 steps.
- Outcome / decision: Kept both loops in the notebook. GD is the honest picture of what a single step size does on this design matrix; Adam is the optimiser used for the MLP later, and now it is written out rather than imported.
- AI tool used?: ChatGPT drafted the loops. The convergence numbers and the conditioning explanation come from running them and from the eigenvalue check.
- Open knowledge gap: I have not derived why Adam's bias correction terms (1 − β^t) are needed; I know they undo the zero initialisation of m and v but have not worked through the expectation argument.

### 2026-09-30 — Section 6: LightGBM is worse than Ridge on the 2021 test set
- What I did: Fitted LightGBM on the same information as Ridge (raw numerics, `type` and `suburb` as categorical), squared error on log price, early stopping on validation, num_leaves chosen on validation (15). Test PPE10 22.5%, MdAPE 20.1%, median bias −18.2%. Ridge had 33.6% / 15.9% / +8.7%.
- Problem / question: The "stronger" model lost by 11 PPE10 points. Bug, or real?
- What I tried / how I verified: (1) Overwrote the test rows' `year`, `cash_rate`, `property_inflation_index` with end-of-2020 values and re-predicted: the largest change in any prediction was 0.0000. The trees price 2021 exactly like late 2020. Training range of the index is 150.9–183.1; test is 183.1–220.1, all on one side of every split. (2) Fitted both models on sales before July 2020 and scored on 2020 H2 (inside the training range): LightGBM 43.7% PPE10 / 12.0% MdAPE vs Ridge 37.1% / 14.5%. So the tree is better when it does not have to extrapolate.
- Outcome / decision: Real, and it is the central finding so far. Trees cannot extrapolate a time trend; a linear model can but overshoots. The fix is to take the trend out of the target (price relative to the price index) — a preview run gave 38.7% PPE10 on test with −3.9% bias. That becomes Section 8. Left the honest LightGBM row in the results table.
- AI tool used?: ChatGPT drafted the section. The two diagnostic checks were run and read by me; the 0.0000 result is what convinced me it was extrapolation and not a coding error.
- Open knowledge gap: I understand LightGBM's categorical handling only at the level of "it groups category values at a split"; I have not read how it orders categories by gradient statistics to find the grouping.

### 2026-09-30 — Section 7: MLP in PyTorch, five seeds
- What I did: 616 → 128 → 64 → 1 ReLU network (87,297 parameters), squared error on log price, mini-batch Adam, dropout 0.1, early stopping on validation loss. Trained with seeds 0–4.
- Problem / question: In a first sweep, configurations that were within 1 point of each other on validation were up to 10 points apart on test (27% to 38% PPE10). Which number is "the" MLP result?
- What I tried / how I verified: Fixed one configuration and varied only the seed. Validation PPE10 35.3 ± 0.9; test PPE10 33.7 ± 2.8; test bias 6.2 ± 4.4 (range +1.8% to +12.7%). The best epoch is 5–12, after which validation loss rises: the network overfits fast on 5,629 rows.
- Outcome / decision: Report mean ± std, and carry the seed with the best *validation* PPE10 (seed 2) into the results table. Test PPE10 33.6%, MdAPE 15.7%, bias +7.6% — same level as Ridge. The network does extrapolate (2021 macro inputs move its predictions by about 26%), which confirms the Section 6 diagnosis: only the tree class is flat outside its range.
- AI tool used?: ChatGPT drafted the training loop. I checked that the loop matches Section 5.2b (forward, loss, backward, Adam step) and that early stopping keeps the best weights rather than the last.
- Open knowledge gap: I do not know why the seed-to-seed spread on *test* is three times the spread on *validation*. My guess is that the extrapolation direction is decided by weights that the validation loss barely constrains, but I have not tested it.

### 2026-09-30 — Section 8: ablations and the final model
- What I did: One harness, `run_ablation`, that trains LightGBM or Ridge with a chosen feature list and target. Experiments: A raw vs log target; B drop macro, then also time inputs; C target = log(price) − log(index) with the index added back; C′ the same with the index lagged one quarter; D drop the suburb categorical; E random split.
- Problem / question: Does the fix for the tree model's extrapolation problem come from the model or from the target? And is the index a legal input on the day of sale?
- What I tried / how I verified: C lifts LightGBM from 22.5% to 37.7% PPE10 and Ridge from 33.6% to 37.3%; both models end within half a point of each other, so the target is what mattered. C′ (index from 91 days earlier, ratio to same-quarter index 1.036 in the test period) costs 1–2 points and 2–3 points of bias, so the fix survives realistic publication lag. D: removing `suburb` costs Ridge 7 points and LightGBM nothing (validation 39.0%, the highest of all rows) — the suburb-level numerics already locate the property for a tree. E: random split shows 42.3% vs 37.7%, about 5 points of inflation.
- Outcome / decision: Final model chosen on validation PPE10: LightGBM, index-adjusted target, property and suburb numerics, `type` categorical, no suburb categorical, no time inputs (620 trees). Test 37.9% / 13.4% / −4.7%. Reported alongside it: the lagged-index deployment score 35.7% / 14.5% / −7.9%.
- AI tool used?: ChatGPT drafted the harness. I read every row of the table before writing the interpretation; the surprising rows (D for LightGBM, Ridge recovering under C) are the ones I checked twice.
- Open knowledge gap: The remaining −4.7% bias under C means the index rose less than this segment's prices in 2021 H2. I have not checked which index this column is (city-wide? all dwellings?) to explain the gap.

### 2026-09-30 — Sections 9 and 10: loss vs objective, quantiles, error analysis, conclusion
- What I did: (9) Wrote out why PPE10 cannot be a training loss (zero gradient, ignores miss size) and measured how the squared-log loss and PPE10 disagree. Trained the final configuration with four LightGBM losses (L2, L1, Huber, Fair). Trained quantile models (τ = 0.1–0.9) with the pinball loss and evaluated calibration and a lender's over-valuation rate. (10) Cut the final model's test errors by price band, distance, type and unseen suburb; listed the worst misses and feature gain; wrote limitations, future work and the conclusion.
- Problem / question: Is the loss/objective mismatch real or academic here?
- What I tried / how I verified: The worst 5% of test sales carry 41.8% of the squared-log loss and none of them are inside 10%. Swapping the loss moves PPE10 by under a point (37.3–38.2%) and the best loss by RMSE-log (L2) is not the best by PPE10 (Fair). Quantiles: 70% coverage for the nominal 80% band, all quantiles shifted down with the point estimate. q25 cuts over-valuation by >10% from 23.6% to 9.7% at a cost of 5 PPE10 points.
- Outcome / decision: Selection stays on PPE10/MdAPE; production should monitor median bias by month; lenders should use a quantile. Error analysis: mid-market (0.8–2 M) 41–46% PPE10, >4 M only 11% with −32% bias, apartments +7%, outer ring −12%. The single city-wide index is the main structural limitation; the worst individual misses are 7–10 bedroom "houses" that passed cleaning.
- AI tool used?: ChatGPT drafted the code and text. I checked the calibration table (share below q_τ should equal τ) and the lender table by hand-reading a few rows before accepting the interpretation.
- Open knowledge gap: I have not proven that the pinball loss minimiser is the τ-quantile; I know the statement and checked it empirically (q10 has 10.9% of prices below it). I also do not know how LightGBM's quantile objective handles leaves with few samples.

### 2026-09-30 — Section 11: deployment function
- What I did: Wrote `estimate_price(num_bed, num_bath, num_parking, property_size, type, suburb, price_index)`. It validates the inputs against the Section 2.3 ranges, looks up the suburb-level figures from a table built on the data, applies `final_model` and the τ = 0.1/0.25/0.5/0.9 quantile models, and returns a point estimate, an 80% interval and a q25 lending value. Demo on four made-up properties and three real test-period sales.
- Problem / question: The specification said the deployment interface existed, but nothing in the notebook could actually be called with one property. Also, the quantile models from Section 9 had not been kept.
- What I tried / how I verified: Stored the quantile models in `quantile_models`. Checked that a 47-bedroom input is rejected with a ValueError, that the Mosman estimate (5.9 M) sits far above Mount Druitt (0.76 M), and that the three real sales fall inside or near their intervals.
- Outcome / decision: The deployment interface of Section 2.1 now has code behind it, and the presentation can include a live call.
- AI tool used?: ChatGPT drafted the function. I checked the validation branch and the sorted-quantile guard against crossing.
- Open knowledge gap: none new.

### 2026-09-30 — Full check of the notebook against the A2 specification
- What I did: Went through the specification (submission rules, criteria A/B/C, clarifications) item by item against the notebook, the log and the repository.
- Problem / question: Three gaps. (1) Section 11 said the suburb lookup table was "built on training data"; the code uses all rows. (2) Criterion B asks for a system overview of the data flow; the notebook had no single place that gave it. (3) Criterion C (Fair) asks for statistical reporting; the final model and baselines were single point values.
- What I tried / how I verified: (1) The suburb table holds static census figures and coordinates, not prices, so using all rows is not label leakage; corrected the sentence rather than the code. (2) Added a "System overview" block before Section 2 with the data flow and section references. (3) Added a 2,000-sample bootstrap of the test set: final model PPE10 95% CI [36.4, 39.4], baseline [19.9, 22.4], gain 16.7 points with CI [14.9, 18.6].
- Outcome / decision: Notebook content complete for criteria A, B, C. Remaining deliverables: the report PDF (with plain-text notebook link and this log), optional slides, and a fresh Colab Run-all of the final notebook.
- AI tool used?: ChatGPT did the item-by-item comparison and drafted the additions. I decided that the suburb-table issue was a wording error and not a leakage error.
- Open knowledge gap: none new.

### 2026-09-30 — Section 11.3: interactive estimator with a year selector
- What I did: Added ipywidgets controls (suburb search box, type, room sliders, land size, year 2016–2030, assumed market growth) that call `estimate_price()` and draw the estimate path over the years, with the q10–q90 band and every recorded sale of the same type in that suburb.
- Problem / question: The deployment function existed but a reader had to write a Python call to try a property, and the natural question "what will it be worth in year X" needs a market level for year X, which Section 8 showed no model here can learn.
- What I tried / how I verified: For 2016–2021 the year selects the observed yearly median index; after 2021 the last value is extended at a growth rate the user sets (default: the index's own 2016–2021 average, 6.7%/yr). The projection is labelled as an assumption and drawn dashed. Rendered two cases offline (Parramatta house 2026; Chatswood apartment 2019) to check the chart. Because the model output is index-free, one prediction per property is enough; the year only rescales it.
- Outcome / decision: Kept the printed demos in 11.1–11.2 because widgets do not render on a static GitHub view. Added a pointer in the notebook header and README.
- AI tool used?: ChatGPT wrote the widget layout. I decided that future years must be shown as a scenario with an explicit growth assumption rather than as a forecast, consistent with the research question's answer.
- Open knowledge gap: none new.

---

## Part 2 — Use of AI tools

I used ChatGPT as an implementation tool during development. I made the decisions about the project topic, dataset selection, problem definition, cleaning rules, split strategy, evaluation metrics, and which experiments to run. AI was used to turn those decisions into code more quickly, fix errors, and draft explanatory text. I ran and checked every code section, and all numerical results in the notebook, this log and the report come from my own notebook runs.

| Area | What I did | Where AI was used |
|------|------------|-------------------|
| Topic and data | Selected Option 2 and the Sydney house-price task; confirmed that the dataset is real market data rather than a toy dataset; decided to train on log(price) | Assisted with the code for the dataset overview |
| Task definition and cleaning | Defined training- and deployment-stage inputs and outputs; decided which types are outside the "single residential property" scope; set the limits for bedrooms, bathrooms, parking and area; decided to keep the genuine AUD 17 M sale | Implemented `clean()` and the input-specification table according to my rules |
| Data split and metrics | Decided on a chronological split rather than a random one and required a leakage comparison; selected PPE10 and MdAPE as primary metrics | Implemented the split and metric functions; I verified `metrics()` on hand-constructed examples |
| Models | Decided to compare Ridge, LightGBM and an MLP, and required hand-written gradient descent and Adam next to the closed-form Ridge solution to show the training process | Drafted the model and training-loop code; I checked agreement with sklearn (7e-15) and that early stopping keeps the best weights |
| Diagnosis and improvement | After LightGBM lost to Ridge, decided to run two diagnostics (macro-value replacement, within-range re-test) to confirm an extrapolation problem rather than a bug; then decided on the index-adjusted target and the lagged-index test | Implemented the ablation harness; I read the ablation table row by row and re-checked the unusual rows |
| Evaluation and deployment | Decided to include the loss-function comparison, quantile intervals, error cuts by price/distance/type, bootstrap intervals, a callable `estimate_price()` and the interactive estimator; required future years to be labelled as scenarios | Implemented these; I checked the calibration table, the input validation and the sorted-quantile guard |
| Documentation | The structure of this log and the report, the discussion, the limitations and the knowledge gaps are in my own words | Drafted parts of the explanatory text, which I rewrote; helped tidy the formatting of the report |

---

## Part 3 — Knowledge gaps

Components I used, can say why they are needed, and checked that they behave as expected, but do not yet fully understand.

| Component | Why it is needed | How I checked it | What I still do not understand |
|-----------|------------------|------------------|--------------------------------|
| `pd.to_datetime(..., format="%d/%m/%y")` | Sale dates are day/month/year (`13/1/16`). A wrong parse would put sales in the wrong year and break the time split. | The printed range is 2016-01-13 to 2022-01-01, which matches the first and last rows of the file. | I have not tried the same call without `format` to see the wrong parse myself. |
| `property_inflation_index` | It lets the final model follow the 2020–2021 price jump. | Checked that it changes by year and measured the cost of using it one quarter late (1–2 PPE10 points). | I have not confirmed when each quarterly value was published, nor which publisher and which dwelling types the index covers. |
| Land-size cap of 10,000 m² | Drops the 7 m² house and the 20,695 m² apartment. | Those two rows are visible in the extremes and are removed by `property_size.between(20, 10000)`. | I do not have an external source for 10,000 rather than, say, 5,000. |
| The 10% threshold in PPE10 | Zillow and the IAAO material use it. | — | I have not checked what tolerance Australian lenders actually accept. |
| Adam's bias-correction terms (1 − β^t) | They undo the zero initialisation of m and v. | Without them convergence was slower; with them the loop reaches the closed form. | I have not worked through the expectation argument for why they are needed. |
| LightGBM categorical handling | Lets a split group suburbs directly instead of 626 one-hot columns. | Results match the information content of the one-hot Ridge. | I know only that it orders category values by gradient statistics before searching groupings; I have not read the algorithm. |
| MLP test spread is three times the validation spread | Decides whether one seed is enough to report. | Five seeds: validation PPE10 ± 0.9, test PPE10 ± 2.8. | My guess is that the extrapolation direction is set by weights the validation loss barely constrains; untested. |
| Pinball loss minimiser is the τ-quantile | Basis of the quantile models and the lending value. | Empirically: 10.9% of test prices fall below q10. | I have not proved it; I also do not know how LightGBM's quantile objective handles leaves with few samples. |
