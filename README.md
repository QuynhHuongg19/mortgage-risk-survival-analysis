# Mortgage Default & Prepayment
## Survival & Competing Risks Analysis

An end-to-end credit risk analytics project analyzing the timing of mortgage default and voluntary prepayment using Freddie Mac Single-Family Loan-Level data.

The project applies survival analysis and competing-risks methods to estimate default risk over the mortgage life cycle, identify key risk drivers, and compare mortgage vintages from 2016 to 2026.

---

## Project Overview

Traditional default analysis often treats mortgage default as a binary outcome. This project instead approaches mortgage default as a time-to-event problem.

A mortgage may terminate through more than one event. In particular, voluntary prepayment prevents a future default from occurring on the same loan. Therefore, prepayment is explicitly modeled as a competing event rather than simply treated as ordinary censoring.

The analytical framework combines:

- Kaplan-Meier survival analysis
- Cox Proportional Hazards model
- Cause-Specific Cox models
- Aalen-Johansen cumulative incidence estimation
- Fine-Gray competing-risks model
- Time-Varying Cox model
- Mortgage vintage analysis

The main risk measure is the cumulative incidence of default:

**Default CIF(t) = P(T ≤ t, J = Default)**

which is interpreted as cumulative default probability over time.

---

## Research Questions

The project focuses on four main questions:

1. How do borrower and loan characteristics affect mortgage default risk?

2. How does cumulative default probability evolve over loan age, particularly at 12-, 24-, and 36-month horizons?

3. How do default-risk estimates change when voluntary prepayment is explicitly treated as a competing event?

4. How does default risk differ across mortgage origination vintages from 2016 to 2026 at comparable loan ages?

---

## Dataset

**Source:** Freddie Mac Single-Family Loan-Level Dataset

**Sample vintages:** 2016–2026

**Performance observation cutoff:** March 31, 2026

### Validated analytical scale

| Component | Observations |
|---|---:|
| Origination records | 512,500 |
| Monthly performance records | 20,097,384 |
| Event-mapped loans | 512,489 |
| Survival-eligible loans | 504,405 |
| Complete baseline model cases | 499,393 |
| Default events | 16,556 |
| Voluntary prepayment events | 206,175 |
| Right-censored loans | 281,674 |

The 2026 vintage is only partially observed because of the March 2026 performance cutoff.

Raw Freddie Mac files and generated large datasets are intentionally excluded from this repository.

---

## Event Definition

Each mortgage is assigned one terminal outcome:

| Code | Event |
|---:|---|
| 0 | Right Censoring |
| 1 | Default |
| 2 | Voluntary Prepayment |

Default is primarily identified using serious delinquency information, with supporting zero-balance and distressed-loan indicators where applicable.

Voluntary prepayment is identified separately as a competing termination event.

When multiple event signals occur in the same month, the project applies a deterministic event-priority rule:

**Default > Prepayment > Censoring**

---

## Methodology

### 1. Kaplan-Meier Survival Analysis

Kaplan-Meier estimation is used to describe survival patterns over loan age.

Under competing risks, however:

**1 - KM(Default) is not treated as the official cumulative default probability.**

Ignoring competing prepayment may overstate cumulative default incidence.

---

### 2. Aalen-Johansen / Cumulative Incidence Function

The competing-risks framework estimates separate cumulative incidence functions for:

- Default
- Voluntary Prepayment

Default CIF is used as the primary cumulative PD measure.

Key horizons include:

- PD 12 months
- PD 24 months
- PD 36 months

A 60-month horizon can be used as supplementary analysis when sufficient follow-up is available.

---

### 3. Cox Proportional Hazards Model

The baseline Cox model evaluates the association between mortgage characteristics and the instantaneous default hazard.

Core covariates include:

- Credit Score
- Original Loan-to-Value ratio
- Original Debt-to-Income ratio
- Original Interest Rate
- Original Loan Term

Model effects are interpreted using hazard ratios.

---

### 4. Cause-Specific Cox Models

Separate cause-specific hazard models are estimated for:

- Default
- Voluntary Prepayment

These models analyze the instantaneous occurrence rate of each event among loans that remain event-free.

---

### 5. Fine-Gray Competing-Risks Model

The Fine-Gray model estimates the subdistribution hazard associated with the cumulative incidence of default while accounting for voluntary prepayment as a competing event.

Fine-Gray subdistribution hazard ratios are interpreted separately from standard Cox hazard ratios and should not be interpreted directly as absolute default probabilities.

---

### 6. Time-Varying Cox Model

Monthly loan performance allows selected borrower states to change through time.

Time-varying covariates include information such as:

- Lagged unpaid principal balance
- Current interest rate
- Delinquency indicators

This extends the analysis beyond origination-only borrower characteristics.

---

### 7. Vintage Analysis

Mortgage cohorts from 2016 through 2026 are compared at equivalent loan ages.

This avoids directly comparing portfolios simply by calendar date and allows differences in cumulative default incidence across origination cohorts to be examined more consistently.

---

## Data & Modeling Pipeline

```text
Freddie Mac ZIP Files
        |
        v
Data Ingestion
        |
        v
Data Cleaning & Standardization
        |
        v
Event Mapping
(Default / Prepayment / Censoring)
        |
        v
Survival Duration Construction
        |
        v
Model Dataset
        |
        +-----------------------+
        |                       |
        v                       v
Survival Models          Competing-Risk Models
KM / Cox PH              Aalen-Johansen / Fine-Gray
        |                       |
        +-----------+-----------+
                    |
                    v
             Vintage Analysis
                    |
                    v
          Validation & Production
                    |
                    v
            Streamlit Dashboard