---
title: 'AWS AI Practitioner - Study Notes #2'
date: 2026-02-21T06:37:49-03:00
draft: false
ShowToC: true
TocOpen: true
cover:
    image: /aws-ai-practitioner/cover-pt2.png
    alt: 'AWS AI Practitioner - Study Notes #2'
    caption: 'AWS AI Practitioner - Study Notes #2'
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

---
