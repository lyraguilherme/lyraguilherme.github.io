---
title: 'AWS AI Practitioner - Study Notes #1'
date: 2026-01-25T15:00:00-03:00
draft: false
tags:
 - AI Practitioner
 - AWS
showTableOfContents: true
featureAlt: "AWS AI Practitioner - Study Notes #1"
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

## 8. Transfer Learning
Transfer Learning is a technique where a model trained on one task is **reused as the starting point** for a different but related task — instead of training from scratch.

* **How it works:** A model pre-trained on a massive general dataset (e.g., millions of images or billions of words) already understands low-level patterns like edges, shapes, or grammar. You take this pre-trained model and adapt it to your specific problem using a much smaller dataset.
* **Why it matters:** Training a model from scratch requires enormous amounts of data and compute. Transfer Learning lets you achieve strong results with a fraction of both.
* **Connection to Foundation Models:** This is the exact principle behind Foundation Models — a large base model is pre-trained once, then adapted (via fine-tuning or prompt engineering) for many downstream tasks.
* **Example:** Instead of training an image classifier from scratch with millions of labeled photos, you start with a model pre-trained on ImageNet and fine-tune it with just a few hundred images of your specific product — getting high accuracy much faster.

**Key Concept:** Transfer Learning is what makes modern AI practical. Without it, every new task would require training from zero — making most projects too expensive and slow.

## 9. Training, Evaluation & Deployment
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

## 10. MLOps (Machine Learning Operations)
Deployment is not the end. MLOps applies **DevOps** principles to ML systems to ensure reliability and scalability.
* **Continuous Integration/Continuous Delivery (CI/CD):** Automating the training and deployment pipelines.
* **Monitoring:** Watching for **Model Drift** (model quality degrades over time) or **Data Drift** (real-world data distribution changes compared to training data).
* **Retraining:** Automatically triggering new training runs when performance drops below a threshold.
* **AWS Tool:** **Amazon SageMaker Model Monitor** (Automatically detects data drift and model quality issues in production).

## 11. No-Code ML with Amazon SageMaker Canvas
Not every ML project requires writing code. **Amazon SageMaker Canvas** provides a **visual, no-code interface** that enables business analysts and non-technical users to build ML models.

* **How it works:** You import your data (from S3, Redshift, or local files), select a target column, and Canvas automatically builds, trains, and evaluates multiple models — then picks the best one.
* **Supported Tasks:** Classification, regression, time-series forecasting, and natural language processing — all without writing a single line of code.
* **When to use it:** Business teams need to generate predictions or forecasts but don't have ML engineering resources.

**Key Concept:** SageMaker Canvas is the **no-code counterpart to SageMaker Studio**. If the exam describes a business analyst who needs to build a model without coding, Canvas is the answer.

## 12. Reinforcement Learning from Human Feedback (RLHF)
RLHF is the training technique that bridges the gap between a raw pre-trained language model and the helpful, safe assistant that end users interact with.

* **The Problem:** A language model trained only to predict the next word can produce outputs that are technically fluent but unhelpful, harmful, or misaligned with human expectations.
* **How it works:**
    1. **Supervised Fine-Tuning (SFT):** The model is first fine-tuned on examples of high-quality, human-written responses.
    2. **Reward Model Training:** Human evaluators rank multiple model outputs for the same prompt. These rankings are used to train a separate **reward model** that scores how "good" a response is.
    3. **Reinforcement Learning:** The language model is further trained using the reward model as a guide — it learns to generate responses that score higher on the reward model's criteria (helpfulness, harmlessness, honesty).
* **Why it matters:** RLHF is how models like Claude are aligned to follow instructions, refuse harmful requests, and produce useful responses — not just statistically likely text.

**Key Concept:** RLHF is the standard technique for **aligning** Foundation Models with human values and expectations. The exam may ask about it in the context of how LLMs are made safe and useful after pre-training.

---