---
layout: post
title: "Which speech codec keeps prosody, and what makes it work?"
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
State the three questions this post goes through in order:
  1. Which codec preserves prosody best? (reconstruction + probing)
  2. What about its design makes that happen? (frame rate, codebooks, semantic distillation, bitrate)
  3. Does a small TTS trained on the winner actually sound better than one trained on a loser?
One sentence on scope: Malay and code-switched speech, small models, one GPU. Not a general benchmark.
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

## Part 3: What makes a codec good at prosody?

<!--
This is the section the title promises. Codecs differ in many things at once, so be careful about attributing a result to one design choice.
Two ways to make it credible:
  a) Matched pairs that differ mainly in one axis:
     - semantic distillation vs none: X-Codec2 vs DAC (or SpeechTokenizer vs EnCodec)
     - frame rate: Mimi (12.5 Hz) vs SNAC / DAC (higher)
     - single codebook vs RVQ: WavTokenizer vs DAC
     - factorised vs entangled: FACodec vs everything else
  b) A simple regression: prosody metric ~ frame rate + codebooks + bitrate + semantic (yes/no). Report coefficients with the caveat that n is small.
Design axes to discuss, one short subsection or paragraph each:
  - Frame rate: fewer frames per second means fewer tokens per phoneme, but can pitch movement survive 12.5 Hz?
  - Number of codebooks and where prosody sits in them: if pitch is only in layer 4+, a TTS LM that predicts layers in order has to get content right first and prosody last.
  - Semantic distillation in codebook 1: helps content, but does it push pitch out of the first layer? (This is the Mimi hint.)
  - Bitrate: does more bitrate buy prosody or just reconstruction fidelity?
  - Single-codebook codecs: easiest for an LM, but is prosody squeezed out?
End with a one-paragraph design recommendation, stated as a hypothesis: "a codec for TTS should have X, Y, Z." Then say you tested it in Part 4.
-->

[FIGURE: prosody metric (F0 corr or probe accuracy) vs frame rate, one point per codec, colour = semantic distillation yes/no, size = bitrate]

[TABLE: matched pairs: pair | design axis that differs | prosody metric A | prosody metric B | difference]

## Part 4: Training a small TTS on the winner

<!--
The payoff. Keep it to what you can afford on one GPU.
Setup (be exact, people will copy it):
  - Data: hours, speakers, language mix. Same data for every run.
  - Model: decoder-only transformer, size (e.g. ~100 to 150M), context length, how codec tokens are flattened (interleaved codebooks like Orpheus, or one codebook like Llasa), text tokenizer.
  - Training: steps, batch, LR, hours on what GPU. Same budget for every run.
Runs: the winner from Part 3, plus at least ONE contrasting codec (the loser on prosody, or the one that differs on the design axis you care most about). One run alone proves nothing; two runs is a comparison.
Evaluation on a held-out set:
  - repeat/skip rate (your detector from the Orpheus post)
  - F0 correlation and prosody similarity against reference audio
  - WER by ASR round trip (content survived?)
  - UTMOS or a small listening test, if you have time. If you do a listening test, say how many listeners and clips.
  - a few audio samples embedded in the post: same sentence, both codecs. A question and an exclamation show prosody best.
Discuss: did the prosody ranking from Parts 1-3 predict the TTS result? If yes, the design recommendation holds. If no, that's the most interesting finding in the post; say what you think broke.
-->

[TABLE: TTS results per codec: repeat/skip % | F0 corr | prosody sim | WER | UTMOS]

[FIGURE: F0 contours for one held-out question: reference vs TTS-with-winner vs TTS-with-contrast]

[AUDIO: 3 or 4 paired samples, same text, winner vs contrast codec]

## What didn't work

<!--
At least three things, honestly. A codec you dropped and why, a probe that didn't converge, a metric that turned out meaningless, a TTS run that collapsed. This section is why people will trust the rest.
-->


## Limitations

<!--
- Probes measure linear accessibility, not what the LM uses.
- Frame rate and bitrate confound comparisons; you controlled for it by tiering, but it's not perfect.
- Malay prosody labels are proxies (F0, punctuation).
- One language family, one test set.
- The design conclusions in Part 3 are correlational: codecs differ in several ways at once and n is small.
- TTS was trained on two (or three) codecs at one small size; a bigger model might close the gap.
-->

## What I'd do next

<!-- Three bullets max. Train a codec with the recommended design yourself, a Malay prosody-labelled set, scaling the TTS up, or whatever surprised you. -->

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
