# Housing Prices During the Full-Scale Invasion in Ukraine

**Ukrainian Catholic University | Faculty of Applied Sciences**  
*IT & Business Analytics Program*  
**Authors:** Olena Tumak, Yuliia Pasternak, Yuliana Vus  
**Date:** April 2026  

---

## 1. Motivation and Hypotheses

* **The Problem**: Standard econometric models fail to explain Ukraine's real estate pricing in the current wartime economy (May 2023 – Jan 2026).
* **The Solution**: A War-Adjusted Pricing Model that quantifies the exact extra value paid for safety and utility independence.

### Stakeholders
* **Developers**: Maximize ROI by identifying the most profitable features (autonomy_power, safety factors).
* **Banks & Fintech (eOselia)**: Refine collateral valuations by factoring in energy and security risks.
* **PropTech (LUN, OLX)**: Enhance platforms by introducing a "Utility & Safety Score".
* **Sellers & Landlords**: Optimize pricing with data-driven markups for autonomous features.

### Core Hypotheses
* **H1 (Standard Structural)**: Basic physical features (area, renovation) still have major impact despite the war.
* **H2 (Autonomy)**: Properties with autonomous utilities have positive impact on price that peaks after severe blackouts.
* **H3 (Physical Security)**: Proximity to subway stations (bomb shelters) and lower floors create distinct safety value.
* **H4 (Institutional)**: The "єОселя" state mortgage program causes sellers to artificially inflate prices due to bureaucratic costs.
* **H5 (Geospatial)**: Frontline-adjacent market (Kyiv) and rear-region market (Lviv) exhibit fundamentally different pricing dynamics.

---

## 2. Literature Review

Our literature review focuses on research methods, including textbooks, papers, and videos about OLS regression and hedonic pricing models. We study variable selection, result interpretation, and implementation of time and city fixed effects. Additionally, we review real estate studies from crisis periods (economic crises, natural disasters) to understand how pricing factors change under extreme conditions.

**Key sources**:
* Article: "The impact of the war in Ukraine on the residential real estate market on the example of Szczecin, Poland"
* *Data Science for Business* (Tom Fawcett)
* *Introductory Econometrics* (Jeffrey Wooldridge)
* *Econometric Analysis of Cross Section and Panel Data* (Jeffrey Wooldridge)
* Christine Jiang data cleaning and analysis video resources

---

## 3. Data Description

### Data Overview
* **Source**: NGO "LUN Misto" (under UCU research agreement)
* **Scope**: Pooled cross-sectional data for Lviv & Kyiv (Sales and Rentals)
* **Timeframe**: May 2023 – January 2026 (Wartime period)
* **Structure**: Monthly snapshots (24th of each month) of unique listings

### Key Variable Categories
* **Core Specs**: Price, total/living area, room count, floor level, year built
* **Location**: District, neighborhood, subway proximity
* **Property Quality**: Renovation type, furniture, parking, pet policies
* **Autonomy (Research Focus)**: Autonomous electricity, heating, water, internet, elevators
* **Market Flags**: "By owner" listings and "eOselya" mortgage eligibility

### Temporal Coverage
Dataset periods with varying completeness:
* **Before 2023-10-24**: Prices only in `price_uah`/`price_usd`, no separate currency field.
* **2023-10-24 to 2025-01-24**: Moderate completeness.
* **After 2025-02-24**: Most complete fields.

---

## 4. Data Preparation

### Initial Data Exploration
Conducted exploratory analysis to:
* Distinguish numerical vs. categorical variables.
* Examine value distributions.
* Identify formatting inconsistencies (especially time variables: `add_time`, `update_time`).
* Detect missing values and anomalies.

### Data Cleaning and Validation

**Logical Consistency Checks**  
Removed observations with:
* Floor exceeding `floor_count`
* Unrealistic floor counts for city context
* `area_total` < `area_living` or `area_total` < `area_living` + `area_kitchen`
* Implausible prices (e.g., placeholder 999999)
* `build_year` later than listing date
* Unrealistic construction years
* Incorrect geographic coordinates

**Outlier Removal**  
Filtered out:
* Extremely small/large total areas
* Implausible price-per-square-meter values
* Unusually expensive properties (non-representative)
* Likely misclassified properties (houses as apartments)

**Deduplication**  
Multi-step process:
1. Identified listings from same building (location + structural features).
2. Within buildings, identified same apartments.
3. Retained observations with different timestamps/prices as distinct market events.

### Handling Missing Values
* **Binary Features**: Missing = 0 (assumption: if not mentioned, feature absent).
* **Categorical**: `wall_type`, `house_type` → "Unknown"; `heat_type` → dummies with central heating baseline.
* **Location**: `microdistrict` imputed via decision-tree logic (street, district, coordinates); insufficient location detail → removed.
* **Numerical**: `build_year` imputed by `house_type` × `district` median; `ceiling_height` by `house_type` median.

### Feature Engineering
Created variables:
* `log(price)`, `log(price per sqm)` — using USD for stability.
* Monthly missile/drone strikes per city (manually from ACLED).
* Average monthly USD exchange rate (from НБУ API).

### Dataset Splitting
Four subsets to account for structural pricing differences:
1. Lviv — Rent
2. Lviv — Sale
3. Kyiv — Rent
4. Kyiv — Sale

### Final Validation
After preprocessing:
* <10% observations removed.
* All regression variables converted to numeric.
* Missingness minimized and controlled.
* Dataset verified for econometric analysis.

---

## 5. Exploratory Data Analysis (EDA)

### Geospatial Price Distribution
* **Kyiv**: Highly centralized high-price core on Right Bank (city center + wartime factors: deeper subway shelters, avoiding bridge crossings during air raids).
* **Lviv**: Strong concentration of expensive properties in historic/central districts; clear price drop in periphery.

> *(Note: You can add your images here by replacing the path)*  
> `![Kyiv Rent](geospatial_kyiv_rent.png)`  
> `![Kyiv Sale](geospatial_kyiv_sale.png)`  
> `![Lviv Rent](geospatial_lviv_rent.png)`  
> `![Lviv Sale](geospatial_lviv_sale.png)`

### Market Dynamics
Prices in all four segments show steady upward trend in USD, following USD/UAH exchange rate. Notably, prices did NOT crash permanently during heavy missile strike months — market adapted by pricing risks into specific features rather than overall value drops.

### Positive Features During Blackouts
Massive price divergence starting late 2024 between apartments WITH autonomous features (lift, internet, water, power) vs. WITHOUT them. Market dynamically re-priced these features after severe infrastructure shocks.

> `![Autonomy Premium Emergence](autonomy_premium_emergence.png)`

### Structural Confounders
Apartments with gas (`has_gas`) or sold by owners (`is_owner`) appear cheaper in simple charts — BUT this is confounded: gas mostly in old, cheap Soviet housing; modern expensive buildings are fully electric. Requires regression controls for building age and renovation quality.

### Institutional Market Distortions
"eOselia"-eligible listings show persistently higher prices, confirming sellers inflate prices to offset 7.5% tax burden and banking bureaucracy.

> `![eOselia Price Markup Effect](eoselia_markup.png)`

### High-Rise Paradox
Despite blackout/drone strike risks, high-rises (25+ floors) remain most expensive, especially in sales.

### Market Liquidity
* **Kyiv**: Severe sales supply (15k–25k active listings) + tight rental market → preference for rental flexibility.
* **Lviv**: Acute "rental squeeze" (sharp drop in rental listings) in late 2025 → recent price surges.

> `![Lviv Rental Market Anomaly Gap](lviv_rental_gap_2025.png)`

---

## 6. Methodology

### Variable Classification
* **Structural**: `log_area_total`, `room_count`, `floor`, `floor_count`, `building_age`, `ceiling_height`, `wall_type`, `house_type`
* **Quality & Amenities**: `is_without_renovation`, `is_babushka_renovation`, `has_furniture`, `has_parking`, `pets_allowed_full`
* **War-Related & Autonomy**: `autonomy_power`, `autonomy_heat`, `autonomy_water`, `autonomy_net`, `n_strikes`
* **Location & Security**: `subway_500m` (shelter proxy for Kyiv), `microdistrict`
* **Financial & Institutional**: `usd_rate_avg`, `is_owner`, `has_eoselia`

**Dependent variable**: `log(price per square meter)`

### Model 1: Pooled OLS
**Objective**: Baseline estimation of market premiums for specific features.

$$ \log(p_i) = \beta_0 + \sum_{k} \beta_k X_{k,i} + \sum_{j} \gamma_j D_{j,i} + \varepsilon_i $$

Where $X_{k,i}$ = continuous structural variables, $D_{j,i}$ = binary dummies.

### Model 2: Two-Way Fixed Effects
**Objective**: Control for unobserved spatial/temporal characteristics.

$$ \log(p_{ist}) = \beta X_{ist} + \alpha_t + \gamma_s + \varepsilon_{ist} $$

$\alpha_t$ = Snapshot Date Fixed Effects, $\gamma_s$ = Microdistrict Fixed Effects.

### Model 3: Dynamic Difference-in-Differences
**Objective**: Evaluate causal impact of shocks over time.

$$ \log(p_{ist}) = \beta X_{ist} + \sum_{y} \tau_y (\text{Treat}_i \times \text{Year}_y) + \alpha_t + \gamma_s + \varepsilon_{ist} $$

$\tau_y$ = additional value for each year relative to baseline.

### Model 4: Time-Series Macro-Interaction
**Objective**: Quantify relationship between security variables and property features.

$$ \log(p_{ist}) = \beta X_{ist} + \theta (\text{Autonomy}_i \times \text{Strikes}_t) + \gamma_s + \varepsilon_{ist} $$

$\theta$ = dynamic premium from "fear".

---

## 7. Results

### Pooled OLS — Lviv
**Model Fit**: R² = 0.249 (Rent), 0.303 (Sale); N = 117,341 (Rent), 133,811 (Sale)

**Energy Autonomy — Core Hypothesis Confirmed**

| Variable | Lviv Rent | Lviv Sale |
| :--- | :--- | :--- |
| `autonomy_power` | +25.6%*** | +15.7%*** |
| `autonomy_net` | +7.0%*** | +6.0%*** |
| `autonomy_lift` | +6.3%*** | +4.4%*** |
| `autonomy_heat` | -5.5%*** | -1.5%* |

Autonomous electricity is the largest positive structural coefficient in both segments. The premium is sharper in rent (+25.6\%) than sale (+15.7\%). The negative `autonomy_heat` reflects redundancy with the heating-system block, not a true negative effect.

**Cross-Segment Sign Reversals**

| Variable | Rent | Sale |
| :--- | :--- | :--- |
| `log_area_total` (elasticity) | -0.19*** | +0.09*** |
| `has_furniture` | -6.3%*** | +10.4%*** |

Rent shows canonical scale discount; Sale shows scale premium. Furniture inverts identically: negative-selection signal in rent, inclusion premium in sale.

**Institutional Effect**  
`has_eoselia` = +10.3%*** (Sale only), confirming seller-markup hypothesis tied to state mortgage program.

### Pooled OLS — Kyiv
**Model Fit**: R² = 0.589 (Rent), 0.575 (Sale); N = 191,439 (Rent), 484,565 (Sale)
* Kyiv R² is approximately 2× the Lviv R² — pricing in Kyiv is far more systematically explained by observable characteristics.

**Energy Autonomy**

| Variable | Kyiv Rent | Kyiv Sale |
| :--- | :--- | :--- |
| `autonomy_power` | +9.0%*** | +7.4%*** |
| `autonomy_lift` | +4.6%*** | +5.6%*** |
| `autonomy_heat` | +1.3%*** | +2.8%*** |
| `autonomy_net` | n.s. | +3.3%*** |

The autonomy premium is roughly one-third of Lviv magnitude. Two explanations: (i) autonomous systems closer to market standard in Kyiv's newer stock; (ii) Lviv premium partly driven by IDP willingness-to-pay for resilience.

**Security Premium — Subway as Shelter**

| Variable | Kyiv Rent | Kyiv Sale |
| :--- | :--- | :--- |
| `subway_500m` | +7.3%*** | +6.7%*** |

Direct empirical confirmation of shelter-proximity hypothesis. Properties within 500m of metro station command 6.7–7.3% premium — one of cleanest results in entire study.

**Institutional Effect**  
`has_eoselia` = +6.1%*** (Sale only), smaller than Lviv (+10.3%) but tightly identified (z = 36.7).

### Fixed Effects Model
**Model Fit**: R² increased modestly across all segments after adding time FE.

**Key Robustness Findings**

| Finding | Pooled OLS | Fixed Effects | Verdict |
| :--- | :--- | :--- | :--- |
| `autonomy_power` Lviv | +25–28%*** | +25%*** | Robust |
| `autonomy_power` Kyiv Rent | +13%*** | insig. | Not robust |
| `subway` × `strikes` | -0.003*** | insig. | Time-driven |
| `floor` × `strikes` Kyiv | -0.001*** | -0.001*** | Robust |
| eOselia markup | +5–11%*** | +9–6%*** | Robust |

* **Critical finding**: `autonomy_power` in Kyiv Rent becomes insignificant after time FE — suggesting effect was captured by time trends rather than structural premium. Subway × strikes interaction is also time-driven.

### Temporal Dynamics
* **Kyiv Sale** — most striking pattern: persistent negative territory throughout 2023–2024. Market was in decline for nearly two years (mid-2023 through early 2025), only beginning recovery in late 2025. This directly reflects buyer uncertainty and risk aversion in frontline-adjacent market.
* **Lviv Sale** — opposite pattern: persistent positive momentum from late 2023 onwards, appreciating by 25% in real USD terms over study period.

### Interaction Model Results
* `n_strikes_lag1` positive everywhere — lagged attacks drive prices up through IDP demand.
* **Autonomy × strikes interaction**: insignificant in all 4 models — premium is structurally embedded, not dynamically responsive to attack intensity.
* **Subway × strikes interaction**: negative in Pooled OLS but becomes insignificant in FE — compression effect was time-driven, not genuine interaction.
* **Floor × strikes interaction**: remains significant in Kyiv (-0.001***) — upper floors additionally penalized during high-strike periods. Robust to time controls.

---

## 8. Conclusions

### Core Findings
1. **Autonomy Premium Confirmed**: Autonomous electricity commands significant premium across all market segments, with Lviv showing roughly 3× higher premium than Kyiv (25.6% vs 9.0% in rent). Premium is structural and stable, not dynamically responsive to strike intensity.
2. **Security Premium (Kyiv)**: Metro proximity commands 6.7–7.3% premium in Kyiv, providing direct evidence of shelter-proximity pricing. Effect robust across specifications.
3. **Institutional Distortion**: eOselia program generates artificial 6–10% price markup, stronger in Lviv than Kyiv.
4. **Market Divergence**: Kyiv sales market experienced sustained two-year decline (2023–2025), only recovering in late 2025. Lviv showed opposite pattern with persistent growth. This reflects fundamental differences in risk perception and buyer confidence between frontline-adjacent and rear markets.
5. **Cross-Segment Reversals**: Area elasticity and furniture premium consistently reverse sign between rent and sale markets, reflecting different valuation logic for short vs. long-horizon contracts.

### Hypothesis Outcomes
* **H1 (Structural)**: Confirmed — standard features remain fundamental price drivers.
* **H2 (Autonomy)**: Confirmed with modification — premium exists but is structurally embedded rather than peaking during attacks.
* **H3 (Security)**: Partially confirmed — shelter proximity premium exists in Kyiv; floor-level safety penalty appears only during high-strike periods.
* **H4 (Institutional)**: Confirmed — eOselia generates price markup.
* **H5 (Geospatial)**: Confirmed — Kyiv and Lviv exhibit fundamentally different pricing dynamics and temporal trajectories.

### Stakeholder Implications
* **Developers**: Autonomous electricity is single most valuable war-related feature. Premium 3× higher in Lviv — prioritize installation in western developments.
* **Banks/eOselia**: Confirmed 6–10% markup exists. Adjust collateral valuations accordingly.
* **PropTech**: Metro proximity and autonomy features should receive explicit scoring in Kyiv/Lviv platforms.
* **Sellers**: Autonomy features command measurable premium — document and advertise them prominently, especially in rental market.

---

## Bibliography

1. The impact of the war in Ukraine on the residential real estate market on the example of Szczecin, Poland
2. Fawcett, T. *Data Science for Business: What You Need to Know about Data Mining and Data-Analytic Thinking*
3. Wooldridge, J. M. *Introductory Econometrics: A Modern Approach*
4. Wooldridge, J. M. *Econometric Analysis of Cross Section and Panel Data*
