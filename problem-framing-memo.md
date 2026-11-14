# ML Problem-Framing Memo: Customer Churn

**Author:** Aathika Begam A
**Date:** 14 November 2026

---

## 1. Summary

Before training any model, I wanted to answer a simple question: do we actually need machine learning to predict which customers will leave? This memo sets out the decision we are trying to support, what we are predicting, what a wrong prediction costs, and what safeguards we need.

My conclusion is that **ML is not justified on this dataset**. The file has only 12 customers, and a few very simple rules already sort the churners from the non-churners perfectly. With so little data, I can't test a model fairly, and a perfect score from any method would make me suspicious rather than happy. For now, I recommend a simple rule, and revisiting ML only once we have a much bigger dataset with proper dates on it.

---

## 2. The decision

The decision is: **which customers should the retention team contact first** with help or an offer, so we reach people who are likely to cancel before they actually do?

The retention (customer success) team would use this. The output should only decide who gets extra attention or an offer. It should never be used to refuse service, change someone's price, or treat any customer worse.

---

## 3. Problem definition

- **Prediction target:** the `churned` column (1 = the customer cancelled, 0 = they stayed). In practice I would define it as "will this customer cancel in the next 30 days?"
- **Unit of observation:** one customer per row, identified by `customer_id`.
- **Features we have:** `tenure_months`, `support_tickets`, `monthly_spend_inr`, `last_login_days` and `plan_type`.
- **Action window:** we make the prediction on a set date. I assume the team needs about 7 days to reach a customer, and the customer has about 30 days before they cancel. So the prediction is only useful if it arrives somewhere in that 7 to 30 day window. A prediction that comes after the customer has already left is no use to anyone.
- **Output:** a simple flag (or risk score) for each customer, which becomes a short "contact these people first" list.

One thing I can't answer from the file is exactly what time period `churned` covers, or when each feature was measured. That needs to be confirmed with whoever owns the data, because it affects the leakage risk in Section 7.

---

## 4. Non-ML baseline

Before reaching for a model, I tried some simple rules that a person could apply by hand. I checked each one against all 12 customers (5 churned, 7 stayed):

| Rule | Churners caught (out of 5) | Loyal customers wrongly flagged (out of 7) |
|---|---|---|
| Last login 10 or more days ago | 5 | 0 |
| 3 or more support tickets | 5 | 0 |
| Last login 10+ days ago, or 3+ tickets | 5 | 0 |
| On the Basic plan | 5 | 0 |
| Tenure of 8 months or less | 5 | 1 |

I chose this as the baseline: **flag a customer if they haven't logged in for 10 or more days.** It's easy to explain, it's based on something the team can actually see, and it doesn't depend on someone's plan or what they pay.

The data is surprisingly clean. Every churner last logged in 12 to 30 days ago, and every customer who stayed last logged in 1 to 9 days ago, so the two groups don't overlap at all. Real customer data is almost never this tidy, so I think this is a small practice dataset. I wouldn't expect these perfect results to carry over to new customers.

A model is only worth using if it clearly beats this baseline on data big enough to show a real difference.

---

## 5. Is ML justified?

It might be useful later. With real data, signals like login recency, support tickets, tenure and plan could combine in ways a single rule misses, and a score would let the team rank customers instead of getting a plain yes or no.

But I don't think it is justified right now, for these reasons:

- **There isn't enough data.** With 12 rows, a train/test split leaves only 3 or 4 customers to test on, so the result would change completely depending on which customers landed there.
- **The baseline can't be beaten.** The simple rule already catches all 5 churners with no false alarms. A model can't do better than perfect, so we couldn't show it adds anything.
- **A model would just memorise the data.** A perfect score on 12 rows would say nothing about how it handles new customers.
- **It adds work and risk for no gain.** A model needs monitoring, fairness checks and maintenance. A one-line rule doesn't.

So my position is: don't train a production model on this dataset. Use the baseline rule, and collect more data first. I'm still happy to train a small model in the notebook to show how it works, as long as it is clearly labelled as a demonstration and not as proof of real performance.

---

## 6. Metrics and cost of mistakes

There are two ways to get it wrong, and they cost different amounts. The figures below are my assumptions and should be replaced with real numbers from the business.

- **False positive** (we flag someone who would have stayed): we waste an offer and some staff time. I assumed roughly one month's discount, about ₹100 to ₹300.
- **False negative** (we miss someone who then leaves): we lose their monthly payment, and every month they would have stayed after that. In this data, monthly spend runs from ₹499 to ₹1,499.

A missed churner is clearly the more expensive mistake, so **recall** (catching the people who will leave) matters most. Precision still matters, though, because we don't want the retention team buried in wrong flags.

The measures I plan to use:

- **Business measures:** estimated revenue saved, cost of unnecessary offers, and the net benefit compared with the baseline.
- **Model measures:** recall, precision, F1 and PR-AUC. I won't rely on accuracy alone, because churn is uneven (5 of 12, or 41.7%, here, and usually much lower in real life).
- **Calibration:** whether a predicted risk like "70%" really means about 70% of those customers leave. We can't judge this with 12 rows.
- **Fairness:** compare error rates across plan types. In this data, all 5 Basic customers churned and none of the Pro or Standard customers did, so any rule or model will end up tied closely to plan. That makes a group-level check important once we have more data.

---

## 7. Data risks

The full detail is in the data card. In short:

- **Leakage.** If `last_login_days` or `support_tickets` were recorded after the customer left, they would give away the answer. A customer who has already left naturally has a long gap since their last login. The model would look great in testing and then fail in real use. This needs checking with the data owner.
- **Plan type almost equals the label.** Every Basic customer churned and no one else did. That is unlikely to be true in real life, and it could lead us to treat Basic customers as a lost cause.
- **Sensitive proxies.** `plan_type` and `monthly_spend_inr` could indirectly reflect how much money someone has. Spend is also almost fixed by plan (Basic ₹499 to ₹799, Standard ₹799 to ₹999, Pro ₹1,299 to ₹1,499), so it adds little on its own.
- **Unknown origin and consent.** The file doesn't say where the data came from or whether customers agreed to this use. I have not assumed an answer.
- **Very small sample.** 12 rows can't support reliable conclusions.
- **The ID column.** `customer_id` should never be used as a feature.

---

## 8. Safeguards

**When to hold back (abstention).** If a customer is missing important information, or is close to the cut-off (say 8 to 11 days since last login), the system shouldn't make an automatic call. It should pass the case to a person.

**Human review.** A retention agent checks every flagged customer before anyone is contacted, and also checks all high-value accounts, such as Pro customers. Nothing happens to a customer automatically.

**Monitoring after launch.**
- Check the recall and precision of the rule each month on customers whose outcome is now known.
- Watch for changes in the data, such as average tenure, spend, login recency and plan mix.
- Compare error rates by plan type.

**When to roll back.** We stop using the rule or model and go back to manual review if:
- recall or precision drops sharply for two months in a row
- a data source or the meaning of a feature changes without warning
- one plan group's error rate is far higher than the others
- the retention team reports a big jump in wrongly flagged customers

---

## 9. Next steps

1. Ask the data owner about the time window for `churned`, when each feature was measured, and whether there is consent and a licence for this data.
2. Finish the responsible data card (`responsible-data-card.md`).
3. Run the baseline notebook, compare the rule with one simple model, and label the model result as a demonstration only.
4. Fill in the risk register, with an owner and a fix for each risk.
5. Collect a much larger dataset, with dates on every feature, before reconsidering ML.

---

*The cost figures in Section 6 are my own assumptions. They should be replaced with real numbers from the business.*
