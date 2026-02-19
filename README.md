# SLFRS-9-Impairment-Engine-for-Dumbara-Micro-Credit
Design and Implementation of an SLFRS 9 Impairment Engine for Dumbara Micro Credit
![Image](https://github.com/user-attachments/assets/2e66765e-2025-4e25-a53a-114ae6c1ee65)

## 📊 Central Bank of Sri Lanka (CBSL) Reporting Alignment
The engine is designed to produce impairment outputs that align with the reporting requirements under SLFRS 9 as expected by the Central Bank of Sri Lanka. It generates key schedules and metrics that can be used to prepare:

* Stage-wise breakdown of loans and ECL allowances

* Reconciliation of the allowance account (opening balance, new provisions, write-offs, etc.)

* Credit quality analysis (past due, impaired, risk concentration)

* Summary of ECL methods and assumptions (PD/LGD tables)

These outputs support the detailed disclosures required by CBSL Directions on credit risk management and impairment. However, final acceptance of the reported figures depends on the institution’s own validation, audit, and governance processes. The engine serves as a robust calculation tool that can be integrated into a compliant SLFRS 9 framework.

## Abstract
The Sri Lanka Financial Reporting Standard 9 (SLFRS 9) introduces a forward-looking expected credit loss (ECL) model that requires financial institutions to recognise credit losses earlier than under previous rules. This article presents a custom-built impairment calculation engine developed in Python for Dumbara Micro Credit, a microfinance institution operating in Sri Lanka. The system implements the three-stage model while incorporating business-specific overrides such as dynamic payment classification, zero-arrears treatment, and granular part-payment rules. The engine processes loan-level data, integrates a risk code lookup table, and produces comprehensive Excel reports, summary statistics, and audit trails, enabling transparent and compliant ECL provisioning. We draw inspiration from open-source resources on IFRS 9 modelling and adapt them to the unique characteristics of Dumbara Micro Credit’s portfolio.

## 1. Introduction
Dumbara Micro Credit provides small loans to underserved communities in Sri Lanka. With the adoption of SLFRS 9 (the local equivalent of IFRS 9), the institution must now measure impairment losses on a forward-looking basis. The standard requires recognition of 12-month ECL at origination (Stage 1), lifetime ECL when credit risk increases significantly (Stage 2), and lifetime ECL with interest calculated on a net basis for credit-impaired assets (Stage 3).

While several open-source initiatives have emerged to demystify IFRS 9 modelling—such as the ifrs9 repository on GitHub (naenumtou/ifrs9) and a Kaggle notebook by Beata Faron—real-world portfolios often require customisation to reflect specific product features, data availability, and business policies. Dumbara Micro Credit needed a solution that could handle its unique data formats, incorporate a dynamic cut-off for stale loans, and apply business overrides based on observed payment behaviours (e.g., part payments, zero arrears).

This article describes the impairment engine we developed for Dumbara Micro Credit. The engine implements the core SLFRS 9 staging logic but extends it with dynamic payment classification, risk code matching, and a hierarchy of overrides that reflect the institution’s practical experience. The result is a flexible, auditable tool that generates detailed ECL allowances, rationales, and diagnostic outputs.

## 2. SLFRS 9 Impairment Model – A Quick Recap
SLFRS 9 requires entities to recognise ECL in three stages:

Stage 1 (12‑month ECL): Upon initial recognition, an entity recognises 12‑month expected credit losses. Interest revenue is calculated on the gross carrying amount.

Stage 2 (Lifetime ECL – not credit‑impaired): If credit risk increases significantly, lifetime ECL is recognised, but interest is still calculated on the gross carrying amount.

Stage 3 (Lifetime ECL – credit‑impaired): For assets that are credit‑impaired, lifetime ECL is recognised and interest revenue is calculated on the net carrying amount (gross less loss allowance).

## 3. Overview of the Dumbara Micro Credit Engine
The engine is written in Python and processes loan‑level data from Excel workbooks provided by Dumbara Micro Credit. It consists of the following main components:

* Data loading and preprocessing – reading the loan book and a risk code file.

* Payment classification – using the system date to determine whether the last payment occurred more than three years ago (Class 1), in which case the present value (PV) of expected cash flows is set to zero.

* Stage assignment – based on the number of days in arrears (≤60 days → Stage 1, 61–90 → Stage 2, >90 → Stage 3).

* Risk code integration – matching loan remarks and text fields against a table of risk situations and codes to flag loans with “risky reasons”.

* PV calculation – starting from a base factor based on customer status (Excellent, Good, Fair, etc.), then applying overrides:

* Zero arrears: PV = Recoverable Balance.

* New loans (≤3 months old, zero arrears, no risk, Good status): PV = 0.

* Stage 3 part‑payment: PV = Recoverable Balance × (40%–50% depending on status and risk).

* No payment behaviour: PV = 0.

* ECL allowance – computed as a percentage of the difference between recoverable balance and PV (RV‑PV), with percentages varying by stage, risk flag, and customer status category.

* PD and LGD assumptions – derived from the same logic and stored for display.

* Output generation – multi‑sheet Excel file, summary CSV, and JSON statistics.

A key feature is the dynamic payment classification threshold: the system date is read at runtime, and any loan whose last paid date is before today - 3 years is automatically classified as Class 1, forcing its PV to zero. This rule reflects Dumbara’s view that loans with no recent payment activity are effectively non‑performing and should be fully provisioned.

## 4. Data Sources and Preprocessing
The engine expects two input files:

* book.xlsx – the main loan portfolio, containing fields such as Account, Member Name, Last Paid Date, Number of Days in Arrears, Total Outstanding, Recoverable Balance, Customer Status, Reason, and optionally Payment_Behavior and Granted Date.

* Risk.xlsx – a lookup table with columns Code, Risk Level, and Situation. Each row defines a risk code, its associated risk level (e.g., “High Risk”, “Medium Risk”), and a textual situation description.

Upon loading, the engine cleans numeric columns, converts dates, and standardises text fields. The arrears column is identified flexibly (by searching for column names containing “arrears”). The Granted Date column is used only for the new‑loan override and is optional.

## 5. Key Features of the Engine
The impairment engine for Dumbara Micro Credit incorporates several practical features that tailor SLFRS 9 requirements to the institution’s portfolio. Below is a high‑level summary of its main components.

### Dynamic Payment Classification
The engine uses the current system date to classify loans based on their last payment. Any loan with a last paid date more than three years before the reporting date is automatically flagged and its expected future cash flows are set to zero. This threshold moves forward each month, eliminating the need for manual updates.

### Stage Determination
Loans are assigned to Stage 1, Stage 2, or Stage 3 solely based on the number of days in arrears:

Stage 1: 0–60 days past due

Stage 2: 61–90 days past due

Stage 3: over 90 days past due

This simple rule aligns with the 90‑day default presumption and reflects Dumbara’s historical experience.

### Risk Code Integration
A separate lookup table (Risk.xlsx) lists risk codes, their severity levels, and textual situation descriptions. The engine scans each loan’s remarks and other text fields to identify any match—whether exact or partial—with these risk situations. If found, the loan is marked as having “risky reasons,” which influences the final ECL calculation.

### Present Value (PV) Calculation with Overrides
The expected future cash flows (PV) start from a base factor linked to the customer’s status (Excellent, Good, Fair, etc.). Several business overrides then adjust this value:

Zero‑arrears: PV equals the full recoverable balance.

New loans: For loans less than three months old, with zero arrears, no risk flags, and a “Good” status, PV is set to zero (a conservative stance).

Stage 3 part‑payment: If a loan is in Stage 3 and shows part‑payment behaviour, PV is set to a percentage of the recoverable balance (40–50%, depending on status and risk).

No payment: Loans with a “no payment” flag receive PV = 0.

### ECL Allowance
The allowance is calculated as a percentage of the difference between the recoverable balance and the PV (called RV‑PV). The percentage varies by stage, risk flag, and customer status category, effectively incorporating probability of default (PD) and loss given default (LGD) assumptions. For Stage 3 loans, the full shortfall is provided.

## 6. Conclusion
We have presented a custom SLFRS 9 impairment engine for Dumbara Micro Credit that balances theoretical rigour with practical flexibility. By leveraging Python’s data processing capabilities and building on insights from open‑source projects, we have created a system that:

Dynamically classifies loans based on the last payment date relative to a rolling three‑year window.

Integrates a risk code lookup with substring matching to capture qualitative risk indicators.

Applies a hierarchy of overrides (zero arrears, new loans, part‑payment, no payment) to refine the present value of expected cash flows.

Calculates ECL allowances with transparent rationales.

Produces comprehensive reports suitable for both internal management and external audit.

The engine is run monthly to reflect updated data and the evolving system date. Its modular design allows easy adjustment of thresholds, factors, and override rules as business needs change.
