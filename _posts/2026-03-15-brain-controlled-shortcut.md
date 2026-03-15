---
title: "Back to VS Code in a blink"
date: 2026-03-15
permalink: /posts/2026/03/brain-controlled-shortcut/
header:
  teaser: /images/blog-toybci/eog_dipole.png
tags:
  - BCI
  - EEG
  - deep learning
  - Python
---

I keep losing my VS Code window. My desktop and browser is always a mess with way too many tabs open because I want to quickly look something up, and I am also guilty of having way too many desktops on my macbook open. Alt-tabbing through everything is annoying. What if I could just... blink my way back?

Good thing I have a Muse S Athena headband sitting on my desk.


## Why blinks show up in EEG

EEG headbands like the Muse S Athena are designed to measure brain activity, but they also pick up a lot of other electrical signals. In clinical and basic neuroscience research, these are actually consideres **artifacts** and researchers, including myself, spend considerable effort removing them. One of the strongest artifact sources are the eyes.

The eyes acts as electrical dipoles: the cornea is positively charged relative to the retina. Every time you blink, your eyelid sweeps across this dipole and briefly rotates it, producing a large voltage deflection that propagates across the scalp. Frontal electrodes (like AF7 and AF8 on the Muse) pick up the strongest signal, while temporal electrodes (TP9, TP10) see a weaker version.

<figure>
  <img src="/images/blog-toybci/eog_dipole.png" alt="Blink artifact waveform">
  <figcaption>Average blink artifact waveform recorded from the Muse S. The frontal electrode (AF7, blue) picks up a large ~125 uV deflection peaking around 100 ms after blink onset, while the temporal electrode (TP10, orange) shows a much weaker response (~35 uV).</figcaption>
</figure>

This is normally a nuisance, but for a BCI, it's a feature. In fact, many modern consumer BCI applications rely heavily on minimally processed EEG that is dominated by peripheral signals: blinks, jaw clenches, heartbeat artifacts, eye movements. From a pure engineering perspective, these signals are incredibly useful. Eye tracking paths alone can reconstruct visual scenes surprisingly well. The downside is that this leads to many claims of "decoding brain activity" when the system is really picking up a mix of peripheral signals. For this project, I'm being explicit about it: we're detecting blinks, not reading minds. Blink artifacts are large (~100-200 uV, compared to ~10-50 uV for actual brain signals), consistent, and trivially voluntary. The trick is distinguishing *intentional* blink sequences from normal blinking.

## The idea

Instead of detecting single blinks (which would trigger constantly), I defined a **blink sequence**: 4 rapid blinks with a distinct temporal sequence in about 1 second. This is distinctive enough that it basically never happens during normal activity, but easy enough to do on purpose. The end-to-end system needs to:

1. Stream EEG from the Muse S Athena in real time
2. Detect blink sequences with low latency
3. Trigger a macOS command (bring VS Code to front)

## Connecting the Muse S Athena on macOS

This turned out to be the hardest part. The Muse S Athena (the latest hardware revision) uses a different BLE GATT profile than older Muses. It multiplexes all sensor data through fewer characteristics (`273e0013` instead of the traditional `273e0003-0007`). Neither BrainFlow nor muselsl support this.

I ended up using `bleak` (a Python BLE library) directly, building on the protocol reverse-engineered by the [amused-py](https://github.com/Amused-EEG/amused-py) project. The initialization sequence looks really weird...

```
v6 -> s -> h -> p21 -> s -> dc001 -> L1 -> h -> p1034 -> s -> dc001 -> L1
```

The EEG data comes as 14-bit packed samples (not 12-bit like older Muses) at 256 Hz across 4 channels.

## Recording training data

I wrote a quick prompter in [PsychoPy](https://www.psychopy.org/) (nostalgic, since I programmed my first psychophysics experiment in PsychoPy during my bachelor's lol) that cues the target blink sequence at jittered intervals. I recorded 6 sessions (~460 seconds of data total, ~90 blink sequences). The EEG streams directly into CSV files alongside event markers.

The challenge is that the prompt only tells you *approximately* when the blinks happened. The exact onset and offset times of each blink within the sequence are unknown, which is why I needed a post hoc labeling strategy.

## Labeling

Using the prompt timestamps as a rough guide, I search for blink peaks in a window after each cue. The labeling algorithm:

1. Bandpass filter the frontal channels (1-15 Hz) to isolate the blink waveform
2. Find peaks using the known inter-blink interval (IBI) distribution (~200-500ms)
3. Score candidate peak combinations by how well they match the expected rhythm
4. Mark the region from first blink onset to last blink offset as positive

<figure>
  <img src="/images/blog-toybci/blink_trial_example.png" alt="Example blink sequence trial">
  <figcaption>Example trial showing a 4-blink sequence across all four Muse channels (TP9, AF7, AF8, TP10). The green shaded region marks the ground truth label. The bottom row shows the binary label (1 during the blink sequence, 0 elsewhere).</figcaption>
</figure>

The result is a binary label for every EEG sample: 1 during a blink sequence, 0 everywhere else. About 8-10% of the data is positive.

## The model

I wanted something tiny and streaming-capable. The model sees a 5-second buffer of EEG and outputs predictions at 16 Hz (one prediction every 62.5ms):

<figure>
  <img src="/images/blog-toybci/architecture.png" alt="StreamingBlinkNet architecture">
  <figcaption>StreamingBlinkNet architecture (68K parameters). Three Conv1D blocks progressively downsample the 4-channel, 5-second EEG input (4x1280) to 64x160 features, followed by a bidirectional GRU and a linear head outputting per-timestep blink probabilities at 16 Hz (1x80).</figcaption>
</figure>

**StreamingBlinkNet**, 68K parameters:
- **3x Conv1D blocks** (kernel sizes 7, 5, 3) with batch norm, ReLU, and max pooling. This downsamples 256 Hz input to 16 Hz features while extracting local temporal patterns.
- **Bidirectional GRU** (64 hidden units) captures longer temporal context across the full 5-second window. Bidirectional because we're okay with ~2 second latency, so the model can look slightly into the "future" relative to the detection window.
- **Linear head** maps to per-timestep blink probability.

The model is intentionally acausal with about 2 seconds of latency. This is a lot, and you could definitely build something much faster with a causal architecture or just a shorter look ahead. But for triggering a keyboard shortcut in this little example, 2 seconds is fine, and letting the model see the complete blink sequence before making a decision dramatically reduces false positives.

## Training

With only 6 sessions of data, training takes about 30 seconds on an M4 Mac. I used a session-wise train/val split with one full session held out for validation.

<figure>
  <img src="/images/blog-toybci/training_curves.png" alt="Training curves">
  <figcaption>Training curves over 25 epochs. Left: BCE loss for train (blue) and validation (orange) sets. Center: validation F1 score reaching 0.986 (green dot marks best checkpoint). Right: validation AUROC saturating at 0.999.</figcaption>
</figure>

The model converges fast and hits **F1 = 0.986** on the held-out session with **AUROC = 0.999**. Early stopping kicks in around epoch 25. The class imbalance (~10% positive) is handled with a pos_weight of ~12x in the BCE loss.

Key training details:
- Adam optimizer, lr=1e-3 with cosine annealing
- Augmentations: Gaussian noise (2-5 uV), amplitude scaling (0.8-1.2x), channel dropout (10%), time shift (+/-50ms)
- Batch size 32, early stopping patience 15

## Does it actually work?

Here's the model's predictions on the held-out validation session, plotted against ground truth:

<figure>
  <img src="/images/blog-toybci/predictions_overlay.png" alt="Model predictions on held-out session">
  <figcaption>Model predictions on a 30-second segment from the held-out validation session. Top: raw 4-channel EEG with blink artifacts visible as large deflections. Middle: ground truth binary label. Bottom: model output probability (blue) with detection threshold (red dashed line). The model produces a sharp, well-localized peak with no false positives.</figcaption>
</figure>

The model cleanly detects blink sequences with sharp probability peaks that closely match the ground truth labels. No false positives in this segment.

## Real-time inference

The real-time system:
1. Connects to the Muse S via BLE
2. Fills a rolling 5-second buffer with incoming EEG
3. Runs inference every 80ms (~12 Hz)
4. Checks the detection window (1.5-3.0s from buffer end) for sustained high probability
5. On detection: plays a sound + runs `osascript -e 'tell application "Visual Studio Code" to activate'`

There's a 3-second cooldown between detections to prevent double-triggers.

The whole thing runs as a tiny always-on-top window: just a status dot, a probability bar, and "Idle" / "Detected".

<figure>
  <video autoplay loop muted playsinline style="width: 100%; border-radius: 6px;">
    <source src="/images/blog-toybci/blinkdemo.mp4" type="video/mp4">
  </video>
  <figcaption>The system in action. Four rapid blinks trigger VS Code to come to the foreground.</figcaption>
</figure>

## Takeaways

The entire project, from first BLE packet to working brain-controlled shortcut, took an afternoon. The model is 68K parameters and trains in 30 seconds. The inference runs comfortably at 12 Hz on CPU.

It's worth noting that this is a personalized model, trained and tested on my own data. One of the hardest problems in deep learning for (neural/bio) signals is cross-participant generalization, which requires much more data and far more sophisticated modeling approaches and augmentation strategies. This is a really simple problem: the signal is strong, the task is binary, and the model only needs to work for one person. That's why a tiny CNN + GRU with 3 minutes of training data gets you to 0.986 F1.

Still, it's incredibly cool to me that you can build something like this in an afternoon. Connecting a consumer EEG headband to a real-time neural network that actually controls your computer is just so much fun to play around with!!
