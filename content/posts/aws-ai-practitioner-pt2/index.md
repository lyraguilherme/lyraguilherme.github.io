---
title: 'AWS AI Practitioner - Study Notes #2'
date: 2026-02-21T06:37:49-03:00
draft: false
tags:
 - AI Practitioner
 - AWS
showTableOfContents: true
featureAlt: "AWS AI Practitioner - Study Notes #2"
---

These are my personal study notes for the **AWS Certified AI Practitioner (AIF-C01)** exam. This part covers **Neural Networks** and **Deep Learning** — the foundational concepts behind modern **Generative AI**.

# Neural Networks and Deep Learning

## 1. Neural Networks
Neural networks trace back to the 1950s, inspired by the structure of the human brain. The idea was simple: if the brain processes information through connected neurons, maybe we could simulate that digitally. A basic network has three layers:

* **Input layer:** Receives the raw data, for example: pixel values of an image, or the words in a sentence
* **Hidden layer:** Nodes with adjustable weights that process the input and detect patterns.
* **Output layer:** Applies an activation function to produce the final result. Examples: "this image is a cat" or "this email is spam"

Think of it like a coffee filter: raw data goes in, gets processed through layers, and a refined answer comes out.

Early networks were limited by available computing power and small datasets. They could handle simple tasks — like recognizing handwritten digits — but struggled with anything more complex. Still, they established the core principles that power modern AI.

## 2. Deep Learning
Deep Learning evolves the neural network structure by stacking many hidden layers — sometimes hundreds. Each layer learns a different level of abstraction from the data. For example, when recognizing a face in a photo:

1. Early layers detect edges and basic shapes
2. Middle layers combine those into features like eyes and noses
3. Final layers assemble those features into a full face recognition

This depth allows models to detect complex, non-linear patterns that humans can't easily identify — and to do it at scale, across millions of examples.

### How Training Works (Backpropagation)
Training a Deep Learning model is an iterative process. Each cycle works like this:

1. The model receives an input and makes a prediction
2. The prediction is compared to the correct answer — the difference is the **loss** (error)
3. The model works backwards through the layers, adjusting each weight to reduce that error
4. This repeats thousands or millions of times until the model reaches acceptable accuracy

A practical example: a spam filter starts by misclassifying many emails. With each correction, it adjusts its internal weights — learning that phrases like "click here to claim your prize" are strong spam signals. Over time, it gets much better at catching spam.

**Key Concept — Backpropagation:** The algorithm that allows a neural network to learn from its mistakes by propagating the error signal backwards through the layers and updating the weights accordingly.

### Real-World Use Cases
Deep Learning is behind most of the AI we interact with daily:

* **Image recognition:** Detecting tumors in medical scans, or identifying objects in self-driving car cameras.
* **Natural language processing:** Chatbots, translation services, and sentiment analysis on customer reviews.
* **Recommendation systems:** Netflix suggesting what to watch next, or Spotify generating personalized playlists.
* **Fraud detection:** Flagging unusual credit card transactions in real time.
* **Speech recognition:** Voice assistants like Alexa or Siri converting audio to text.

## 3. Key Deep Learning Architectures
Not all deep learning models are structured the same way. The architecture you choose depends on the type of data and problem. Two foundational architectures appear frequently on the exam.

### Convolutional Neural Network (CNN)
CNNs are purpose-built for **visual data** — images and videos. Instead of looking at an image as one flat list of pixels, a CNN slides small filters (called **kernels**) across the image to detect local patterns.

**How it works:**
1. **Convolutional layers** scan the image with filters to detect low-level features like edges, corners, and textures.
2. **Pooling layers** reduce the size of the data by keeping only the most important information (e.g., "there's an edge here" but discarding the exact pixel positions).
3. **Fully connected layers** at the end combine all detected features to make a final prediction.

* Use case: A self-driving car's camera system. Early layers detect edges and lines, middle layers recognize shapes like stop signs or pedestrians, and final layers classify the full scene — all in real time.
* Use case: Medical imaging. A CNN trained on thousands of X-rays can detect early-stage pneumonia by recognizing subtle patterns in lung tissue that a human radiologist might miss.

**Key Concept:** CNNs are the default architecture for any **image or video** task. If the exam describes a scenario involving visual data (object detection, image classification, content moderation), the answer involves a CNN.

### Recurrent Neural Network (RNN)
RNNs are designed for **sequential data** — any data where order matters, such as text, time series, or audio. Unlike standard neural networks that treat each input independently, RNNs maintain a **hidden state** (a form of memory) that carries information from previous steps in the sequence.

**How it works:**
* The network processes one element at a time (e.g., one word, one time step) and passes its hidden state to the next step — creating a chain where earlier inputs influence later outputs.
* This allows the model to understand context: in the sentence "The clouds were dark, so I grabbed my ___", the RNN's memory of "clouds" and "dark" helps it predict "umbrella."

**The Problem — Vanishing Gradient:**
Standard RNNs struggle with long sequences because the influence of earlier inputs fades as the sequence gets longer — like a game of telephone where the message degrades with each pass.

**The Solution — LSTM (Long Short-Term Memory):**
LSTM is an improved version of RNN that adds "gates" to control what information to **remember**, **forget**, and **output** at each step. This allows it to maintain context over much longer sequences.

* Use case: Stock price forecasting. An LSTM processes months of daily price data in order, remembering long-term trends (like seasonal patterns) while also reacting to recent changes.
* Use case: Real-time speech recognition. As you speak, the RNN processes each audio frame in sequence, using context from earlier words to better predict the current one — which is why "recognize speech" doesn't get transcribed as "wreck a nice beach."

**Key Concept:** RNNs (especially LSTMs) are the go-to for **sequential and time-series data**. However, for most NLP tasks, **Transformers** have largely replaced RNNs because they process all tokens in parallel rather than one at a time — making them much faster to train.

### Architecture Comparison

| Architecture | Best For | Key Advantage | Limitation |
|---|---|---|---|
| CNN | Images, video, visual data | Detects spatial patterns efficiently | Not designed for sequential data |
| RNN / LSTM | Time series, sequences, audio | Maintains memory across steps | Slow to train (processes one step at a time) |
| Transformer | Text, NLP, large-scale generation | Processes all tokens in parallel | Requires large amounts of data and compute |

## 4. Training Concepts
Understanding how models are trained — and how to control the training process — is essential for the exam.

### Hyperparameters
Hyperparameters are settings that you configure **before** training begins. They control *how* the model learns, not *what* it learns. The model cannot adjust these on its own — you must set them.

* **Learning Rate:** How big of a step the model takes when adjusting its weights after each error. Too high and the model overshoots the optimal solution (bouncing around without converging). Too low and training takes forever or gets stuck.
    * Think of it like tuning a radio dial: large turns skip past the station, tiny turns take forever to find it.
* **Batch Size:** How many training examples the model looks at before updating its weights. A batch size of 32 means the model sees 32 examples, calculates the average error, and then adjusts.
    * Larger batches = more stable updates but more memory required. Smaller batches = noisier updates but faster iteration.
* **Epochs:** The number of times the model passes through the **entire** training dataset. One epoch = the model has seen every example once.
    * Too few epochs = the model hasn't learned enough (underfitting). Too many epochs = the model starts memorizing the training data (overfitting).

**Key Concept:** Hyperparameter tuning is the process of finding the best combination of these settings. **Amazon SageMaker Automatic Model Tuning** automates this by running multiple training jobs with different configurations and picking the best result.

### Regularization
Regularization is a set of techniques used to **prevent overfitting** — when the model performs well on training data but poorly on new data.

* **L1 Regularization (Lasso):** Adds a penalty based on the **absolute value** of the model's weights. This pushes some weights all the way to zero, effectively removing irrelevant features. Useful when you suspect many features are unnecessary.
    * Example: A house price model with 200 features. L1 might zero out features like "number of closets" or "distance to nearest library" that don't meaningfully affect price — simplifying the model automatically.
* **L2 Regularization (Ridge):** Adds a penalty based on the **squared value** of the weights. Instead of eliminating features entirely, it shrinks all weights toward zero — keeping every feature but reducing the influence of any single one.
    * Example: The same house price model, but instead of dropping features, L2 ensures that no single feature (like "square footage") dominates the prediction disproportionately.
* **Dropout:** During training, randomly "turns off" a percentage of neurons in each layer, forcing the remaining neurons to learn independently rather than relying on specific connections. Only used during training — all neurons are active during inference.
    * Example: Like randomly removing team members during practice drills so everyone learns to cover multiple positions — the team becomes more resilient and adaptable.
* **Early Stopping:** Monitors the model's performance on the validation set during training and **stops training when performance starts to degrade** — even if there are more epochs remaining. This catches the exact point where the model transitions from learning to memorizing.
    * Example: A model's validation accuracy improves for 50 epochs, plateaus, then starts dropping at epoch 60. Early stopping would halt training around epoch 55, keeping the best version.

**Key Concept:** The exam may describe a scenario where a model has high training accuracy but low test accuracy — this is overfitting. The solution involves regularization techniques (L1, L2, Dropout) or early stopping, not collecting more features or increasing model complexity.

### Data Augmentation
Data Augmentation is the technique of artificially **expanding your training dataset** by creating modified versions of existing data — without collecting new data.

* **For Images:** Rotating, flipping, cropping, adjusting brightness, adding slight blur, or zooming in on existing images. Each transformation creates a "new" training example that teaches the model to recognize the same object under different conditions.
    * Example: You have 1,000 photos of cats for a pet classification model. By applying random rotations, horizontal flips, and brightness changes, you effectively create 5,000+ training images — helping the model recognize cats regardless of angle, lighting, or orientation.
* **For Text:** Synonym replacement, random word insertion, back-translation (translating to another language and back). These create slight variations that preserve meaning.
    * Example: The sentence "The movie was excellent" can be augmented into "The film was outstanding" — giving the sentiment model more diverse examples of positive reviews.

**Key Concept:** Data Augmentation is especially valuable when your dataset is **small or imbalanced**. Instead of spending time and money collecting more data, you generate variety from what you already have. It directly combats overfitting by exposing the model to more diverse examples.

---
