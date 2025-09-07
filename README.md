# Source DB Prediction from Genomic Transcript Data

## Project Overview
This project performs an end-to-end machine learning workflow on a raw genomic transcript dataset from the NCBI database. The goal is to predict the `source_db` (the source database of a transcript, like 'est' or 'mrna') based on features like transcript length (`bases`) and the number of transcripts per gene. The project addresses significant challenges including data leakage, severe class imbalance, and the precision-recall trade-off.

---

## Dataset
* **Source:** NCBI Genomes - Canis lupus familiaris (BUILD.3.1)
* **Description:** The dataset contains information about various RNA transcripts, including their accession numbers, gene IDs, lengths, and source databases. The initial dataset consisted of over 326,000 rows.
* **Link:** https://ftp.ncbi.nlm.nih.gov/genomes/MapView/Canis_lupus_familiaris/objects/BUILD.3.1/initial_release/org_transcript.q.gz

---

## Methodology & Workflow

1.  **Data Cleaning & EDA:**
    * Loaded the raw data and performed an initial assessment of column types and null values.
    * Dropped irrelevant or mostly-empty columns (`strain`, `is_RefSeq`, etc.).
    * Discovered and analyzed a data leakage issue between the `is_EST` and `source_db` columns, leading to a pivot in the project goal.

2.  **Feature Engineering:**
    * Applied a **log1p transformation** to the heavily skewed `bases` column to normalize its distribution, creating `bases_log`.
    * Engineered a new feature, `transcripts`, representing the number of unique transcripts associated with each gene (`unigene_id`). This was also log-transformed.

3.  **Modeling & Evaluation:**
    * **Baseline Model:** Started with a simple Logistic Regression model using undersampling to handle class imbalance, which resulted in underfitting.
    * **Improved Baseline:** Implemented Logistic Regression with the `class_weight='balanced'` parameter, which yielded a much better model for finding the rare class (`recall=0.80` for 'mrna').
    * **Advanced Model:** Transitioned to a more powerful `XGBoost Classifier`.
    * **Hyperparameter Tuning:** Used `GridSearchCV` to find the optimal hyperparameters for the XGBoost model, focusing on `max_depth`, `n_estimators`, and `learning_rate`.
    * **Threshold Tuning:** Explored the precision-recall trade-off by manually adjusting the prediction threshold to create a high-precision model and a high-recall model, in addition to the F1-optimized model from `GridSearchCV`.

---

## Results & Key Findings

The final tuned XGBoost model achieved an overall accuracy of **99.51%**. More importantly, by tuning the prediction threshold, we could optimize for different business needs:

* **High-Recall Model (Threshold = 0.71):** Excellent for scenarios where finding every potential 'mrna' case is critical, even at the cost of some false positives.
    * Recall for 'mrna' : **0.74**
    <img width="450" height="450" alt="image" src="https://github.com/user-attachments/assets/60d554f0-54eb-4809-8445-d4a4a561cb17" />
* **High-Precision Model (Threshold = 0.99):** Ideal for cases where you need to be very confident in your 'mrna' predictions, minimizing false alarms.
    * Precision for 'mrna' : **0.77**
    <img width="450" height="450" alt="image" src="https://github.com/user-attachments/assets/c66d7c98-797d-4772-b5b8-1e27a1c63609" />
* **F1-Optimized Model (Threshold tuned):** Ideal for cases where you need a model good enough for 'mrna' predictions & recall, maximizing overall performance.
    * Precision for 'mrna' : **0.81**
    * F1 Score for 'mrna' : **0.59**
    <img width="674" height="701" alt="2cc41bc9-1a72-49b7-ae37-1731b9175ac5" src="https://github.com/user-attachments/assets/959ccb6e-7ac9-4d0a-8103-585b1479c83c" />
---

## Conclusion
This project successfully demonstrates a robust workflow for handling a real-world, imbalanced dataset. By engineering meaningful features and tuning an advanced model like XGBoost, it's possible to build a highly effective classifier that can be optimized for either precision or recall depending on the specific use case.
