# Commercial Bank Credit Risk & NPL Monitoring Dashboard

[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![DAX](https://img.shields.io/badge/DAX-Data_Analysis_Expressions-blue?style=for-the-badge)](https://learn.microsoft.com/en-us/dax/)
[![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)](#)

An interactive, executive-grade Business Intelligence dashboard developed in **Power BI** to monitor credit risk exposure, evaluate portfolio health, and uncover default drivers across a **$312.43M** commercial loan portfolio.

---

## Dashboard Preview

![Dashboard Screenshot](Asset/Dashboard_preview.png)

---

## Business Problem & Context

In the banking and credit industry, unmonitored **Non-Performing Loans (NPL)** severely impact capital adequacy, net interest margins, and operational profitability.

Risk management and credit underwriting teams require a unified analytical tool to:
1. Track the macro portfolio health and total capital at risk (**$77.13M** default exposure).
2. Segment high-risk debtor groups across credit grades (**A–G**), loan purposes, and demographic profiles.
3. Establish data-backed debt-to-income (**DTI**) and interest rate thresholds to minimize future loan default rates.

---

## Data Dictionary

| Column | Description | Type |
|--------|-------------|------|
| `person_age` | Borrower age | Integer |
| `person_income` | Annual income (USD) | Integer |
| `person_home_ownership` | Housing status: RENT, OWN, MORTGAGE, OTHER | Categorical |
| `person_emp_length` | Employment length in years | Float |
| `loan_intent` | Loan purpose: PERSONAL, EDUCATION, MEDICAL, VENTURE, HOMEIMPROVEMENT, DEBTCONSOLIDATION | Categorical |
| `loan_grade` | Credit grade: A (best) to G (worst) | Categorical (Ordinal) |
| `loan_amnt` | Loan amount (USD) | Integer |
| `loan_int_rate` | Interest rate (%) | Float |
| `loan_status` | Target: 1 = Default (NPL), 0 = Performing | Binary |
| `loan_percent_income` | Debt-to-Income ratio (loan_amnt / person_income) | Float |
| `cb_person_default_on_file` | Historical default flag: Y = Yes, N = No | Binary |
| `cb_person_cred_hist_length` | Credit history length in years | Integer |

---

## Key Portfolio Metrics (KPIs)

* **Total Disbursed Volume:** $312.43M
* **Overall NPL Ratio (%):** 24.69%
* **Total Non-Performing Exposure:** $77.13M
* **Weighted Average Interest Rate:** 11.01%
* **Total Portfolio Borrowers:** 32,581 debtors

---

## Methodology

**NPL Definition:** Loans with `loan_status = 1` are classified as Non-Performing Loans (defaulted).

**NPL Ratio Calculation:**
```
NPL Ratio = SUM(loan_amnt WHERE loan_status = 1) / SUM(loan_amnt) * 100
```

**Weighted Average Interest Rate:**
```
Weighted Avg Rate = SUM(loan_amnt * loan_int_rate) / SUM(loan_amnt)
```

**Data Quality Notes:**
- No missing values in key fields (`loan_status`, `loan_grade`, `loan_amnt`)
- One missing `loan_int_rate` value (row 39) handled via DAX `DIVIDE` with fallback
- `person_emp_length` contains values up to 123 years (likely data entry anomalies); analysis uses bucketed ranges

---

## Key Insights & Analytical Findings

### 1. Severe Risk Inversion in Subprime Grades (D – G)
While Grades **A** and **B** represent **~63%** of total disbursed capital ($196.5M) with relatively controlled NPL rates (11.1% and 18.3% respectively), Grades **D to G** exhibit alarming default rates ranging from **58% to over 99%**.

| Grade | Portfolio Share | NPL Rate | NPL Exposure |
|-------|----------------|----------|--------------|
| A | 29.5% | 11.1% | $10.2M |
| B | 33.4% | 18.3% | $19.1M |
| C | 19.0% | 22.8% | $13.6M |
| D | 12.6% | 58.0% | $22.8M |
| E | 4.0% | 62.9% | $7.8M |
| F | 1.1% | 70.4% | $2.5M |
| G | 0.4% | 99.9% | $1.1M |

> **Implication:** Grades D–G represent only 18% of portfolio volume but contribute **44% ($34.2M)** of total NPL exposure.

---

### 2. Vulnerability in Tenant Demographics (Status: RENT)
Borrowers who rent their homes show a significantly higher default risk (**38.3% NPL rate**) compared to mortgage (**13.7%**) and home-owning (**5.6%**) segments. In emergency loans (e.g., *Medical* for *Renters*), the NPL ratio climbs to **41.3%**.

| Home Ownership | Borrowers | NPL Rate | NPL Exposure |
|----------------|-----------|----------|--------------|
| RENT | 16,446 | 38.3% | $55.9M |
| MORTGAGE | 13,444 | 13.7% | $19.5M |
| OWN | 2,584 | 5.6% | $1.3M |
| OTHER | 107 | 37.3% | $0.4M |

> **Implication:** Renters hold 46.6% of portfolio volume but account for **72.4% ($55.9M)** of NPL exposure.

---

### 3. Debt-to-Income (DTI) Default Threshold
Correlation analysis confirms that when a borrower's `loan_percent_income` (DTI) exceeds **0.25 (25%)**, default density increases substantially. Borrowers with DTI > **0.35** paired with interest rates above **14%** demonstrate a default probability of **85.2%**.

| DTI Threshold | Borrowers | Default Rate |
|---------------|-----------|--------------|
| DTI > 0.25 | 6,280 | 52.9% |
| DTI > 0.35 | 2,176 | 71.9% |
| DTI > 0.35 + Rate > 14% | 466 | **85.2%** |

---

### 4. Historical Default Flag is a Strong Predictor
Borrowers with `cb_person_default_on_file = Y` (previous defaults) have a **37.8%** default rate vs **18.4%** for clean-file borrowers — a **2.05x risk multiplier**.

| Previous Default | Borrowers | Default Rate |
|------------------|-----------|--------------|
| Y (Yes) | 5,745 | 37.8% |
| N (No) | 26,836 | 18.4% |

> **Implication:** This flag should be a mandatory gate in underwriting scorecards.

---

### 5. Loan Purpose Risk Profile
**Debt Consolidation** loans carry a **33.1%** NPL rate ($16.6M exposure), significantly above portfolio average. **Medical** loans for renters peak at **41.3%**.

| Loan Intent | Borrowers | NPL Rate | NPL Exposure |
|-------------|-----------|----------|--------------|
| DEBTCONSOLIDATION | 5,212 | 33.1% | $16.6M |
| MEDICAL | 6,431 | 28.7% | $14.2M |
| PERSONAL | 5,636 | 24.1% | $12.1M |
| VENTURE | 4,892 | 21.8% | $9.8M |
| EDUCATION | 5,805 | 19.2% | $8.9M |
| HOMEIMPROVEMENT | 4,605 | 15.3% | $5.5M |

---

## Data Pipeline & DAX Architecture

All core business calculations are centralized within an isolated `_Measures` table using optimized DAX expressions:

```dax
// 1. Total Disbursed Loan Volume
Total Loan Amount = 
SUM(credit_risk_dataset[loan_amnt])

// 2. Total Non-Performing Loan Exposure
NPL Amount = 
CALCULATE(
    SUM(credit_risk_dataset[loan_amnt]),
    credit_risk_dataset[loan_status] = 1
)

// 3. Gross NPL Ratio (%)
NPL Ratio % = 
DIVIDE([NPL Amount], [Total Loan Amount], 0)

// 4. Weighted Average Portfolio Interest Rate
Weighted Avg Int Rate = 
VAR TotalAmt = [Total Loan Amount]
VAR WeightedSum = 
    SUMX(
        credit_risk_dataset,
        credit_risk_dataset[loan_amnt] * (credit_risk_dataset[loan_int_rate] / 100)
    )
RETURN
    DIVIDE(WeightedSum, TotalAmt, 0)

// 5. Total Borrowers Count
Total Borrowers = 
COUNTROWS(credit_risk_dataset)
---

## Strategic Recommendations

1. **Underwriting Policy Revision:** Immediately cap or require full collateral backing for unsecured credit applications classified under Grades E, F, and G.

2. **DTI Hard Limits:** Implement a maximum allowable DTI threshold of **30%** for applicants under the **RENT** home ownership status.

3. **Historical Default Gate:** Auto-decline or require enhanced collateral for any applicant with `cb_person_default_on_file = Y`.

4. **Proactive Collections & Restructuring:** Focus early warning collection workflows on **Debt Consolidation** and **Medical** loan facilities showing NPL rates above 26%.

5. **Portfolio Rebalancing:** Gradually reduce exposure to Grade D–G segments; redirect origination capacity toward Grade A–B with DTI < 0.25.

---

## How to Run the Project

1. Clone this repository:
   ```bash
   git clone https://github.com/Nofangagah/Commercial-Bank-Credit-Risk-NPL-Monitoring-Project.git
   ```

2. Download and install [Power BI Desktop](https://powerbi.microsoft.com/desktop/).

3. Open the file `Bank Commercial Loan & Credit Risk (NPL) Monitoring Dashboard.pbix`.

4. If prompted, refresh the dataset connection pointing to `Data/credit_risk_dataset.csv`.

---

## Repository Structure

```
├── Data/
│   └── credit_risk_dataset.csv          # Raw source dataset (32,581 rows)
├── Asset/
│   └── Dashboard_preview.png            # Dashboard screenshot preview
├── Bank Commercial Loan & Credit Risk (NPL) Monitoring Dashboard.pbix  # Power BI project file
└── README.md                            # Project documentation
```

---

## Limitations & Future Work

- **Static snapshot:** Dashboard reflects a single period; time-series trend analysis requires historical vintages.
- **No macroeconomic variables:** Unemployment rate, GDP growth, and interest rate environment not incorporated.
- **Employment length anomalies:** Values > 50 years likely data errors; consider capping at retirement age.
- **Future enhancements:** 
  - Add PD/LGD/EAD modeling for IFRS 9 / CECL compliance
  - Integrate macroeconomic stress testing scenarios
  - Build automated retraining pipeline for credit scorecards

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details (if applicable).

## Contact

For questions or collaboration, reach out via [GitHub Issues](https://github.com/Nofangagah/Commercial-Bank-Credit-Risk-NPL-Monitoring-Project/issues).
