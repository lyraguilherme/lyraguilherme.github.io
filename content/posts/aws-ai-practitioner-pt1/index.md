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

### Data Types
Understanding your data type determines which services, storage, and algorithms are appropriate.
* **Structured Data:** Organized in rows and columns with a clear schema — like spreadsheets, relational databases, or CSV files. Each field has a defined type and meaning. Examples: sales records, customer databases, sensor readings.
* **Unstructured Data:** No predefined format or schema — raw content that doesn't fit neatly into tables. Examples: images, videos, audio files, emails, social media posts, PDF documents.
* **Semi-Structured Data:** Has some organizational structure (like tags or keys) but doesn't follow a strict tabular schema. Examples: JSON, XML, log files.

**Key Concept:** Most real-world AI projects deal with a mix of all three. Structured data works well with traditional ML algorithms (like XGBoost or Linear Regression). Unstructured data typically requires Deep Learning (like CNNs for images or Transformers for text).

### Problem Definition
* **Target Variable:** What specifically are we predicting? (e.g., "Will this user click?" or "What will the house price be?").
* **Learning Type:**
    * **Supervised:** Using labeled historical data (e.g., Spam vs. Not Spam).
    * **Unsupervised:** Finding hidden structures in unlabeled data (e.g., Customer Segmentation).
    * **Reinforcement Learning:** An agent learns by interacting with an environment and receiving rewards or penalties (e.g., AWS DeepRacer, game-playing AI).
    * **Semi-Supervised:** A mix of a small amount of labeled data and a large amount of unlabeled data.
* **Task Type:**
    * **Classification:** Predicting a category (e.g., Spam vs. Not Spam, cat vs. dog).
    * **Regression:** Predicting a continuous number (e.g., house price, temperature tomorrow).
    * **Clustering:** Discovering natural groupings in unlabeled data (e.g., customer segments).
    * **Anomaly Detection:** Identifying data points that deviate significantly from normal patterns. Unlike classification, you typically have very few (or zero) examples of the "abnormal" class — the model learns what "normal" looks like and flags anything that doesn't fit.
        * Use case: A network security system that monitors server traffic patterns. The model learns the baseline of normal traffic and flags sudden spikes, unusual access patterns, or connections from unexpected locations as potential intrusions.
        * Use case: Manufacturing quality control. A model trained on images of non-defective products flags any product that looks different — scratches, dents, or misalignments — without needing examples of every possible defect type.
        * **AWS Tool:** **Amazon Lookout for Metrics** (Automatically detects anomalies in business metrics like revenue drops, traffic spikes, or conversion rate changes) and **Amazon Lookout for Equipment** (Detects abnormal equipment behavior using sensor data to enable predictive maintenance).

## 3. Common ML Algorithms
Now that you know the task types, the next step is understanding which algorithms solve each one. You don't need to know the math — but you do need to know **which algorithm to pick for a given problem**.

### Classification Algorithms
These algorithms predict **which category** something belongs to.

* **Logistic Regression:** Despite the name, this is a classification algorithm — not regression. It predicts the probability that an input belongs to one of two categories (binary classification).
    * Use case: Predicting whether a customer will churn (yes/no). The model outputs a probability (e.g., 78% likely to churn), and you set a threshold to make the final decision.
    * When to use it: Simple binary classification problems with a clear relationship between features and outcome. It's fast, interpretable, and a good baseline before trying more complex models.

* **Naive Bayes:** A probabilistic algorithm based on Bayes' theorem. It assumes that all features are independent of each other (the "naive" part) — which is rarely true in practice but still works surprisingly well for text-based tasks.
    * Use case: Email spam detection. The model learns that words like "prize," "winner," and "click here" appear far more often in spam than in legitimate emails, and uses those probabilities to classify new messages.
    * When to use it: Text classification tasks where speed matters and the dataset is small. It's one of the fastest algorithms to train.

* **Support Vector Machine (SVM):** Finds the best boundary (called a hyperplane) that separates data into categories with the **maximum margin** — the widest possible gap between the closest points of each class.
    * Use case: Image classification — distinguishing between handwritten digits (0-9). SVM finds boundaries in the feature space that separate each digit class as clearly as possible.
    * When to use it: Small to medium datasets with clear separation between classes. It handles high-dimensional data well (e.g., many features) but gets slow on very large datasets.

* **K-Nearest Neighbors (KNN):** Classifies a new data point based on the **majority vote of its K closest neighbors** in the training data. No model is actually "trained" — it just memorizes the data and compares at prediction time.
    * Use case: A movie recommendation system. To predict whether you'll like a new movie, KNN looks at the K users most similar to you and checks what they rated it. If most of them liked it, the model predicts you will too.
    * When to use it: Small datasets where the relationship between data points is spatial or distance-based. It's simple and intuitive but becomes very slow on large datasets because it has to compare every new point against all stored data.

### Regression Algorithms
These algorithms predict a **continuous numerical value**.

* **Linear Regression:** The simplest regression algorithm. It fits a straight line through the data to model the relationship between input features and a numeric output.
    * Use case: Predicting house prices based on square footage. The model learns that each additional square foot adds roughly $X to the price — a simple linear relationship.
    * When to use it: When the relationship between features and the target is roughly linear. It's fast, easy to interpret, and serves as a strong baseline for any regression task.

* **Decision Tree:** Splits data into branches based on a series of if/then questions about the features, forming a tree-like structure. Each leaf node gives a prediction. Can be used for both classification and regression.
    * Use case: Estimating car insurance premiums. The tree might first split on driver age (over/under 25), then on accident history (yes/no), then on vehicle type — each path through the tree ending at a predicted premium amount.
    * When to use it: When you need a model that's easy to visualize and explain to non-technical stakeholders. Decision trees are inherently interpretable — you can trace exactly why a prediction was made.

* **Random Forest:** An **ensemble** method that trains many decision trees on random subsets of the data and averages their predictions. This reduces the overfitting problem that individual decision trees suffer from.
    * Use case: Predicting customer lifetime value. Each tree in the forest sees a slightly different view of the data, and the final prediction is the average of all trees — producing a more stable and accurate estimate than any single tree.
    * When to use it: When you want better accuracy than a single decision tree without much extra tuning. Random Forest is one of the most reliable "general-purpose" algorithms in traditional ML.

* **XGBoost (Extreme Gradient Boosting):** Another ensemble method, but instead of training trees independently (like Random Forest), it trains them **sequentially** — each new tree focuses specifically on correcting the errors of the previous ones.
    * Use case: Fraud detection in financial transactions. XGBoost excels here because fraudulent transactions are rare (imbalanced data), and the sequential correction mechanism helps the model learn from its mistakes on these hard-to-catch cases.
    * When to use it: When you need maximum predictive accuracy and are willing to spend more time tuning. XGBoost dominates ML competitions and is often the go-to for structured/tabular data problems.

### Clustering Algorithms
These algorithms find **natural groupings** in unlabeled data (unsupervised learning).

* **K-Means Clustering:** Partitions data into **K groups** by iteratively assigning each data point to the nearest cluster center, then recalculating the centers. You must specify the number of clusters (K) in advance.
    * Use case: Customer segmentation for a retail company. K-Means might discover that your customers naturally fall into 4 groups — budget shoppers, seasonal buyers, premium loyal customers, and one-time visitors — allowing you to tailor marketing strategies for each segment.
    * When to use it: When you need to discover natural groupings in your data and have a rough idea of how many groups to expect. It's fast and scales well to large datasets.

### Choosing the Right Algorithm

| Task Type | Start With | Upgrade To |
|---|---|---|
| Binary Classification (yes/no) | Logistic Regression | Random Forest, XGBoost |
| Text Classification | Naive Bayes | SVM |
| Regression (predict a number) | Linear Regression | Random Forest, XGBoost |
| Grouping unlabeled data | K-Means | — |
| Need explainability | Decision Tree | Random Forest (less explainable but more accurate) |

**Key Concept:** The exam won't ask you to do math — but it will describe a scenario and expect you to pick the right algorithm. Start with the task type (classification, regression, or clustering), then match it to the algorithm that fits the constraints (accuracy vs. interpretability, dataset size, structured vs. text data).

## 4. The "80/20" Rule of ML
This is a key concept that states that 80% of an AI project is spent on **Data Preparation** and only 20% is spent on actual model training.
* **GIGO (Garbage In, Garbage Out):** The quality of your model is strictly limited by the quality of your data.
* **Sequential Process:** Data must be collected, cleaned, and engineered *before* the model can begin the learning phase.

## 5. Data Collection & Integration
Gathering raw data from scattered sources into a centralized repository, often a **Data Lake**.
* **The Process:** Fetching logs, database records, or real-time streams (often referred to as **ETL**: Extract, Transform, Load).

**AWS Services to Remember:**
* **Amazon S3:** Go-to storage for unstructured data (photos, raw logs).
* **Amazon Kinesis:** Used for ingesting real-time, high-speed data streams.
* **AWS Glue:** A serverless service for crawling, discovering, and cataloging data from various sources.

## 6. Data Preprocessing
Data Preprocessing is the process of converting "messy" raw data into a clean, standardized format that algorithms can process efficiently.

**Common Tasks:**
* **Cleaning:** Removing duplicate entries and fixing structural errors.
* **Imputation:** Handling missing values by filling gaps (e.g., using the mean/median) or dropping incomplete rows.
* **Scaling/Normalization:** Adjusting numerical values to a common range (e.g., 0 to 1) so large numbers don't disproportionately influence the model.
* **AWS Tool:** **AWS Glue DataBrew** (A visual, no-code tool for cleaning and normalizing data with 250+ built-in transformations).

## 7. Feature Engineering
Feature Engineering is the "art" of selecting, creating, or modifying variables (**features**) to maximize the model's predictive performance.

**Key Techniques:**
* **Selection:** Keeping relevant columns (e.g., "Price") but dropping useless ones (e.g., "Internal ID").
* **Creation:** Combining existing data to make new insights (e.g., turning "Date" and "Time" into "Is_Holiday").
* **Transformation/Encoding:** Converting text categories (e.g., "High", "Low") into numerical values (e.g., 1, 0) via techniques like **One-Hot Encoding**.
* **AWS Tool:** **Amazon SageMaker Data Wrangler** (The primary tool for preparing features specifically for ML within SageMaker).

## 8. Data Visualization
Representing data graphically to identify patterns, trends, and anomalies before training.
* **EDA (Exploratory Data Analysis):** The practice of spotting errors, biases, or data imbalances early in the lifecycle.
* **Correlation:** Identifying how variables relate to one another (e.g., "Temperature" vs. "Ice Cream Sales").
* **AWS Tool:** **Amazon QuickSight** (The primary cloud-native BI service used for creating dashboards and visualizing ML insights).

## 9. Transfer Learning
Transfer Learning is a technique where a model trained on one task is **reused as the starting point** for a different but related task — instead of training from scratch.

* **How it works:** A model pre-trained on a massive general dataset (e.g., millions of images or billions of words) already understands low-level patterns like edges, shapes, or grammar. You take this pre-trained model and adapt it to your specific problem using a much smaller dataset.
* **Why it matters:** Training a model from scratch requires enormous amounts of data and compute. Transfer Learning lets you achieve strong results with a fraction of both.
* **Connection to Foundation Models:** This is the exact principle behind Foundation Models — a large base model is pre-trained once, then adapted (via fine-tuning or prompt engineering) for many downstream tasks.
* **Example:** Instead of training an image classifier from scratch with millions of labeled photos, you start with a model pre-trained on ImageNet and fine-tune it with just a few hundred images of your specific product — getting high accuracy much faster.

**Key Concept:** Transfer Learning is what makes modern AI practical. Without it, every new task would require training from zero — making most projects too expensive and slow.

## 10. Training, Evaluation & Deployment
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

**Metrics:** Model performance is measured using metrics like Accuracy, Precision, Recall, F1-Score, and AUC-ROC for classification tasks, and MSE, RMSE, MAE for regression tasks. These are covered in detail with examples in [Part 3](/posts/aws-ai-practitioner-pt3/#evaluation-metrics-for-fms).

### Deployment
* Hosting the model on an endpoint (**Inference**) so applications can make real-world predictions.
* **AWS Tool:** **Amazon SageMaker** (The comprehensive, end-to-end platform for the entire ML lifecycle).

## 11. MLOps (Machine Learning Operations)
Deployment is not the end. MLOps applies **DevOps** principles to ML systems to ensure reliability and scalability.
* **Continuous Integration/Continuous Delivery (CI/CD):** Automating the training and deployment pipelines.
* **Monitoring:** Watching for **Model Drift** (model quality degrades over time) or **Data Drift** (real-world data distribution changes compared to training data).
* **Retraining:** Automatically triggering new training runs when performance drops below a threshold.
* **AWS Tool:** **Amazon SageMaker Model Monitor** (Automatically detects data drift and model quality issues in production).

## 12. No-Code ML with Amazon SageMaker Canvas
Not every ML project requires writing code. **Amazon SageMaker Canvas** provides a **visual, no-code interface** that enables business analysts and non-technical users to build ML models.

* **How it works:** You import your data (from S3, Redshift, or local files), select a target column, and Canvas automatically builds, trains, and evaluates multiple models — then picks the best one.
* **Supported Tasks:** Classification, regression, time-series forecasting, and natural language processing — all without writing a single line of code.
* **When to use it:** Business teams need to generate predictions or forecasts but don't have ML engineering resources.

**Key Concept:** SageMaker Canvas is the **no-code counterpart to SageMaker Studio**. If the exam describes a business analyst who needs to build a model without coding, Canvas is the answer.

## 13. Reinforcement Learning from Human Feedback (RLHF)
RLHF is the training technique that bridges the gap between a raw pre-trained language model and the helpful, safe assistant that end users interact with.

* **The Problem:** A language model trained only to predict the next word can produce outputs that are technically fluent but unhelpful, harmful, or misaligned with human expectations.
* **How it works:**
    1. **Supervised Fine-Tuning (SFT):** The model is first fine-tuned on examples of high-quality, human-written responses.
    2. **Reward Model Training:** Human evaluators rank multiple model outputs for the same prompt. These rankings are used to train a separate **reward model** that scores how "good" a response is.
    3. **Reinforcement Learning:** The language model is further trained using the reward model as a guide — it learns to generate responses that score higher on the reward model's criteria (helpfulness, harmlessness, honesty).
* **Why it matters:** RLHF is how models like Claude are aligned to follow instructions, refuse harmful requests, and produce useful responses — not just statistically likely text.

**Key Concept:** RLHF is the standard technique for **aligning** Foundation Models with human values and expectations. The exam may ask about it in the context of how LLMs are made safe and useful after pre-training.

---