# Customer Loyalty Analytics — Full Analytical Report

**Case study:** Turtle Games (games retailer)
**Analytical basis:** Python EDA notebook · Business questions notebook · R EDA & regression notebook · R script
**Dataset:** `turtle_reviews.csv` — 2,000 customer records, 200 unique products
**Programme:** LSE — Advanced Analytics for Organisational Impact · June 2026

---

## Headline metrics

| Metric | Value | Note |
|---|---|---|
| Mean loyalty points (overall) | 1,578 | |
| Peak loyalty (age 26–35, high pay) | 3,410 | 2.2× overall mean |
| Mean VADER score | 0.645 | Broadly positive |
| Unique products | 200 | 5 flagged for urgent attention |
| Loyalty variance explained by pay + spend (MLR) | 82.7% | |
| Customer segments identified (K-Means) | 5 | |
| Loyalty Gini coefficient | 0.415 | High inequality |
| NPS proxy score | 74.3 | Excellent brand health |

---

# Part 1 — Executive summary

## Project objective

The analytics team has answered four core business questions using data from 2,000 customer reviews, demographic profiles, loyalty records, and product sentiment. The objective is to improve overall sales performance by understanding how loyalty is accumulated, how customers should be segmented for targeted marketing, how review text data can inform campaigns, and whether loyalty data is statistically suitable for predictive modelling.

## Key findings

The customer data reveal a loyalty programme that is working well for a concentrated elite but leaving significant commercial value on the table. The following five findings are the most consequential for business strategy:

1. **Loyalty is driven by two completely independent variables** — spending score and remuneration — which together explain 82.7% of all loyalty variation. Age has virtually zero influence, making the loyalty programme effectively age-neutral.

2. **Five distinct customer segments exist** with dramatically different commercial profiles. The top segment (18% of customers) generates 4× the loyalty of the average, while a high-income disengaged segment represents the greatest untapped revenue opportunity in the entire dataset.

3. **Customer sentiment is strongly positive overall** (89% positive reviews, NPS proxy 74.3), but five specific products have critically low sentiment scores combined with high-value customers — creating a hidden churn risk that requires immediate executive attention.

4. **Loyalty inequality is substantial:** the top 25% of customers hold 54% of all loyalty points (Gini = 0.415). Losing the top 200 customers would remove 30% of total programme value overnight.

5. **The age 26–35 cohort with high income** (top pay quartile) generates 3,410 loyalty points on average — 2.2× the overall mean. This demographic is the single highest-ROI acquisition target for the loyalty programme.

---

# Part 2 — Data quality and preparation review

## Dataset overview

| Attribute | Detail | Assessment |
|---|---|---|
| Source file | `turtle_reviews.csv` | Single source — no data integration required |
| Records | 2,000 customer rows | Moderate size — sufficient for regression and clustering |
| Original columns | 11 variables | 2 dropped, 3 renamed, 1 engineered (`log_loyalty`) |
| Missing values | Zero across all columns | Excellent data quality — no imputation required |
| Duplicate reviews | Present (not removed) | Justified decision — different customers may post identical generic reviews |
| Date / time data | Not present | Significant limitation — no temporal trend analysis possible |
| Transaction data | Not present | Loyalty used as engagement proxy — direct sales data unavailable |

## Data cleaning steps performed

| Step | Action | Justification |
|---|---|---|
| 1 | Dropped `language` and `platform` columns | Both contained a single value (EN / Web) — zero analytical variance |
| 2 | Renamed `remuneration(k£)` to `pay` | Improves code readability and reduces error risk in formulas |
| 3 | Renamed `spending_score(1-100)` to `spend` | Same rationale — cleaner variable names throughout |
| 4 | Renamed `loyalty_points` to `loyalty` | Consistency across Python and R notebooks |
| 5 | Created `log_loyalty = log(loyalty)` | Addresses right-skew (1.465) — improves OLS model validity |
| 6 | Lowercased review and summary text | Normalises tokens for NLP frequency analysis |
| 7 | Removed punctuation from text | Prepares text for tokenisation and wordcloud generation |
| 8 | Saved clean DataFrame as `reviews.csv` | Ensures consistent starting point across all modules |

## Variables excluded or limited in value

| Variable | Decision | Reason |
|---|---|---|
| `language` | Dropped | 100% English — no variation, no analytical signal |
| `platform` | Dropped | 100% Web — no variation, no analytical signal |
| `age` | Retained but deprioritised | r = -0.042 with loyalty — not a meaningful predictor; useful only in cohort analysis |
| `gender` | Retained for completeness | t-test p = 0.363 — no significant loyalty difference; campaigns should be gender-neutral |
| `product` | Retained for product-level analysis | 200 unique IDs — only 10 reviews per product on average, limiting product-level inference |

> **Concern:** The absence of transaction dates and direct sales figures is the most significant data limitation. Loyalty points are used as a proxy for commercial engagement throughout this analysis, which is appropriate but introduces a structural bias: loyalty reflects programme mechanics (points awarded on purchase) as well as genuine brand loyalty. These two components cannot be separated without additional transactional data.

---

# Part 3 — Exploratory data analysis review

## Descriptive statistics — key variables

| Variable | Mean | Std dev | Min | Max | Skewness | Kurtosis | Distribution |
|---|---|---|---|---|---|---|---|
| Age | 39.5 yrs | 13.6 | 17 | 72 | 0.609 | -0.189 | Mildly right-skewed, approx. normal |
| Pay (remuneration) | £48.1k | £23.1k | £12.3k | £112.3k | 0.413 | -0.406 | Near-normal, mild positive skew |
| Spend (score 1–100) | 49.0 | 26.6 | 1 | 100 | -0.042 | -0.889 | Near-uniform — very flat distribution |
| Loyalty (raw) | 1,578 pts | 1,283 | 25 | 6,847 | 1.465 | 1.716 | Highly right-skewed — **not normal** |
| Log(loyalty) | 7.19 | 0.71 | 3.22 | 8.83 | 0.070 | -0.151 | Near-normal after transformation |

## Distribution findings

- Loyalty points exhibit significant right-skew (skewness = 1.465) and positive excess kurtosis (1.716). Shapiro-Wilk and D'Agostino-Pearson tests both confirm non-normality. **This is the critical distributional finding that underpins all modelling decisions.**
- Log-transformation reduces skewness from 1.465 to 0.070 — a 95% improvement — making the transformed variable suitable for OLS regression.
- Spending score is remarkably close to uniform distribution (skewness = -0.042, kurtosis = -0.889), suggesting the scoring mechanism was designed to produce an even spread across customers.
- Pay distribution is approximately bell-shaped (skewness = 0.413), indicating a relatively healthy income spread across the customer base without extreme concentration.
- Age distribution is mildly right-skewed but broadly representative of an adult consumer population.

## Correlation analysis

| Variable pair | Pearson r | Spearman ρ | Significance | Business interpretation |
|---|---|---|---|---|
| Spend vs loyalty | 0.672 | 0.668 | p < 0.001 | Strongest single loyalty driver |
| Pay vs loyalty | 0.616 | 0.601 | p < 0.001 | Second strongest loyalty driver |
| Age vs loyalty | -0.042 | -0.045 | p = 0.059 | Not significant — exclude from models |
| Pay vs spend | 0.002 | N/A | p > 0.05 | Zero correlation — fully independent predictors |
| Loyalty vs log(loyalty) | 0.978 | N/A | N/A | Log transform preserves ranking structure |

> **Finding:** Pay and spend are entirely uncorrelated (r = 0.002, VIF = 1.00). This is a critical and unusual finding: high-income customers do not automatically spend more. The two variables provide completely independent information, making the MLR model particularly powerful and free from multicollinearity concerns.

## Demographic analysis findings (R notebook)

| Dimension | Finding | Statistical evidence | Business implication |
|---|---|---|---|
| Gender | No significant loyalty difference | t-test p = 0.363; Mann-Whitney p = 0.027 (negligible effect r = 0.045) | Design gender-neutral campaigns — avoid wasted demographic targeting |
| Education | Significant loyalty variation | ANOVA F = 7.23, p < 0.001; Kruskal-Wallis H = 37.4 | Education is a pay proxy — target graduates (n = 900, largest group) |
| Age (cohort) | Inverted-U pattern | ANOVA F = 61.4, p < 0.001 | 26–35 group generates 2.2× overall mean loyalty — prime acquisition target |
| Pay quartile | Each quartile roughly doubles loyalty | Q1 = 592 pts, Q2 = 1,294, Q3 = 1,818, Q4 = 2,733 | Q4 earns 362% more than Q1 with identical spend scores |

---

# Part 4 — Regression model review

## Complete model comparison — Python and R notebooks

| Model | Tool | Predictors | Outcome | R² | Adj R² | Status | Recommendation |
|---|---|---|---|---|---|---|---|
| M1 | Python | spend | loyalty | 0.452 | 0.452 | Heteroscedastic | Baseline only |
| M2 | Python | spend | log_loyalty | 0.524 | 0.523 | Reduced heteroscedasticity | Acceptable |
| M3 | Python | pay | loyalty | 0.380 | 0.379 | Heteroscedastic | Baseline only |
| M4 | Python | pay | log_loyalty | 0.280 | 0.280 | Homoscedastic | Valid but weak |
| M5 | Python | age | loyalty | 0.002 | 0.001 | Not significant | Rejected |
| **M6 (MLR)** | Python | pay + spend | loyalty | **0.827** | 0.827 | Heteroscedastic | Strong — caution |
| **M7 (MLR)** | Python | pay + spend | log_loyalty | **0.810** | 0.809 | Improved | **Recommended OLS** |
| M4 (R) | R | spend + rem | loyalty | 0.826 | 0.826 | Heteroscedastic | Cross-validated |
| M5 (R) | R | spend + rem + age | loyalty | 0.826 | 0.826 | Age adds nothing | Confirms age exclusion |
| Interaction | Python | pay × spend | loyalty | **0.977** | 0.977 | Multiplicative effect | Advanced model |

## Best model — multiple linear regression (M6 / M7)

> **Recommended production model**
>
> `loyalty = -1,700.3 + 33.98 × pay + 32.89 × spend`
>
> R² = 0.827 · Adj R² = 0.827 · VIF = 1.00 · Both predictors p < 0.001
>
> **Interpretation:** Every £1,000 increase in annual pay adds approximately 34 loyalty points. Every 1-point increase in spending score adds approximately 33 loyalty points. Pay and spend are entirely independent drivers with equal importance.

## Model assumptions assessment

| Assumption | Test used | Result | Verdict |
|---|---|---|---|
| Linearity | Scatter plots, residual plots | Clear linear trends for pay and spend | Met |
| No multicollinearity | VIF | VIF = 1.00 for both predictors | Met — excellent |
| Normal residuals | Q-Q plots, Shapiro-Wilk, D'Agostino | Residuals approximately normal (skew = 0.10) | Substantially met |
| Homoscedasticity | Breusch-Pagan | p < 0.001 — heteroscedasticity present | **Violated — use robust SEs** |
| Model validity (log-transformed) | Breusch-Pagan on M7 | p > 0.05 for pay-only model | Improved substantially |

> **Concern:** Heteroscedasticity is present in all models using raw loyalty as the outcome. Standard errors are underestimated and confidence intervals are unreliable. The recommended fix is to use HC3 robust standard errors for reporting, or to use log(loyalty) as the outcome variable for all OLS-based reporting. For production deployment in a CRM system, ensemble models (Random Forest R² = 0.94) are preferred as they do not require normality or homoscedasticity assumptions.

## Advanced model findings

- **Interaction regression (pay × spend)** achieves R² = 0.977 — confirming loyalty is multiplicative, not additive. High pay amplifies the effect of spend, meaning targeting both dimensions simultaneously is far more powerful than targeting either alone.
- **Lasso regularisation** zeroes out age and review length, confirming pay and spend as the only substantive predictors. Consistent across both Python and R analyses.
- **Decision Tree** (pruned, max_depth = 4) achieves R² = 0.943 on the test set — outperforming OLS without requiring normality assumptions. The critical threshold **spend = 67** is identified as the single most important decision boundary.
- **Random Forest and Gradient Boosting** achieve R² > 0.94 with stable 5-fold cross-validation — these are the recommended models for production deployment.
- The R notebook independently validates all Python findings: Model 4 in R (spend + remuneration) achieves Adj R² = 0.826, confirming cross-platform consistency.

---

# Part 5 — Customer segmentation analysis

## K-Means clustering — methodology and validation

K-Means clustering was applied to spending score and remuneration (standardised) to identify natural customer groupings. Both the Elbow Method (inertia analysis) and Silhouette Method independently confirmed k = 5 as optimal. Visual inspection of scatterplots at k = 3, 4 and 5 further validated this choice, with five non-overlapping clusters clearly visible. All cluster means are validated with 95% bootstrap confidence intervals.

## Five customer segments — complete profiles

| Segment | Pay range | Spend range | Mean loyalty | Size | Silhouette | CLV tier |
|---|---|---|---|---|---|---|
| A — VIP (high pay & high spend) | ~£73k | ~82/100 | 3,988 pts | 356 (18%) | 0.61 | Platinum |
| B — Growth (mid pay & mid spend) | ~£44k | ~50/100 | 1,420 pts | 774 (39%) | 0.55 | Gold |
| C — Untapped (high pay, low spend) | ~£75k | ~17/100 | 912 pts | 269 (13%) | 0.58 | Silver\* |
| D — Engaged (low pay, high spend) | ~£20k | ~79/100 | 972 pts | 330 (17%) | 0.52 | Silver |
| E — Dormant (low pay & low spend) | ~£20k | ~20/100 | 275 pts | 271 (14%) | 0.54 | Bronze |

\* *Cluster C CLV is Silver tier despite high income, due to low spend engagement.*

### Segment A — VIP (n = 356, 18% of customer base)

- **Customer profile:** High income (mean £73.2k) combined with high spending behaviour (score 82/100). Mean age 35.6 years. Predominantly graduates and postgraduates.
- **Commercial value:** Generates 3,988 mean loyalty points — 2.5× the overall average. Estimated CLV: Platinum tier. Top 18% of customers representing the highest programme value.
- **Retention strategy:** VIP programme with exclusive early product access, dedicated account management, personalised rewards, and quarterly loyalty bonus events. Cost of losing one VIP customer is equivalent to acquiring 15 Dormant customers.
- **Growth strategy:** Invite to brand ambassador programme. Use positive review language from this segment in marketing materials. Introduce referral incentives — their network likely has similar income profiles.

### Segment B — Growth (n = 774, 39% of customer base)

- **Customer profile:** Mid-range across all variables (pay £44.4k, spend score 50/100). Largest single segment. Mean age 42.1 — slightly older than VIP customers. Most diverse education mix.
- **Commercial value:** Generates 1,420 mean loyalty points. Gold CLV tier. Accounts for the largest share of total programme volume by size. Converting even 10% to VIP tier adds significant revenue.
- **Retention strategy:** Tier progress notifications showing distance to next loyalty tier. Personalised product recommendations based on purchase history. Targeted spend multiplier events.
- **Growth strategy:** The primary growth lever. Loyalty multiplier promotions targeting customers with spend scores of 50–67 (just below the critical spend = 67 threshold). A 17-point spend increase moves this segment into premium loyalty territory.

### Segment C — Untapped (n = 269, 13% of customer base)

- **Customer profile:** High income equivalent to VIP segment (mean £74.8k) but minimal spending engagement (score only 17/100). Mean age 31.6 — youngest segment. High education level.
- **Commercial value:** Despite earning as much as VIP customers, this segment generates only 912 loyalty points — 77% less than Segment A. Estimated CLV uplift potential: 6.5× if activated to VIP spend behaviour. **This is the single largest commercial opportunity in the dataset.**
- **Retention strategy:** These customers are not yet churned — they are simply not engaged. Exit survey or preference questionnaire to understand barriers. Premium trial offers at no cost to demonstrate product quality.
- **Growth strategy:** Premium bundle campaigns, exclusive first-look product invitations, concierge-style onboarding. Target with LinkedIn advertising (high income, young professional demographic).

### Segment D — Engaged (n = 330, 17% of customer base)

- **Customer profile:** Low income (mean £20.4k) but high spending engagement (score 79/100). Actively participates in the loyalty programme despite financial constraints. Mean age 40.7.
- **Commercial value:** Generates 972 loyalty points — near the overall mean despite low income. High NPS from this segment; these customers are genuine brand advocates. Limited CLV potential due to income ceiling.
- **Retention strategy:** Value promotions, cashback loyalty offers, spend-more-save-more mechanics. Price-sensitive but highly engaged — protect with value-based rewards rather than premium experiences.
- **Growth strategy:** Referral programme — their enthusiasm can attract higher-income customers. Family and social gifting promotions around peak retail periods.

### Segment E — Dormant (n = 271, 14% of customer base)

- **Customer profile:** Low income and low spending engagement. Mean loyalty only 275 points — 83% below the overall average. Mean age 43.5. Typically diploma or basic education level.
- **Commercial value:** Lowest CLV tier. Minimal current programme contribution. However, 271 customers represent acquisition headroom if effectively re-engaged with the right entry-level offers.
- **Retention strategy:** Low-investment retention — automated win-back email sequences, entry-level loyalty sign-up bonus. Do not over-invest premium retention resources in this segment.
- **Growth strategy:** Broad awareness campaigns. Consider whether acquisition cost for upgrading this segment is justified given the low income ceiling.

---

# Part 6 — NLP and sentiment analysis

## Sentiment analysis overview

Two sentiment frameworks were applied to 2,000 customer reviews and their summary fields: **TextBlob** (polarity and subjectivity scoring) and **VADER** (Valence Aware Dictionary and sEntiment Reasoner). Both methods were used to cross-validate findings. The analysis includes word frequency analysis, wordcloud generation, and a critical examination of the "Five Stars" neutral scoring limitation.

| Metric | TextBlob | VADER | Interpretation |
|---|---|---|---|
| Mean sentiment score | 0.20 (polarity) | 0.645 (compound) | Both confirm broadly positive customer sentiment |
| Positive reviews | ~65% | 89.0% (> 0.05) | VADER is more sensitive to nuanced positivity |
| Neutral reviews | ~20% | 4.5% | TextBlob overstates neutral — Five Stars problem |
| Negative reviews | ~15% | 6.5% (< -0.05) | Consistent signal across both tools |
| Sentiment–loyalty correlation | N/A | -0.025 | No direct relationship — loyalty is behavioural |
| Sentiment NPS proxy | N/A | 74.3 | Excellent brand health benchmark |

## Critical limitation — "Five Stars" neutral scoring

> **Concern:** Both VADER and TextBlob score the phrase "Five Stars" as neutral (compound = 0.0). Investigation reveals this affects a significant subset of summary reviews. When Five Stars records are removed, the proportion of genuinely positive summaries increases substantially. This means neutral sentiment is overrepresented and positive sentiment is underrepresented in summary-level analysis.
>
> **Recommendation:** Train a domain-specific gaming retail NLP model, or manually curate a custom lexicon that scores "Five Stars", "Five star" and "5 stars" as strongly positive.

## Top 15 most frequent review words (stopwords removed)

| Rank | Word | Signal | Marketing application |
|---|---|---|---|
| 1 | game | Product | Core brand identity — use in all campaign headlines and SEO |
| 2 | great | Positive | Primary testimonial language — feature in product listings |
| 3 | good | Positive | General satisfaction — use in review summary widgets |
| 4 | fun | Positive | Key emotional benefit — anchor for family product campaigns |
| 5 | play | Product | Gameplay experience central — feature in product descriptions |
| 6 | love | Positive | Strong emotional engagement — use in loyalty programme messaging |
| 7 | easy | Product | Ease of use valued — highlight in product spec sheets |
| 8 | quality | Mixed | Appears positively and negatively — monitor context with NER |
| 9 | kids | Demographic | Family segment prominent — dedicated family gaming campaign |
| 10 | board | Product | Board games most reviewed — prioritise in marketing spend allocation |
| 11 | waste | Negative | Quality concern — flag for product review pipeline |
| 12 | price | Business | Value for money mentioned — review pricing communication strategy |
| 13 | pieces | Product | Component quality concern — address in product descriptions |
| 14 | time | Mixed | Delivery or engagement time — add clarity to product listings |
| 15 | perfect | Positive | High satisfaction marker — use in testimonials and star ratings |

## Product sentiment analysis — performance matrix

Product-level sentiment was cross-referenced with product-level mean loyalty to create a four-quadrant performance matrix. The key analytical insight is that **high loyalty combined with low sentiment represents the highest churn risk** — not the worst brand health metric. This is because loyalty is driven by customer profile (pay, spend), not product quality. A VIP customer who purchases a poorly reviewed product retains their high loyalty score — but their disappointment represents a future churn risk disproportionate to the loyalty points at stake.

| Quadrant | Products | Description | Priority action |
|---|---|---|---|
| Q1 — Star products | 41 | High sentiment + high loyalty — best of breed | Spotlight in marketing, use as testimonials |
| Q2 — Churn risk | 48 | Low sentiment + high loyalty — VIP customers disappointed | **Urgent:** quality fix before high-value churn |
| Q3 — Underperformers | 70 | High sentiment + low loyalty — liked but wrong audience | Reposition or cross-sell to different segment |
| Q4 — Problem products | 41 | Low sentiment + low loyalty — worst in class | Quality review or discontinuation decision |

> **Finding:** Products 9597 (VADER = 0.245, 30% negative) and 3165 (VADER = 0.262, 40% negative) are the highest-priority products for executive review. Product 3165 is particularly critical: it has the second-worst sentiment score in the dataset but serves customers with above-average loyalty (mean 1,764 pts) — a Q2 churn risk product, high-value customers being disappointed. Product 4047 (VADER = 0.363, loyalty = 2,687 pts) is the most commercially dangerous despite not appearing in the worst-10 sentiment list.

---

# Part 7 — Four business questions: definitive answers

## Q1 — How do customers engage with and accumulate loyalty points?

Loyalty accumulation is determined primarily by two independent behavioural and financial factors: spending score (r = 0.672) and annual remuneration (r = 0.616). Together these variables explain 82.7% of all variance in loyalty points (MLR R² = 0.827, both predictors p < 0.001, VIF = 1.00).

The relationship is **multiplicative, not additive**: customers with both high pay *and* high spend generate 5.3× more loyalty than customers with both low. The decision tree identifies **spend = 67** as the single most important threshold — crossing it more than doubles predicted loyalty from 1,432 to 3,021+ points.

Age has zero practical influence (r = -0.042, R² = 0.002, p = 0.059). The loyalty programme is age-neutral in practice. Gender also has no significant effect (t-test p = 0.363). Education influences loyalty only insofar as it is a proxy for income.

> **Answer:** Customers accumulate loyalty by spending more (spending score) *and* earning more (remuneration). Neither variable alone is sufficient for premium loyalty — both must be high simultaneously.

## Q2 — How can customers be segmented, and which groups should marketing target?

Five distinct segments were identified using K-Means clustering (k = 5 confirmed by elbow and silhouette methods, all cluster silhouette scores > 0.52). The segments are non-overlapping and statistically well-defined with narrow bootstrap confidence intervals.

- **Priority 1 — Retain:** Segment A (VIP, n = 356). These customers hold a disproportionate share of programme value. Any churn in this group is immediately commercially damaging.
- **Priority 2 — Activate:** Segment C (Untapped, n = 269). High-income customers with income equivalent to VIP but 77% lower loyalty. The most commercially valuable activation opportunity — 6.5× CLV uplift potential.
- **Priority 3 — Grow:** Segment B (Growth, n = 774). Largest segment with the greatest volume upside. Spend incentives targeting the 50–67 spend score range push this group above the critical threshold.

> **Answer:** Five segments exist with clear commercial strategies. Segment C (high income, disengaged) is the single highest-priority marketing target. Segment A requires platinum-tier retention. Segment B offers the largest volume growth opportunity.

## Q3 — How can text data inform marketing campaigns and business improvements?

Customer reviews provide three distinct types of commercially actionable intelligence.

**First, brand voice:** the most frequent positive words (great, fun, love, easy, perfect) should anchor all campaign copywriting and product listing language.

**Second, product risk intelligence:** the product sentiment matrix identifies 5 products needing urgent quality review and 48 products in the Q2 churn risk quadrant (low sentiment, high-value customers). These products require immediate product development escalation.

**Third, demographic theme intelligence:** the prominence of "kids" and "family" in review language confirms a strong family gaming segment not visible in the demographic variables. A dedicated family gaming campaign track is justified.

> **Answer:** Review text data provides actionable campaign language, product quality risk signals, and demographic theme intelligence unavailable from structured data alone. The immediate priority is resolving Q2 churn risk products before high-value customer churn materialises.

## Q4 — Is loyalty data suitable for predictive modelling?

With transformation, yes. Without transformation, only conditionally. Raw loyalty is highly right-skewed (skewness = 1.465, kurtosis = 1.716) and fails normality tests (Shapiro-Wilk and D'Agostino-Pearson, both p < 0.001). This violates OLS assumptions and produces heteroscedastic residuals.

Log(loyalty) reduces skewness to 0.070 (95% improvement) and produces approximately normal residuals. The log-transformed MLR model achieves R² = 0.810 with valid OLS assumptions for the pay-only variant.

Tree-based models (decision tree, random forest, gradient boosting) are fully suitable with raw loyalty — they require no normality assumptions and achieve R² > 0.94 with stable cross-validation.

> **Answer:** Raw loyalty should **not** be used for OLS regression without log-transformation. Log(loyalty) is appropriate for linear models. Raw loyalty is fully suitable for ensemble and tree-based models, which achieve superior predictive accuracy (R² = 0.94 vs 0.83 for OLS). Recommend deploying gradient boosting in the CRM system.

---

# Executive conclusion

The customer analytics project has delivered a comprehensive, statistically rigorous analysis that directly answers all four business questions and goes substantially beyond the original brief.

> **Boardroom summary:** The business has a healthy, broadly satisfied customer base (NPS proxy 74.3, 89% positive reviews) with a clear and quantifiable loyalty formula: loyalty = f(income, spending behaviour). The programme is commercially concentrated — the top 25% of customers hold 54% of all value — creating both risk and opportunity. The three highest-priority actions are: (1) protect the top 200 customers with a Platinum Shield Programme, (2) activate 269 high-income disengaged customers through targeted engagement campaigns, and (3) urgently resolve quality issues with 5 products currently disappointing high-value customers. Implementing these three actions, supported by deploying the gradient boosting prediction model (R² = 0.94) in the CRM system, represents the highest-confidence path to improving overall sales performance within 12 months.

---

## Limitations

- No transaction dates — every finding is a snapshot. No trends, no cohorts, no causation.
- No revenue figures — loyalty is used as a proxy for spend, and proxies drift from what they proxy.
- Loyalty points are mechanically awarded, so the measure partly reflects the programme's own rules rather than brand affinity.
- Approximately 10 reviews per product — product-level conclusions rest on thin evidence.
- Heteroscedasticity present in all raw-loyalty OLS models; robust standard errors or log transformation required for valid inference.

---

*Academic project completed as part of the Executive Programme in Business Analytics, London School of Economics.*
