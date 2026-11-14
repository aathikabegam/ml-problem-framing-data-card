# Responsible Data Card

**Dataset:** `customer-churn-training.csv`
**Prepared by:** Aathika Begam A
**Date:** 14 November 2026

---

## Dataset purpose

This dataset can help with one decision: which customers the retention team should contact first with help or an offer, so fewer of them cancel.

It must **not** be used to:
- refuse service, support or refunds to a customer
- set or change an individual customer's price
- disadvantage customers because of their plan type or how much they spend
- run a production ML model, because the dataset is far too small (see Quality checks)

Realistically, it is only suitable for learning, prototyping and testing a baseline.

---

## Provenance and permission

- **How it was created:** The file doesn't say. The assignment calls it a "small labeled dataset for responsible baseline modeling", and the data looks synthetic or heavily simplified, but I haven't been able to verify that.
- **Who may use it:** Not stated. I'm assuming it is for this internship exercise only.
- **Consent and licence:** Unknown. Nothing in the file tells us whether customers agreed to their data being used to predict churn, or what licence applies. If this is real customer data, consent and a legal basis (for example under India's Digital Personal Data Protection Act, 2023) would need to be confirmed before any real use.
- **Personal data:** There are no names, contact details or locations, only a customer ID. Even so, whoever holds the customer records could link an ID back to a real person, so a file like this shouldn't be published if it comes from a real business.
- **When it was collected:** Not stated.

---

## Population and representation

- **Size:** 12 customers (rows) and 7 columns.
- **Who is represented:** three plan types: Basic (5 customers), Standard (4) and Pro (3).
- **Who is missing:** there's nothing on age, gender, region or customer segment, and `tenure_months` is the only hint about how long someone has been a customer. We can't tell whether these 12 customers represent the wider customer base.
- **Sampling:** 12 customers is far too few to represent a real customer base, and we don't know how they were picked.
- **Imbalance:** 5 customers churned (41.7%) and 7 stayed (58.3%). Real churn is usually much lower, so this split probably isn't realistic.
- **A pattern to watch:** all 5 Basic customers churned, and none of the 3 Pro or 4 Standard customers did. This is probably a side effect of a tiny or constructed sample, but a model could easily learn to judge people mostly by their plan.

---

## Features and target

**`customer_id`**: a unique ID for each customer (C001 to C012). It is only a label, so it should not be used as a feature.

**`tenure_months`**: how many months the customer has been with the company (1 to 30). In this data, newer customers are more likely to have churned.

**`support_tickets`**: how many support tickets the customer raised (0 to 5). There is a leakage risk here, because it's unclear whether tickets were counted before or after the customer decided to cancel. A rise in tickets before someone leaves is a fair signal, but tickets raised during the cancellation itself are not.

**`monthly_spend_inr`**: what the customer pays per month in rupees (499 to 1,499). This could act as a proxy for income or affordability. It is also almost fixed by plan (Basic ₹499 to ₹799, Standard ₹799 to ₹999, Pro ₹1,299 to ₹1,499), so it adds little beyond `plan_type`.

**`last_login_days`**: days since the customer last logged in (1 to 30). This is another leakage risk. If it was measured after the customer left, it would be large simply because they left, and we wouldn't have that information at the time we need to make a prediction. It must be recorded as of the prediction date.

**`plan_type`**: the subscription level, Basic, Standard or Pro. It is also a proxy for ability to pay, and it lines up perfectly with the label in this sample.

**`churned`** (the target): 1 means the customer cancelled and 0 means they stayed. The time period it covers isn't documented.

**Possible leakage, in short:** `last_login_days` and `support_tickets` need their measurement dates confirmed, and `monthly_spend_inr` could change or drop to zero after cancellation depending on how it was recorded.

**Sensitive proxies:** `plan_type` and `monthly_spend_inr` (income and affordability). There are no direct sensitive attributes such as gender, religion, caste or health.

---

## Quality checks

I ran these on the full file:

- **Size:** 12 rows and 7 columns.
- **Missing values:** none in any column.
- **Duplicates:** no duplicate rows and no duplicate customer IDs.
- **Class balance:** 5 churned (41.7%) and 7 stayed (58.3%).
- **Outliers:** none that stand out. Tenure runs from 1 to 30 months, tickets from 0 to 5, spend from ₹499 to ₹1,499 and last login from 1 to 30 days. With only 12 rows, a formal outlier test wouldn't mean much anyway.
- **Train/test separation:** this isn't really possible here. A split leaves about 3 or 4 customers in the test set, so the result would depend on which ones ended up there. I can show cross-validation or leave-one-out for demonstration, but it shouldn't be presented as proof of performance.

**How cleanly the groups separate:** churners last logged in 12 to 30 days ago and raised 3 to 5 tickets. Customers who stayed last logged in 1 to 9 days ago and raised 0 to 2 tickets. The two groups don't overlap on either feature, which is unusually clean for real data.

---

## Risks and safeguards

**Bias: Basic customers treated as lost causes.** A model could learn "Basic plan means they'll leave". To guard against this, check error rates by plan, never rely on plan alone, and have a person review before anyone is contacted.

**Bias: affordability.** Spend acts as a stand-in for income, so people who can least afford the service might be targeted or ignored. Limit or remove spend and plan as features, and never use the output for pricing or service decisions.

**Privacy.** Real customer data could be shared or published without consent. Confirm consent and the legal basis, remove or hash IDs before sharing, and don't put real data in a public repository.

**Misuse.** Churn predictions could be used to cut or deny service. Write down the intended use and limit the output to retention outreach only.

**False positive.** An offer goes to someone who would have stayed. Keep offers low cost, have a person review flags, and track what wrong flags cost.

**False negative.** A customer who is about to leave is missed. Focus on recall, review missed churners every month, and keep a manual review as a fallback.

**Leakage.** Features that already know the outcome make results look better than they are, and the system then fails in real use. Confirm when each feature was measured and only use values available on the prediction date.

**Over-trusting a tiny sample.** Label every result as a demonstration only, and gather much more data before any real use.

---

## Intended evaluation

I'm setting these out before any training:

- **Baseline:** flag a customer if `last_login_days` is 10 or more. On this file it catches all 5 churners with 0 false alarms. That perfect score is what you'd expect from 12 rows and won't carry over to new data.
- **Performance:** recall, precision, F1 and PR-AUC. I'll report accuracy but won't rely on it.
- **Calibration:** whether predicted probabilities are trustworthy. This can't be assessed with 12 rows and needs a much larger dataset.
- **Fairness:** compare false-positive and false-negative rates across plan types.
- **Error analysis:** list every customer the model gets wrong and work out why (a data problem, leakage, or genuine behaviour).
- **Cost view:** total the cost of false positives and false negatives, using the assumptions in the problem-framing memo.
- **Decision rule:** only adopt an ML model if it beats the baseline on these measures, on a dataset large enough to show a real difference.
