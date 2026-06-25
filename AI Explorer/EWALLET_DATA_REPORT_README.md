# Automated Visual Data Report: E-Wallet Adoption Growth in the Philippines and Its Implications for Financial Inclusion (2020–2025)

> A portfolio submission documenting AI-assisted data cleaning, analytical visualization, and human-interpreted policy analysis for the Mindanao Regional Development Council. Prepared by a Senior Data Analyst in partial fulfillment of university data analytics portfolio requirements.

**Analyst Role:** Senior Data Analyst, Regional Development Council — Mindanao
**Dataset Period:** 2020–2025
**Geographic Focus:** Philippines, with emphasis on Mindanao (Region X, XI, XII, CARAGA, BARMM)
**Tools Applied:** AI-assisted data cleaning pipeline, Chart.js visualization, human analytical interpretation
**Policy Alignment:** BSP Digital Payments Transformation Roadmap 2020–2023; National Strategy for Financial Inclusion 2022–2028

---

## Table of Contents

1. [Dataset Focus](#dataset-focus)
2. [Data Cleaning Protocol Log](#1-data-cleaning-protocol-log)
3. [Cleaned Dataset Preview](#2-cleaned-dataset-preview)
4. [Visualization 1 — E-Wallet Adoption Growth (2020–2025)](#3-visualization-1--e-wallet-adoption-growth-20202025)
5. [Visualization 2 — E-Wallet Usage by Age Group](#4-visualization-2--e-wallet-usage-by-age-group)
6. [Human Analytical Narrative](#5-human-analytical-narrative)
7. [Policy Implications](#6-policy-implications)
8. [Limitations of AI Analysis](#7-limitations-of-ai-analysis)
9. [Conclusion](#8-conclusion)

---

## Dataset Focus

This report analyzes a composite dataset tracking e-wallet adoption rates, active user counts, and demographic usage patterns across Philippine regions from 2020 to 2025. The dataset draws from simulated survey-based and transactional records consistent with the data architecture of the Bangko Sentral ng Pilipinas (BSP) Financial Inclusion Surveys (2019, 2021) and BSP e-Money Issuer Reports, supplemented with regional disaggregations from the Philippine Statistics Authority (PSA).

E-wallet adoption has emerged as one of the most consequential structural shifts in Philippine financial behavior since the passage of the National Payment Systems Act (RA 11127) in 2018. Platforms such as GCash and Maya have moved beyond peer-to-peer remittances into merchant payments, government cash transfer disbursements (particularly under the Ayuda programs of 2020–2021), microinsurance, and small-ticket lending. For the approximately 56 percent of Filipino adults who remained unbanked as of the 2021 BSP Financial Inclusion Survey, e-wallets have functioned as a first point of contact with formal financial infrastructure — a bridging mechanism rather than a substitute for full banking access.

For policymakers in Mindanao — where financial exclusion rates are structurally higher than the national average, connectivity gaps persist in island and upland communities, and the MSME sector remains predominantly cash-dependent — the trajectory of e-wallet adoption is not merely a technology story. It is a development story, with direct implications for poverty reduction, MSME resilience, disaster-response cash transfer efficiency, and the long-term viability of LGU-level digital governance programs. Understanding the pace, distribution, and demographic composition of adoption is a prerequisite for designing interventions that close rather than widen the digital financial divide.

---

## 1. Data Cleaning Protocol Log

### Raw Input Problems

Upon initial ingestion of the raw CSV file (`ph_ewallet_adoption_raw.csv`), automated diagnostic checks identified the following data quality issues:

| Issue Category | Description | Affected Records | Severity |
|---|---|---|---|
| Missing Values | `active_users` field blank for 14 records; `adoption_rate_pct` null for 6 records across BARMM rows (2020–2021) | 20 records | High |
| Duplicate Records | 7 exact-duplicate rows identified (same region, year, and metric values), likely from a double-export event during data compilation | 7 records | High |
| Inconsistent Percentage Formats | `adoption_rate_pct` field contained mixed formats: decimal (0.312), percentage string ("31.2%"), and integer (31) across different regional entries | 38 records | Medium |
| Formatting Corruption | 3 region name entries contained trailing whitespace or encoding artifacts: `"Region XI "`, `"Davao  Region"`, `"CARAGA\xa0"` — causing groupby operations to create phantom duplicate categories | 3 records | Medium |
| Mixed Date Formats | `survey_year` field used both 4-digit integer (2021) and string formats ("CY2021", "FY 2021") inconsistently across source batches | 12 records | Medium |
| Outlier Values | Two `active_users` entries for NCR (2023) recorded at 10x the plausible range relative to prior years, suggesting a unit conversion error (thousands vs. units) | 2 records | High |

**Total raw records:** 312
**Records with at least one quality issue:** 74 (23.7%)

---

### AI Cleaning Instruction

The following prompt was submitted to the AI data cleaning pipeline after manual diagnostic review:

```
SYSTEM: You are a data cleaning specialist. The attached CSV file contains regional e-wallet adoption data for the Philippines from 2020 to 2025.

Perform the following cleaning operations in order and log each action taken:

1. MISSING VALUES:
   - For numeric fields (adoption_rate_pct, active_users): impute missing values using the regional mean for the same year where at least 3 comparable records exist. If fewer than 3 comparators exist (as in BARMM 2020), flag the record as [IMPUTED - INSUFFICIENT COMPARATORS] and use the national mean for that year as a fallback. Do not silently fill; every imputed value must be logged.

2. DUPLICATES:
   - Remove exact duplicate rows (all fields identical). Retain the first occurrence. Log removed duplicates with row index references.

3. PERCENTAGE FORMAT STANDARDIZATION:
   - Convert all adoption_rate_pct values to a consistent two-decimal float (e.g., 0.312 → 31.20, "31.2%" → 31.20, 31 → 31.00). Final field unit: percentage points, not decimal proportion.

4. REGION NAME NORMALIZATION:
   - Strip all leading/trailing whitespace. Remove non-standard Unicode characters. Standardize to PSA official region names: "Davao Region" (not "Region XI"), "Northern Mindanao" (not "Region X"), "SOCCSKSARGEN" (not "Region XII"), "CARAGA", "BARMM".

5. DATE FORMAT STANDARDIZATION:
   - Convert all survey_year values to 4-digit integer format. Remove "CY", "FY", and whitespace prefixes.

6. OUTLIER TREATMENT:
   - Flag active_users values exceeding 3 standard deviations from the regional 5-year mean. Do not automatically remove; present flagged records for human review with a notation: [OUTLIER - HUMAN REVIEW REQUIRED].

7. VALIDATION:
   - Confirm adoption_rate_pct is bounded between 0 and 100 for all records.
   - Confirm survey_year is within the range 2020–2025.
   - Confirm no null values remain in key fields after imputation.
   - Output a cleaning summary report alongside the cleaned CSV.
```

---

### Cleaning Actions Performed

**Missing Value Treatment**

Fourteen records with missing `active_users` values were imputed using the regional mean for the corresponding survey year, calculated from all non-null records within the same region. For BARMM 2020 and 2021 (where the regional mean was unavailable due to fewer than three comparator records), the national mean active user count for those years was applied as a fallback. All imputed values are flagged in the cleaned dataset with the notation `[IMPUTED]` in the `data_quality_flag` column. Six null `adoption_rate_pct` values in the BARMM rows were treated identically.

**Duplicate Removal**

Seven exact-duplicate rows were identified and removed, retaining the first occurrence per row index. Removed row indices were logged in `cleaning_log.txt`. Post-removal record count: 305.

**Data Normalization**

All `adoption_rate_pct` values were standardized to two-decimal percentage point format (range: 0.00–100.00). The three format variants (raw decimal, percentage string, integer) were converted using a conditional parsing function. Post-normalization, the field range was validated as 8.20 (BARMM, 2020) to 73.40 (NCR, 2025), consistent with plausible adoption trajectories.

**Format Standardization**

Region names were normalized to PSA-standard nomenclature. Three encoding-corrupted entries were corrected. Survey year values were standardized to 4-digit integers across all 12 affected records.

**Validation Checks**

Post-cleaning validation confirmed: zero null values in `adoption_rate_pct`, `active_users`, `region`, and `survey_year`; all percentage values within 0–100 bounds; all year values within 2020–2025. Two NCR active_users records were flagged `[OUTLIER - HUMAN REVIEW REQUIRED]` and retained pending analyst review.

---

### Result Summary

| Metric | Pre-Cleaning | Post-Cleaning |
|---|---|---|
| Total Records | 312 | 305 |
| Records with Quality Issues | 74 (23.7%) | 2 (0.7% — outliers pending review) |
| Null Values in Key Fields | 20 | 0 |
| Duplicate Records | 7 | 0 |
| Non-Standard Region Names | 3 | 0 |
| Non-Standard Date Formats | 12 | 0 |
| Non-Standard Percentage Formats | 38 | 0 |

The cleaned dataset (`ph_ewallet_adoption_cleaned.csv`) is ready for analysis and visualization. Two outlier records in the NCR active_users field remain flagged for human analyst review before inclusion in aggregate national statistics.

---

## 2. Cleaned Dataset Preview

The following table presents a representative sample of 8 records from the cleaned dataset, selected to illustrate geographic and temporal range. All values reflect post-cleaning standardization.

| Year | Region | E-Wallet Adoption Rate (%) | Active Users (thousands) | Data Quality Flag |
|---|---|---|---|---|
| 2020 | NCR | 38.50 | 4,820 | CLEAN |
| 2020 | Davao Region | 18.30 | 892 | CLEAN |
| 2020 | BARMM | 8.20 | 143 | IMPUTED |
| 2021 | Davao Region | 26.70 | 1,341 | CLEAN |
| 2021 | CARAGA | 14.90 | 318 | CLEAN |
| 2022 | Northern Mindanao | 31.40 | 1,108 | CLEAN |
| 2023 | SOCCSKSARGEN | 34.80 | 987 | CLEAN |
| 2024 | Davao Region | 51.20 | 2,673 | CLEAN |
| 2025 | BARMM | 29.60 | 612 | IMPUTED |
| 2025 | CARAGA | 47.30 | 1,089 | CLEAN |

*Active user counts are expressed in thousands. BARMM 2020 and 2025 values were imputed using national mean fallback due to insufficient regional comparators; interpret with appropriate caution.*

---

## 3. Visualization 1 — E-Wallet Adoption Growth (2020–2025)

### Chart Description

This line chart plots e-wallet adoption rates (expressed as a percentage of the adult population with at least one active e-wallet account) from 2020 to 2025 across five Mindanao regions — Davao Region, Northern Mindanao, SOCCSKSARGEN, CARAGA, and BARMM — alongside the national (Philippines) average for comparison. Each region is represented by a distinct line series. The horizontal axis plots survey year (2020–2025); the vertical axis plots adoption rate in percentage points (0–80%). A vertical dashed reference line at 2021 marks the primary COVID-19 pandemic inflection point. A horizontal reference line at 50% marks the BSP DPTR target threshold for digital payment participation.

![E-Wallet Adoption Trend by Region (2020–2025)](images/chart1_adoption_trend.png)

*Figure 1. E-Wallet Adoption Rate (%) by Mindanao Region and National Average, 2020–2025. Source: Simulated dataset based on BSP Financial Inclusion Survey architecture. Dotted vertical line = 2021 pandemic inflection; horizontal line = BSP 50% target threshold.*

---

#### Key Insight

The chart reveals a structural bifurcation in Mindanao's e-wallet adoption trajectory. All five Mindanao regions show a pronounced acceleration in adoption rate between 2020 and 2022 — a pattern consistent with the documented behavioral shift driven by pandemic-era cash transfer programs (Ayuda, SAP) and the enforced reduction in physical cash transactions. However, the post-2022 growth rate diverges sharply between regions. Davao Region and Northern Mindanao approach or cross the 50 percent BSP threshold by 2024–2025, suggesting that urbanization level and existing digital infrastructure are the primary moderators of sustained adoption beyond the initial pandemic-era surge. BARMM, despite registering the strongest relative growth rate from its low 2020 baseline, remains the furthest from the national average throughout the period — a finding that reflects the compounding effect of infrastructure deficits, lower formal financial literacy baselines, and the structural disruption of conflict-affected communities on digital financial behavior. The chart does not show convergence; it shows parallel growth with a persistent and widening absolute gap between Mindanao's most and least digitally included regions.

---

## 4. Visualization 2 — E-Wallet Usage by Age Group

### Chart Description

This grouped bar chart presents e-wallet usage rates by age cohort for Mindanao (aggregated across Region X, XI, XII, CARAGA, and BARMM) for two reference years: 2021 and 2024. Six age cohorts are represented on the horizontal axis: 18–24, 25–34, 35–44, 45–54, 55–64, and 65+. Each cohort displays two bars — one for 2021 (light blue) and one for 2024 (deep blue) — enabling direct comparison of cohort-level adoption change over the three-year period. The vertical axis represents usage rate as a percentage of respondents within each age cohort who reported owning and actively using at least one e-wallet in the prior 30 days.

![E-Wallet Usage Rate by Age Group — Mindanao (2021 vs. 2024)](images/chart2_age_demographics.png)

*Figure 2. E-Wallet Active Usage Rate (%) by Age Cohort, Mindanao Regions, 2021 vs. 2024. Source: Simulated dataset based on BSP Financial Inclusion Survey methodology. "Active use" defined as at least one transaction in prior 30 days.*

---

#### Key Insight

The chart reveals a consistent pattern of age-gradient adoption that has not significantly flattened between 2021 and 2024, despite aggregate adoption growth across all cohorts. The 18–24 and 25–34 cohorts lead adoption in both years, with usage rates approaching or exceeding 65 percent by 2024 — consistent with the demographic profile of GCash and Maya's primary user base as documented in BSP quarterly payment system reports. The critical finding is in the 45–54, 55–64, and 65+ cohorts: while all three show measurable growth from 2021 to 2024, the absolute gap between the youngest and oldest cohorts has widened rather than narrowed. The 65+ cohort in 2024 registers usage rates below the 2021 levels of the 18–24 cohort. This suggests that platform-level growth — driven by younger urban adopters — is masking a persistent inclusion gap among older Mindanao residents, many of whom operate in agricultural and fishing livelihoods where cash remains the dominant transaction medium and where peer-network adoption effects (the primary driver of e-wallet uptake in younger cohorts) are structurally weaker.

---

## 5. Human Analytical Narrative

### Interpreting the Data: Beyond the Growth Headline

The aggregate trajectory of e-wallet adoption in the Philippines — and specifically in Mindanao — is frequently summarized in policy discourse through a single, optimistic headline: adoption has grown dramatically since 2020. That summary is accurate. It is also insufficient. The more consequential story embedded in this dataset is not the rate of growth but its distribution: who is being included in the digital financial system, at what pace, and with what depth of engagement.

The first visualization establishes that the COVID-19 pandemic functioned as an adoption accelerant across all Mindanao regions, compressing what might have been a multi-year gradual adoption curve into an 18-month structural shift. This acceleration was not primarily driven by consumer choice or market maturation; it was driven by necessity and government policy. The Ayuda and Social Amelioration Program cash transfers of 2020–2021 disbursed billions of pesos through GCash and Maya wallets to recipients who had never previously held a digital financial account. For a significant portion of first-time e-wallet users in Mindanao, the government was the onboarding agent, not a commercial platform. This has a critical implication that the adoption rate alone does not capture: account activation is not the same as financial inclusion. A recipient who received a one-time cash transfer through a government-provisioned GCash account and subsequently ceased using the platform has been counted in adoption statistics but has not been integrated into the digital financial ecosystem in any durable sense. The distinction between nominal account ownership and active, habitual digital financial participation is the analytical gap that policy must address.

The second visualization makes this gap concrete through its demographic disaggregation. The persistent and widening usage gap between younger and older cohorts in Mindanao reflects structural barriers that monetary policy and platform design alone cannot resolve. Older residents in agricultural communities — particularly the smallholder farmers, fisherfolk, and informal market vendors who constitute the economic backbone of rural Mindanao — have lower average educational attainment, lower smartphone ownership rates, lower exposure to peer networks that model e-wallet use, and greater reliance on cash-based trust relationships (paluwagan, suki credit systems) that the current e-wallet product architecture does not accommodate. For these populations, the marginal cost of digital financial adoption is not primarily a platform fee or an account minimum; it is the cognitive and social transition cost of replacing deeply embedded financial practices with unfamiliar digital alternatives.

The economic significance of this bifurcation extends beyond financial statistics. Access to digital payments is increasingly a prerequisite for participation in the formal economy: GCash-integrated merchant payment systems, government benefit disbursements, agricultural loan applications through Landbank's e-channels, and disaster-response cash assistance are all moving toward digital-first delivery. A Mindanao resident who remains outside the digital financial system in 2025 is not merely inconvenienced by having to use cash; they are progressively excluded from the administrative and commercial infrastructure of the formal economy. The adoption growth visible in the charts, if unevenly distributed, may ultimately deepen rather than reduce economic inequality in the region — a paradox that demands careful policy attention.

For Mindanao communities specifically, the relevance is compounded by the region's disaster exposure profile. Typhoon Odette (December 2021) and successive flooding events in the Davao Region demonstrated both the potential and the limitations of digital financial infrastructure as a disaster-response mechanism. In barangays where mobile signal was disrupted, digital cash transfers could not be disbursed; in communities where adoption was lowest (older residents, interior barangays), alternative delivery mechanisms had to be maintained at significant logistical cost. A robust financial inclusion strategy for Mindanao must address not only the adoption rate but the resilience and redundancy of the digital financial system under disaster conditions — a dimension that aggregate adoption statistics do not capture.

---

## 6. Policy Implications

The following recommendations are grounded in the analytical findings of this report. They are organized by responsible implementing actor and are designed to be actionable within existing institutional mandates and fiscal constraints.

---

### Recommendation 1 — BSP: Mandate Disaggregated Regional Reporting in the Financial Inclusion Survey

**Actor:** Bangko Sentral ng Pilipinas — Financial Inclusion Office
**Basis:** The current BSP Financial Inclusion Survey provides national and some regional breakdowns but does not consistently publish Mindanao-level disaggregations by age cohort, livelihood type, or barangay classification (urban/rural/agricultural). Without this data, policymakers designing Mindanao-specific interventions are working from national averages that systematically underweight the region's structural exclusion patterns.
**Action:** BSP should expand the FIS sampling frame to enable statistically significant Mindanao-level estimates by province and age cohort, and publish a dedicated Mindanao Financial Inclusion Data Report annually, aligned with the NEDA Regional Development Plan reporting cycle.

---

### Recommendation 2 — LGUs: Integrate E-Wallet Onboarding into Barangay-Level Service Delivery Points

**Actor:** City and Municipal LGUs in Mindanao, in coordination with DILG-XI/XII/X
**Basis:** The data shows that older residents and agricultural community members have the lowest adoption rates and are the most dependent on government cash transfers — the same transfers that are increasingly disbursed digitally. LGUs are the primary touchpoint between government and these populations.
**Action:** Designate Barangay Health Centers, Lingap Centers, and DSWD Pantawid Pamilya registration sites as assisted e-wallet registration hubs, staffed by trained Barangay DigiSugo facilitators (a peer-trust agent model). Allocate Barangay Development Fund allocations for digital literacy facilitation in communities where the 45+ age cohort adoption gap exceeds 30 percentage points.

---

### Recommendation 3 — Financial Institutions: Design Interoperable Products for Agricultural Livelihoods

**Actor:** Rural Banks, Cooperatives, Landbank, and Licensed E-Money Issuers operating in Mindanao
**Basis:** The persistent exclusion of older agricultural workers and fisherfolk reflects a product-market mismatch: current e-wallet architectures are designed for retail consumption patterns (merchant QR payments, online shopping, food delivery) that do not map to the transaction patterns of smallholder farmers (seasonal income, cooperative-mediated sales, crop insurance disbursements).
**Action:** Financial institutions should develop QR Ph-compatible wallet products integrated with existing cooperative and Landbank lending platforms, enabling farmers to receive crop loan disbursements, sell produce through digital payment channels, and access microinsurance through the same interface. Interoperability with RCEF (Rice Competitiveness Enhancement Fund) disbursement channels should be a baseline requirement.

---

### Recommendation 4 — Community Organizations: Deploy Paluwagan-Linked Digital Savings Pilots

**Actor:** Community-Based Organizations, Women's Cooperatives, Barangay Micro Business Enterprise (BMBE) associations
**Basis:** Informal trust-based savings mechanisms (paluwagan) remain the dominant savings instrument for many Mindanao MSMEs and low-income households, particularly among women and older residents. These networks represent existing financial behavior and social trust infrastructure that digital financial platforms have not yet engaged.
**Action:** Partner with GCash's GCredit/GSave product teams and Maya's savings products to pilot paluwagan-linked digital savings circles, where the paluwagan organizer (kasali) functions as a peer-trust financial agent. Formalize these pilots through DTI-XI's BMBE registration program to bring participants into the formal financial system incrementally. Evaluate pilots against a six-month digital transaction frequency metric rather than account creation alone.

---

### Recommendation 5 — BSP and DICT: Prioritize Connectivity Infrastructure in Adoption-Lagging Provinces

**Actor:** BSP (regulatory advocacy), Department of Information and Communications Technology (DICT), NTC
**Basis:** The BARMM adoption gap visible in Visualization 1 cannot be closed by financial product design or community facilitation alone. In municipalities where 4G connectivity is unreliable or absent, e-wallet adoption is structurally constrained regardless of financial literacy investment.
**Action:** BSP should formally endorse DICT's National Broadband Plan implementation in BARMM and CARAGA as a financial inclusion infrastructure priority, and advocate for offline-capable e-wallet transaction protocols (similar to QR-based offline payment pilots in Indonesia) to be mandated for all BSP-licensed e-money issuers operating in Class 4 and Class 5 municipalities.

---

### Recommendation 6 — DTI-XI and LGUs: Tie MSME Digital Onboarding Metrics to Negosyo Center Performance Targets

**Actor:** DTI-XI, Negosyo Centers across Mindanao regions
**Basis:** Negosyo Centers are the most widely distributed MSME service delivery infrastructure in Mindanao. They currently track business registration and loan facilitation metrics but do not systematically monitor digital payment adoption among their MSME clients.
**Action:** Incorporate e-wallet activation rate and 90-day active usage rate as Key Performance Indicators (KPIs) in the Negosyo Center Annual Performance Review framework. Provide Negosyo Center staff with standardized digital onboarding facilitation training accredited by BSP's Financial Education program. Report results to the Regional Development Council as part of the MSME Digital Inclusion Dashboard.

---

## 7. Limitations of AI Analysis

### Data Bias and Representational Gaps

The dataset used in this report is constructed from simulated records modeled on BSP Financial Inclusion Survey architecture. Even when working with authentic BSP data, a structural representational bias exists: financial inclusion surveys tend to oversample urban and peri-urban respondents where field enumeration is logistically easier, resulting in systematic underrepresentation of interior rural barangays, upland communities, and conflict-affected areas in BARMM. AI cleaning and imputation procedures — including the mean-based fallback applied to BARMM 2020 records in this report — cannot correct for this sampling bias; they can only standardize the data that exists. Policy recommendations derived from underrepresentative data will systematically favor already-served populations.

### Missing Contextual Factors

The dataset captures adoption rates and active user counts but does not encode the contextual variables that most strongly predict whether digital financial adoption translates into durable financial inclusion: frequency and volume of transactions, diversity of financial products used (savings, credit, insurance, remittances), household income volatility, and the degree to which digital payments have displaced versus supplemented cash. An AI analysis operating on adoption rate data alone cannot distinguish between a user who transacts digitally five times per week and one who activated an account for a single government transfer and has not used it since. Both appear identically in the adoption statistics; their financial inclusion implications are categorically different.

### Correlation vs. Causation

The positive correlation between COVID-19 pandemic period and e-wallet adoption acceleration, visible in Visualization 1, should not be interpreted causally without additional analysis. The pandemic period coincided with multiple simultaneous adoption drivers: government cash transfer programs, enforced merchant digitalization, platform promotional campaigns, and general behavioral disruption of cash-handling norms. Attributing adoption growth to any single factor — or to "the pandemic" as an undifferentiated event — overstates analytical certainty. Similarly, the age-cohort gradient in Visualization 2 reflects the correlation between age and adoption but cannot, from this dataset alone, establish whether age-related barriers are primarily cognitive (digital literacy), social (peer network effects), economic (smartphone ownership), or structural (product design mismatch). Policy interventions require this disaggregation; the AI visualization cannot provide it.

### Why Human Oversight Remains Necessary

AI-assisted data analysis excels at pattern detection, format standardization, and visualization at scale — tasks that are time-consuming and error-prone when performed manually across large, multi-source datasets. However, the interpretive layer — understanding why a pattern exists, what it means for a specific community, and what policy response is proportionate — requires contextual knowledge that no AI system trained on global datasets can reliably supply for Mindanao-specific conditions. The BARMM adoption gap means something different to a researcher who knows that Marawi City was under siege in 2017, that BARMM's formal financial infrastructure was extensively disrupted, and that trust in formal institutions remains structurally lower in conflict-affected communities than the adoption rate itself communicates. Human oversight is not a quality assurance step performed after AI analysis; it is an irreplaceable analytical capacity that determines whether the output of AI processing is meaningfully useful or merely technically accurate.

---

## 8. Conclusion

This report has demonstrated a complete AI-assisted data analytics workflow — from raw CSV ingestion and systematic data cleaning through visualization production and human-interpreted policy analysis — applied to the question of e-wallet adoption and financial inclusion in Mindanao and the Philippines. The data reveals a regional story that is more complex and more consequential than aggregate national growth rates suggest: adoption is accelerating, but its distribution is uneven across geography and age cohort in ways that, left unaddressed, risk deepening the financial exclusion of the populations that financial inclusion policy exists to serve.

The six policy recommendations advanced in this report are grounded in the specific patterns identified in the data, calibrated to the institutional landscape of Mindanao, and designed to be actionable within existing LGU, BSP, DTI-XI, and community organization mandates. They share a common analytical thread: that the transition from nominal e-wallet account ownership to durable, habitual digital financial participation requires deliberate, targeted investment in the communities and age cohorts where adoption barriers are structural rather than incidental.

The methodological contribution of this report — systematic AI-assisted data cleaning with full audit logging, human oversight flags, and explicit documentation of AI limitations — reflects a professional standard for data analytics practice in Philippine development research. As e-wallet and broader digital finance datasets become richer and more granular, the value of this standard will increase: the risk of acting on uncleaned, unverified, or AI-misinterpreted data in policy contexts is not abstract — it translates directly into misallocated resources, missed populations, and policy interventions that solve the problem as the data described it rather than as the community lives it.

---

*Automated Visual Data Report v1.0*
*Prepared for University Data Analytics Portfolio Submission*
*Regional Development Council — Mindanao*
*Aligned with BSP Digital Payments Transformation Roadmap, NEDA Regional Development Plan 2023–2028, and National Strategy for Financial Inclusion 2022–2028*
*All dataset values are simulated for portfolio purposes based on BSP Financial Inclusion Survey architecture. Charts reference placeholder image files pending dashboard implementation.*
