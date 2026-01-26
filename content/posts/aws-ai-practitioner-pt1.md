---
title: 'AWS AI Practitioner - Study Notes #1'
date: 2026-01-25T15:00:00-03:00
draft: false
ShowToC: true
TocOpen: true
cover:
    image: /aws-ai-practitioner/cover.jpg
    alt: 'AWS AI Practitioner - Study Notes #1'
    caption: 'AWS AI Practitioner - Study Notes #1'
---

# The Machine Learning Workflow

## 1. The "80/20" Rule of ML
This is a key concept that states that 80% of an AI project is spent on **Data Preparation** and only 20% is spent on actual model training.
* **GIGO (Garbage In, Garbage Out):** The quality of your model is strictly limited by the quality of your data.
* **Sequential Process:** Data must be collected, cleaned, and engineered *before* the model can begin the learning phase.

## 2. Data Collection & Integration
Gathering raw data from scattered sources into a centralized **Data Lake** or "source of truth".
* **The Process:** Fetching logs, database records, or real-time streams (often referred to as **ETL**: Extract, Transform, Load).

**AWS Services to Remember:**
* **Amazon S3:** Go-to storage for unstructured data (photos, raw logs).
* **Amazon Kinesis:** Used for ingesting real-time, high-speed data streams.
* **AWS Glue:** A serverless service for crawling, discovering, and cataloging data from various sources.

## 3. Data Preprocessing
Data Preprocessing is the of converting "messy" raw data into a clean, standardized format that algorithms can process efficiently.

**Common Tasks:**
* **Cleaning:** Removing duplicate entries and fixing structural errors.
* **Imputation:** Handling missing values by filling gaps (e.g., using the mean/median) or dropping incomplete rows.
* **Scaling/Normalization:** Adjusting numerical values to a common range (e.g., 0 to 1) so large numbers don't disproportionately influence the model.
* **AWS Tool:** **AWS Glue DataBrew** (A visual, no-code tool for cleaning and normalizing data with 250+ built-in transformations).

## 4. Feature Engineering
Feature Engineering is the "art" of selecting, creating, or modifying variables (**features**) to maximize the model's predictive performance.

**Key Techniques:**
* **Selection:** Keeping relevant columns (e.g., "Price") but dropping useless ones (e.g., "Internal ID").
* **Creation:** Combining existing data to make new insights (e.g., turning "Date" and "Time" into "Is_Holiday").
* **Transformation/Encoding:** Converting text categories (e.g., "High", "Low") into numerical values (e.g., 1, 0) via techniques like **One-Hot Encoding**.
* **AWS Tool:** **Amazon SageMaker Data Wrangler** (The primary tool for preparing features specifically for ML within SageMaker).

## 5. Data Visualization
Representing data graphically to identify patterns, trends, and anomalies before training.
* **EDA (Exploratory Data Analysis):** The practice of spotting errors, biases, or data imbalances early in the lifecycle.
* **Correlation:** Identifying how variables relate to one another (e.g., "Temperature" vs. "Ice Cream Sales").
* **AWS Tool:** **Amazon QuickSight** (The primary cloud-native BI service used for creating dashboards and visualizing ML insights).

## 6. Training, Evaluation & Deployment
* **The "After" Stages:** Once data is prepped, the model-building phase begins.
* **Model Training:** The algorithm analyzes the prepared data to find mathematical patterns and weights.
* **Evaluation:** Testing the model against a "Validation Dataset" to ensure it hasn't **Overfit** (memorized the data instead of learning).
* **Deployment:** Hosting the model on an endpoint so applications can make real-world predictions.
* **AWS Tool:** **Amazon SageMaker** (The comprehensive, end-to-end platform for the entire ML lifecycle).


