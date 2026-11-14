# ML Problem Framing & Responsible Data Card

**Author:** Aathika Begam A
**Task:** Internship Task 02, ML Problem Framing & Responsible Data Card
**Date:** 14 November 2026

This repository is my work for deciding whether machine learning is justified for a real decision, and documenting the data, harms and fallback plan *before* training any model. The decision I looked at is which customers a retention team should contact first, using a small customer churn dataset.

## The short answer

**ML is not justified on this dataset.** It has only 12 customers, so a model can't be tested fairly. A one-line rule (flag a customer who hasn't logged in for 10 or more days) already catches all 5 churners with no false alarms, so a model has nothing to improve on. I recommend using the simple rule for now and collecting a much larger, properly dated dataset before reconsidering ML.

## What's in this repo

| Deliverable | File | What it is |
|---|---|---|
| ML problem-framing memo | [problem-framing-memo.md](problem-framing-memo.md) | The decision, prediction target, unit of observation, action window, non-ML baseline, cost of mistakes and safeguards |
| Responsible data card | [responsible-data-card.md](responsible-data-card.md) | Provenance, consent, sensitive attributes, missing data, leakage risks, quality checks and intended evaluation |
| Baseline notebook | [baseline.ipynb](baseline.ipynb) | Data checks, the rule-based baseline, a small demonstration model, a fairness check and a cost comparison |
| Risk register | [risk-register.md](risk-register.md) | 12 risks with likelihood, impact, fixes and owners, plus abstention, human review, monitoring and rollback conditions |
| Dataset | [customer-churn-training.csv](customer-churn-training.csv) | The customer churn data provided with the assignment (12 rows, 7 columns) |

## Main findings

- The data is clean: no missing values and no duplicates. 5 of 12 customers churned (41.7%).
- Customers who churned last logged in 12 to 30 days ago. Customers who stayed last logged in 1 to 9 days ago.
- All 5 Basic-plan customers churned and none of the Pro or Standard customers did, which is a bias concern if a model learns to judge by plan.
- A small demonstration model (logistic regression, checked with leave-one-out) matches the simple rule but doesn't beat it, and three of its scores sit close to the cut-off.
- Two features, `last_login_days` and `support_tickets`, might have been recorded after the customer left. This leakage risk needs the data owner to confirm.

## Things I could not confirm

The dataset doesn't say where it came from, whether customers consented to this use, or what time period the `churned` column covers. I've marked these as unknown in the documents instead of guessing. The cost figures in the memo and notebook (about ₹200 per wrong offer, 6 months of lost revenue per missed customer) are my own assumptions and would need real numbers from a business.

## Running the notebook

The notebook has already been run, so the results are visible when you open it on GitHub. To run it yourself:

1. Install Python with `pandas`, `numpy` and `scikit-learn`.
2. Download `baseline.ipynb` and `customer-churn-training.csv` and keep them in the same folder.
3. Open `baseline.ipynb` in Jupyter, VS Code or Google Colab (in Colab, upload the CSV to the file panel).
4. Run all cells from top to bottom.

## A note on the results

Everything here comes from a dataset of 12 customers. A perfect score on 12 rows tells us very little about how any method would do on new customers, so please read the results as a worked example of the process, not as evidence of real-world performance.
