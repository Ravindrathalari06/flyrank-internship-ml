# Prioritizing Potentially Declining Content with Machine Learning: An Honest Client-Holdout Evaluation on Anonymized Search Data

## 1. Abstract

Content teams often have thousands of pages that could potentially require investigation, but reviewing every page manually is inefficient. This study asks whether a machine-learning model can help prioritize potentially declining content pages for human review using observable search and engagement signals. We use an anonymized FlyRank dataset containing 30,000 content-page records and define a binary decision-support label in which pages with `trend_direction = down` are treated as declining. We compare a transparent hand-written baseline with Logistic Regression, Decision Tree, and Random Forest models using Precision@50, with evaluation performed under client-holdout validation. The learned models substantially outperform the Week-4 baseline in the evaluated dataset: Logistic Regression and Decision Tree each achieve Precision@50 of 1.00, Random Forest achieves 0.94, while the baseline achieves 0.46. A subsequent grouped client validation produces Precision@50 of 1.00 for the Logistic Regression model, unchanged from the original Week-5 result, with zero client overlap between training and test sets. These results provide directional evidence that learned ranking can support prioritization of potentially declining pages, while the findings should not be interpreted as proof of universal or future performance.

**Keywords:** search intelligence, content refresh, machine learning, ranking, Precision@50, content decay, client-holdout validation, decision support

---

# 2. Introduction / Problem

Content teams frequently need to decide which pages deserve attention first. A website may contain thousands of pages, but available editorial and SEO resources are limited. The practical question is therefore not simply whether a page is declining, but **which pages should be investigated first**.

A simple manual approach can rank pages using signals such as visibility, freshness, search position, click-through rate, and content depth. This approach is transparent and easy to explain, but it relies on manually chosen rules.

Machine learning offers another approach. Instead of specifying every ranking rule manually, a model can learn a relationship between observable page characteristics and an observed outcome. The resulting scores can then be used to create a prioritized review queue.

This work investigates the following research question:

> **Can a learned machine-learning model provide useful decision support for prioritizing potentially declining content pages compared with a transparent hand-written baseline?**

The central result is that, on the evaluated anonymized dataset and client-holdout split, the learned models achieve substantially higher Precision@50 than the hand-written baseline. In particular, Logistic Regression achieves a measured Precision@50 of **1.00** compared with **0.46** for the Week-4 baseline.

However, the objective of this study is deliberately narrower than predicting search-engine ranking behavior. The model identifies pages associated with an observed decline label in the supplied dataset. Therefore, the results should be interpreted as **measured, directional decision-support evidence** rather than a claim that the model predicts Google's ranking algorithm.

---

# 3. Data

## 3.1 Dataset

The analysis uses the anonymized FlyRank content-refresh dataset provided through the internship program.

The analyzed dataset contains:

* **30,000 rows**
* **44 original columns**
* **32 clients**
* Search-performance, engagement, content, freshness, and ranking-related signals

The dataset contains an observed `trend_direction` field with five categories:

| Trend direction |       Rows |
| --------------- | ---------: |
| down            |     16,262 |
| stable          |      5,962 |
| up              |      4,388 |
| new             |      2,236 |
| flat            |      1,152 |
| **Total**       | **30,000** |

Thus, 16,262 of the 30,000 records are labeled as declining under the binary decision-support definition, corresponding to approximately **54.2%** of the dataset.

## 3.2 Unit of analysis

The unit of analysis is an individual **content page record**.

Each row represents one page-level observation containing signals describing search visibility, engagement, content characteristics, freshness, and related attributes.

## 3.3 Label definition

For the modeling task, the observed trend field was converted into a binary label:

```text
is_declining_label = 1  if trend_direction == "down"
                     0  otherwise
```

This means that the model is not predicting an independently collected future outcome. Instead, it learns to rank pages according to their relationship with the observed `trend_direction` field.

This distinction is important when interpreting the results.

## 3.4 Candidate signals

The final model used **38 feature fields**.

There were:

* **29 numeric features**
* **9 categorical features**

Examples include:

* `search_volume`
* `competition`
* `competition_level`
* `cpc`
* `content_type`
* `main_intent`
* `word_count`
* `char_count`
* `impressions_90d`
* `clicks_90d`
* `pageviews_90d`
* `sessions_90d`
* `users_90d`
* `engaged_sessions_90d`
* `days_since_last_update`
* `content_age_days`
* `ctr`
* `avg_position`
* `engagement_rate`
* `scroll_rate`
* `ai_traffic_pct`

The final feature set excluded:

* `trend_direction`
* `trend_pct`
* `content_id`
* `client_id`
* `provider_used`
* `model_used`

This exclusion was part of the leakage and feature-safety workflow.

---

# 4. Methodology

## 4.1 Baseline

Before applying machine learning, a transparent hand-written ranking rule was established.

The baseline combines four intuitive dimensions:

1. **Visibility**
2. **Freshness**
3. **Position opportunity**
4. **Content-depth gap**

The baseline uses signals including:

* `impressions_90d`
* `days_since_last_update`
* `avg_position`
* `word_count`
* `ctr`

It also produces interpretable reason codes such as:

* `stale_visible_page`
* `thin_visible_page`
* `page_one_opportunity`
* `low_ctr_visible_page`
* `general_refresh_review`

No target-derived field such as `trend_direction` or `trend_pct` is used in the baseline score.

This baseline provides an important reference point because it represents a plausible rule-based workflow that a content team could implement without machine learning.

## 4.2 Machine-learning models

Three supervised learning methods were evaluated:

1. **Logistic Regression**
2. **Decision Tree**
3. **Random Forest**

Logistic Regression was implemented using preprocessing appropriate for mixed numerical and categorical data.

For numerical features:

* missing values were imputed using the median;
* features were standardized.

For categorical features:

* missing values were imputed using the most frequent category;
* categorical values were one-hot encoded.

The Logistic Regression model used a maximum of 1,000 iterations.

## 4.3 Evaluation metric

The primary evaluation metric is **Precision@50**.

Precision@50 measures the fraction of genuinely declining pages among the 50 pages receiving the highest model scores.

```text
Precision@50 =
number of declining pages among top 50 / 50
```

This metric matches the practical decision problem: if a content team can investigate only a small number of pages, how useful is the model's highest-priority queue?

For example, a Precision@50 of 0.80 means that 40 of the top 50 recommended pages are labeled as declining.

## 4.4 Client-holdout validation

The Week-5 modeling workflow used a client-holdout split.

There were **32 clients** in total, with:

* **25 clients** in training
* **7 clients** in testing

The original Week-5 split contained:

* **26,581 training rows**
* **3,419 test rows**

The later ML-09 validation audit used an honest grouped client split containing:

* **26,496 training rows**
* **3,504 test rows**
* **0 client overlap**

The absence of client overlap in the ML-09 grouped evaluation is important because pages belonging to the same client can share characteristics. Separating clients between training and testing provides a more realistic test of performance on held-out client groups.

## 4.5 Leakage audit

A final leakage audit was performed on the 38-feature model.

The excluded fields were:

```text
['trend_direction',
 'trend_pct',
 'content_id',
 'client_id',
 'provider_used',
 'model_used']
```

The audit therefore excluded the observed target field, trend percentage, page identity, client identity, and provider/model metadata from the final model features.

The final feature configuration contained:

* **38 features**
* **29 numeric features**
* **9 categorical features**

This reduces the risk of directly using target or identity information in the learned model.

---

# 5. Results

## 5.1 Model comparison

The learned models were compared with the Week-4 hand-written baseline using Precision@50.

| Method              | Precision@50 |
| ------------------- | -----------: |
| Logistic Regression |     **1.00** |
| Decision Tree       |     **1.00** |
| Random Forest       |     **0.94** |
| Week-4 Baseline     |     **0.46** |

The Week-5 model comparison identified **Logistic Regression as the best learned model**, with a measured Precision@50 of **1.00**.

The Decision Tree also achieved **1.00**, while Random Forest achieved **0.94**.

The Week-4 baseline achieved **0.46**.

For Logistic Regression, the measured difference over the baseline is:

```text
1.00 - 0.46 = 0.54
```

Therefore, the measured Precision@50 improvement is **0.54 absolute points**.

The results are measured on the evaluated dataset and should not be interpreted as a universal performance guarantee.

## 5.2 Base-rate check

The ML-09 grouped test set contains **3,504 pages**.

The target distribution was:

| Class         |     Pages | Proportion |
| ------------- | --------: | ---------: |
| Non-declining |     1,987 | **56.71%** |
| Declining     |     1,517 | **43.29%** |
| **Total**     | **3,504** |   **100%** |

The majority-class base rate is therefore **56.71%**.

This base rate is reported alongside Precision@50 because a ranking metric should be interpreted in the context of how common the target class is in the evaluation set.

The model's measured Precision@50 of **1.00** means that all 50 pages in the evaluated top-50 queue were declining according to the project's target definition.

## 5.3 ML-09 grouped validation audit

The Week-5 Logistic Regression model was subsequently evaluated using the grouped client split as part of the ML-09 validation audit.

| Validation result          |    Value |
| -------------------------- | -------: |
| Training clients           |       25 |
| Test clients               |        7 |
| Training rows              |   26,496 |
| Test rows                  |    3,504 |
| Client overlap             |    **0** |
| Week-5 Precision@50        | **1.00** |
| ML-09 grouped Precision@50 | **1.00** |
| Change                     | **0.00** |

The measured Precision@50 therefore remained unchanged under the tested grouped client evaluation.

This is useful evidence because the result did not decrease when the validation explicitly enforced client separation.

However, unchanged performance under one grouped split does not establish robustness across all future clients, time periods, industries, or datasets.

## 5.4 Main finding

The main finding of this study is:

> **Within the evaluated anonymized FlyRank dataset, learned ranking models provided substantially stronger top-50 prioritization than the hand-written baseline, and the Logistic Regression result remained at Precision@50 = 1.00 under the tested grouped client-holdout validation.**

The strongest interpretation is therefore that machine learning can provide useful **prioritization support** for this specific content-review workflow.

---

# 6. Limitations & Honest Framing

This study has several important limitations.

## 6.1 The target is an observed label

The binary target is derived directly from `trend_direction`.

Therefore, the model is learning to identify pages associated with an observed decline category rather than predicting a future decline from a genuinely prospective forecasting setup.

## 6.2 The dataset is anonymized and limited

The analysis uses the supplied anonymized dataset. Although it contains 30,000 records, it represents only the available evaluation dataset.

The findings should not automatically be generalized to all websites, industries, clients, or search environments.

## 6.3 No explicit calendar-time split

The validation audit tests generalization across clients but does not fully test temporal generalization.

A future study should evaluate a time-aware split in which earlier observations are used for training and later observations are reserved for testing.

## 6.4 High Precision@50 should be interpreted carefully

A Precision@50 of 1.00 is an excellent measured result, but it does not mean that the model will always identify the 50 most important pages in production.

Precision@50 evaluates the top 50 records in the particular test set. It does not measure:

* long-term business impact;
* causal effect of content changes;
* future search-engine ranking changes;
* performance on unseen time periods;
* performance after deployment.

## 6.5 No causal interpretation

The model identifies statistical relationships between available signals and the observed label.

It does not establish that changing a particular feature will cause a page to recover.

For example, a relationship between freshness and decline should not automatically be interpreted as proof that updating a page will improve its performance.

## 6.6 No claim about Google's algorithm

This work does not reverse-engineer or predict Google's proprietary ranking algorithm.

The appropriate interpretation is narrower:

> The model provides decision support for prioritizing pages associated with an observed decline signal in the analyzed dataset.

---

# 7. Ranked Recommendations

The modeling results suggest the following practical workflow for a content team.

## Recommendation 1 — Review the highest-scoring pages first

Use the model's ranking score to create a prioritized review queue.

The immediate operational objective should be to focus limited human attention on the highest-ranked pages rather than attempting to review the entire content inventory simultaneously.

## Recommendation 2 — Preserve human review

The model should be treated as a **triage system**, not an automatic content-update system.

A high score should trigger investigation.

The reviewer can then determine whether the page actually needs:

* a content refresh;
* improved search-intent alignment;
* stronger topical coverage;
* improved content depth;
* title or snippet review;
* technical investigation;
* no action.

## Recommendation 3 — Combine model ranking with interpretable signals

The baseline reason codes remain useful even when the learned model performs better.

For example, a highly ranked page that is both stale and highly visible may deserve immediate investigation because the model ranking and human-readable reason agree.

## Recommendation 4 — Add temporal validation before production use

Before treating the ranking system as a production workflow, evaluate it on a future time window.

A strong next experiment would be:

```text
Earlier observations → Training
Later observations   → Validation/Test
```

This would answer a different and important question:

> Does the model continue to prioritize declining pages when applied to a later period?

## Recommendation 5 — Track outcomes after intervention

A future version of the project should connect recommendations to actual post-intervention outcomes.

For example:

```text
Model ranking
      ↓
Human review
      ↓
Content change
      ↓
Future performance measurement
```

This would allow the project to move from ranking evaluation toward measuring whether the recommendations actually create useful business outcomes.

---

# 8. Reproducibility

The analysis is implemented in the public GitHub repository:

**FlyRank Internship ML — Ravindra Thalari**

https://github.com/Ravindrathalari06/flyrank-internship-ml

The repository contains the assignment notebooks used throughout the workflow, including :
w01_research_question.ipynb
w02_ml_task_framing.ipynb
w03_data_contract.ipynb
w03_feature_leakage_check.ipynb
w04_signal_audit.ipynb
w04_baseline_score.ipynb
w05_model.ipynb
w06_validation_audit.ipynb
w07_action_playbook.ipynb
capstone.ipynb

The main capstone notebook is:

```text
work/notebooks/capstone.ipynb
```

The repository documents the workflow as:

```text
problem framing
      ↓
data preparation
      ↓
leakage checking
      ↓
baseline
      ↓
machine-learning models
      ↓
Precision@50 evaluation
      ↓
grouped client validation
      ↓
ranked recommendations
```

The capstone workflow also creates supporting artifacts for the paper, including:

* ranked action queue;
* model-versus-baseline results table;
* Precision@50 comparison figure;
* suggested-action distribution figure.

For reproducibility, future reruns should preserve:

* the feature list;
* label definition;
* preprocessing procedure;
* client split;
* Precision@50 definition;
* leakage exclusions.

---

# 9. Acknowledgments & Data Credit

Built on the [FlyRank](https://flyrank.ai) ML Internship dataset.
