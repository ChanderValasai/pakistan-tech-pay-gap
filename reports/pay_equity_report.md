# Pakistan Tech Compensation Audit: Does a Pay Gap Persist After Controlling for Experience?

**Author:** Chander
**Data source:** 2025 Stack Overflow Developer Survey (public release)
**Analysis period:** July 2026

---

## 1. Business Problem

Compensation disparities in tech are frequently discussed but rarely tested rigorously. Most 
informal comparisons stop at "Group A earns less than Group B" without asking the question that 
actually matters to HR leaders, policymakers, and affected developers themselves: **does the gap 
survive once you control for the legitimate drivers of pay, like experience?**

This project was originally scoped as a gender pay gap analysis. After downloading the 2025 
survey data, no gender field was present in the public release (Stack Overflow appears to have 
removed optional demographic questions in recent survey years). The project was reframed around 
a comparison with direct personal relevance: **Pakistan-based developers vs. the rest of the 
world**, using the same statistical framework originally planned for the gender comparison.

**Business question:** Do Pakistani developers earn less than global peers, and if so, does that 
gap persist after controlling for experience — or is it explained away by legitimate factors?

**Population of inference:** Respondents to the 2025 Stack Overflow Developer Survey. This is a 
self-selected, internet-active, largely English-fluent population — not a representative sample 
of the Pakistani or global tech workforce. All findings below are scoped accordingly.

---

## 2. Data & Methodology

### 2.1 Source and Sample

The raw dataset contains 48,882 respondents across ~173 fields. After filtering to respondents 
with non-null `Country`, `ConvertedCompYearly` (annual compensation, USD-converted), and 
`YearsCode`, the working sample was 23,838 rows.

### 2.2 Cleaning Decisions and Reasoning

| Decision | Reasoning |
|---|---|
| Salary bounds: $1,000–$2,000,000 | Global data included implausible values (min $1, max $50,000,000). Rather than a mechanical statistical rule (e.g., IQR), which performs poorly on heavily right-skewed income data, bounds were set using domain judgment: no full-time developer role plausibly pays below $1,000/year or above $2,000,000/year. |
| Investigated Pakistan-specific impact of the floor | Pakistan lost 20/132 rows (15.2%) to the floor vs. 3.05% overall — a 5x higher removal rate, worth investigating rather than accepting silently. Inspection of the excluded values showed several exact duplicate entries (e.g., five respondents all reporting $351), indicating a likely systematic unit-conversion error rather than genuine low salaries. This supported keeping the floor as-is. |
| YearsCode capped at ≤50 | Values above 50 (up to 100) were concentrated in round/repeated numbers consistent with the survey's "more than 50 years" option being converted to a sentinel value rather than a true reported figure. Affected 0.33% of rows, entirely within Rest-of-World — no impact on the Pakistan subgroup. |

**Final sample:** 23,036 respondents — 112 Pakistan, 22,924 Rest of World.

### 2.3 Known Data Quality Concern: Non-Response Bias

Only ~49% of all respondents reported compensation (a standard limitation of this survey). Within 
the Pakistan subgroup specifically, respondents who reported salary had a higher median experience 
(7 years) than those who did not (5.5 years) — suggesting salary reporters skew more senior. This 
means the final Pakistan sample likely overrepresents more experienced, higher-earning individuals 
relative to the true underlying respondent population. Directionally, this bias would tend to 
*understate* rather than overstate any gap favoring Rest-of-World, since it inflates Pakistan's 
observed compensation relative to its true distribution.

---

## 3. Unadjusted Findings

### 3.1 Descriptive Statistics

| Group | n | Median | Mean | Std Dev |
|---|---|---|---|---|
| Pakistan | 112 | $8,428 | $15,413 | $21,977 |
| Rest of World | 22,924 | $78,000 | $96,384 | $95,240 |

![Median compensation comparison](../figures/01_median_comparison.png)

Both distributions are heavily right-skewed, confirmed via histogram, boxplot, and Q-Q plot 
inspection. Variances are highly unequal between groups, ruling out a standard Student's t-test.

![Distribution Shape](../figures/02_distibution_shape.png)

### 3.2 Hypothesis Testing

Three tests were run to triangulate the result across different distributional assumptions:

| Test | Result |
|---|---|
| Welch's t-test (raw) | t = -37.32, p < .001 |
| Welch's t-test (log-transformed) | t = -19.09, p < .001 |
| Mann-Whitney U (non-parametric) | p < .001 |

All three agree: the unadjusted compensation difference is statistically significant.

### 3.3 Effect Size

Cohen's d = 0.85 (raw scale), **1.87 (log scale — considered more reliable** given raw-scale 
variance is inflated by extreme high earners in the Rest-of-World group). By conventional 
benchmarks (0.2/0.5/0.8 = small/medium/large), this is a large effect — not merely a statistically 
detectable one, but a substantively meaningful one.

**Caution:** with n=22,924 on one side, statistical significance alone was near-guaranteed even 
for a trivial difference. The large effect size, not the p-value, is what makes this finding 
worth reporting.

---

## 4. Confounder-Adjusted Findings

The unadjusted gap does not yet answer the core question — it doesn't account for the fact that 
Pakistan respondents are, on average, less experienced (median 7 years vs. 15 years for 
Rest-of-World). Two independent methods were used to test whether the gap survives this control.

### 4.1 Method 1: Stratified Comparison

![Compensation gap by experience band](../figures/03_gap_by_experience.png)

| Experience Band | Pakistan n | RoW n | Pakistan Median | RoW Median | Ratio |
|---|---|---|---|---|---|
| 0–5 years | 33 | 1,993 | $2,950 | $23,203 | 7.9x |
| 5–10 years | 50 | 5,215 | $10,861 | $51,071 | 4.7x |
| 10–15 years | 17 | 4,662 | $16,856 | $75,410 | 4.5x |
| 15–25 years | 7 | 6,107 | $25,284 | $92,812 | 3.7x* |
| 25+ years | 5 | 4,947 | $18,000 | $111,435 | 6.2x* |

*Small Pakistan sample — directional only, not statistically reliable.

The gap persists in every single experience band without exception. The relative gap narrows 
somewhat from entry-level (~8x) to mid-career (~4.5x) before the small top-band samples make the 
pattern less certain.

### 4.2 Method 2: OLS Regression

Model: `log(ConvertedCompYearly) ~ YearsCode + CountryGroup`

| Term | Coefficient | p-value |
|---|---|---|
| CountryGroup (Rest of World) | 1.6713 | < .001 |
| YearsCode | 0.0397 | < .001 |

R² = 0.167. Result confirmed robust to heteroskedasticity-consistent (HC3) standard errors 
(coefficient and significance essentially unchanged).

**Translated:** holding years of experience constant, Rest-of-World respondents earn 
approximately **5.3x** what Pakistan respondents earn. This sits within the range produced by 
the stratified method (3.7x–7.9x across bands), providing convergent evidence from two 
independent approaches.

Each additional year of coding experience is associated with ~4.05% higher compensation, holding 
country group constant — confirming experience is a genuine driver, just not one that explains 
away the country-level gap.

**Model limitation:** R² = 0.167 means experience and country group together explain only ~17% 
of salary variance. The majority of variation is driven by factors not included in this model — 
see Section 6.

---

## 5. Business Insights

| Insight | Confidence |
|---|---|
| The raw compensation gap is large and statistically robust | High — confirmed by 3 independent tests, large effect size |
| The gap persists after controlling for experience | High — confirmed by 2 independent methods (stratification, regression) converging on a consistent estimate |
| The gap narrows somewhat at mid-career before becoming uncertain at senior levels | Low-Moderate — senior bands have very small Pakistan samples (n=5–7) |
| Experience and country together explain only ~17% of salary variation | High — direct model statistic; other factors clearly matter |

---

## 6. Recommendations

**1. Treat this as a starting signal for further investigation, not a standalone conclusion.**
Evidence: large, robust gap surviving experience control. Confidence: high that a gap exists, low 
on root cause. Assumption: nominal USD, not purchasing-power adjusted. Impact: prevents both 
premature over-claiming (discrimination) and under-reacting (dismissing a real signal). Risk: 
either extreme is a real risk if this analysis is over-interpreted.

**2. Expand the model to include role/seniority, company size, and remote-work status before 
drawing stronger conclusions.**
Evidence: R² = 0.167 — most variance is currently unexplained. Confidence: high that these 
variables would materially change the estimate. Impact: necessary before this could inform actual 
compensation policy.

**3. Adjust for purchasing power parity (PPP) before using this for market benchmarking.**
Evidence: nominal USD comparison conflates real pay differences with cost-of-living differences. 
Confidence: high that PPP-adjustment would narrow the apparent gap substantially. Impact: prevents 
both underpaying local talent and overestimating true disparity.

---

## 7. Interpretation Caveats

- **Correlation/association, not causation.** This analysis rules out "experience alone explains 
  the gap." It does not identify *why* the remaining gap exists, and does not establish 
  discrimination or any specific causal mechanism.
- **Nominal USD, not purchasing power.** A substantial nominal gap is expected between Pakistan 
  and higher cost-of-living countries independent of any equity concern.
- **Statistical significance ≠ practical significance, and vice versa.** Significance here was 
  near-guaranteed by sample size; the large effect size is what makes the finding substantively 
  meaningful.
- **If a stakeholder claims this "proves" discrimination:** the correct response is that "not 
  explained by experience" is a narrower, defensible claim than "caused by unfair treatment." 
  Confirming the latter would require controlling for role, seniority, company size, and 
  purchasing power at minimum — none of which were available in this dataset.

---

## 8. Limitations & Future Work

- Small Pakistan sample size in senior experience bands (n=5–7) limits confidence in that portion 
  of the findings
- Salary non-response correlates with experience within the Pakistan subgroup, likely biasing the 
  final sample toward more senior respondents than the true population
- Self-selected survey population — findings apply to Stack Overflow respondents, not the tech 
  workforce broadly, in Pakistan or elsewhere
- Future iterations should incorporate `DevType` (role), `OrgSize` (company size), remote-work 
  status, education level, and a purchasing-power-parity adjustment to better isolate the source 
  of the remaining gap
- Gender-based analysis (the project's original scope) was not possible due to the absence of a 
  gender field in the 2025 survey's public release; this remains a potential avenue if a future 
  survey year restores that field

---

## Appendix: Reproducibility

All analysis code, cleaning logic, and statistical outputs are available in the accompanying 
notebooks (`notebooks/01_data_cleaning.ipynb` through `04_confounder_analysis.ipynb`). See the 
repository root `README.md` for setup instructions.
