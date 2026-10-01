# l1ght14

Machine learning engineer working on applied ML and LLM systems - classification,
forecasting, retrieval, and agent safety - with the evaluation treated as the part that
actually matters.

## Selected work

Four projects, built around one recurring question: **how do you know your evaluation isn't
lying to you?** Each one found a real defect in its own measurement before it found anything
interesting about the model.

### [Retrieval Evaluation Harness](https://github.com/l1ght14/retrieval-eval-harness) - FiQA, 57,638 documents

Measures whether a retriever found the right passage, and keeps that separate from whether
the model used it correctly - because the two have different fixes and different costs, and
collapsing them into one "accuracy" destroys the information you need.

- Dense retrieval beats BM25 by **+0.1767 recall@10, 95% CI [+0.1182, +0.2344]** - the one
  comparison in the project that is not in doubt
- BM25 fusion contributes **nothing at any weighting**: at BM25 weight 0.3 the delta is exactly
  `+0.0000` with a zero-width interval. The lexical arm changed no query's top-10
- Cross-encoder reranking costs **400x the latency** (22 ms to 8,810 ms) for a gain the
  interval cannot separate from zero
- **Smaller chunks were worse.** 512 > 256 > 128, because FiQA's median document is 90 words
  and splitting it severs the term from its context. The best setting is effectively *no
  chunking* - the opposite of the usual advice

I wrote the wrong conclusion first and corrected it twice. The first draft claimed fusion and
reranking were *harmful* on point estimates of -0.007 and -0.025; the paired tests showed every
interval containing zero. Then a bug was found in my own reranker path - it scored the
retrieved chunk while the generator was shown the whole document - which moved reranking from
0.4470 to **0.4863** and reversed the sign. `eval/report.py` now generates the expected claim
text from stored results and fails if the prose disagrees.

175 tests, 90% coverage. The CI gate fails on regression, on an unknown config, and on corpus
provenance drift. Re-runs are bit-identical.

### [Prompt Injection Guardrails](https://github.com/l1ght14/prompt-injection-guardrails) - 60 attacks, 8 families

A recruiting agent reads resumes and writes back to an ATS. Every document was written by
someone with an incentive to manipulate the outcome.

The thesis: **filtering the phrase about ignoring previous instructions is theatre.** The
control that matters is limiting what the agent may do *after* it has been fooled.

| | undefended | guarded |
|---|---|---|
| Attack success rate | **0.8667** | **0.0000** |
| Containment of attempted attacks | - | **1.0000** |
| False positives (100 benign resumes) | - | **0.0000** |

While untrusted content is in scope the capability ceiling is `Risk.SCORE`, so an agent that
has genuinely been convinced cannot reach `ats_write` or `send_email`. **No detection rule is
load-bearing** - delete every pattern and containment is unchanged, because the ceiling never
consults them.

Three findings worth reading:

- **Score laundering defeated the capability layer, and the ceiling was right anyway.**
  `score_candidate` is inside the untrusted ceiling because refusing to score means refusing to
  do the job. Closed with a *different* control - a value-provenance check. The attacker-chosen
  score is blocked; an agent-computed score still passes.
- **My first baseline flattered the guardrails.** The undefended arm still required a
  confirmation token, so every email-targeting attack scored 0.0000 *before any guardrail
  existed*. Real baseline: 0.53 to **0.87**.
- **8 of 60 attacks are reported as unattempted, not contained.** My regex simulator cannot
  read a base64 payload. They count against the success rate but are not credited to the
  guardrails, and containment is computed over attempts only.

Three bypasses documented as executable before/after evidence. 70 tests, 92% coverage.

### [Customer Churn Prediction](https://github.com/l1ght14/customer-churn-prediction) - Telco, 7,032 subscribers

Binary classification on a 26% positive class, where accuracy is a trap - predicting "nobody
churns" scores 73%.

The substance is four ways this evaluation was quietly wrong, each found by adversarial review
and each now pinned by a test:

- model selection was reading holdout labels
- campaign thresholds were derived from the very holdout they were scored against, which pins
  recall to whatever you targeted and measures nothing
- the threshold came from *in-sample* train scores, where the model has partly memorised its
  own labels - it missed its own 80% recall target by 2.7 points on holdout
- the shipped call list was graded by a model that had memorised it, inflating lift from an
  honest 2.86x to 3.08x

157 tests, `pyflakes` clean, byte-reproducible. `tests/test_docs.py` asserts every number
quoted in the README still matches the generated JSON, so a model change that moves a figure
breaks the build rather than silently making the documentation wrong.

Business read: 87% of revenue at risk sits in month-to-month contracts churning at 42.7%
against 2.8% for two-year.

### [Bike Demand Forecasting](https://github.com/l1ght14/bike-demand-forecasting) - UCI, 731 days

Seven-day-ahead daily demand, where the interesting result is the one that does not flatter:

- **the model loses to the seasonal naive on holdout** - MASE 1.098 against 1.000
- 80% prediction intervals achieve **69% actual coverage**, not 80%
- holdout bias is **+472.8 units**; the seasonal naive is nearly as bad at +459.7
- weather features remove **35.5%** of gradient-boosting error but only **5.2%** of Poisson
  error - most of the apparent weather gain was an artefact of which model was used

Reported because the negative result is the transferable one. 132 tests, byte-reproducible.

## How I work

Every project above reports what was **wrong** alongside what was right, and ships a test or a
check that fails if that defect returns. Where a difference turned out not to be significant,
the write-up says so rather than quoting the point estimate - twice above, that correction
changed the conclusion.