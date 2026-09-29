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
