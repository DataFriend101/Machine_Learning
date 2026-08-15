# Capstone Report

**Author:** Sofia Bourjeily  
**Lane:** Refresh / Content Opportunity Scoring  
**Repo:** https://github.com/DataFriend101/Machine_Learning  
**Date:** 15/08/2026

---

## 1. Problem framing

This project asks whether content at risk of declining can be identified early enough to prioritize which pages should receive attention.

The unit of analysis is a **content item/page**. The output is a **ranked priority score** that helps content teams decide which pages to review first.

A human editor can use the ranking to investigate a page and decide whether a refresh or other action is appropriate. A wrong call can lead to time being spent reviewing a page that does not need attention, or a potentially important page being missed.

Data and ML help by combining multiple observable performance and content signals into a consistent ranking rather than relying only on a simple rule or manually reviewing every page.

## 2. Data safety

This analysis uses the FlyRank ML Internship dataset, release `flyrank_pseudonymized_warehouse_release_v20260703`, using the anonymized `content_refresh_anonymized.csv` slice provided for the internship.

The data contains observed search and engagement metrics, content metadata, age/freshness fields, and derived comparison windows. It contains no client names, domains, URLs, page titles, keywords, or raw search queries.

`client_id` and `content_id` were used only for grouping and validation and were not model features.

I deliberately excluded `trend_direction` and `trend_pct` from the model because they are related to the observed decline label and could create leakage. Product decision flags and composite scores were also not used.

I checked for label-derived, future/overlapping information, and decision-derived features. `content_age_days` and `days_since_last_update` showed no obvious leakage, while `impressions_90d`, `ctr`, and `avg_position` require timeline verification because they are performance measures that could overlap with the period used to define the observed trend.

No client-identifying information appears in the published analysis.

## 3. Baseline

The Week 4 baseline ranked pages using a simple combination of recent visibility and CTR opportunity.

It provides a transparent comparison because it addresses the same prioritization task as the Random Forest and uses the same **Precision@50** metric on the same client-grouped test split.

The reported results were:

- **Base rate:** 0.511
- **Week 4 baseline Precision@50:** 0.440
- **Random Forest Precision@50:** 0.680

## 4. Model / analysis

I used a **Random Forest classifier** because it can capture nonlinear relationships and interactions between multiple content-performance signals.

The five model features were:

- `impressions_90d`
- `ctr`
- `avg_position`
- `content_age_days`
- `days_since_last_update`

`client_id` and `content_id` were used only for grouping and validation. `trend_direction` and `trend_pct` were excluded because of leakage risk.

The target was `is_declining_label`, where **1 = `trend_direction == "down"` and 0 = otherwise**. The positive rate in the full dataset was **54.2%**.

The Random Forest used **200 trees**, `random_state=42`, and **balanced class weights**.

## 5. Evaluation

The primary evaluation used an **80/20 client-grouped split** with `random_state=42`, keeping pages from the same client entirely in either training or testing.

On this test split, the Random Forest achieved **0.680 Precision@50**, compared with **0.440** for the baseline and a **0.511** test-set base rate.

The Random Forest therefore placed **34 declining-labeled pages in its top 50**, compared with **22** for the baseline.

The approach was also evaluated using **5-fold client-grouped validation**. Precision@50 across the folds was:

- **Fold 1:** 0.800
- **Fold 2:** 0.840
- **Fold 3:** 0.600
- **Fold 4:** 0.820
- **Fold 5:** 0.780

The mean was **0.768** with a standard deviation of **0.086**.

The variation across folds shows that performance differs across client groups.

## 6. Interpretation

The strongest Random Forest feature importances were:

- `impressions_90d`: **0.322**
- `avg_position`: **0.291**
- `content_age_days`: **0.207**
- `ctr`: **0.133**
- `days_since_last_update`: **0.047**

This means recent visibility, observed ranking, and content age contributed most strongly to the model's ranking.

These are **feature importance values, not causal effects**. The results show measured associations with the observed decline label rather than evidence that changing any feature will cause performance to improve.

## 7. Recommendation

The model should be used as a **prioritization layer for human review**.

The recommended workflow is:

**rank → investigate → review → decide → act**

First, review the highest-scoring pages. Then investigate their visibility and ranking signals, followed by content age and freshness.

The model should not automatically rewrite, publish, delete, redirect, or otherwise change content. Final decisions remain with a human editor.

The results are **directional** and should be treated as **decision support**. They may not generalize to every client, content type, or search environment.

## 8. Reproducibility

The analysis is organized across the project notebooks covering the research question, task framing, data contract, leakage checks, baseline, signal audit, model training, validation, action playbook, and paper deployment.

The main model uses `random_state=42`, with an **80/20 client-grouped split**. The environment should be recreated using the project's `requirements.txt`
