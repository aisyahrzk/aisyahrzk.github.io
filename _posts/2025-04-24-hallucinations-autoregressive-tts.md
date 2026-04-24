---
layout: post
title: "What Actually Causes Hallucinations in Autoregressive TTS? A Mechanistic Breakdown of Failure Modes"
date: 2025-04-24 10:00 +0700
categories: research ml
---

Autoregressive text-to-speech (TTS) models have revolutionized speech synthesis, but they're not without their flaws. One of the most intriguing issues is "hallucination" — where the model generates content that wasn't in the input text.

## What Are Hallucinations?

In TTS, hallucinations occur when the model:
- Generates words or phrases not present in the input text
- Repeats phrases unnaturally
- Produces nonsensical utterances that sound confident but wrong

## The Root Causes

### 1. Exposure Bias

Autoregressive models are trained with teacher forcing, but during inference they must rely on their own predictions. This mismatch creates a compounding error — the model has never seen its own mistakes during training.

### 2. Attention Collapse

The attention mechanism sometimes gets stuck in a loop, attending to the same token repeatedly. This leads to the dreaded "stuttering" effect where a single syllable or word repeats.

### 3. Latent Space Drift

Over long sequences, the model's internal representation can drift away from the training distribution, especially for:
- Long input texts
- Unusual character combinations
- Out-of-vocabulary words

## Mitigation Strategies

1. **Non-autoregressive models** — Trade some quality for consistency
2. **Constrained decoding** — Limit the model's freedom during generation
3. **Attention regularization** — Penalize repetitive attention patterns

The trade-off between fluency and reliability remains an open question in TTS research.

---

*This is a simplified overview — the actual mechanisms are even more fascinating (and complex).*
