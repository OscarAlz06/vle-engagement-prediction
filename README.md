# VLE Engagement Prediction

## Project Overview

This project aims to predict whether a video lecture will achieve **high engagement** before publication, using only features available prior to user interaction.

The dataset used is the **VLE Dataset v1 (12k)**, a large-scale dataset built from aggregated consumption data of video lectures from [VideoLectures.net](http://videolectures.net/).

While the original dataset provides continuous engagement metrics (NET, MNET, ANET), this project reformulates the problem as a **binary classification task** to address the *item cold-start problem* in educational recommender systems.

---

## Dataset Overview

The **VLE Dataset v1**:

- Contains **11,568 English video lectures**
- Includes aggregated data from over **1.1 million users**
- Provides engagement metrics based on **Normalized Engagement Time (NET)**
- Is designed to study population-level engagement in educational videos
- Targets applications in recommendation systems and educational quality analysis

The dataset includes multiple feature types: metadata, textual content, Wikipedia-based semantic features, and video-specific variables.

---

## Problem Formulation

The original dataset provides continuous labels such as:

- MNET (Median Normalized Engagement Time)
- ANET (Average Normalized Engagement Time)
- SMNET / SANET
- View count
- Average star rating

In this project:

- A binary target variable `engagement` was created
- Due to the absence of a clear structural breakpoint, a **quantile-based approach (75–25)** was used to define the positive class
- The resulting dataset is **class-imbalanced (75% – 25%)**, which is considered during modeling

This setup simulates a real-world scenario where engagement must be predicted before user interaction occurs.

---

## Feature Groups

The VLE dataset contains four main groups of features. This project uses only those available prior to publication.

### Metadata Features

- `categories` (grouped as *stem* or *misc*)
- `type` (lecture type: lecture, tutorial, invited talk, etc.)

These variables were anonymized and grouped in the original dataset.

---

### Content-Based Features (Transcript)

Extracted from lecture transcripts using linguistic and NLP-based metrics:

- `document_entropy`
- `easiness` (Flesch–Kincaid Easiness)
- `fraction_stopword_presence`
- `normalization_rate`
- `pronoun_rate`

These features capture:

- Topic coverage  
- Readability  
- Presentation style  

---

### Wikipedia-Based Features

Generated using *Entity Linking* between lecture transcripts and Wikipedia concepts.

Two groups:

**1. Topic Authority (PageRank-based)**

- `auth_topic_rank_1_score`
- `auth_topic_rank_2_score`
- `auth_topic_rank_3_score`
- `auth_topic_rank_4_score`
- `auth_topic_rank_5_score`

**2. Topic Coverage (Cosine similarity)**

- `coverage_topic_rank_1_score`
- `coverage_topic_rank_2_score`
- `coverage_topic_rank_3_score`
- `coverage_topic_rank_4_score`
- `coverage_topic_rank_5_score`

These features quantify semantic relevance and depth of content.

---

### Video-Specific Features

- `duration`
- `silent_period_rate`
- `freshness`

These capture structural and temporal characteristics of the lectures.

---

## Exploratory Data Analysis (EDA)

The exploratory analysis included:

- Distribution of variables  
- Detection of outliers  
- Correlation analysis  
- Target variable balance  

Key findings:

- Many variables show **highly skewed distributions**, typical in text-derived metrics  
- Moderate correlations were observed among some Wikipedia-based features  

These insights guided preprocessing and feature selection.

---

## Data Preprocessing

### Scaling

**StandardScaler** was applied to normalize numerical features for scale-sensitive models such as logistic regression.

### Dimensionality Reduction

Three dataset versions were evaluated:

1. **Full dataset**
2. **Reduced dataset** (removing highly correlated variables)
3. **PCA-transformed dataset**

This allowed analysis of how dimensionality impacts model performance.

---

## Model Generalization Analysis

To evaluate generalization, performance was compared between **train and test sets** using **ROC AUC**.

The **AUC gap** was defined as:

AUC gap = AUC (train) - AUC (test)

Results:

- **Logistic Regression**: excellent generalization (gap ≈ 0)
- **Decision Tree**: slight overfitting
- **Random Forest**: moderate overfitting (~0.11 gap)
- **XGBoost**: strong balance between fit and generalization

---

## Model Evaluation

Models were evaluated using:

- **ROC AUC**
- **Recall**
- **Precision**
- **Accuracy**

Additionally, visual tools were used:

- ROC curves  
- Precision–Recall curves  
- Confusion matrices  

Since the goal is to **minimize false negatives**, **Recall** was prioritized.

---

## Decision Threshold Tuning

The default classification threshold (0.5) was adjusted for each model to achieve:

**Recall ≥ 0.8**

This ensures that most high-engagement videos are correctly identified, allowing a moderate increase in false positives.

---

## Final Model Selection

After threshold tuning, the best-performing models were compared.

The selected final model:

**XGBoost trained on the full dataset**

### Results:

| Metric    | Value |
|----------|------|
| Recall   | 0.80 |
| Precision| 0.52 |
| Accuracy | 0.75 |
| AUC      | 0.86 |

This model provides the **best balance between detecting high-engagement videos and controlling false positives**.

---

## Conclusions

The results show that it is possible to **predict the engagement potential of a video lecture before publication** using only content and metadata features.

Key insights:

- Linear models show better generalization  
- Tree-based models capture more complex relationships  
- **XGBoost achieves the best overall performance**

This type of model can be integrated into **educational recommender systems**, helping prioritize content with higher engagement potential.
