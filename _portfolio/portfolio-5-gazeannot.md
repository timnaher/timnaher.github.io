---
title: "GazeAnnot: Interactive Eye Movement Labeling & U'n'Eye Training"
excerpt: "PyQt6 desktop app for labeling saccades, microsaccades, and PSO in eye tracking data, with integrated U'n'Eye deep learning model training and evaluation."
collection: portfolio
order: 5
header:
  teaser: blog-gazeannot/logo.png
tags:
  - PyQt6
  - PyTorch
  - Eye Tracking
  - Deep Learning
---

![GazeAnnot Demo](/images/blog-gazeannot/demo.gif)

Interactive desktop application for labeling eye movement events (saccades, microsaccades, post-saccadic oscillations) and training [U'n'Eye](https://github.com/berenslab/uneye) deep learning models for automatic saccade detection.

Three integrated views: **Label** (interactive time series annotation with undo/redo), **Train** (full training or k-fold CV with live loss curves and per-class metrics), and **Evaluate** (predictions, error histograms, confusion browser). Supports CSV, MATLAB v7.3, and NumPy data formats with auto-detection of file pairs and existing labels.

Built with PyQt6, pyqtgraph, and PyTorch.

[GitHub Repository](https://github.com/timnaher/gazeannot) \| [Blog Post](/posts/2026/03/gazeannot/)
