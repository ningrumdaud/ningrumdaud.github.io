---
layout: post
title:  "Using NLP to Track ESG Controversies"
author: Ningrum
categories: [ portfolio ]
image: assets/images/AILoveMySkin.png
---


# Building an AI Sentinel: A Real-Time ESG Controversy Radar

As a data scientist, I’m passionate about turning unstructured data into actionable intelligence. The investment world is flooded with text—news articles, regulatory filings, earnings call transcripts. Hidden in this deluge are early signals of risk and opportunity that traditional models often miss.

This led me to a question: **Can we build a system that automatically detects corporate controversies in real-time, like a seasoned analyst reading the news?**

The result is my personal project: an end-to-end NLP pipeline that acts as an automated sentinel for ESG (Environmental, Social, and Governance) risks. This is not just sentiment analysis; it is a scientifically rigorous system that brings clarity and speed to a fast-moving problem.

---

## Why This Matters

ESG is more than a buzzword. Companies facing environmental disasters, labor strikes, or governance scandals face regulatory fines, reputational damage, and stock declines. These events often break first in the news, long before quarterly reports. No analyst can read every headline for every company in a portfolio—this is where automation becomes critical.

---

## A Peek Under the Hood: The Science Behind the Sentinel

This project was designed for accuracy, transparency, and robustness. Every step is backed by proven scientific rationale.

### 1. Building a High-Quality Labeled Dataset

**Challenge:** High-quality labeled data is scarce for financial ESG news. Manual labeling is costly and subjective.  

**Solution:** I used a committee of open-source LLMs (Gemma2, LLaMA 3.1, Mistral) with few-shot prompts to classify headlines into E, S, G, or Non-ESG. Using **majority voting across models**, I created a high-quality silver-standard dataset.  

*Rationale:* This “model distillation” approach reduces human bias while leveraging the reasoning power of multiple models.

---

### 2. A Specialized ESG Classifier

LLMs are powerful but slow for real-time use.  

**Solution:** I fine-tuned a **DistilBERT model** on the LLM-labeled dataset. This smaller, task-specific model is fast and efficient, making it ideal for real-time ESG news classification.

*Rationale:* Knowledge distillation retains the expertise of large models in a lightweight, deployable form.

---

### 3. Understanding Context: Aspect-Based Sentiment Analysis (ABSA)

Knowing a headline is about Governance is useful—but knowing the exact issue and sentiment is *actionable*.  

**Solution:** I integrated a pre-trained ABSA model to identify specific aspects (e.g., “carbon emissions,” “board diversity”) and assign sentiment scores. This allows the system to differentiate minor complaints from major scandals.

*Rationale:* Detailed, structured insights improve risk assessment and decision-making beyond simple classification.

---

### 4. Dashboard: From Text to Intelligence

The final demo uses a **Gradio interface** to display:

- **ESG Classification:** E, S, G, or Non-ESG
- **Confidence Scores:** Model certainty for each prediction
- **Aspect & Sentiment:** Specific issues mentioned and their sentiment
- **Visual Analytics:** Overview of news flow by category

*Rationale:* Converts raw text into structured, quantifiable risk reports, making insights easy to digest.

---

## Future Vision: Real-Time ESG Intelligence

The demo uses a static dataset. Here’s the roadmap for a live system:

### Phase 1: Real-Time Data Ingestion
- Connect to live news feeds via APIs or web scrapers.
- Use a message broker (e.g., Apache Kafka) to handle high-throughput streams.
- Implement entity linking to tag news items with companies or stock tickers.

### Phase 2: Alerting and Portfolio Integration
- Configurable notifications (Slack, email) for high-severity events.
- Portfolio-level ESG health dashboards highlighting companies with negative news.
- Historical analysis linking controversy signals to stock movements.

### Phase 3: Continuous Learning
- Human-in-the-loop feedback to validate predictions.
- Continuous fine-tuning to improve model accuracy over time.

---

## Conclusion: Democratizing Quantitative Insight

This project is more than a demo; it’s a blueprint for the future of investment research. By systematically parsing ESG news, the system augments human analysts, providing faster, more comprehensive monitoring.  

In the information age, **reading the news is not enough—understanding it in a structured way is a competitive advantage.** This AI sentinel translates headlines into risk insights, and that is a language every investor needs to speak.

