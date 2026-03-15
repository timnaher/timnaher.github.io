---
permalink: /
title: "About"
author_profile: true
redirect_from:
  - /about/
  - /about.html
---

I am a NeuroAI researcher currently finishing my PhD in Systems Neuroscience at [Radboud University](https://www.ru.nl/en), supervised by [Prof. Pascal Fries](https://www.kyb.tuebingen.mpg.de/person/59524/2549) at the [Max Planck Institute for Biological Cybernetics](https://www.kyb.tuebingen.mpg.de/en) (Tübingen) and the [Ernst Strüngmann Institute](https://www.esi-frankfurt.de/) (Frankfurt). My thesis, *Latent Structure in Brain and Behavior: From Geometric Brain State Decoding to Saccadic Rhythmicity*, combined Riemannian geometry, causal deep learning, and stochastic modeling to decode brain states and characterize neural dynamics across species.

After my internship last year, I will join [Meta](https://about.meta.com/realitylabs/) in June 2026 as a Research Scientist for peripheral neural interfaces in New York, where I will develop real-time deep learning models for peripheral neural interface decoding.


## Research Interests

- Neural decoding & real-time models
- Deep learning for neuroscience
- Multimodal Brain-computer interfaces (fNIRS, EEG, EMG)
- Riemannian geometry for neural data
- Saccade dynamics & visual neuroscience
- Sleep, memory & closed-loop neurostimulation

## Featured Projects

<style>
  .featured-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 1.2em;
    margin-top: 0.8em;
  }
  .featured-card {
    border: 1px solid #e0e0e0;
    border-radius: 10px;
    overflow: hidden;
    transition: transform 0.2s, box-shadow 0.2s;
    background: #fff;
  }
  .featured-card:hover {
    transform: translateY(-3px);
    box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  }
  .featured-card a {
    text-decoration: none;
    color: inherit;
    display: block;
  }
  .featured-card img {
    width: 100%;
    height: 160px;
    object-fit: cover;
    border-bottom: 1px solid #eee;
  }
  .featured-card-body {
    padding: 0.9em 1.1em;
  }
  .featured-card-body h3 {
    margin: 0 0 0.3em 0;
    font-size: 0.95em;
  }
  .featured-card-body p {
    color: #555;
    font-size: 0.82em;
    line-height: 1.5;
    margin: 0 0 0.6em 0;
  }
  .feat-tags {
    display: flex;
    flex-wrap: wrap;
    gap: 0.3em;
  }
  .feat-tag {
    background: #f0f0f0;
    color: #555;
    padding: 0.15em 0.5em;
    border-radius: 4px;
    font-size: 0.72em;
    font-weight: 500;
  }
</style>

<div class="featured-grid">
  <div class="featured-card">
    <a href="/portfolio/portfolio-1-sodeep/">
      <img src="/images/sodeep-architecture.png" alt="SODEEP">
      <div class="featured-card-body">
        <h3>SODEEP: Real-Time Slow Oscillation Detection</h3>
        <p>Causal CNN (~100k params) achieves F1 of 0.967 with 39 ms latency, replacing classical methods.</p>
        <div class="feat-tags">
          <span class="feat-tag">PyTorch</span>
          <span class="feat-tag">Causal CNN</span>
          <span class="feat-tag">Real-Time</span>
        </div>
      </div>
    </a>
  </div>
  <div class="featured-card">
    <a href="/portfolio/portfolio-2-fnirs-method/">
      <img src="/images/fnirs-method-figure.png" alt="fNIRS Riemannian">
      <div class="featured-card-body">
        <h3>Riemannian Geometry Boosts fNIRS Classification</h3>
        <p>Block diagonal SPD covariance kernels for fNIRS. 89% two-class and 70% eight-class accuracy.</p>
        <div class="feat-tags">
          <span class="feat-tag">SPD Manifolds</span>
          <span class="feat-tag">pyRiemann</span>
          <span class="feat-tag">fNIRS</span>
        </div>
      </div>
    </a>
  </div>
  <div class="featured-card">
    <a href="/portfolio/portfolio-3-fnirs-clinical/">
      <img src="/images/fnirs-clinical-figure.png" alt="fNIRS Clinical">
      <div class="featured-card-body">
        <h3>fNIRS Awareness Detection in Disorders of Consciousness</h3>
        <p>Individualized Riemannian pipeline. 100% sensitivity and 89% specificity at bedside.</p>
        <div class="feat-tags">
          <span class="feat-tag">Clinical BCI</span>
          <span class="feat-tag">Riemannian</span>
          <span class="feat-tag">fNIRS</span>
        </div>
      </div>
    </a>
  </div>
  <div class="featured-card">
    <a href="/portfolio/portfolio-4-riemannian-infant/">
      <img src="/images/riemannian-figure.png" alt="Riemannian Infant">
      <div class="featured-card-body">
        <h3>Riemannian Manifold Analysis of Infant Memory</h3>
        <p>SPD covariance matrices reveal convergence of voice representations after sleep in 3-month-olds.</p>
        <div class="feat-tags">
          <span class="feat-tag">pyRiemann</span>
          <span class="feat-tag">Laplacian Eigenmaps</span>
          <span class="feat-tag">EEG</span>
        </div>
      </div>
    </a>
  </div>
</div>
