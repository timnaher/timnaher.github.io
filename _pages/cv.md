---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

[Download CV (docx)](/files/TimNaherCV.docx){: .btn .btn--primary}

Education
======
* **Ph.D. in Systems Neuroscience**, Radboud University, Nijmegen, Netherlands, 2021 -- 2026
  * Thesis: *Latent Structure in Brain and Behavior: From Geometric Brain State Decoding to Saccadic Rhythmicity*
  * MPI for Biological Cybernetics (Tübingen) & Ernst Strüngmann Institute (Frankfurt). Supervisor: Pascal Fries.
* **M.Sc. Research Master -- Clinical and Cognitive Neuroscience**, Maastricht University, Netherlands, 2019 -- 2021
  * *summa cum laude*. WSUM Master Thesis Prize (best thesis, Faculty of Neuroscience & Psychology 2021).
* **B.Sc. Psychology**, Trier University, Germany / Troy University, AL, 2013 -- 2019

Work Experience
======
* **Jun 2026 -- Present: Research Scientist, Peripheral Neural Interfaces**
  * Meta Reality Labs, New York, NY
  * Develop and deploy real-time deep learning models for peripheral neural interface decoding, enabling sub-millisecond latency input for next-generation wearable human--computer interaction.
  * Design on-device inference pipelines for continuous neural signal processing, optimizing model architectures for throughput and power constraints in wearable form factors.

* **May 2025 -- Nov 2025: Research Scientist Intern**
  * Meta Reality Labs, New York, NY
  * Developed novel deep learning architectures for real-time neuromotor decoding from EMG, advancing state-of-the-art accuracy for wearable neural input.
  * Re-engineered modeling infrastructure and experimental pipelines, cutting iteration cycles and enabling rapid architecture search.

* **Jul 2024 -- May 2025: PhD Researcher & Co-Founder (Biofeedback Startup)**
  * Max Planck Institute for Biological Cybernetics, Tübingen, Germany
  * ML pipelines using Riemannian geometry on fNIRS covariance manifolds for awareness detection in disorders of consciousness. Won the IEEE Brain Discovery Workshop Poster Prize (1st place).
  * Built a causal ResidualEncoder + LSTMDecoder model for real-time EEG sleep oscillation detection.

* **Oct 2021 -- Jul 2024: PhD Researcher**
  * Ernst Strüngmann Institute for Neuroscience, Frankfurt, Germany
  * Gamma renewal process model with rate-rescaling for inter-saccade interval rhythmicity across three primate species.
  * Vector-field decompositions of large-scale macaque ECoG arrays.
  * cTBS-over-rFEF experiments with concurrent EEG; stochastic differential equation framework for saccade timing.

* **Nov 2022 -- Nov 2023: Statistics Instructor**
  * Goethe University Frankfurt, Interdisciplinary M.Sc. Neuroscience
  * Taught statistical modeling, data analysis, and interpretation; developed interactive Python and MATLAB course materials.

Skills
======
* **Neural Decoding & Real-Time Models:** Causal CNNs, LSTMs, sequence-to-sequence, Viterbi/CRF decoding, on-device inference, EMG neuromotor decoding, EEG sleep oscillation detection, closed-loop neurostimulation
* **Machine Learning & Deep Learning:** PyTorch, PyTorch Lightning, Transformers, contrastive/self-supervised learning, foundation-model pretraining, transfer learning, CNNs, RNNs, GANs, autoencoders, Bayesian optimization
* **Neuroimaging & Signal Processing:** Riemannian geometry (pyRiemann/SPDLearn), EEG/fNIRS/EMG/ECoG pipelines, time-frequency analysis, connectivity analysis, aperiodic spectral decomposition, MNE-Python
* **Sensor Fusion & Wearables:** Multimodal integration (EMG, EEG, fNIRS, eye tracking, HRV, respiration, motion), real-time biofeedback systems, wearable neural interface prototyping
* **Infrastructure:** Python, MATLAB, JavaScript, SQL, Linux/HPC (SLURM), Git, Docker

Awards
======
* **IEEE Brain Discovery & Neurotechnology Workshop Poster Prize** -- 1st place, ML & Computer Paradigms (Riemannian geometry for fNIRS-BCIs), 2024
* **IEEE SMC BR4IN.IO BCI Hackathon** -- Closed-loop wearable EEG BCI for panic attack detection with tVNS biofeedback, 2024
* **WSUM Master Thesis Prize** -- Best thesis, Faculty of Neuroscience & Psychology, Maastricht University, 2021

Publications
======
  <ul>{% for post in site.publications reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>
