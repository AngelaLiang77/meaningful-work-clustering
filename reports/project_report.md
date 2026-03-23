# Project Report: Dual Clustering Analysis of Meaningful Work

**Authors:** Jung Shan Liang and Man-Chen Chen
**Date:** 2025
**Type:** Original Applied Study

---

## 1. Introduction

Understanding the drivers of meaningful work is a cornerstone of organizational psychology and HR analytics, as it directly impacts engagement, retention, and well-being. However, meaningful work is a multidimensional construct — it cannot be reduced to a single metric. Instead, it emerges from a synergy between scale-based metrics and open-ended qualitative reflections.

This study asks: when individuals articulate the underlying "why" behind their professional purpose, do they coalesce into homogeneous groups with shared perspectives, or do they provide distinct, idiosyncratic reasons? Additionally, we consider the role of personality as a potential differentiator in how meaningful work is conceptualized across different employee segments.

### Research Questions

- **RQ1 (Primary):** Do participants who cluster similarly on work-meaning survey items (CMWS + WAMI) also produce semantically similar narratives about meaningful work?
- **RQ2 (Secondary):** Does personality-based clustering (NEO) show any association with narrative clustering or with work-meaning clustering?

---

## 2. Data Description

### 2.1 Dataset

The dataset was collected from Harvard Dataverse (Morrison & Kale, 2019). It contains open-ended narrative responses alongside a battery of validated psychometric scales from full-time workers.

- **Observations (n):** 194 (after listwise deletion of 2 rows with missing scale items; raw dataset = 196)
- **Features (p):** 75 total columns (64 numerical scale items + text response + metadata)
- **Outcome:** Unsupervised — no labeled outcome variable

### 2.2 Psychometric Scales

**CMWS (28 items, range 1–5):** The Comprehensive Meaningful Work Scale evaluates seven dimensions — Unity with Others, Serving Others, Expressing Full Potential, Developing Inner Self, Reality, Inspiration, and Balancing Tensions.

**WAMI (10 items, range 1–10):** The Work and Meaning Inventory assesses three facets — Positive Meaning, Meaning Making through Work, and Greater Good Motivations.

**NEO-FFI Short Form (10 items, range 1–6):** Measures the Big Five personality traits — Reserved, Trusting, Lazy, Handles Stress, Few Artistic Interests, Outgoing, Fault Others, Thorough, Nervous, Imagination.

**Additional scales (not used for clustering):** Affective Commitment (6 items), Portrait Values Questionnaire (10 items).

**Qualitative data:** An open-ended text response (Overall_Why) specifically designated for text clustering.

### 2.3 Preprocessing

- Removed rows with any missing values on scale items (2 rows removed, 1.0% of data)
- Standardized all numerical features to zero mean and unit variance (StandardScaler) within each feature group before clustering
- No PCA applied to numerical features — clustering is performed directly in the standardized feature space
- Text responses extracted as a list of 194 documents for sentence embedding

---

## 3. Methodology

### 3.1 Overview

The analysis proceeds in three parallel tracks:

1. **Clustering A (Work-Meaning):** KMeans on 38 standardized items from CMWS + WAMI
2. **Clustering B (Personality):** KMeans on 10 standardized NEO items
3. **Text Clustering:** Sentence embedding → UMAP → HDBSCAN on open-ended narratives

All three clusterings share the same random seed (RANDOM_STATE = 42) for reproducibility.

### 3.2 Baseline

For numerical clustering, k = 1 (treating all participants as one group with a single centroid) serves as the degenerate baseline. Clustering is considered meaningful only when the silhouette score substantially exceeds the degenerate case and when cluster profiles are theoretically interpretable.

### 3.3 Numerical Clustering: Model Specification

Both Clustering A and Clustering B use KMeans, minimizing the within-cluster sum of squared Euclidean distances. Features are first standardized (z-scored) to ensure that scales with different response ranges (CMWS: 1–5, WAMI: 1–10, NEO: 1–6) contribute equally to the distance metric.

### 3.4 Numerical Clustering: k Selection

We selected k via two complementary criteria over k ∈ {2, ..., 10} with n_init=10:

- **Silhouette coefficient:** Measures how similar each observation is to its own cluster compared to the nearest other cluster. Higher is better (range [-1, 1]).
- **Elbow method:** Inertia (within-cluster sum of squares) plotted against k; the "elbow" indicates diminishing returns.

Final k selection also incorporated theoretical interpretability: a solution was retained only if cluster profiles mapped onto meaningful constructs in the literature.

### 3.5 Text Clustering: Sentence Embedding

Open-ended narrative responses were encoded using the pre-trained **all-mpnet-base-v2** model (Reimers & Gurevych, 2019), a 110M-parameter transformer that maps variable-length text to 768-dimensional dense vectors. This model was selected for its strong performance on semantic similarity benchmarks.

### 3.6 Text Clustering: UMAP Dimensionality Reduction

UMAP was applied to reduce the 768-dimensional embeddings to a lower-dimensional space suitable for density-based clustering. We tested n_components ∈ {5, 10, 15, 20, 30, 50} with fixed n_neighbors=15 and metric='cosine', evaluating each via HDBSCAN silhouette on the resulting clusters.

**n_components = 10** was selected as it produced the highest silhouette score (0.5374). The final UMAP was fit with min_dist=0.0 and random_state=42.

### 3.7 Text Clustering: HDBSCAN

HDBSCAN is a density-based algorithm that identifies clusters of arbitrary shape and assigns low-density points as outliers (label = −1). Unlike KMeans, it does not require specifying k in advance. The key hyperparameter is min_cluster_size (MCS).

We swept MCS ∈ {5, 6, 7, 8, 10, 12, 15, 20} and evaluated each setting via **Bootstrap Adjusted Rand Index (ARI)** — the mean ARI between full-data labels and labels produced on five 80% random subsamples.

**min_cluster_size = 7** achieved the highest Bootstrap ARI (0.5934) and was selected. This produced 5 clusters with 26.3% outliers.

### 3.8 Text Cluster Interpretation

cTF-IDF (class-based TF-IDF) and representative documents (closest to embedding centroid) were used to support interpretation. We initially labeled these clusters independently; we then met to resolve discrepancies and reached a final consensus on the thematic labels.

### 3.9 Cross-Clustering Comparison: Chi-Squared Test

To assess whether cluster memberships across modalities are statistically associated, we used the chi-squared test of independence on contingency tables. Three comparisons were performed:

1. Work-Meaning clusters × Personality clusters (n = 194; all participants)
2. Work-Meaning clusters × Text clusters (n = 143; outliers excluded)
3. Personality clusters × Text clusters (n = 143; outliers excluded)

Cells with expected count < 5 reduce the reliability of the chi-squared approximation; we flagged comparisons where more than 20% of cells fell below this threshold.

---

## 4. Evaluation Strategy

Because this is an unsupervised study, there is no labeled outcome for held-out prediction. Model evaluation relies on internal validity metrics (silhouette coefficient, inertia elbow) and external validity via cross-modality comparison. For text clustering, bootstrap stability (Bootstrap ARI over 5 resamples at 80%) served as the primary selection criterion for min_cluster_size.

### Evaluation Metrics

- **Silhouette Coefficient** (higher is better, range [−1, 1]): Measures cohesion vs. separation for each observation.
- **Bootstrap ARI** (higher is better, range [0, 1]): Measures pairwise label agreement between full-data and bootstrap-subsample clustering solutions.
- **Chi-squared (χ²):** Tests statistical independence between two cluster assignment vectors. A p-value < 0.05 indicates non-random association.

---

## 5. Results

### 5.1 Clustering A: Work-Meaning

KMeans with k=2 identified two profiles (silhouette = 0.40):

- **Cluster 0 — "Self-Estranged Contribution"** (n=153, 78.9%): Higher scores across all CMWS dimensions and WAMI facets (CMWS mean: 3.99, WAMI mean: 7.10). Top distinguishing features: CMWS_INSPIRED (z=+0.39), WAMI_KNOW_WHY (z=+0.38), CMWS_IMPORTANT (z=+0.37).
- **Cluster 1 — "Isolated Introspector"** (n=41, 21.1%): Systematically lower on all dimensions (CMWS mean: 2.85, WAMI mean: 5.68). Top distinguishing features: CMWS_INSPIRED (z=−1.45), WAMI_KNOW_WHY (z=−1.41), CMWS_IMPORTANT (z=−1.38).

### 5.2 Clustering B: Personality

KMeans with k=3 identified three Big Five profiles (silhouette = 0.13):

- **Cluster 0 — High-Strained Achiever:** High stress sensitivity, high thoroughness
- **Cluster 1 — Unconstrained Free-Spirit:** High laziness, low thoroughness (low conscientiousness)
- **Cluster 2 — Emotionally Stable Engager:** Handles stress well, outgoing, trusting

k=3 was chosen over k=2 (despite lower silhouette) because it isolated a distinct low-conscientiousness profile meaningful for workplace behavior prediction.

### 5.3 Text Clustering

HDBSCAN with min_cluster_size=7 identified 5 thematic clusters (26.3% outliers, Bootstrap ARI = 0.593). Clusters were labeled via cTF-IDF top terms and representative documents.

### 5.4 Cross-Clustering Comparisons

| Comparison | n | χ² | df | p-value | % Cells Expected < 5 |
|---|---|---|---|---|---|
| Work-Meaning × Personality | 194 | 8.62 | 2 | 0.0134* | < 20% |
| Work-Meaning × Text | 143 | 9.28 | 4 | 0.0544 | 30% |
| Personality × Text | 143 | 19.87 | 8 | 0.0108* | 27% |

**Comparison 1 — Work-Meaning × Personality:** Significant (p = 0.013). Isolated Introspectors are overrepresented in the Unconstrained Free-Spirit personality cluster, suggesting that low conscientiousness may reduce the experience of work-meaning.

**Comparison 2 — Work-Meaning × Text:** Marginally non-significant (p = 0.054). Psychometric work-meaning profiles and narrative text clusters capture largely independent variance — participants with the same survey-based meaning profile can narrate their work meaning in quite different ways.

**Comparison 3 — Personality × Text:** Significant (p = 0.011). High-Strained Achievers are strongly associated with one particular narrative theme. This suggests that personality traits may predict narrative style more strongly than psychometric work-meaning scores.

---

## 6. Discussion

The finding that Work-Meaning × Text falls just short of significance while Personality × Text is significant is counterintuitive at first glance, but interpretable. Personality traits — being stable dispositional characteristics — may systematically influence the topics and domains a person gravitates toward professionally, leading to occupation-specific clustering of both narratives and personality profiles.

By contrast, psychometric work-meaning scores may be higher-order evaluations that any occupational narrative can satisfy through different routes. Put differently: how one narrates meaningful work (what domain, what relationships) may be more predictive of personality than of abstract meaning evaluations.

The significant Work-Meaning × Personality association (p = 0.013) confirms that psychometric meaning scores are not independent of personality. The over-representation of low-meaning workers in the low-conscientiousness cluster is consistent with engagement research linking conscientiousness to meaning appraisal.

---

## 7. Limitations

1. **Sample size (N = 194):** Contingency tables contain many cells with expected counts below 5. Chi-squared results should be treated as exploratory.
2. **HDBSCAN outlier exclusion (~26%):** Excluded participants may systematically differ from included participants, introducing potential selection bias.
3. **NEO short form (10 items):** Limited reliability and precision for personality clustering. The low silhouette (max = 0.13) reflects this limitation.
4. **Single embedding model:** The all-mpnet-base-v2 was selected for benchmark performance, but different models may yield different cluster solutions.
5. **Cross-sectional design:** Meaningful work is a dynamic construct. These clusters represent a single temporal snapshot.

---

## 8. Conclusion

This study applied a dual clustering approach to a corpus of work-meaning survey and narrative data, producing three parallel cluster solutions and three cross-modal comparisons. The key takeaway for organizations: collecting both quantitative survey data and open-ended narratives provides complementary rather than redundant information about how employees experience and articulate the meaning of their work.

---

## References

- Grant, A., & Shin, J. (2012). Work motivation. In *The Oxford Handbook of Human Motivation*, pp. 505–519.
- Grootendorst, M. (2022). BERTopic: Neural topic modeling with a class-based TF-IDF procedure.
- Lips-Wiersma, M., & Wright, S. (2012). Comprehensive Meaningful Work Scale. *Group & Organization Management*, 37(5), 655–685.
- McInnes, L., Healy, J., & Astels, S. (2017). HDBSCAN. *JOSS*, 2(11), 205.
- McInnes, L., Healy, J., & Melville, J. (2018). UMAP. *arXiv:1802.03426*.
- Morrison, M., & Kale, S. (2019). The Work Stories Corpus. *Harvard Dataverse*, V1.
- Reimers, N., & Gurevych, I. (2019). Sentence-BERT. *EMNLP 2019*.
- Steger, M. F., Dik, B. J., & Duffy, R. D. (2012). WAMI. *Journal of Career Assessment*, 20(3), 322–337.
