# Risk Register: Customer Churn Prediction

**Prepared by:** Aathika Begam A
**Date:** 14 November 2026

This is my list of the main things that could go wrong with a churn prediction system, how likely each one is, how much it would matter, and what I would do about it. It goes with my problem-framing memo, the responsible data card and the baseline notebook.

A quick note on how to read it. Likelihood and impact are rated Low, Medium or High, and those ratings are my own judgement from a 12-customer dataset, not measured numbers. The owners are roles (like "retention lead") because the assignment doesn't say who would run this in a real team.

---

## 1. Risks

| ID | What could go wrong | Likelihood | Impact | What I would do | Owner |
|---|---|---|---|---|---|
| R1 | **Not enough data.** With only 12 customers, any result, even a perfect score, can't be trusted on new customers. | High | High | Don't train a production model on this file. Call every model result a demonstration. Get a much bigger dataset first. | Data analyst |
| R2 | **Leakage.** `last_login_days` or `support_tickets` might have been recorded after the customer left. That would give away the answer, and the system would fail in real use. | Medium | High | Ask the data owner when each feature was measured. Only use values that exist on the day we make the prediction. | Data owner |
| R3 | **False positive.** A loyal customer gets flagged and receives an offer they didn't need. | Medium | Low | Keep offers cheap (I assumed about ₹100 to ₹300). Have a person check each flag. Track what wrong flags cost every month. | Retention lead |
| R4 | **False negative.** A customer who is about to leave gets missed, and we lose their monthly payment and every month after it. | Medium | High | Focus on recall. Look at missed churners each month. Keep manual review as a backup. | Retention lead |
| R5 | **Plan-type bias.** All 5 Basic customers churned and nobody else did, so a model might write Basic customers off as lost causes. | High | Medium | Never judge by plan alone. Compare error rates by plan type. Have a person review before anyone is contacted. | Data analyst |
| R6 | **Affordability proxy.** `monthly_spend_inr` and `plan_type` stand in for income, so people who can least afford the service could be ignored or unfairly targeted. | Medium | Medium | Leave spend and plan out of the model (the notebook does this). Never use the output to set prices or decide service. | Data analyst |
| R7 | **Misuse.** Someone could use churn flags to cut service or change a customer's price. | Low | High | Write down the intended use: retention outreach only. Limit who can see the flags. | Retention lead |
| R8 | **Privacy and consent.** The file doesn't say whether customers agreed to this use, and customer IDs could be traced back to real people. | Medium | High | Confirm consent and the legal basis (for example under India's Digital Personal Data Protection Act, 2023). Hash or remove IDs before sharing. Don't publish real customer data. | Data owner |
| R9 | **Unrealistic churn rate.** 41.7% of these customers churned, which is far higher than normal, so the metrics and cut-offs may not carry over. | High | Medium | Check the balance again on real data. Rely on recall, precision and PR-AUC instead of accuracy. | Data analyst |
| R10 | **Fragile model scores.** In the demonstration model, three customers (C003, C010 and C012) sit close to the 0.5 cut-off, so a small change could flip them. | Medium | Low | Use the simple login rule as the baseline. Send borderline cases to a person (see Abstention below). | Data analyst |
| R11 | **Drift after launch.** Customer behaviour, plans or prices change over time, and the rule or model slowly stops working. | Medium | Medium | Monitor every month (see Monitoring below) and re-check the cut-off regularly. | Data analyst |
| R12 | **Unknown data origin.** We don't know how or when the file was made, so we can't tell whether it's real, simulated or out of date. | High | Medium | Ask the data owner. Until I get an answer, treat it as practice data only. | Data owner |

---

## 2. Abstention: when the system should hold back

The system shouldn't make an automatic call, and should hand the case to a person, when:

- a key field is missing for that customer
- `last_login_days` is close to the cut-off, which I define as **8 to 11 days** (the cut-off itself is 10)
- the customer's data looks odd for their plan, such as an extreme value
- the data is out of date or can't be matched to a known customer

I chose these because borderline or incomplete records are where wrong calls are most likely, and a person can use context the data doesn't have.

---

## 3. Human review

- A retention agent checks **every** flagged customer before anyone is contacted. Nothing goes out automatically.
- A person also checks all high-value accounts (for example Pro customers) and every customer the system held back on.
- The agent can overrule a flag. Those overrules get recorded so we can learn from them and spot patterns.
- A flag is only a nudge to look more closely. It never decides price, service or support for anyone.

---

## 4. Monitoring after launch

Each month, I would check:

- **Recall and precision** of the rule or model, using customers whose outcome is now known. This shows whether it still catches churners without flooding the team with wrong flags.
- **The cost of wrong flags and of missed churners**, so performance is tied back to money.
- **Error rates by plan type**, to catch unfair gaps between groups (R5 and R6).
- **Data drift**, meaning average tenure, spend, login recency and the mix of plans (R11). It gives an early warning that the data has changed.
- **How many cases go to abstention and human review.** A sudden rise usually points to a data problem.
- **How often agents overrule a flag.** Lots of overrules would mean people don't trust the flags or don't find them useful.

I would also check **whenever a data source or feature definition changes**, not just monthly. A quiet change in how a column is measured can break everything (R2).

---

## 5. Rollback conditions

We stop using the rule or model and go back to manual review by the retention team if **any** of these happen:

- Recall or precision falls sharply for **two months in a row**. A real threshold, such as recall below 80%, should be agreed with the business before launch. That number is only my suggestion.
- A data source, or the meaning of a feature, changes without warning.
- One plan group's error rate is far higher than the others.
- The retention team reports a big jump in wrongly flagged customers.
- We find out a feature was recorded after the customer left (leakage, R2).
- Consent for this use turns out to be missing or has been withdrawn (R8).

**Who decides:** the retention lead, together with the data owner.

**What happens next:** stop using the flags, tell the retention team, write down what went wrong, and only restart once the cause is fixed and checked again.

---

## 6. Open questions

1. What time period does `churned` cover, and when was each feature measured?
2. Did customers consent to this use, and is there a licence for the data?
3. Where did the data come from, and is it real or simulated?
4. What do a wrong offer and a lost customer actually cost?
5. Who would take on each of the roles above in a real team?
