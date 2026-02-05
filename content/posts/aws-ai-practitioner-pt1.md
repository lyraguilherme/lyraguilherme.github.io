---
title: 'AWS AI Practitioner - Study Notes #1'
date: 2026-01-25T15:00:00-03:00
draft: false
ShowToC: true
TocOpen: true
cover:
    image: /aws-ai-practitioner/cover.png
    alt: 'AWS AI Practitioner - Study Notes #1'
    caption: 'AWS AI Practitioner - Study Notes #1'
---

These are my personal study notes for the **AWS Certified AI Practitioner (AIF-C01)** exam. In this first part, we cover the **Machine Learning Workflow** — from data preparation to deployment and MLOps. The goal is to understand how each stage of the pipeline connects and which AWS services are relevant at each step.

# The Machine Learning Workflow

## 1. Business Goal Identification
Before touching data or algorithms, you must define the **value** of the project.
* **Define Success:** Establish clear, measurable outcomes (e.g., "Reduce customer churn by 5%" or "Detect 90% of fraudulent transactions").
* **Is ML Necessary?:** Not every problem needs AI. If a simple rule-based system works, use it. ML is chosen when patterns are too complex for hard-coded rules.

## 2. ML Problem Framing
Translating the business goal into a specific Machine Learning task.
* **Target Variable:** What specifically are we predicting? (e.g., "Will this user click?" or "What will the house price be?").
* **Learning Type:**
    * **Supervised:** Using labeled historical data (e.g., Spam vs. Not Spam).
    * **Unsupervised:** Finding hidden structures in unlabeled data (e.g., Customer Segmentation).
    * **Reinforcement Learning:** An agent learns by interacting with an environment and receiving rewards or penalties (e.g., AWS DeepRacer, game-playing AI).
    * **Semi-Supervised:** A mix of a small amount of labeled data and a large amount of unlabeled data.
* **Task Type:** Classification (categories), Regression (numbers), or Clustering (groups).

## 3. The "80/20" Rule of ML
This is a key concept that states that 80% of an AI project is spent on **Data Preparation** and only 20% is spent on actual model training.
* **GIGO (Garbage In, Garbage Out):** The quality of your model is strictly limited by the quality of your data.
* **Sequential Process:** Data must be collected, cleaned, and engineered *before* the model can begin the learning phase.

## 4. Data Collection & Integration
Gathering raw data from scattered sources into a centralized repository, often a **Data Lake**.
* **The Process:** Fetching logs, database records, or real-time streams (often referred to as **ETL**: Extract, Transform, Load).

**AWS Services to Remember:**
* **Amazon S3:** Go-to storage for unstructured data (photos, raw logs).
* **Amazon Kinesis:** Used for ingesting real-time, high-speed data streams.
* **AWS Glue:** A serverless service for crawling, discovering, and cataloging data from various sources.

## 5. Data Preprocessing
Data Preprocessing is the process of converting "messy" raw data into a clean, standardized format that algorithms can process efficiently.

**Common Tasks:**
* **Cleaning:** Removing duplicate entries and fixing structural errors.
* **Imputation:** Handling missing values by filling gaps (e.g., using the mean/median) or dropping incomplete rows.
* **Scaling/Normalization:** Adjusting numerical values to a common range (e.g., 0 to 1) so large numbers don't disproportionately influence the model.
* **AWS Tool:** **AWS Glue DataBrew** (A visual, no-code tool for cleaning and normalizing data with 250+ built-in transformations).

## 6. Feature Engineering
Feature Engineering is the "art" of selecting, creating, or modifying variables (**features**) to maximize the model's predictive performance.

**Key Techniques:**
* **Selection:** Keeping relevant columns (e.g., "Price") but dropping useless ones (e.g., "Internal ID").
* **Creation:** Combining existing data to make new insights (e.g., turning "Date" and "Time" into "Is_Holiday").
* **Transformation/Encoding:** Converting text categories (e.g., "High", "Low") into numerical values (e.g., 1, 0) via techniques like **One-Hot Encoding**.
* **AWS Tool:** **Amazon SageMaker Data Wrangler** (The primary tool for preparing features specifically for ML within SageMaker).

## 7. Data Visualization
Representing data graphically to identify patterns, trends, and anomalies before training.
* **EDA (Exploratory Data Analysis):** The practice of spotting errors, biases, or data imbalances early in the lifecycle.
* **Correlation:** Identifying how variables relate to one another (e.g., "Temperature" vs. "Ice Cream Sales").
* **AWS Tool:** **Amazon QuickSight** (The primary cloud-native BI service used for creating dashboards and visualizing ML insights).

## 8. Training, Evaluation & Deployment
Once data is prepped, the model-building phase begins.

### Data Splitting
Before training, data is split into separate sets to measure real performance:
* **Training Set (~70-80%):** The data the model learns from.
* **Validation Set (~10-15%):** Used during training to tune **hyperparameters** and detect overfitting early.
* **Test Set (~10-15%):** Held out completely and used only at the end for a final, unbiased performance check.

### Model Training
* The algorithm analyzes the training data to find mathematical patterns and weights.
* **Hyperparameter Tuning:** Adjusting settings that control *how* the model learns (e.g., learning rate, number of layers) to find the best configuration.
* **AWS Tool:** **Amazon SageMaker Automatic Model Tuning** (Runs multiple training jobs to find the optimal hyperparameters).

### Evaluation
Testing the trained model to measure how well it generalizes to new data.

**Key Concepts:**
* **Overfitting:** The model learned noise and specifics of the training data rather than generalizable patterns, resulting in high training accuracy but poor performance on new data.
* **Underfitting:** The model is too simple to capture the underlying patterns, resulting in poor performance on both training and new data.
* **Bias-Variance Tradeoff:** High bias leads to underfitting; high variance leads to overfitting. The goal is to find the right balance.

**Common Metrics:**
* **Accuracy:** Overall percentage of correct predictions.
* **Precision:** Of all positive predictions, how many were actually correct? (Important when false positives are costly).
* **Recall (Sensitivity):** Of all actual positives, how many did the model catch? (Important when false negatives are costly).
* **F1-Score:** The harmonic mean of Precision and Recall, useful when you need a balance between both.
* **AUC-ROC:** Measures the model's ability to distinguish between classes across all thresholds.
* **RMSE (Root Mean Squared Error):** Common metric for regression tasks, measuring the average prediction error.

### Deployment
* Hosting the model on an endpoint (**Inference**) so applications can make real-world predictions.
* **AWS Tool:** **Amazon SageMaker** (The comprehensive, end-to-end platform for the entire ML lifecycle).

## 9. MLOps (Machine Learning Operations)
Deployment is not the end. MLOps applies **DevOps** principles to ML systems to ensure reliability and scalability.
* **Continuous Integration/Continuous Delivery (CI/CD):** Automating the training and deployment pipelines.
* **Monitoring:** Watching for **Model Drift** (model quality degrades over time) or **Data Drift** (real-world data distribution changes compared to training data).
* **Retraining:** Automatically triggering new training runs when performance drops below a threshold.
* **AWS Tool:** **Amazon SageMaker Model Monitor** (Automatically detects data drift and model quality issues in production).

---