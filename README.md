# Assignment 2: Multiclass Text Classification

**Name:** Fong Lieu 
**Date:** March 2026  
**Dataset:** SASB-Aligned ESG Sentences

---

## Overview

This project builds a **multiclass text classifier** to categorize corporate ESG report sentences into one of six mutually exclusive categories.

**What is being classified:** Individual sentences from corporate ESG (Environmental, Social, and Governance) disclosure reports, sourced from SASB-aligned company filings.

---

## Dataset Details

| Property | Value |
|---|---|
| Source | [Kaggle — SASB-Aligned ESG Sentences](https://www.kaggle.com/datasets/edwardjunprung/sasb-aligned-esg-sentences) |
| Total rows | 6,460 sentences |
| Total columns | 3 (Text, Parent Label, Child Label) |
| Missing values | 0 |
| Task type | Multiclass (6 classes) |
| Train / Test split | 80% / 20% (stratified) |

**Class Distribution:**

| Class | Count | % of Dataset |
|---|---|---|
| Non-ESG | 3,546 | 54.9% |
| Social Capital | 882 | 13.7% |
| Leadership & Governance | 721 | 11.2% |
| Environment | 567 | 8.8% |
| Human Capital | 466 | 7.2% |
| Business Model & Innovation | 278 | 4.3% |

> The dataset is heavily imbalanced — Non-ESG accounts for over half the data. Class imbalance was addressed using `class_weight='balanced'` for all models.

---

## Best Model Results

**Model:** Naive Bayes + CountVectorizer  
**Feature approach:** CountVectorizer (max 7,000 features, 1-2 grams)

| Metric | Value |
|---|---|
| Weighted F1 | 0.813 |
| Weighted Precision | 0.817 |
| Weighted Recall | 0.810 |
| Accuracy | 0.810 |
| Training Time | 0.028s |

**Per-class breakdown (Naive Bayes):**

| Class | Precision | Recall | F1 |
|---|---|---|---|
| Business Model & Innovation | 0.64 | 0.62 | 0.63 |
| Environment | 0.78 | 0.84 | 0.81 |
| Human Capital | 0.67 | 0.61 | 0.64 |
| Leadership & Governance | 0.74 | 0.73 | 0.73 |
| Non-ESG | 0.92 | 0.88 | 0.90 |
| Social Capital | 0.62 | 0.73 | 0.67 |

---

## Important Class: Environment

**Justification:** The Environment class is the most critical category in this dataset for the following reasons:

- **Regulatory risk:** Missing an environmental disclosure can constitute non-compliance with TCFD, SEC climate disclosure rules, or EU CSRD requirements.
- **Greenwashing risk:** Incorrectly classifying non-environmental content as environmental can falsely inflate a company's perceived climate action.
- **Investor impact:** ESG-focused funds rely on accurate environmental classification to assess climate transition risk.

**Priority metric: Recall.** Missed environmental disclosures (false negatives) are more costly than false alarms (false positives), as undetected disclosures can lead to incomplete risk assessments or overlooked compliance obligations.

Naive Bayes achieves the highest Environment recall (0.84) of all three models, making it the strongest choice for this use case.

---

## Model Comparison (5 Criteria)

| Criterion | Logistic Regression | Naive Bayes | Linear SVM |
|---|---|---|---|
| 1. Weighted F1 | 0.794 | **0.813** | 0.813 |
| 2. Training Time | 1.11s | **0.028s** | 0.11s |
| 3. Environment F1 | 0.772 | **0.809** | 0.786 |
| 4. Interpretability | High (coefficients) | Medium (log-prob) | Low (no probs) |
| 5. Scalability | Good | **Excellent** | Good |

---

## Custom Inference Summary

**Score: 13/20 correct (65%)**

| Difficulty | Correct | Total |
|---|---|---|
| Easy | 8 | 10 |
| Tricky | 4 | 5 |
| Out-of-domain | 1 | 5 |

**Key findings:**
- The model performs well on clearly worded ESG sentences with strong domain-specific vocabulary (e.g., "Scope 1 emissions", "audit committee")
- The main failure mode for easy/tricky examples was **cross-category confusion** — sentences mentioning executive compensation tied to ESG goals were sometimes classified as Human Capital instead of Leadership & Governance
- Out-of-domain performance was poor (1/5) — the model relies heavily on vocabulary patterns from corporate ESG reports and struggles when sentences come from different industries or writing styles
- A vague "commitment to responsible business practices" sentence was incorrectly labeled as Environment, showing the model picks up on ESG-adjacent language even when no specific disclosure is made

---

## Recommendation

**Deploy Naive Bayes + CountVectorizer** as a first-pass ESG disclosure triage tool.

**Justification:**
- Ties with Linear SVM on weighted F1 (0.813) but achieves significantly higher Environment recall (0.84 vs. 0.79)
- Trains 40x faster than Logistic Regression, making it practical for large-scale document processing
- CountVectorizer is the correct pairing for MultinomialNB, which requires non-negative count inputs

**Deployment caveat:** The model should not be used as a fully automated system. Low-confidence predictions should be routed to human reviewers, particularly for the Environment and Business Model & Innovation classes which showed the most classification difficulty. Future improvements should include more diverse training data and potentially a context-aware model (e.g., word embeddings) to better handle linguistic variation across industries.

