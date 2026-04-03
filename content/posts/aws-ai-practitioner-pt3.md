---
title: 'AWS AI Practitioner - Study Notes #3'
date: 2026-03-07T05:35:00-03:00
draft: false
ShowToC: true
TocOpen: true
cover:
    image: /aws-ai-practitioner/cover-pt3.png
    alt: 'AWS AI Practitioner - Study Notes #3'
    caption: 'AWS AI Practitioner - Study Notes #3'
---

These are my personal study notes for the **AWS Certified AI Practitioner (AIF-C01)** exam. This part covers **Generative AI** and AWS pre-built AI/ML services.

# Generative AI

## 1. Generative AI Model Architectures
Before diving into services and concepts, we need to understand the four main model types that power Generative AI.

### Generative Adversarial Network (GAN)
GANs are defined by "competition." They consist of two neural networks playing a zero-sum game.

**How it works:**
* **The Generator:** Tries to create fake data (like an image) that looks real.
* **The Discriminator:** Tries to distinguish between the "real" training data and the "fake" data produced by the generator.

**Key Concept:** Through this competition, the generator gets better at fooling the discriminator until the output is indistinguishable from reality.

**Best For:** High-resolution image generation, style transfer, and deepfakes.

### Variational Autoencoder (VAE)
VAEs focus on the "latent space" — a compressed mathematical representation of data.

**How it works:**
* **The Encoder:** Compresses the input data into a lower-dimensional "bottleneck" (the latent space).
* **The Decoder:** Reconstructs the data from this compressed version.

**Key Concept:** Unlike standard autoencoders, VAEs add probabilistic noise to the latent space. This allows them to sample new, slightly varied data points from that space.

**Best For:** Image denoising, signal processing, and generating variations of existing data (like facial expressions).

### Transformer Model
This is the "brain" behind Large Language Models (LLMs) like Claude and GPT. It revolutionized AI by moving away from processing data sequentially.

**How it works:**
* **Self-Attention Mechanism:** This is the most important term for the exam. It allows the model to weigh the importance of different words in a sentence, regardless of their distance from each other.
* **Parallelization:** Transformers process all words in a sequence at once (unlike older RNNs), making them much faster to train.

**Key Concept:** The "Attention" mechanism is what allows the model to understand context and long-range dependencies in text.

**Best For:** Natural Language Processing (NLP), translation, and text summarization.

### Diffusion Model
These are the current gold standard for high-quality text-to-image generation (like Stable Diffusion or Midjourney).

**How it works:**
* **Forward Process:** Gradually adds Gaussian noise to an image until it's just pure static.
* **Reverse Process (Reverse Diffusion):** The model learns to "undo" the noise, step-by-step, to recover a clean image from the static.

**Key Concept:** The model isn't "editing" an image; it's denoising random noise to reveal a structure it was prompted to find.

**Best For:** High-fidelity image generation and creative art tools.

### Architecture Comparison

| Model Type | Core Mechanism | Primary Strength |
|---|---|---|
| GAN | Competition (Generator vs. Discriminator) | Realistic image synthesis |
| VAE | Latent space / Probability | Data compression and reconstruction |
| Transformer | Self-Attention / Parallel processing | Text and sequence understanding |
| Diffusion | Iterative Denoising | State-of-the-art text-to-image |

## 2. Foundation Models and Large Language Models
Now that we know the architectures, the next step is understanding what's built on top of them.

### Foundation Models (FMs)
A Foundation Model is a large-scale model that has been **pre-trained on massive, broad datasets** (books, websites, code, images) and can then be **adapted to a wide range of downstream tasks** without being trained from scratch each time.

* **Key Concept:** FMs are not built for one specific task. They learn general patterns first, then get specialized later through fine-tuning or prompt engineering.
* **Example:** A single FM can be adapted to do summarization, translation, code generation, and question answering — all from the same base model.

### Large Language Models (LLMs)
A Large Language Model (LLM) is a specific type of Foundation Model built on the **Transformer architecture**, specialized in understanding and generating text.

* **Tokens:** LLMs don't read words — they process **tokens**, which are chunks of text (roughly 3-4 characters each). A sentence like "The dog sat on the mat" might be split into 7 tokens. Token limits define how much text a model can process in a single request (the **context window**).
* **Training Data:** LLMs are trained on massive collections of text. The quality and diversity of this data directly affect the model's knowledge and biases.

### Multi-Modal Models
Some Foundation Models go beyond text and can process multiple types of input and output:
* **Text + Image:** Understand images and answer questions about them (e.g., "What's in this photo?").
* **Text + Audio:** Transcribe speech or generate audio from text.
* **Key Concept:** Multi-modal models can accept and generate across different data types — text, images, audio, video — within the same model.

## 3. How Generative AI Produces Output
Understanding the mechanics of how models generate text is heavily tested on the exam.

### Inference
Inference is the process of using a trained model to generate predictions or outputs from new input. When you send a prompt to an LLM and get a response, that's inference.

### Inference Parameters
These settings control the behavior and creativity of the model's output:
* **Temperature:** Controls randomness. Low temperature (e.g., 0.1) produces focused, deterministic answers. High temperature (e.g., 0.9) produces more creative, varied responses.
* **Top-p (Nucleus Sampling):** Instead of a fixed number of options, the model considers the smallest set of tokens whose combined probability exceeds `p`. A top-p of 0.9 means the model picks from the tokens that together cover 90% of the probability mass.
* **Top-k:** The model only considers the `k` most likely next tokens. A top-k of 50 means it picks from the 50 most probable options.
* **Max Tokens:** Limits the length of the generated response.
* **Stop Sequences:** Specific strings that tell the model to stop generating.

**Key Concept:** Temperature, Top-p, and Top-k all control the tradeoff between **creativity and consistency**. For factual tasks (e.g., summarizing a report), use low temperature. For creative tasks (e.g., writing a story), use higher temperature.

### Hallucinations
A hallucination occurs when a model generates output that is **plausible-sounding but factually incorrect or fabricated**. The model isn't "lying" — it's predicting the most likely next tokens based on patterns, not retrieving verified facts.

* **Why it happens:** LLMs are pattern-matching machines, not knowledge databases. They can confidently produce wrong answers because the sequence "sounds right."
* **How to mitigate:** Use RAG (covered below), provide context in prompts, lower temperature for factual tasks, or add human review.

## 4. Prompt Engineering
Prompt engineering is the practice of crafting inputs to get the best possible output from a Foundation Model — without changing the model itself.

### Prompting Techniques
* **Zero-Shot:** Asking the model to perform a task with no examples. Example: *"Classify this review as positive or negative: 'The food was amazing!'"*
* **Few-Shot:** Providing a handful of examples in the prompt so the model understands the expected format. Example: *"Review: 'Great product!' → Positive. Review: 'Terrible experience.' → Negative. Review: 'It was okay.' → ?"*
* **Chain-of-Thought (CoT):** Asking the model to reason step-by-step before giving a final answer. This improves accuracy on complex or multi-step problems. Example: *"Think step by step. If a train travels 60 km/h for 2.5 hours, how far does it go?"*

### Prompt Engineering Best Practices
* **Be specific:** Vague prompts produce vague answers. Include context, constraints, and the desired output format.
* **Assign a role:** Telling the model to act as an expert (e.g., "You are a senior AWS architect...") often improves response quality.
* **Provide context:** The more relevant information you include, the better the output.
* **Use delimiters:** Separate instructions from data using markers like `---` or XML tags to avoid ambiguity.

**Key Concept:** Prompt engineering is the **cheapest and fastest** way to improve model output — no training, no infrastructure changes, just better inputs.

## 5. Customizing Foundation Models
When prompt engineering alone isn't enough, there are three main strategies to make a model more useful for a specific domain. Knowing when to use each is critical for the exam.

### Retrieval-Augmented Generation (RAG)
RAG combines an FM with an **external knowledge base** to ground its responses in real, up-to-date data.

**How it works:**
1. The user sends a query.
2. The system searches a knowledge base (e.g., company documents, databases) for relevant information.
3. The retrieved context is injected into the prompt alongside the user's question.
4. The FM generates a response grounded in the retrieved data.

* **Key Concept:** RAG reduces hallucinations by giving the model **real data to reference** instead of relying solely on its training. It does not modify or retrain the model.
* **When to use it:** You need the model to answer questions using **current, proprietary, or domain-specific data** without the cost of retraining.
* **Embeddings:** RAG relies on **vector embeddings** — numerical representations of text that capture semantic meaning. Similar concepts are stored close together in vector space, enabling the system to find relevant documents for a query.
* **AWS Tool:** **Amazon Bedrock Knowledge Bases** (Connects FMs to your data sources like S3, databases, or web crawlers to enable RAG workflows).

### Fine-Tuning
Fine-tuning takes a pre-trained FM and performs **additional training on a smaller, task-specific dataset** to change its behavior, tone, or domain expertise.

* **When to use it:** You need the model to adopt a specific **style, format, or specialized vocabulary** that prompt engineering alone can't achieve (e.g., training a model to write in your company's brand voice, or to understand medical terminology).
* **Key Concept:** Fine-tuning **modifies the model's weights**. It's more expensive and time-consuming than RAG or prompt engineering, but produces deeper behavioral changes.
* **AWS Tool:** **Amazon Bedrock Custom Model Training** (Fine-tune supported FMs with your own labeled data).

### Continued Pre-Training
Goes even further than fine-tuning by training the FM on a **large corpus of unlabeled, domain-specific data** — essentially teaching the model an entirely new knowledge domain.

* **When to use it:** The model needs deep familiarity with a specialized field (e.g., legal, financial, or scientific language) that wasn't well-represented in its original training data.
* **Key Concept:** This is the most expensive and resource-intensive option. Use it only when RAG and fine-tuning aren't sufficient.

### Choosing the Right Approach

| Approach | Modifies Model? | Cost | Best When |
|---|---|---|---|
| Prompt Engineering | No | Lowest | You need better outputs with no infrastructure changes |
| RAG | No | Low-Medium | You need answers grounded in current or proprietary data |
| Fine-Tuning | Yes (weights) | Medium-High | You need to change the model's style or domain behavior |
| Continued Pre-Training | Yes (weights) | Highest | You need deep domain-specific knowledge |

## 6. Selecting and Evaluating Foundation Models
Choosing the right FM for a use case is a common exam scenario. You won't always use the biggest or most expensive model — the right choice depends on the task requirements.

### Selection Criteria
* **Task Type:** What do you need the model to do? Text generation, image creation, code completion, embeddings for search? Not all FMs support all tasks.
* **Performance vs. Cost:** Larger models are generally more capable but cost more per inference call. A simple classification task doesn't need a 100B-parameter model.
* **Latency:** Real-time applications (e.g., chatbots) need fast response times. Smaller models or optimized endpoints may be required.
* **Context Window:** How much text does the model need to process at once? Summarizing a 50-page document requires a large context window.
* **Modality:** Do you need text-only, or text + image, or text + code? Choose a model that supports your required input/output types.
* **Customizability:** Some FMs support fine-tuning through Bedrock, others don't. If you need to customize, check availability first.

### Evaluation Metrics for FMs
After selecting a model, you need to measure how well it performs. The exam tests these metrics:

**For Text Generation / Summarization:**
* **ROUGE (Recall-Oriented Understudy for Gisting Evaluation):** Compares the model's output to a reference text by measuring overlap of n-grams (words or phrases). High ROUGE = the model captured the key content from the reference.
    * Use case: Evaluating a summarization model. If the model's summary contains most of the important phrases from the original text, it will have a high ROUGE score.
* **BLEU (Bilingual Evaluation Understudy):** Originally designed for machine translation. Measures how many n-grams in the model's output match the reference. High BLEU = the generated text closely matches the expected translation.
    * Use case: Evaluating a language translation model. If the model produces accurate and fluent translations, it should have a high BLEU score.
* **BERTScore:** Uses embeddings to compare semantic similarity between generated and reference text, rather than exact word matching. More forgiving of paraphrasing than ROUGE or BLEU.
    * Use case: Evaluating a chatbot’s response generation. If the model generates responses that are semantically similar to the user’s input, it should have a high BERTScore.

**For Classification Tasks:**
The same metrics from traditional ML apply:
* **Accuracy:** Overall percentage of correct predictions.
    * Use case: Evaluating a spam detection model. If the model accurately identifies 90% of spam emails as such, it has an accuracy score of 0.9.
* **Precision:** Of all positive predictions, how many were actually correct? (Important when false positives are costly).
    * Use case: Evaluating a fraud detection model. If the model flags 100 transactions as fraudulent, but only 80 are actually fraudulent, the precision is 0.8.
* **Recall (Sensitivity):** Of all actual positives, how many did the model catch? (Important when false negatives are costly).
    * Use case: Evaluating a medical diagnosis model. If there are 100 patients with a disease and the model correctly identifies 90 of them, the recall is 0.9.
* **F1-Score:** The harmonic mean of Precision and Recall, useful when you need a balance between both.
    * Use case: Evaluating a sentiment analysis model. If the model has a precision of 0.8 and a recall of 0.9, the F1-Score would be approximately 0.85, indicating good overall performance.

**Human Evaluation:**
* **Key Concept:** Automated metrics don't capture everything. Human evaluation is used to judge qualities like **helpfulness, harmlessness, and honesty** — especially for open-ended generation where there's no single "correct" answer.
* **AWS Tool:** **Amazon Bedrock Model Evaluation** (Allows you to run automatic and human-based evaluations on FMs directly within Bedrock).

## 7. AWS Services for Generative AI
Now let's tie everything back to the AWS ecosystem — the exam will test you on which service to use in which scenario.

### Amazon Bedrock
The **central, fully managed service** for building Generative AI applications on AWS. Think of it as the "one-stop shop" for Foundation Models.
* **Access to Multiple FMs:** Provides API access to models from Anthropic (Claude), Amazon (Titan), Meta (Llama), AI21 Labs, Cohere, Stability AI, and others — without managing infrastructure.
* **Bedrock Knowledge Bases:** Enables RAG by connecting FMs to your data sources.
* **Bedrock Agents:** Allows FMs to execute multi-step tasks by calling APIs and interacting with external systems.
* **Bedrock Guardrails:** Defines safety policies to filter harmful content, block sensitive topics, and control model behavior.
* **Custom Model Training:** Fine-tune or continue pre-training supported FMs with your own data.
* **Key Concept:** Bedrock is **serverless** — you don't manage any infrastructure. You pay per API call. If a question asks about the easiest way to access multiple FMs, the answer is Bedrock.

### Amazon Titan
Amazon's own family of Foundation Models, available through Bedrock:
* **Titan Text:** Text generation, summarization, and conversational AI.
* **Titan Embeddings:** Converts text into vector embeddings for search and RAG.
* **Titan Image Generator:** Creates and edits images from text prompts.
* **Key Concept:** Titan models are **first-party AWS models** — built by Amazon and optimized for the AWS ecosystem.

### Amazon SageMaker JumpStart
A **model hub** within SageMaker that provides access to hundreds of pre-trained models (including open-source FMs like Llama and Falcon) that you can deploy with a few clicks.
* **Key Difference from Bedrock:** JumpStart gives you more control — you deploy models to your own SageMaker endpoints and manage the infrastructure. Bedrock is fully managed and serverless.
* **When to use it:** You need to host, customize, or fine-tune open-source models with full control over the underlying compute.

### Amazon Q
AWS's AI-powered assistant, available in multiple flavors:
* **Amazon Q Business:** An AI assistant that connects to your company's internal data (documents, wikis, Slack, etc.) to answer employee questions.
* **Amazon Q Developer:** An AI coding assistant integrated into IDEs that helps with code generation, debugging, and understanding code.
* **Key Concept:** Amazon Q is purpose-built for specific roles (business users, developers) — it's not a general-purpose FM like Bedrock models.

### Amazon PartyRock
A **no-code playground** for experimenting with Generative AI applications. You can build and share simple FM-powered apps in a browser without writing any code or having an AWS account.
* **When to use it:** Prototyping, learning, and experimenting with Generative AI concepts.

## 8. AWS AI/ML Application Services
Not every AI problem needs a Foundation Model. AWS offers a set of **pre-built, task-specific AI services** that are ready to use out of the box — no ML expertise required. The exam frequently tests which service to pick for a given scenario.

### Vision
* **Amazon Rekognition:** Analyzes images and videos — detects objects, faces, text, scenes, and inappropriate content. Use cases: facial verification, content moderation, PPE detection on construction sites.
* **Amazon Textract:** Extracts text, tables, and form data from scanned documents (PDFs, images). Goes beyond simple OCR by understanding document structure. Use cases: invoice processing, medical records extraction, ID verification.

### Language
* **Amazon Comprehend:** Natural Language Processing (NLP) service that extracts insights from text — sentiment analysis, entity recognition, key phrase extraction, language detection, and topic modeling. Use cases: analyzing customer reviews, detecting PII in documents.
* **Amazon Translate:** Real-time language translation supporting 75+ languages. Use cases: translating user-generated content, multilingual customer support.
* **Amazon Lex:** Builds conversational interfaces (**chatbots**) with voice and text. The same technology that powers Alexa. Use cases: customer service bots, automated order taking.
* **Amazon Kendra:** Intelligent **enterprise search** powered by ML. Indexes documents from multiple sources and returns precise answers to natural language questions (not just keyword matches). Use cases: internal knowledge base search, IT helpdesk.

### Speech
* **Amazon Transcribe:** Converts speech to text (**Speech-to-Text / STT**). Supports real-time and batch transcription, speaker identification, and custom vocabularies. Use cases: call center analytics, meeting transcription, subtitles.
* **Amazon Polly:** Converts text to lifelike speech (**Text-to-Speech / TTS**). Supports multiple languages and voice styles (including Neural TTS for more natural-sounding output). Use cases: audiobook generation, voice notifications, accessibility.

### Data & Predictions
* **Amazon Personalize:** Builds real-time **recommendation engines** using the same ML technology as Amazon.com. Use cases: product recommendations, personalized content feeds, targeted marketing.
* **Amazon Forecast:** Generates **time-series forecasts** using ML — no ML experience needed. Use cases: demand planning, inventory forecasting, resource capacity planning.
* **Amazon Fraud Detector:** Detects potentially **fraudulent online activities** (e.g., fake accounts, payment fraud) using ML models trained on your data. Use cases: new account fraud, online payment verification.

### Choosing Between Pre-Built Services and Foundation Models

| Use... | When... |
|---|---|
| **Pre-Built AI Service** | You have a well-defined task (e.g., "extract text from invoices," "detect faces") and want a ready-made solution with no model management |
| **Foundation Model (Bedrock)** | You need flexible, general-purpose AI capabilities (e.g., open-ended Q&A, content generation, complex reasoning) or want to customize behavior |

**Key Concept:** Pre-built services are **faster to deploy and cheaper to run** for their specific tasks. Don't use a general-purpose FM when a purpose-built service already solves the problem.

---
