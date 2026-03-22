---
title: "Does the Muse S Athena's fNIRS actually improve brain decoding?"
date: 2026-03-22
permalink: /posts/2026/03/multimodal-bci-fnirs/
header:
  teaser: /images/blog-multimodal/multimodal_2_strategies.png
tags:
  - BCI
  - EEG
  - fNIRS
  - Riemannian geometry
  - Python
---

The new [Muse S Athena](https://choosemuse.com/) wearable EEG headband is now multimodal! Besides EEG it also includes optical sensors to measure fNIRS. This is a pretty big deal, as we are entering an era where consumer wearables ship with truly multimodal brain sensing out of the box. And there's good reason for excitement: EEG and fNIRS measure fundamentally complementary aspects of brain activity. EEG captures fast, synchronous neural oscillations on a millisecond timescale, while fNIRS tracks the slower hemodynamic response, which are changes in blood oxygenation that follow neural activation with a delay of several seconds. These two signals are linked through neurovascular coupling but sensitive to different physiology and different artifacts, making them natural candidates for fusion ([Hong & Khan, 2017](https://www.frontiersin.org/journals/human-neuroscience/articles/10.3389/fnhum.2017.00503/full); [Chiarelli et al., 2022](https://pmc.ncbi.nlm.nih.gov/articles/PMC9371171/)).

Having a Muse S Athena at home, I wanted to test if this new modality actually adds benefits for simple BCI applications.

## The setup

The Muse S Athena includes 4 EEG electrodes (TP9, AF7, AF8, TP10 at 256 Hz) and 8 optical channels (4 long-distance fNIRS + 4 short-distance PPG at 64 Hz). After preprocessing the optics through the modified Beer-Lambert law and short-channel regression, I get 2 channels of oxygenated hemoglobin (**HbO**) and 2 channels of deoxygenated hemoglobin (**HbR**).

Since the Muse S covers frontal cortex, I chose a task contrast that targets exactly that region: **mental arithmetic vs. rest**. This is a classic cognitive load paradigm that has been shown to produce robust prefrontal activation, something we've previously shown can be decoded well even in multi-class settings using full research-grade fNIRS setups and Riemannian geometry classifiers ([Näher et al., 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12523035/)). The question is whether a consumer device with just 2 fNIRS channels can capture enough of that signal to actually help the decoding.

I chose to implement a simple data collection strategy: A blocked design, of randomly interleaved trials with either 10 seconds or REST or 10 seconds of CALCULATE with a large and jittered ITI to allow the fNIRS to return to baseline between trials. During the CALCULATE trials, a three-digit number appears on screen and I mentally subtract 7 repeatedly for 10 seconds (487, 480, 473, ...). In REST trials, I stare at a fixation cross for the same duration. I recorded 80 trials across multiple sessions (balanced classes). 


The question: **does adding hemodynamic data from fNIRS improve classification beyond what EEG alone can do?**

## Why this matters

Hybrid EEG-fNIRS systems have been shown to improve classification accuracy by around 5% on average compared to either modality alone ([Fazli et al., 2012](https://www.sciencedirect.com/science/article/pii/S1053811911008792)). However, this number typically comes from research-grade setups with dense optode arrays covering large cortical areas. A consumer headband with 2 fNIRS channels covering a small frontal patch is a very different story. More sensors don't automatically mean better decoding, each additional modality adds dimensionality, noise, and estimation cost. If the fNIRS channels don't carry information that EEG is missing, they make the Muse S Athena more of a gimmick than actually a true useful multimodal brain wearable.

## Riemannian geometry classifiers

As a classifier, I used [pyriemann](https://pyriemann.readthedocs.io/) to classify covariance and kernel matrices. The key insight is that covariance/kernel matrices live on a curved manifold, not in flat Euclidean space, and respecting this geometry gives better classification, especially with small datasets like mine.

The EEG signal is bandpass-filtered into **theta** (4-8 Hz), **alpha** (8-13 Hz), and **beta** (13-30 Hz). Each band produces a 4x4 covariance matrix. For fNIRS, I split HbO and HbR into separate covariance blocks (2 channels each) rather than combining them into one matrix. This follows the approach we introduced in [Näher et al. (2025)](https://pmc.ncbi.nlm.nih.gov/articles/PMC12523035/), where we showed that HbO and HbR contain different information structures and benefit from independent optimization of kernel selection and regularization parameters.

Both EEG and fNIRS blocks are searched over the same space of covariance estimators (SCM, Ledoit-Wolf, OAS) and kernel methods (RBF, linear), each with optional shrinkage regularization. All matrices are trace-normalized before fusion so that modalities with different signal magnitudes (EEG in µV, fNIRS in µM) contribute equally.

## Three fusion strategies

How you combine modalities matters as much as *what* you combine. I tested three strategies:

**Strategy A: Block EEG + block fNIRS.** Each EEG band (4x4) and each fNIRS component (2x2) gets its own covariance block, assembled into a 16x16 block-diagonal matrix. This assumes the modalities are independent, no cross-modal correlations.

**Strategy B: Full EEG + block fNIRS.** All 12 EEG channels (3 bands × 4 channels) get one full 12x12 covariance matrix that captures cross-band relationships within EEG. The fNIRS components are added as separate 2x2 blocks. This is a hybrid: cross-band EEG correlations yes, cross-modal correlations no.

**Strategy C: Full covariance.** All 16 channels (12 EEG + 2 HbO + 2 HbR) go into one 16x16 covariance matrix. This captures cross-modal correlations, e.g., whether alpha power and HbO concentration co-vary differently during arithmetic vs rest. If EEG-fNIRS coupling carries discriminative information, this strategy should win.

## Results

<figure>
  <img src="/images/blog-multimodal/multimodal_1_all_conditions.png" alt="All conditions ranked by accuracy">
  <figcaption>All five conditions ranked by stratified 5-fold cross-validation accuracy. Block EEG (4+4+4) achieves the highest accuracy at 68%. Color indicates fusion strategy; hatched bars include fNIRS. None of the fNIRS strategies surpass EEG-only baselines.</figcaption>
</figure>

Here's the punchline: In this setup and paradigm **fNIRS adds nothing.** Despite the promising multimodal setup, not a single fusion strategy beats plain EEG.

- **Block EEG (68%)** wins overall, treating each frequency band as an independent covariance block works best
- **Strategy B (67%)** is the best fNIRS approach, closely matching the full EEG baseline but not exceeding it. It uses RBF kernels for both EEG and fNIRS blocks
- **Strategy C (64%)**, the full cross-modal covariance, actually *hurts*. Cross-modal EEG×fNIRS correlations likely add noise, not signal
- **Strategy A (62%)**, all block-diagonal, is the worst fNIRS approach

## Statistical testing

I used a permutation test (1,000 permutations) to assess whether the differences are significant. Under the null hypothesis, each fold's accuracy difference is equally likely to be positive or negative, and we permute the signs to build a null distribution.

<figure>
  <img src="/images/blog-multimodal/multimodal_3_stats.png" alt="Permutation test results">
  <figcaption>Accuracy difference (in percentage points) when adding fNIRS to EEG, with permutation test p-values. All three bars are negative or zero, fNIRS consistently fails to help. None reach significance (p < 0.05).</figcaption>
</figure>

None of the three comparisons reach significance (all p > 0.5). The direction is consistently negative or zero.

## Why doesn't fNIRS help?

A few factors likely explain this:

**Spatial resolution.** The Muse S has only 2 long-distance fNIRS channels (left and right forehead). Research-grade fNIRS systems use 20-50+ channels with dense source-detector grids. With just 2 channels, there's not enough spatial information to capture the distributed hemodynamic pattern of cognitive load.

**EEG already captures the signal.** Mental arithmetic modulates alpha and theta power strongly and reliably. On a 4-channel consumer EEG, this spectral signature is already sufficient, the fNIRS doesn't carry complementary information that EEG is missing.

**Curse of dimensionality.** Adding fNIRS channels increases the covariance matrix size without proportionally adding discriminative features. With only 80 trials, the larger matrices become harder to estimate reliably.

## What I learned

**More sensors are not necessarily better.** The engineering intuition that more data means better classification breaks down when the additional modality doesn't carry complementary information relative to its dimensionality cost.

**Per-modality kernel tuning matters.** An interesting finding: EEG alpha performed best with RBF kernels (`ker:rbf+shr0.01`), not standard covariance estimators. fNIRS also benefited from kernel methods. Using the same estimator for both would miss this. I wrote about the per-block kernel tuning approach in my [pyriemann fNIRS example](https://pyriemann.readthedocs.io/en/latest/auto_examples/fnirs/plot_classif_fnirs.html).

**Trace normalization is essential** for multimodal fusion. Without it, the modality with larger absolute values dominates the block-diagonal matrix, making the other modality invisible.

**Full cross-modal covariance hurts.** Strategy C, computing one big covariance over all channels, performed worst. The cross-modal EEG×fNIRS correlations are noisy on a consumer device and add dimensionality without adding signal. Block-diagonal fusion (keeping modalities separate) is more robust.

## The honest takeaway

On a consumer-grade Muse S Athena with 4 EEG channels and 2 fNIRS channels, adding fNIRS does not improve mental arithmetic vs rest classification. This doesn't mean multimodal sensing is useless, or that the Muse S fNIRS sensors are useless. This conclusion is limited to my specific setup, task, paradigm and maybe also to my brain. With more fNIRS channels,  or tasks where EEG is weaker, the answer might be different. But for this combination of device, task, and data size, the 4-channel EEG already captures everything the classifier needs. I still think its cool that companies like Muse start to think more in the multimodal sense for wearable. Ultimately I believe that each modality has certain advantages and in my opinion, an ideal BCI/HCI setup leverages all these different strengths to build the best possible application, without stressing too much about only using brain data.

