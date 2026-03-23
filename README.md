# Dual Clustering Analysis of Meaningful Work

**Comparing Psychometric and Narrative Measurement Modalities**

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3+-orange.svg)](https://scikit-learn.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> Do people who score similarly on work-meaning surveys also *narrate* their work in similar ways? This project applies unsupervised ML to find out.

---

## Overview

Understanding what makes work meaningful is central to organizational psychology and HR analytics. But meaningful work is multidimensional — it emerges from the interplay of survey-based metrics and how people actually talk about their jobs.

This project runs **three parallel clustering analyses** on 194 workers and compares them cross-modally to test whether quantitative and qualitative measurement capture the same underlying structure.

### Research Questions

1. **RQ1 (Primary):** Do participants who cluster similarly on work-meaning survey items (CMWS + WAMI) also produce semantically similar narratives?
2. **RQ2 (Secondary):** Does personality-based clustering (NEO Big Five) show any association with narrative or work-meaning clustering?

---

## Methods

| Clustering Track | Features | Algorithm | Key Hyperparameters |
|---|---|---|---|
| **A: Work-Meaning** | 38 items (CMWS + WAMI) | KMeans | k=2, n_init=10 |
| **B: Personality** | 10 items (NEO-FFI) | KMeans | k=3, n_init=10 |
| **Text (Narrative)** | Open-ended responses | Sentence Embedding → UMAP → HDBSCAN | all-mpnet-base-v2, n_components=10, min_cluster_size=7 |

**Cross-modal comparison:** Chi-squared tests of independence on all three pairwise contingency tables.

### Pipeline Architecture

```
Raw Survey Data (n=194, 75 features)
│
├── Numerical Features ──► StandardScaler ──► KMeans
│   ├── Clustering A: 38 work-meaning items → 2 clusters
│   └── Clustering B: 10 personality items  → 3 clusters
│
├── Text Responses ──► SentenceTransformer (768D)
│   └── UMAP (10D) ──► HDBSCAN → 5 clusters + outliers
│
└── Cross-Modal Comparison ──► Chi-squared tests (3 pairs)
```

---

## Key Findings

**Clustering A** identified two distinct work-meaning profiles:
- **Cluster 0 — "Self-Estranged Contribution"** (n=153, 78.9%): Higher scores across all CMWS and WAMI dimensions
- **Cluster 1 — "Isolated Introspector"** (n=41, 21.1%): Systematically lower engagement and purpose scores

**Clustering B** revealed three personality profiles:
- High-Strained Achiever, Unconstrained Free-Spirit, and Emotionally Stable Engager

**Cross-modal results** showed a nuanced pattern:

| Comparison | χ² | p-value | Interpretation |
|---|---|---|---|
| Work-Meaning × Personality | 8.62 | 0.013* | Meaning scores are not independent of personality |
| Work-Meaning × Text | 9.28 | 0.054 | Survey and narrative clusters capture largely independent variance |
| Personality × Text | 19.87 | 0.011* | Personality predicts narrative style more strongly than survey scores do |

The key insight: **how one narrates meaningful work is more predictive of personality than of abstract meaning appraisals.** Survey data and narratives offer complementary — not redundant — windows into meaningful work.

---

## Project Structure

```
meaningful-work-clustering/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── work-stories-corpus-clean.csv     # 194 participants, 75 features
├── notebooks/
│   └── dual_clustering_analysis.ipynb    # Full analysis pipeline
└── reports/
    └── project_report.md                 # Detailed methodology & findings
```

---

## Quick Start

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/meaningful-work-clustering.git
cd meaningful-work-clustering

# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run the notebook
jupyter notebook notebooks/dual_clustering_analysis.ipynb
```

---

## Tech Stack

- **NLP:** Sentence-Transformers (all-mpnet-base-v2), BERTopic (cTF-IDF)
- **Dimensionality Reduction:** UMAP
- **Clustering:** KMeans, HDBSCAN
- **Evaluation:** Silhouette analysis, Bootstrap ARI, Chi-squared tests
- **Visualization:** Matplotlib, Seaborn

---

## Dataset

The [Work Stories Corpus](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.5683/SP2/MIOVSG) (Morrison & Kale, 2019) from Harvard Dataverse contains open-ended narratives and validated psychometric scales from 196 full-time workers. After listwise deletion of 2 rows with missing values, the analysis uses n=194 participants.

**Scales included:** CMWS (28 items), WAMI (10 items), NEO-FFI (10 items), Affective Commitment (6 items), Portrait Values Questionnaire (10 items), plus demographic metadata and free-text narratives.

---

## Model Selection Rationale

- **KMeans k selection:** Silhouette coefficient + elbow method + theoretical interpretability. For Clustering B, k=3 was chosen over k=2 (despite lower silhouette) because it isolated a distinct low-conscientiousness profile meaningful for workplace behavior prediction.
- **UMAP n_components:** Grid search over {5, 10, 15, 20, 30, 50} evaluated by downstream HDBSCAN silhouette. n_components=10 selected (silhouette=0.537).
- **HDBSCAN min_cluster_size:** Sweep over {5–20} evaluated by Bootstrap ARI (5 resamples at 80%). min_cluster_size=7 selected (ARI=0.593, highest stability).

---

## Limitations

- **Small sample** (N=194): Some contingency table cells fall below the expected count threshold of 5, making chi-squared results exploratory rather than confirmatory.
- **HDBSCAN outlier exclusion** (~26%): Excluded participants may systematically differ from those retained.
- **Single embedding model:** Results may vary with different sentence transformers.
- **Cross-sectional design:** Meaningful work is dynamic; these clusters represent a single snapshot.

---

## References

- Lips-Wiersma, M., & Wright, S. (2012). Comprehensive Meaningful Work Scale. *Group & Organization Management*, 37(5), 655–685.
- Steger, M. F., Dik, B. J., & Duffy, R. D. (2012). Work and Meaning Inventory. *Journal of Career Assessment*, 20(3), 322–337.
- Morrison, M., & Kale, S. (2019). The Work Stories Corpus. *Harvard Dataverse*, V1.
- Reimers, N., & Gurevych, I. (2019). Sentence-BERT. *EMNLP 2019*.
- McInnes, L., Healy, J., & Astels, S. (2017). HDBSCAN. *JOSS*, 2(11), 205.
- McInnes, L., Healy, J., & Melville, J. (2018). UMAP. *arXiv:1802.03426*.

---

## Authors

Jung Shan Liang and Man-Chen Chen

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.
