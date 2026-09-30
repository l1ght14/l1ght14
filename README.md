# l1ght14

<!-- Replace the line below with your name. Everything else here is factual. -->

Machine learning engineer working on applied ML — classification and forecasting on real
business problems, with the evaluation treated as the part that actually matters.

## Selected work

These two are the ones I'd want reviewed. Both are built around the same question: **how do
you know your evaluation isn't lying to you?**

### [Customer Churn Prediction](https://github.com/l1ght14/customer-churn-prediction) · Telco, 7,032 subscribers

Binary classification on a 26% positive class, where accuracy is a trap — predicting "nobody
churns" scores 73%.

The substance is four ways this evaluation was quietly wrong, each found by adversarial
review and each now pinned by a test that fails if the bug returns:

- model selection was reading holdout labels
- campaign thresholds were derived from the very holdout they were scored against, which
  pins recall to whatever you targeted and measures nothing
- the threshold came from *in-sample* train scores, where the model has partly memorised its
  own labels — it missed its own 80% recall target by 2.7 points on holdout
- the shipped call list was graded by a model that had memorised it, inflating lift from an
  honest 2.86x to 3.08x

157 tests, `pyflakes` clean, byte-reproducible artefacts. `tests/test_docs.py` asserts every
number quoted in the README still matches the generated JSON, so a model change that moves a
figure breaks the build rather than silently making the documentation wrong.

Business read: 87% of revenue at risk sits in month-to-month contracts churning at 42.7%
against 2.8% for two-year.


### [Bike Demand Forecasting](https://github.com/l1ght14/bike-demand-forecasting) · UCI, 731 days

Seven-day-ahead daily demand forecasting — the horizon a bike operator actually faces when
committing staff and rebalancing capacity.

**The headline is a loss, and the README leads with that.** The model beats the seasonal naive
on 231 development days at MASE 0.796 and *loses* to it on the 84 scored holdout days at
MASE 1.098. Rather than quote the development number, the gap is diagnosed: holdout bias is
+472.8 rentals against the seasonal naive's +459.7 on the same rows, so the model adds almost
no bias of its own — it inherits a level problem, because a tree ensemble cannot extrapolate a
falling trend. Residual autocorrelation at lag 7 is 0.049, so the weekly cycle *is* captured.
The error is level, not pattern.

Also worth noting:

- features are built off a horizon-shifted target, so leakage is structurally impossible rather
  than merely checked. `y[i-1]` is the most predictive value in the series and is deliberately
  excluded, because at a seven-day lead nobody knows it
- weather columns are contemporaneous actuals, so two feature sets are built and the assumption
  is measured: weather removes **35.5%** of the error for gradient boosting and **5.2%** for
  Poisson, and without a weather forecast the best model is worse than naive
- the per-horizon error table turned out to be confounded with day of week, and is labelled as
  such on every row rather than quoted as a lead-time finding

132 tests, byte-reproducible, and hermetic — an earlier suite deleted the project's own
committed deliverable while reporting green.


## Earlier work

Around 25 further projects covering NLP, recommendation, clustering, forecasting and
experiment design — visible in the repository list. Those are earlier and more exploratory;
the two above are where I work to a standard I'd defend in a review.

## Contact

<!-- Add a contact method. -->

## Stack

Python · pandas · NumPy · scikit-learn · matplotlib · pytest
