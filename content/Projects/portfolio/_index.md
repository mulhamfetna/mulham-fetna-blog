---
title: "Technical Portfolio"
description: "Engineering and research projects by Mulham Fetna — CNN fault diagnosis on real motor data, merged OpenCV contributions, WiFi-radar SLAM research, and shipped open-source tools."
keywords: ["Mulham Fetna projects", "PMSM fault diagnosis CNN", "OpenCV contributor", "WiFi radar SLAM", "robotics portfolio Syria"]
showDate: false
showAuthor: false
showTableOfContents: true
---

Every project here is public, and every claim links to the code. Where a result is weak or a project is early-stage, it says so.

---

## Fault Diagnosis in PMSM Motors — CNN on Wavelet Scalograms

**[github.com/mulhamfetna/pmsm-fault-diagnosis-cnn-scalogram](https://github.com/mulhamfetna/pmsm-fault-diagnosis-cnn-scalogram)** · Python · MIT

Detecting inter-turn stator faults in permanent-magnet synchronous motors by converting raw sensor signals into continuous-wavelet scalograms and classifying them with a CNN.

Trained on the **real KAIST motor dataset** (DOI `10.17632/rgn5brrgrn.5`) — not synthetic data.

**Results, reported honestly:**

| Sensor channel | Balanced accuracy |
|---|---|
| Vibration @ 25.6 kHz | **1.00** |
| Current @ 100 kHz | **0.69** |

The vibration result is excellent. The current-channel result is not, and the repository says so. The dataset contains only four healthy recordings, so the perfect vibration score may not generalize — that caveat is in the README, not buried in a footnote.

38 unit tests. This is the project I would want to be judged on: real data, a strong result, a weak result, and no hiding of either.

---

## Merged Open-Source Contributions — OpenCV & OpenDR

Three pull requests accepted into major open-source projects. Two landed in **OpenCV core**, one of the most widely used computer-vision libraries in the world.

| Pull request | Project | What it fixed |
|---|---|---|
| [opencv#28935](https://github.com/opencv/opencv/pull/28935) | OpenCV (core) | Unicode temp-path handling on Windows |
| [opencv#28880](https://github.com/opencv/opencv/pull/28880) | OpenCV (highgui/Qt) | UTF-8 window names preserved in fallback paths |
| [opendr#522](https://github.com/opendr-eu/opendr/pull/522) | OpenDR | Python 3.12 compatibility |

Both OpenCV fixes concern **non-ASCII path and text handling** — the class of bug that quietly breaks the library for everyone outside the English-speaking world, and that is easy to miss if you never work in Arabic.

Contributing to a library used by millions means most submissions are rejected. I have opened {{< fact key="oss_submitted" >}} pull requests across OpenCV, opencv-python, Ultralytics, and OpenDR; these three were accepted.

---

## WiFi as a Radar Replacement for Automotive SLAM

**[github.com/mulhamfetna/wifi-radar-slam](https://github.com/mulhamfetna/wifi-radar-slam)** · Python · AGPL-3.0
Archived on Zenodo — DOI [10.5281/zenodo.21247288](https://doi.org/10.5281/zenodo.21247288)

Can ambient WiFi substitute for radar in automotive localization and mapping? A simulation-first feasibility study using Sionna ray tracing, testing whether the channel-state information available from commodity WiFi carries enough structure for SLAM.

**Status: active research, feasibility stage.** The literature review and simulation infrastructure are complete; there are **no results yet**. I publish it as it develops rather than after the fact.

---

## Trading Strategy Finder

**[github.com/mulhamfetna/trading-strategy-finder](https://github.com/mulhamfetna/trading-strategy-finder)** · Python · MIT

A backtesting engine comparing scalping, day, and intraday strategies on NQ futures, with a machine-learning filter layer over signal generation.

Frozen at tag `v1.0.0` with reproducible results: **$633.65 net profit, 54.5% win rate, 2.62 profit factor.**

Modest, believable numbers on a small account — which is the point. The engineering (versioning, tests, reproducibility) is the deliverable here, not a claim of a trading edge.

---

## OpenCode Presentations

**[github.com/mulhamfetna/opencode-presentations-skill](https://github.com/mulhamfetna/opencode-presentations-skill)** · JavaScript · MIT · published on npm

A Marp-based presentation generator with 20 design styles and live preview. Small, finished, and actually shipped — a working tool anyone can install and use today.

---

## Geopolymer Design System

**[github.com/mulhamfetna/geopolymer-design-system](https://github.com/mulhamfetna/geopolymer-design-system)** · Jupyter · Apache-2.0 · archived on Zenodo with a DOI

A Gradio application pairing a materials database with a RandomForest predictor and a Nelder–Mead optimizer, to design geopolymer mixes for nuclear-waste adsorption.

**Caveat stated up front:** the model is trained on **synthetic data**, not laboratory measurements. It is a design-exploration tool, and explicitly not a validated substitute for experimental work.

---

## Embedded Systems — Ala'a Screens

Professional work, 2022–2023. Embedded systems design for P10 DMD LED display applications, including a basketball stadium scoreboard clock and a school-bus information display. Firmware and hardware, shipped and deployed.

---

## Research papers

Three robotics papers are **under peer review**: distributed model-predictive control for multi-UAV formation, a modernized Bees Algorithm for dynamic path planning, and hybrid inverse kinematics with learned uncertainty estimation.

They are submitted, not accepted, and their code is early-stage. See **[Research](/research/)** for detail — submission is not publication, and this site will not pretend otherwise.

---

**Code:** [github.com/mulhamfetna](https://github.com/mulhamfetna) · **Contact:** contact@mulhamfetna.com
