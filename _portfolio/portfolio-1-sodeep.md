---
title: "SODEEP: A Lightweight Causal CNN for Real-Time Slow Oscillation Detection"
excerpt: "Causal CNN (~100k params) achieves F1 of 0.967 with 39 ms latency, replacing classical methods for closed-loop sleep stimulation."
collection: portfolio
order: 1
header:
  teaser: sodeep-architecture.png
tags:
  - PyTorch
  - Causal CNN
  - LSTM
  - Real-Time
---

![SODEEP Architecture](/images/sodeep-architecture.png)

A lightweight causal convolutional neural network for real-time detection of sleep slow oscillations in EEG signals. With only ~100k parameters, SODEEP achieves an F1 score of 0.967 at just 39 ms latency, making it suitable for closed-loop neurostimulation experiments targeting memory consolidation during sleep.

The model uses a ResidualEncoder + LSTMDecoder architecture with Viterbi/CRF decoding, focal loss, and event-specific evaluation metrics. It replaces classical threshold-based methods with a learned, causal approach that generalizes across subjects.

[GitHub Repository](https://github.com/timnaher/SODeep)
