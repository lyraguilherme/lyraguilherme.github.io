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

# The Machine Learning Lifecycle

## 1. Business Goal Identification
Before touching data or algorithms, you must define the **value** of the project.
* **Define Success:** Establish clear, measurable outcomes (e.g., "Reduce customer churn by 5%" or "Detect 90% of fraudulent transactions").
* **Is ML Necessary?:** Not every problem needs AI. If a simple rule-based system works, use it. ML is chosen when patterns are too complex for hard-coded rules.

## 2. ML Problem Framing
Translating the business goal into a specific Machine Learning task.
* **Target Variable:** What specifically are we predicting? (e.g., "Will this user click?" or "What will the house price be?").
* **Learning Type:**
    * **Supervised:** using labeled historical data (e.g., Spam vs. Not Spam).
    * **Unsupervised:** Finding hidden structures in unlabelled data (e.g., Customer Segmentation).
* **Task Type:** Classification (categories), Regression (numbers), or Clustering (groups).

## 3. The "80/20" Rule of ML
This implies that roughly 80% of an AI project is spent on **Data Preparation** and only 20% on actual model training.
* **GIGO (Garbage In, Garbage Out):** The quality of your model is strictly limited by the quality of your data.
* **Sequential Process:** Data must be collected, cleaned, and engineered *before* effective learning can occur.

## 4. Data Collection & Integration
Gathering raw data from scattered sources into a centralized **Data Lake** or "source of truth".
* **The Process:** Fetching logs, database records, or real-time streams (often referred to as **ETL**: Extract, Transform, Load).

**AWS Services to Remember:**
* **Amazon S3:** Go-to storage for unstructured data (photos, raw logs).
* **Amazon Kinesis:** Used for ingesting real-time, high-speed data streams.
* **AWS Glue:** A serverless service for crawling, discovering, and cataloging data.

## 5. Data Preprocessing
Data Preprocessing is the process of converting "messy" raw data into a clean, standardized format that algorithms can process efficiently.

**Common Tasks:**
* **Cleaning:** Removing duplicate entries and fixing structural errors.
* **Imputation:** Handling missing values by filling gaps (e.g., using the mean/median) or dropping incomplete rows.
* **Scaling/Normalization:** Adjusting numerical values to a common range (e.g., 0 to 1) so large numbers don't disproportionately influence the model.
* **AWS Tool:** **AWS Glue DataBrew** (A visual, no-code tool for cleaning and normalizing data with 250+ built-in transformations).

## 6. Feature Engineering
Feature Engineering is the discipline of selecting, creating, or modifying variables (**features**) to maximize the model's predictive performance.

**Key Techniques:**
* **Selection:** Keeping relevant columns (e.g., "Price") but dropping useless ones (e.g., "Internal ID").
* **Creation:** Combining existing data to create new signals (e.g., turning "Date" and "Time" into "Is_Holiday").
* **Transformation/Encoding:** Converting text categories (e.g., "High", "Low") into numerical values via techniques like **One-Hot Encoding**.
* **AWS Tool:** **Amazon SageMaker Data Wrangler** (The primary tool for preparing features specifically for ML within SageMaker).

## 7. Data Visualization
Representing data graphically to identify patterns, trends, and anomalies before training.
* **EDA (Exploratory Data Analysis):** The practice of spotting errors, biases, or data imbalances early in the lifecycle.
* **Correlation:** Identifying how variables relate to one another (e.g., "Temperature" vs. "Ice Cream Sales").
* **AWS Tool:** **Amazon QuickSight** (The primary cloud-native BI service used for creating dashboards and visualizing ML insights).

## 8. Training, Evaluation & Deployment
* **Model Training:** The algorithm analyzes the prepared data to find mathematical patterns and weights.
* **Evaluation:** Testing the model against a "Validation Dataset" to ensure it hasn't **Overfit** (memorized the data instead of learning).
* **Deployment:** Hosting the model on an endpoint (Inference) so applications can make real-world predictions.
* **AWS Tool:** **Amazon SageMaker** (The comprehensive, end-to-end platform for the entire ML lifecycle).

## 9. MLOps (Machine Learning Operations)
Deployment is not the end. MLOps applies **DevOps** principles to ML systems to ensure reliability and scalability.
* **Continuous Integration/Continuous Delivery (CI/CD):** Automating the training and deployment pipelines.
* **Monitoring:** Watching for **Model Drift** (quality degrades over time) or **Data Drift** (real-world data changes compared to training data).
* **Retraining:** Automatically triggering new training runs when performance drops below a threshold.


