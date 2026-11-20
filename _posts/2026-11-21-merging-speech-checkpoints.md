---
layout: post
title: "One model from many: merging speech checkpoints without training"
date: 2026-11-21 11:00 +0800
categories: research speech
published: false
---

<!--
HOW TO USE THIS DRAFT
- Each section has a comment saying what goes there. Delete the comment once you've written the section.
- [FIGURE: ...], [TABLE: ...] and [AUDIO: ...] are placeholders. Images go in /assets/images/.
- When it's ready, delete the `published: false` line in the front matter and push.
- Target length: 2,000 to 3,000 words. Plots over prose. One main claim per section.
-->

<!-- TL;DR: 4 to 6 bullets, written LAST, one number per bullet. -->
**TL;DR**

- Finding 1
- Finding 2
- Finding 3
- When merging is good enough to skip training, and when it isn't

## Why I looked at this

<!--
Start from the practical problem, not the method.
- You end up with a shelf of checkpoints from the same base: one fine-tuned for Malay, one for Arabic, one for noisy telephony, one for code-switching, maybe one for a specific domain. Each is good at one thing.
- Serving all of them is expensive and routing is brittle. Fine-tuning on everything at once means re-collecting data and re-training every time one capability changes. Sequential fine-tuning forgets.
- The question: can I add the checkpoints together and get one model that keeps most of each capability, with zero training?
- One sentence on scope: ASR (Whisper-family or your own encoder-decoder), a handful of capabilities, evaluated on held-out sets you trust. TTS merges are a short section at the end or a follow-up.
-->

## How merging works, in two paragraphs

<!--
Written for someone who trains models but hasn't merged any. Keep it short; link out for details.
- Task vector = fine-tuned weights minus base weights. Merging = base + some combination of task vectors.
- The methods you'll compare, one line each:
  - plain weight averaging (model soup)
  - task arithmetic: base + lambda * sum of task vectors
  - TIES: trim small values, resolve sign conflicts, then merge
  - DARE: randomly drop and rescale task-vector entries, then merge (often combined with TIES)
  - SLERP for pairwise merges
  - optionally a low-rank / sparse merge (LoRS-style) if you have time
- The one hard requirement: every checkpoint must come from the same base with the same architecture and tokenizer. Say what happens if you break this (nothing useful).
- One diagram.
-->

[FIGURE: base model + task vectors A, B, C -> merged model; show trimming / sign resolution for TIES in a small inset]

## The checkpoints

<!--
Table. Be exact about what each was fine-tuned on, because the whole post depends on the capabilities being genuinely different.
Mention how long each took to train, so the "free" in "merging is free" has a number next to it.
-->

[TABLE: checkpoint | base | fine-tuned for | data (hours, domain, language) | training cost (GPU-hours) | its own eval set]

## Setup

<!--
- Base model and size.
- Methods and hyperparameters: lambda sweep for task arithmetic, trim density for TIES, drop rate for DARE. State the sweep ranges.
- Tooling: mergekit if it supports your architecture; if you had to write the merge yourself (likely for Whisper or a custom encoder-decoder), say so and link the code.
- Evaluation: one held-out set per capability, same metric throughout (WER, plus your hallucination rate if relevant). Also a "nothing should change" set: plain English or a domain none of the checkpoints touched, to catch collateral damage.
- Compute for the whole study: merging is seconds; evaluation is where the time goes. Give the number.
-->

## Part 1: Two at a time

<!--
Pairwise merges of every pair of checkpoints, every method.
The number to report is "retained": merged WER on capability A relative to specialist A's WER, as a percentage. 100% means the merge is as good as the specialist. Also report the base model so people see how far the specialists are from it.
Key figure: a heatmap, rows = capability evaluated, columns = pair merged, cells = retained %. One heatmap per method, or one with the best method and a table for the rest.
Point to make: which pairs merge cleanly and which fight. Expect language pairs to interfere more than language + noise-robustness.
-->

[FIGURE: heatmap of retained % for pairwise merges, best method]

[TABLE: pairwise results, all methods: pair | method | retained A | retained B | collateral set WER]

### What this says

<!-- Which capabilities are compatible, which aren't, and whether the method choice matters more than the pair choice. -->

## Part 2: All of them at once

<!--
Merge all N checkpoints. This is where naive methods fall apart.
- Task arithmetic: show the lambda sweep. There's usually a narrow window where it works and cliffs on both sides.
- TIES and DARE-TIES: expect these to be more stable. Report retained % per capability.
- Plot: retained % per capability as a function of number of checkpoints merged (2, 3, ... N), one line per method. Does it degrade smoothly or collapse?
-->

[FIGURE: retained % vs number of checkpoints merged, one line per method, one panel per capability]

[FIGURE: task-arithmetic lambda sweep, WER per capability vs lambda]

### What this says

<!-- Is there a merged model you'd actually ship? Which method, which settings, and what it costs you on each capability. -->

## Part 3: Where does the interference come from?

<!--
The insightful section. Everything above is a ranking; this is an explanation.
Cheap analyses that usually pay off:
  - Cosine similarity between task vectors, per checkpoint pair. Do the pairs that merge badly have the most negative or the most aligned vectors?
  - Per-layer: compute task-vector norm and pairwise similarity for each layer. Is the conflict in the encoder (acoustics) or the decoder (language)? For language pairs expect decoder; for noise-robustness expect encoder.
  - Sign conflicts: what fraction of parameters have opposite signs across task vectors, per layer. This is what TIES is fixing; show where it's needed.
  - A targeted merge: merge only the encoder from the noise-robust checkpoint and only the decoder from the language checkpoints. Does the hand-built merge beat the automatic ones?
Plot: per-layer sign-conflict rate or similarity, x = layer index, with encoder/decoder boundary marked.
-->

[FIGURE: per-layer task-vector similarity or sign-conflict rate, encoder vs decoder marked]

[TABLE: targeted (layer-selective) merge vs best automatic merge, retained % per capability]

### What this says

<!-- One paragraph: the picture of where capabilities live in the network and why some combinations are free and others aren't. -->

## Part 4: The honest comparison

<!--
Merging has to be compared with the thing it replaces: one multi-task fine-tune on the union of all the data, same base, reasonable budget.
Table: specialists | best merge | multi-task fine-tune, WER per capability plus total GPU-hours.
The question to answer plainly: how much of the trained model's quality does the free merge get you, and is that enough for your use case? Give the answer as a number, e.g. "the merge closes X% of the gap between base and multi-task, for 0 GPU-hours instead of Y".
If you don't have budget for the multi-task baseline, say so and use the specialists as the ceiling instead. Don't skip the section.
-->

[TABLE: specialists vs best merge vs multi-task fine-tune: WER per capability, hallucination rate, GPU-hours]

## A note on TTS

<!--
Optional. If you have TTS checkpoints from one base (different voices, languages or styles), try a pairwise merge and report whether the merged model can do both, with two audio samples. If it just produces an averaged voice, that's still worth two sentences. If you haven't tried it, cut this section and put it under "What I'd do next".
-->

[AUDIO: two or three samples, merged TTS, if attempted]

## What didn't work

<!--
At least three. A method that collapsed, a pair that could not be merged at any lambda, a tokenizer mismatch you discovered the hard way, an eval set that turned out to overlap with training data.
-->

## Limitations

<!--
- A handful of checkpoints from one base; results may not transfer to other bases or sizes.
- Held-out sets are small; report confidence intervals or at least say how many hours each is.
- Merging methods have hyperparameters; you tuned them on the eval sets, so the retained % is optimistic. Say by how much if you held out a second split.
- Analysis in Part 3 is correlational.
-->

## What I'd do next

<!-- Three bullets max: merge LoRA adapters instead of full checkpoints, merging across model sizes via distillation, TTS merges, evolutionary search over layer-wise weights. -->

## Reproduce it

<!-- Repo link, the merge script, the eval script, which command produces which figure. If the checkpoints can be released, link them; if not, say which public Whisper fine-tunes reproduce the same story. -->

---

<!--
Related work: one short paragraph at the end, with links. Don't open with it.
General merging:
- Model soups (Wortsman et al., 2022): https://arxiv.org/abs/2203.05482
- Task arithmetic (Ilharco et al., 2023): https://arxiv.org/abs/2212.04089
- TIES-Merging (Yadav et al., 2023): https://arxiv.org/abs/2306.01708
- DARE (Yu et al., 2023): https://arxiv.org/abs/2311.03099
- mergekit: https://github.com/arcee-ai/mergekit
Speech-specific (check these before writing; they're close to this post):
- LoRS-Merging, low-rank and sparse merging for multilingual ASR: https://arxiv.org/abs/2502.17380
- Benchmark of 11 merging methods across 10 ASR domains (European Portuguese): https://arxiv.org/abs/2603.05354
- Merging Whisper fine-tunes for code-switching ASR; finds TIES most stable, task arithmetic and DARE unstable with 3+ models: https://arxiv.org/abs/2606.05846
- Task arithmetic to transfer from high-resource to low-resource Whisper languages: https://arxiv.org/abs/2601.07038
Positioning: these show merging works for ASR. What they don't do is explain where interference comes from (Part 3) or compare against the multi-task fine-tune on an equal budget (Part 4). That's your angle.
-->
