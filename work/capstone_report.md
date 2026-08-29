# Capstone Report — <your lane>

- **Author:**
- **Lane:**
- **Repo:**
- **Date:**

> Copy this file to `work/capstone_report.md` and fill it in as you build. Sections 1–8
> mirror the Pass / Needs-Work rubric axes, so nothing here is optional. Sections 0 and 9
> are **paper sections**: your deployed research paper must carry both, and they're here so
> you never rebuild them from memory at ship time.

## 0. Abstract
FlyRank needs a practical way to prioritize content pages that may require investigation or refresh. This capstone frames that problem as a ranking task: given page-level content and performance signals from an anonymized content-refresh dataset, identify which pages should be investigated first. I performed a data and feature-leakage audit, established a baseline, developed a ranking model, and evaluated it using Precision@50 under a grouped client split designed to test performance on unseen clients. The grouped evaluation achieved a Precision@50 of 1.00 in this experiment, matching the original Week-5 result and exceeding the Week-4 baseline of 0.46. The result supports using the model as a prioritization aid for content review, while the grouped evaluation and limited client coverage mean that further validation on additional unseen clients is needed before treating the result as evidence of broad generalization.
## 1. Problem framing

### Decision

The system supports the decision of **which content pages should be investigated first for possible refresh or improvement**. Instead of treating every page equally, the goal is to prioritize the pages most likely to need attention.

### Unit of analysis

The unit of analysis is **one content page**. Each row in the dataset represents a page-level observation with content, performance, and other page-related signals.

### Output

The output is a **ranking/score of content pages**, allowing pages to be ordered by priority for investigation.

### Human action

A content or SEO team can use the ranking to **review the highest-priority pages first**, inspect the underlying performance and content signals, and decide whether a page should be refreshed, monitored, or left unchanged.

### Cost of a wrong call

A false positive can cause a team to spend time reviewing or refreshing a page that did not require attention. A false negative can be more costly because a page that actually needs attention may be missed or reviewed too late, potentially allowing declining content performance to continue.

### Why data/ML helps

A content team may have many pages to review, making manual prioritization difficult and inconsistent. Data provides measurable signals such as search and traffic performance that can be used systematically. Machine learning can combine these signals into a ranking that helps the team focus its limited review time on the pages most likely to deserve attention. The model is therefore intended as a **decision-support tool**, not as an automatic replacement for human content decisions.


## 2. Data safety

The analysis uses the anonymized FlyRank content-refresh dataset provided for the internship. The dataset does not require publishing personal information or identifying individual users.

Only aggregated or page-level content and performance signals are used for the analysis. No personally identifying information, credentials, private customer information, or raw user-level behavioral data are included in the report.

The dataset is used only for the purpose of demonstrating the machine-learning methodology and supporting the content-prioritization case study. Results are reported at an aggregate level and should not be interpreted as revealing information about individual users or private clients.

The analysis also includes a feature-leakage check to reduce the risk of using information that would not legitimately be available when making a prioritization decision.



## 3. Baseline

The baseline provides a simple reference point for judging whether the machine-learning approach adds value beyond a straightforward prioritization strategy.

For this project, the Week-4 baseline achieved a **Precision@50 of 0.46**. This means that 46% of the pages selected in the baseline's top 50 were relevant according to the evaluation target.

The baseline is important because a model should not be considered useful simply because it produces a ranking. Its performance should be compared with a simpler reference approach using the same evaluation metric.

The baseline therefore provides the reference point for evaluating whether the later model improves or maintains useful top-ranked results.


## 4. Model / analysis

The analysis uses a machine-learning ranking approach to prioritize content pages for investigation. The input consists of page-level content and performance signals available in the anonymized FlyRank dataset.

Before modeling, the available features were reviewed for potential target or identity leakage. The leakage audit identified no direct target or identity leakage fields, leaving **38 features** for the modeling workflow.

The model was developed to produce a ranking of pages rather than an automatic refresh decision. This distinction is important because the intended workflow keeps a human content team in the decision loop: the model identifies pages to review first, while the team determines the appropriate action.

The modeling workflow builds on the earlier baseline and evaluates whether the learned ranking can maintain useful top-ranked results when evaluated using a grouped client split. The grouped split separates clients between training and test data so that no client appears in both sets, providing a more realistic test of performance on unseen clients.


## 5. Evaluation

## 5. Evaluation

The primary evaluation metric for the ranking task is **Precision@50**. This metric measures the proportion of relevant pages among the 50 highest-ranked pages selected by the model. It directly reflects the intended use case: helping a content team identify a small set of pages to investigate first.

The grouped test set contains 3,504 pages: 1,987 non-declining pages (56.71%) and 1,517 declining pages (43.29%). The majority-class base rate is therefore **56.71%**.

The Week-4 baseline achieved a Precision@50 of **0.46**.

The original Week-5 model achieved a Precision@50 of **1.00**.

To test whether this result depended on having the same clients represented in both training and test data, the model was also evaluated using an honest grouped client split. The dataset contained **32 clients** in total, with **25 clients used for training and 7 clients held out for testing**. This produced **26,496 training rows and 3,504 test rows**, with **zero client overlap** between the two sets.

Under this grouped client evaluation, the model achieved a Precision@50 of **1.00**. The result was unchanged from the original Week-5 evaluation.

This result should be interpreted carefully. The unchanged score shows that the measured top-50 precision remained strong under the grouped client split used in this experiment. However, it does not prove that the model will generalize to every future client or dataset. The evaluation covers the available client groups, and additional validation on more unseen clients would provide stronger evidence of generalization.

### Evaluation summary

| Evaluation                      | Precision@50 |
| ------------------------------- | -----------: |
| Majority-class base rate        |       0.5671 |
| Week-4 baseline                 |         0.46 |
| Week-5 model                    |         1.00 |
| ML-09 grouped client evaluation |         1.00 |

The comparison shows a substantial improvement over the baseline and no measured drop from the original Week-5 model when evaluated using the grouped client split.


## 6. Interpretation

The results indicate that the machine-learning ranking approach was effective at identifying **declining pages** within the top 50 positions under the evaluation used in this project. The model achieved a Precision@50 of 1.00, compared with 0.46 for the Week-4 baseline.

The grouped client evaluation is particularly important because clients were separated between training and testing, with zero client overlap. The model maintained a Precision@50 of 1.00 under this split, so the measured top-50 performance did not decrease when evaluated on the held-out client groups used in the experiment.

However, the result should be interpreted within the limits of the evaluation. A Precision@50 of 1.00 means that the evaluated top 50 predictions were **declining pages according to the project's target definition**; it does not mean that every page is correctly ranked or that the model will achieve the same performance on all future data.

The result also does not establish causality. The model identifies pages associated with the target pattern using the available signals, but it does not prove that changing a particular page will cause its performance to improve.

For a content team, the most useful interpretation is therefore that the model can serve as a **prioritization mechanism**: it can narrow a large collection of pages into a smaller set for human investigation. The final content decision should still consider context that may not be represented in the dataset.


## 7. Recommendation

The model should be used as a **decision-support and prioritization tool** for content teams. Its primary role is to help reduce a large set of content pages into a smaller group that can be investigated first.

A practical workflow would be:

1. Generate the model's ranking of content pages.
2. Select the highest-priority pages for review.
3. Inspect the page's underlying performance and content signals.
4. Have a content or SEO specialist determine whether a refresh, monitoring, or no action is appropriate.
5. Record the outcome and use future results to evaluate and improve the prioritization workflow.

The model should not automatically trigger content changes based only on its ranking. Human review remains important because business context, search intent, content quality, and other factors may not be represented in the available dataset.

The evaluation also suggests that further validation is worthwhile. The model maintained a Precision@50 of 1.00 under the grouped client split used in this experiment, but additional testing on unseen clients and future data would provide stronger evidence about real-world generalization.

Therefore, the recommended next step is to treat the model as a **ranking aid for human investigation**, validate it on additional unseen data, and monitor whether the pages prioritized by the model lead to useful content decisions over time.


## 8. Reproducibility

The analysis is implemented in the notebooks contained in the repository under `work/notebooks/`. The main capstone workflow is available in `work/notebooks/capstone.ipynb`, with the earlier research, task-framing, leakage-check, baseline, modeling, and validation work preserved in the preceding notebooks.

The dataset used for the analysis is the anonymized FlyRank content-refresh dataset provided for the internship. The modeling workflow includes data preparation, feature inspection, leakage checking, baseline evaluation, model development, and grouped client validation.

The final evaluation uses a grouped client split with no client overlap between training and test data. The reported metrics in this paper should be reproduced by running the corresponding notebook cells using the same dataset and evaluation procedure.

All analysis code, report files, and supporting notebooks are committed to the project repository so that the workflow can be inspected and reproduced.


## 9. Acknowledgments & data credit
Built on the FlyRank ML Internship dataset — [FlyRank](https://flyrank.ai)
