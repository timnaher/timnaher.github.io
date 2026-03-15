---
title: "I Built a Brain-Controlled Shortcut in an Afternoon"
date: 2026-03-15
permalink: /posts/2026/03/brain-controlled-shortcut/
tags:
  - BCI
  - EEG
  - deep learning
  - Python
---

I keep losing my VS Code window. I'll be deep in some code, switch to the browser to look something up, and suddenly I have 40 tabs open and no idea where my editor went. Alt-tabbing through everything is annoying. What if I could just... blink my way back?

Good thing I have a Muse S headband sitting on my desk.

![Pipeline](/images/blog-toybci/pipeline.png)

## Why blinks show up in EEG

EEG headbands like the Muse S are designed to measure brain activity — but they also pick up a lot of other electrical signals. In clinical and basic neuroscience research, these are called **artifacts** and researchers spend considerable effort removing them. One of the strongest artifact sources is the eye.

The eye acts as an electrical dipole: the cornea is positively charged relative to the retina. Every time you blink, your eyelid sweeps across this dipole and briefly rotates it, producing a large voltage deflection that propagates across the scalp. Frontal electrodes (like AF7 and AF8 on the Muse) pick up the strongest signal, while temporal electrodes (TP9, TP10) see a weaker version.

![Blink waveform](/images/blog-toybci/eog_dipole.png)

This is normally a nuisance — but for a BCI, it's a feature. Blink artifacts are large (~100-200 uV, compared to ~10-50 uV for brain signals), consistent, and trivially voluntary. The trick is distinguishing *intentional* blink sequences from normal blinking.

## The idea

Instead of detecting single blinks (which would trigger constantly), I defined a **blink sequence**: 4 rapid blinks in about 1 second. This is distinctive enough that it basically never happens during normal activity, but easy enough to do on purpose. The system needs to:

1. Stream EEG from the Muse S in real time
2. Detect blink sequences with low latency
3. Trigger a macOS command (bring VS Code to front)

## Connecting the Muse S Athena on macOS

This turned out to be the hardest part. The Muse S Athena (the latest hardware revision) uses a different BLE GATT profile than older Muses — it multiplexes all sensor data through fewer characteristics (`273e0013` instead of the traditional `273e0003-0007`). Neither BrainFlow nor muselsl support this.

I ended up using `bleak` (a Python BLE library) directly, with a very specific initialization sequence reverse-engineered from the device's protocol:

```
v6 -> s -> h -> p21 -> s -> dc001 -> L1 -> h -> p1034 -> s -> dc001 -> L1
```

The EEG data comes as 14-bit packed samples (not 12-bit like older Muses) at 256 Hz across 4 channels. Once I figured out the protocol, the connection is rock solid.

## Recording training data

I wrote a quick PsychoPy experiment that prompts me to blink:

- Random inter-trial interval (5-15s)
- Audio countdown: "3... 2... 1..."
- "NOW!" — do 4 rapid blinks
- Repeat for 15 trials

Each block takes about 3 minutes. I recorded 6 sessions (~460 seconds of data total, ~90 blink sequences). The EEG streams directly into CSV files alongside event markers.

## Labeling

Since I know exactly when the "NOW" prompt appeared, I can search for blink peaks in a window after each prompt. The labeling algorithm:

1. Bandpass filter the frontal channels (1-15 Hz) to isolate the blink waveform
2. Find peaks using the known inter-blink interval (IBI) distribution (~200-500ms)
3. Score candidate peak combinations by how well they match the expected rhythm
4. Mark the region from first blink onset to last blink offset as positive

![Example trial](/images/blog-toybci/blink_trial_example.png)

The result is a binary label for every EEG sample: 1 during a blink sequence, 0 everywhere else. About 8-10% of the data is positive.

## The model

I wanted something tiny and streaming-capable. The model sees a 5-second buffer of EEG and outputs predictions at 16 Hz (one prediction every 62.5ms):

![Architecture](/images/blog-toybci/architecture.png)

**StreamingBlinkNet** — 68K parameters:
- **3x Conv1D blocks** (kernel sizes 7, 5, 3) with batch norm, ReLU, and max pooling. This downsamples 256 Hz input to 16 Hz features while extracting local temporal patterns.
- **Bidirectional GRU** (64 hidden units) captures longer temporal context across the full 5-second window. Bidirectional because we're okay with ~2 second latency — the model can look slightly into the "future" relative to the detection window.
- **Linear head** maps to per-timestep blink probability.

The model is intentionally acausal with about 2 seconds of latency. This means it sees the *complete* blink sequence before making a decision, which dramatically reduces false positives. For triggering a keyboard shortcut, 2 seconds of delay is totally acceptable.

## Training

With only 6 sessions of data, training takes about 30 seconds on an M1 Mac (MPS backend). I used a session-wise train/val split — one full session held out for validation.

![Training curves](/images/blog-toybci/training_curves.png)

The model converges fast and hits **F1 = 0.986** on the held-out session with **AUROC = 0.999**. Early stopping kicks in around epoch 25. The class imbalance (~10% positive) is handled with a pos_weight of ~12x in the BCE loss.

Key training details:
- Adam optimizer, lr=1e-3 with cosine annealing
- Augmentations: Gaussian noise (2-5 uV), amplitude scaling (0.8-1.2x), channel dropout (10%), time shift (+/-50ms)
- Batch size 32, early stopping patience 15

## Does it actually work?

Here's the model's predictions on the held-out validation session, plotted against ground truth:

![Predictions](/images/blog-toybci/predictions_overlay.png)

The model cleanly detects blink sequences with sharp probability peaks that closely match the ground truth labels. No false positives in this segment.

## Real-time inference

The real-time system:
1. Connects to the Muse S via BLE
2. Fills a rolling 5-second buffer with incoming EEG
3. Runs inference every 80ms (~12 Hz)
4. Checks the detection window (1.5-3.0s from buffer end) for sustained high probability
5. On detection: plays a sound + runs `osascript -e 'tell application "Visual Studio Code" to activate'`

There's a 3-second cooldown between detections to prevent double-triggers.

The whole thing runs as a tiny always-on-top window — just a status dot, a probability bar, and "Idle" / "Detected".

## Takeaways

The entire project — from first BLE packet to working brain-controlled shortcut — took an afternoon. The model is 68K parameters and trains in 30 seconds. The inference runs comfortably at 12 Hz on CPU.

The key insight is that you don't need a sophisticated model when the signal is this strong. Blink artifacts are 10x larger than the brain signals that EEG headbands are designed to measure. A tiny CNN + GRU with 3 minutes of training data gets you to 0.986 F1.

What would make this better:
- **More commands**: Different blink patterns (2 blinks vs 4, left eye vs right eye) for multiple shortcuts
- **Adaptive thresholds**: Auto-calibrate to each user's blink amplitude
- **Online learning**: Fine-tune on false positives/negatives during use

The code is at [github.com/timnaher/toyBCI](https://github.com/timnaher/toyBCI).
