---
layout: post
title: "What do codec tokens encode, and why does it matter for TTS?"
date: 2026-11-21 10:00 +0800
categories: research speech
published: false
---

<!--
HOW TO USE THIS DRAFT
- Each section has a short note in an HTML comment like this one saying what goes there. Delete the comment once you've written the section.
- [FIGURE: ...] and [TABLE: ...] are placeholders. Images go in /assets/images/ and are linked as ![caption](/assets/images/name.png).
- When it's ready, delete the `published: false` line in the front matter and push.
- Target length: 2,000 to 3,000 words. Plots over prose. One main claim per section.
-->

<!-- TL;DR: 4 to 6 bullets. Write this LAST. One line per finding, each with a number in it. -->
**TL;DR**

- Finding 1
- Finding 2
- Finding 3
- What this means for anyone training codec-based TTS

## Why I looked at this

<!--
Start from a failure, not from the literature. The Orpheus repeat/skip story: three attention heads tracking position, ablation breaks 99.6% of clips. The question that led here: is the model struggling because of what the codec gives it?
State the two questions this post separates:
  1. Can the codec *reconstruct* prosody?
  2. Can an autoregressive LM *model* prosody through those tokens?
Say plainly that this post answers question 1 and most of question 2 (probing), and that training TTS over each codec is the follow-up.
-->

## What a neural codec actually gives the language model

<!--
Short explainer, written for someone who trains LLMs but hasn't touched audio. Keep it to ~300 words.
- Encoder, residual vector quantisation (RVQ), decoder. One diagram.
- Frame rate (12.5 Hz vs 50 Hz vs 75 Hz) and number of codebooks: why both matter for an autoregressive model (sequence length, how many tokens per phoneme).
- "Semantic" first codebook (SpeechTokenizer, Mimi, X-Codec) vs purely acoustic (DAC, EnCodec).
- One line each on why the common assumption is "early codebooks = content, later = detail", and that nobody has pinned down where prosody lives.
-->

[FIGURE: block diagram of encoder -> RVQ (stacked codebooks) -> decoder, with frame rate and codebook count labelled]

## The codecs I compared

<!--
Table, then one line per codec saying what it was built for and what TTS model uses it (SNAC -> Orpheus, Mimi -> Moshi, X-Codec2 -> Llasa, BiCodec -> Spark-TTS, etc.).
Note licences here: some weights are non-commercial.
-->

[TABLE: codec | frame rate (Hz) | codebooks | bitrate (kbps) | sample rate | semantic distillation? | used by | licence]

## Data and setup

<!--
Keep this honest and short.
- Test audio: how many hours, Malay / English / code-switched split, how many speakers, which domains (from the Malaysian Speech Benchmark?).
- F0 extraction: name the one extractor you used everywhere and why (consistency matters more than which one).
- Voiced frames only for F0 metrics.
- Compute: how long on what GPU, so people know this is cheap to reproduce.
- Link to the repo.
-->

## Part 1: Can the codec reconstruct prosody?

<!--
Encode -> decode each codec, compare to the original.
Metrics, one row per codec:
  - F0 correlation and RMSE (voiced frames)
  - Energy contour correlation
  - Prosody / emotion similarity from a prosody encoder (say which)
  - UTMOS (naturalness)
  - WER by ASR round trip (does content survive)
Group by bitrate tier. The point to make: reconstruction fidelity and prosody fidelity are not the same ranking.
Pick one surprising row and show the F0 contour overlay for it.
-->

[TABLE: reconstruction metrics per codec, grouped by bitrate tier]

[FIGURE: F0 contour overlays, original vs reconstructed, for 2 or 3 codecs on the same Malay utterance, ideally a question]

### What this says

<!-- Two or three sentences. Which codec keeps pitch best, which flattens it, whether code-switched utterances are worse. -->

## Part 2: Where does prosody live in the tokens?

<!--
Linear probes on each codebook layer and on cumulative layers (1, 1-2, 1-3, ...).
Targets:
  - phoneme (content)
  - speaker ID
  - F0 bin per frame (prosody)
  - energy bin per frame
  - question vs statement (utterance level, from punctuation)
  - emphasis, if you have any labels; otherwise say you skipped it
Controls: shuffled tokens, randomly initialised codec. Say what chance level is.
Plot: probe accuracy vs codebook index, one line per target, one panel per codec. This is the figure people will share.
-->

[FIGURE: probe accuracy vs codebook layer, one panel per codec, lines for content / speaker / F0 / energy]

### Temporal accessibility

<!--
The probe the literature doesn't do. How much context does a probe need to recover F0 at frame t: tokens at t only, +-2 frames, +-8 frames, +-32 frames.
Why it matters: an autoregressive LM has to produce prosody causally. If pitch is only recoverable from a wide window, the LM has to carry it in its own state.
Plot: F0 probe accuracy vs context window, one line per codec.
-->

[FIGURE: F0 probe accuracy vs context window size, one line per codec]

### What this says

<!-- Which codecs put prosody early and local, which smear it across layers and time. Tie back to the Mimi-and-pitch hint from the DualCodec paper if your results agree or disagree. -->

## Part 3: Does this predict TTS failures?

<!--
If you have trained TTS over more than one codec, this is the payoff section:
  - same model size, same data, same steps, different codec
  - repeat/skip rate from the detector
  - F0 correlation to reference, WER, prosody similarity
  - hypothesis: codecs with prosody accessible early and locally give fewer repeat/skip failures and better prosody
If you have NOT trained them yet, keep this section as a short "what I expect and why", clearly labelled as a hypothesis, and say it's the next post. Don't pad it.
-->

[TABLE: TTS results per codec: repeat/skip %, F0 corr, WER, prosody sim]

## What didn't work

<!--
At least three things. Probes that didn't converge, a codec you dropped, an extractor that gave garbage on code-switched audio, a metric that turned out meaningless. This section is why people will trust the rest.
-->

## Limitations

<!--
- Probes measure linear accessibility, not what the LM uses.
- Frame rate and bitrate confound comparisons; you controlled for it by tiering, but it's not perfect.
- Malay prosody labels are proxies (F0, punctuation).
- One language family, one test set.
-->

## What I'd do next

<!-- Three bullets max. The TTS-over-each-codec experiment, a Malay prosody-labelled set, and whatever surprised you. -->

## Reproduce it

<!-- Repo link, which script makes which figure, how long it takes. Checkpoints or cached tokens on Hugging Face if you can release them. -->

---

<!--
Related work: keep it to one short paragraph here at the end, with links. Don't open the post with it.
- Sadok et al., Bringing Interpretability to Neural Audio Codecs (Interspeech 2025): https://www.isca-archive.org/interspeech_2025/sadok25_interspeech.pdf
- Probing Neural Audio Codecs with Speaker Traits and Speech Characteristics: https://arxiv.org/abs/2610.20955
- Speech Codec Probing from Semantic and Phonetic Perspectives: https://arxiv.org/abs/2603.10371
- Recent Advances in Discrete Speech Tokens: A Review: https://arxiv.org/abs/2502.06490
- AudioCodecBench: https://arxiv.org/abs/2509.02349
- DualCodec (note on Mimi and pitch): https://arxiv.org/abs/2505.13000
- AffectCodec (prosody similarity numbers): https://arxiv.org/abs/2605.11098
-->
