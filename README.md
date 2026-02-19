# SLFRS-9-Impairment-Engine-for-Dumbara-Micro-Credit
Design and Implementation of an SLFRS 9 Impairment Engine for Dumbara Micro Credit
<img width="1536" height="1024" alt="Image" src="https://github.com/user-attachments/assets/726f0d8a-e47e-4466-ae44-d2850019726f" />
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

The ECL is typically computed as:
