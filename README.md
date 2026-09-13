# Explainable Fraud Detection for Transactions

*This is for whoever has to manually sift through a transaction export looking for fraud — the report gives them a shortlist of the transactions worth a closer look, instead of scanning everything by hand.*

## 1. The demo

I run `python fraudsense.py score data/sample_transactions.csv`. It cleans the data, scores all 500 transactions, and prints:

Scored 500 transactions in 2.3s.
14 flagged as high-risk (score > 0.75).
Report written to reports/fraud_report.csv.

I open "reports/fraud_report.csv" and see the 14 flagged rows, each with a transaction id, a risk score, and a short reason. I then run `python fraudsense.py explain --id T00231` on one of them, and it prints the features that pushed the score up:


Transaction T00231 — risk score 0.91
  amount $4,200 is 8x this customer's typical transaction ($525)
  6 transactions in the last 10 minutes (typical: 1)

## 2. The shape


in            a CSV of transactions — amount, timestamp, customer_id,
              location, payment_method

out           a report of flagged transactions, each with a risk score
              and the top reasons it was flagged

in between    clean and validate the data; build per-customer behavioural
              features (typical spend, transaction frequency, location
              and payment-method patterns); score each transaction with
              a trained model plus a few simple rules; keep the
              highest-scoring transactions and their reasons


Model training and comparison happens once, offline, before this pipeline runs on new data — it is not something that happens on every run.

## 3. The size

**First useful version**

* Reads a transaction CSV and cleans it, exiting with a clear error if a required column is missing.
* Computes per-customer behavioural features: typical spend, transaction frequency, location and payment-method patterns.
* Applies a small set of rule-based checks (e.g. an amount far above the customer's average, many transactions in a short window).
* Scores every transaction with a trained ML model, chosen by comparing at least two candidate models offline using precision, recall, F1 and PR-AUC.
* Writes `reports/fraud_report.csv` listing the high-risk transactions, their scores and reasons.
* Lets me look up any transaction by id and see the specific features and rules behind its score.

**Not this term**

* Real-time fraud detection.
* Connecting to a real bank or payment system.
* Blocking transactions automatically.
* Using real private customer data.
* Building a full web application.
* Deploying the system online.
* Making it work on millions of transactions.

## 4. How we would know it works

1. Given a file missing a required column (e.g. amount or customer_id), the program exits with an error naming the missing column.
2. Given a test file containing an obviously unusual transaction an amount far above a customer's typical spend, or many transactions in a short time window — that transaction receives a risk score higher than the file's normal transactions.
3. Given a transaction flagged as high-risk, selecting it shows the specific features or rules that pushed its score up (e.g. "amount 8x customer average" or "5 transactions in 10 minutes").

## 5. What could stop this

The biggest risk is the data. Real bank transaction data is not available to me, so I will use a public dataset or build a small synthetic one if the public data does not contain the fields I need. Either way, the data and results can be shown in class.

Fraud cases are usually far less common than normal transactions, so accuracy alone would give a misleading picture — I will rely on precision, recall, F1 and PR-AUC instead.

I do not yet know which behavioural features will hold up on the dataset I choose — for example, I may not have enough history to calculate a reliable "normal spending pattern" per customer. I will adapt the features to what the data actually supports rather than forcing ones it can't.

The explanation feature is also untested territory. If model-based explanations turn out to be too complex for the first version, I will fall back to explanations based on the rules and the model's raw feature weights.
