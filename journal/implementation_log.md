# Implementation Log — A2 Sydney House Price Estimation

> Purpose (from the A2 spec): document challenges & solutions, use of AI tools, and knowledge gaps.
> Write an entry **every working session**. Short, honest and dated. This log is your protection in the Q&A.

Entry template:

```
### YYYY-MM-DD — <one-line topic>
- What I did:
- Problem / question:
- What I tried / how I verified:
- Outcome / decision:
- AI tool used? (what I asked, what it produced, how I checked it):
- Open knowledge gap (if any):
```

---

## Part 1 — Development entries

### 2026-09-30 — Project setup and first look at the data
- What I did: chose Option 2 (practical ML system). Task: estimate the sale price of a Greater Sydney residential property. Dataset: `domain_properties.csv` (11,160 Domain.com.au sales, 2016–2022). Created the project skeleton (`data/`, `notebook/`, `journal/`, `report/`) and notebook sections 0–1 (environment, data acquisition, first look).
- Problem / question: Is this dataset acceptable under the spec ("real, non-trivial, not a toy dataset")?
- What I tried / how I verified: profiled the CSV — 17 columns, no missing values, 637 suburbs, 16 property types, price range 0.225 M – 60 M AUD. It is real scraped market data with time and macro features, not a textbook dataset.
- Outcome / decision: accepted. Noted data-quality issues to handle in Section 2 (47-bedroom / 7 m² rows, non-residential types such as *Vacant land*), and a distribution shift issue (48 % of rows are from the 2021 boom year).
- AI tool used?: Cursor (Claude) helped design the project plan against the rubric and drafted notebook section 0–1 code. I ran every cell locally and checked the printed statistics against my own `pandas` inspection of the CSV.
- Open knowledge gap: whether `property_inflation_index` is known at sale time (publication lag) — must check the source of the index before deciding to use it as an input feature.

### 2026-09-30 — Published the notebook and checked it runs on Colab
- What I did: pushed the repository to https://github.com/AlexHu1210/uts-ml-a2-sydney-house-price (public). Opened the notebook in Google Colab and used Runtime → Run all.
- Problem / question: Colab cannot see the file on my laptop, so the download URL has to work.
- What I tried / how I verified: Run all finished with no error and showed the two price histograms.
- Outcome / decision: the notebook is self-contained. First `git push` failed because the Mac keychain offered a different GitHub account; the second attempt, with a personal access token for AlexHu1210, succeeded.
- AI tool used?: Cursor walked through the GitHub steps. I did the token creation and the push myself.
- Open knowledge gap: none new.

### 2026-09-30 — Section 2: task boundary and cleaning rules
- What I did: defined training and deployment inputs/outputs, and removed non-dwellings plus physically implausible rows. 11,160 → 10,864.
- Problem / question: should a 17 M AUD house be deleted as an outlier?
- What I tried / how I verified: looked at the extreme rows. 47 bedrooms / 7 m² are entry errors. The 17 M AUD Woollahra sale has 3 bedrooms and 325 m², which is a real expensive house. Price is therefore not capped; the log target will be used instead.
- Outcome / decision: drop non-residential types, bedrooms outside 0–10, bathrooms outside 0–8, parking outside 0–8, size outside 20–10,000 m².
- AI tool used?: Cursor drafted the cleaning function. I need to be able to state each bound and why, without reading the code.
- Open knowledge gap: whether 10,000 m² is the right land-size cap for outer-suburb houses.

<!-- add new entries above this line, newest at the bottom of Part 1 -->

---

## Part 2 — Use of AI tools (summary, to be finalised before submission)

| Where | What AI produced | How I verified / what I changed |
|-------|------------------|---------------------------------|
| Notebook §0–1 | boilerplate for env setup, data download with local fallback, summary tables | ran locally, compared numbers with manual `pandas` checks |

---

## Part 3 — Knowledge gaps (to be finalised before submission)

For each item: why the component is needed · how I verified its correctness · what I did to understand it.

| Component | Why needed | How verified | Attempts to understand |
|-----------|-----------|--------------|------------------------|
| (fill in as the project progresses) | | | |
