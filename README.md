# QSS 20 Final Project — SNAP/TANF Receipt and Educational Outcomes

**Author:** Molly Ryan
**Course:** QSS 20: Modern Statistical Computing, Spring 2026

## Research question

How does childhood receipt of SNAP/Food Stamps and AFDC/TANF relate to long-term educational outcomes (high-school completion, college attainment) for children of NLSY79 mothers?

The literature (Pilkauskas 2023; Heflin et al. 2022; Aizer et al. 2022) suggests both programs may support children's development through different mechanisms — SNAP via food security, AFDC/TANF via household income stability. This project compares four exposure groups: **Neither**, **SNAP only**, **TANF only**, and **Both SNAP and TANF**.

## Data source

[NLSY79 Children and Young Adults (NLS-CYA)](https://www.nlsinfo.org/content/cohorts/nlsy79-children) — a national longitudinal study following the biological children of NLSY79 mothers from birth into adulthood. Data covers 1979–2018 across biennial waves.

Public-use extract was pulled from the [NLS Investigator](https://www.nlsinfo.org/investigator/pages/login). The full raw extract (`nlscya_all_1979-2018.csv`, ~2.8 GB, 82,318 variables × 11,545 children) is **excluded from this repository** via `.gitignore` because of GitHub's file size limits. The repo contains:

- `data/nlscya_subset.csv` — slim 22-variable extract used by all notebooks (0.75 MB, tracked in repo)
- `data/nlscya_cleaned.csv` — analysis-ready cleaned dataset with constructed exposure groups (tracked)
- Raw `.csv`, `.cdb`, `.R`, `.sas`, `.dat`, `.NLSCYA`, `.sdf` files — gitignored

To rebuild the slim subset from scratch, download the same six files from NLS Investigator (variable list documented in `output/welfare_variables_inventory.csv`) and re-run `code/01_pull_data.ipynb`.

## Code structure

The three notebooks run end-to-end and should be executed in order:

| Notebook | Purpose | Inputs | Outputs |
|----------|---------|--------|---------|
| [`code/01_pull_data.ipynb`](code/01_pull_data.ipynb) | Load the slim 22-variable subset, inspect raw NLS missing codes, verify sample size | `data/nlscya_subset.csv` | (interactive) |
| [`code/02_clean_construct.ipynb`](code/02_clean_construct.ipynb) | Recode NLS missing codes (`-1`, `-2`, `-3`, `-7` → `NaN`), build SNAP exposure indicator (1994–2006), build AFDC/TANF exposure indicator (1994–1998), assign each child to one of four program groups, build 3-level education outcome | `data/nlscya_subset.csv` | `data/nlscya_cleaned.csv` |
| [`code/03_analyze_visualize.ipynb`](code/03_analyze_visualize.ipynb) | Descriptive comparison of education attainment across the four exposure groups; produces Figures 1–2 and Table 1 | `data/nlscya_cleaned.csv` | `output/fig1_education_distribution.png`, `output/fig2_hs_completion_by_group.png`, `output/table1_hs_rate_by_group.csv` |

## Key variables

**Welfare exposure**

- `FOODSTAMPS_1994` through `FOODSTAMPS_2016` — biennial indicators of household food-stamp receipt
- `AFDC_1994`, `AFDC_1996`, `AFDC_1998` — AFDC receipt (program was replaced by TANF in 1997)
- `snap_ever` (constructed) — 1 if ever received food stamps 1994–2006, 0 if never, NaN if all waves missing
- `tanf_ever` (constructed) — same logic for AFDC 1994–1998
- `program_group` (constructed) — `Neither` / `SNAP only` / `TANF only` / `Both SNAP and TANF`

**Education outcomes (cross-round summary)**

- `HSDIPLOMA_EVER` — ever received HS diploma (1/0)
- `GED_EVER` — ever received GED (1/0)
- `PROFDEGREE_EVER` — ever received a college/professional degree (1/0)
- `edu_level` (constructed) — 0 = No HS or GED, 1 = HS or GED, 2 = Has professional degree

**Demographics**

- `CPUBID`, `MPUBID` — child and mother IDs
- `CSEX`, `CRACE` — child sex and race/ethnicity

## Headline result (Milestone 2)

HS/GED completion rate by program-exposure group (children with valid responses on all exposure and education variables):

| Group | N | HS/GED rate | 95% CI half-width |
|---|---|---|---|
| Neither | 1,567 | 92.1% | ±1.3 pp |
| SNAP only | 574 | 85.4% | ±2.9 pp |
| TANF only | 7 | 85.7% | ±25.9 pp |
| Both SNAP and TANF | 127 | 81.9% | ±6.7 pp |

There is a clear monotonic gradient: more welfare-program exposure is associated with lower HS-completion rates. **This is descriptive, not causal** — the literature is clear that welfare-receiving households differ from non-receiving households on many background characteristics (parental education, income, family structure, neighborhood) that independently affect children's education. Milestone 3 will add demographic controls and regression-based adjustment to begin to disentangle these patterns.

See `output/fig1_education_distribution.png` for the full education distribution by group and `output/fig2_hs_completion_by_group.png` for the rate plot with 95% confidence intervals.

## Known limitations

1. **Sparse TANF-only cell.** Only 7 children fall into the TANF-only group with valid education data, so the 95% CI is uninformative for that cell. The Both / SNAP-only / Neither comparisons are well-powered.
2. **Self/spouse reporting.** The AFDC and food-stamp variables in the NLSY79-CYA file ask about the respondent's own / their spouse's receipt, not their mother's during their childhood. The interpretation is therefore "respondents who entered welfare programs as young adults" rather than "children of welfare-receiving mothers." A future round will link the mother's welfare history from the NLSY79 main file.
3. **Sample restriction.** Of 11,545 NLS-CYA children, only ~2,275 have non-missing values for both the SNAP and TANF exposure indicators. Most missingness is because the child was too young to be asked welfare-receipt questions in the 1994–2006 windows.
4. **No causal claim.** Group comparisons reflect a mix of program effect, selection into program participation, and unobserved confounders. The Milestone 3 logistic-regression step (per Unit 11 methods) will add the controls but cannot fully resolve selection bias in observational data.

## Environment

Python 3.10+ with `pandas`, `numpy`, `matplotlib`.
Install: `pip install pandas numpy matplotlib`.

## Course skills used (per the QSS 20 rubric)

- Unit 3 (pandas): boolean masking, `groupby`, `apply`, stacked bar / errorbar plots
- Unit 4 (user-defined functions): `assign_group(row)`, `edu_level(row)`
- Unit 5 (workflows): GitHub repository with `code/`, `data/`, `output/` directories
- Unit 11 (supervised ML): planned for Milestone 3 — logistic regression of HS completion on program-group + controls

## Milestone progress

- [x] Milestone 1 — research question, data sources, two starter visualizations
- [x] Milestone 2 — public repo, 3 working notebooks, cleaned data, headline figures + table
- [ ] Milestone 3 — regression with controls, expanded education outcome
- [ ] Final paper
