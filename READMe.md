# Pakistan Tech Compensation Analysis: Does a Pay Gap Persist After Controlling for Experience?

**A statistical audit of developer compensation using the 2025 Stack Overflow Developer Survey**

## Why This Project

As a Pakistan-based CS graduate entering the data/tech job market, I wanted to move beyond a 
"raw numbers look different" observation and actually test whether a compensation gap between 
Pakistan-based and globally-distributed developers survives rigorous statistical control — the 
same standard a real People Analytics team would apply before making any claim about pay equity.

*(Originally scoped as a gender pay gap analysis; pivoted after confirming the 2025 survey does 
not include a gender field in its public release — see [Limitations](#limitations--future-work).)*

## Key Finding

> Pakistan-based developers report a median annual compensation of **$8,428**, compared to 
> **$78,000** for Rest-of-World respondents. This gap **persists — at roughly 5.3x — even after 
> statistically controlling for years of coding experience**, using two independent methods 
> (stratified comparison and OLS regression) that converge on a consistent estimate.

![Median compensation comparison](figures/01_median_comparison.png)

**This does not establish discrimination.** It establishes that the gap is not explained by 
experience alone. See [Interpretation & Caveats](#interpretation--caveats) below — this distinction 
is central to the whole analysis, not a footnote.

## Business Question

Do Pakistani developers earn less than global peers, and if so, does that gap persist after 
controlling for experience — or is it explained away by legitimate factors?

**Statistical hypotheses tested:**
1. Unadjusted: is there a significant mean/median salary difference between groups?
2. Adjusted: does the difference persist after controlling for `YearsCode`?

## Methodology Summary

| Step | Approach |
|---|---|
| Data source | 2025 Stack Overflow Developer Survey (public release) |
| Sample | 23,036 respondents (112 Pakistan, 22,924 Rest of World) after cleaning |
| Cleaning | Removed rows missing Country/Compensation/YearsCode; applied plausibility-based outlier bounds ($1,000–$2,000,000 salary; ≤50 years coding) — see `data/processed/README.md` for full derivation |
| Unadjusted testing | Welch's t-test (raw + log-transformed), Mann-Whitney U — all agree, p < .001 |
| Effect size | Cohen's d = 1.87 (log scale) — large effect |
| Confounder control | (1) Stratified comparison across 5 experience bands, (2) OLS regression: `log(salary) ~ YearsCode + CountryGroup`, robustness-checked with heteroskedasticity-consistent standard errors |

Full statistical reasoning, assumption checks, and decision log are documented inline in the 
notebooks (see [Repository Structure](#repository-structure)) — every cleaning and modeling 
choice is explained with its rationale, not just executed silently.

## Results at a Glance

![Compensation gap by experience band](figures/03_gap_by_experience.png)

| Experience Band | Pakistan Median | Rest-of-World Median | Ratio |
|---|---|---|---|
| 0–5 years | $2,950 | $23,203 | 7.9x |
| 5–10 years | $10,861 | $51,071 | 4.7x |
| 10–15 years | $16,856 | $75,410 | 4.5x |
| 15–25 years* | $25,284 | $92,812 | 3.7x |
| 25+ years* | $18,000 | $111,435 | 6.2x |

*Small Pakistan sample (n≤7) — directional only, not statistically reliable.

**Regression estimate (full sample, controlling for experience continuously):** Rest-of-World 
earns ~5.3x Pakistan's compensation, holding years of experience constant (p < .001, robust to 
heteroskedasticity correction). Model R² = 0.167 — experience and country group together explain 
only ~17% of salary variance; substantial variation remains unexplained (see Limitations).

## Interpretation & Caveats

- **Correlation/association, not causation.** This analysis rules out "experience differences 
  alone explain the gap." It does not identify *why* the remaining gap exists.
- **Nominal USD, not purchasing power.** Compensation is compared in raw USD terms, not adjusted 
  for cost of living or purchasing power parity. A meaningful nominal gap is expected between 
  Pakistan and higher cost-of-living countries independent of any equity concern.
- **Unmeasured confounders.** Role/seniority, company size, industry, education, and remote-work 
  status were not included in this model — see Future Work.
- **Self-selected survey population.** Findings apply to Stack Overflow survey respondents, not 
  the tech workforce broadly.

*If you take one thing from this project: "not explained by experience" is a defensible, narrow 
claim. "Caused by unfair treatment" is a much larger claim this analysis cannot make.*

## Limitations & Future Work

- Small Pakistan sample size in senior experience bands (n=5–7) limits confidence in that portion 
  of the finding
- Salary non-response correlates with experience in the Pakistan subgroup (reporters have higher 
  median experience than non-reporters), suggesting the final sample likely skews toward more 
  senior respondents than the true population
- Future iterations should incorporate `DevType` (role), `OrgSize` (company size), remote-work 
  status, and a purchasing-power-parity adjustment to better isolate the source of the remaining gap
- Originally scoped as a gender pay gap study; the 2025 survey does not include a public gender 
  field, so the project was reframed around country/region — a decision documented in the 
  commit history rather than silently absorbed

## Repository Structure
pakistan-tech-pay-gap/
├── README.md                          # this file
├── data/
│   ├── raw/                           # not committed — see data/raw/README.md to reproduce
│   └── processed/                     # cleaned dataset + derivation log
├── notebooks/
│   ├── 01_data_cleaning.ipynb         # inclusion criteria, outlier handling, documented reasoning
│   ├── 02_eda.ipynb                   # distribution analysis, normality checks
│   ├── 03_hypothesis_testing.ipynb    # unadjusted tests: t-test, Mann-Whitney, effect size
│   ├── 04_confounder_analysis.ipynb   # stratified comparison + regression (core analysis)
│   └── 05_visualizations.ipynb        # publication-quality figures for report/README
├── reports/
│   ├── pay_equity_report.md           # full findings, business insights, recommendations
│   ├── pay_equity_report.pdf          # exported version, downloadable
│   └── executive_summary.md           # 1-page non-technical summary
└── figures/                           # exported charts, referenced in README + report

## Reproducing This Analysis

1. Download the 2025 Stack Overflow Developer Survey from [insights.stackoverflow.com/survey](https://insights.stackoverflow.com/survey)
2. Place `survey_results_public.csv` in `data/raw/` (see `data/raw/README.md`)
3. Install dependencies: `pip install -r requirements.txt`
4. Run notebooks in order (01 → 05); each saves outputs consumed by the next

## Tech Stack

Python · pandas · NumPy · SciPy · statsmodels · Seaborn · Matplotlib

## Author

Chander — [https://github.com/ChanderValasai](#) · [https://linkedin.com/in/mrchandervalasai](#)
*Built as a portfolio project while completing DataCamp's Data Analyst with Python track.*