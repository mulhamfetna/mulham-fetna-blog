---
title: "Technical Projects Portfolio"
description: "Comprehensive portfolio of technical projects in AI, robotics, and embedded systems."
showDate: false
showAuthor: false
showTableOfContents: true
---

# Technical Projects Portfolio

> Showcase of technical projects spanning AI/ML, robotics, embedded systems, and data engineering.

---

## Research Projects

### 1. UAV Distributed MPC Control

| Field | Details |
|-------|---------|
| **Type** | Robotics & Control Systems |
| **Status** | Submitted to Elsevier Robotics and Autonomous Systems |
| **Manuscript ID** | ROBOT-D-26-01147 |
| **Benchmark** | 100% success rate (7/7 scenarios) |

**Description:** Distributed Model Predictive Control for multi-UAV quadrotor formation with consensus-based coordination and geometric SO(3) attitude control.

**Key Features:**
- Local MPC solver with formation constraints
- Consensus protocols (ring/mesh/star topology)
- Geometric SO(3) controller with quaternion representation
- Formation planner (grid, line, circle, wedge)

**Technical Stack:** Python, NumPy, ROS/Gazebo

---

### 2. Modernized Bees Algorithm

| Field | Details |
|-------|---------|
| **Type** | Swarm Intelligence & Optimization |
| **Status** | Submitted to Elsevier Applied Soft Computing |
| **Manuscript ID** | ASOC-D-26-06746 |
| **Success Rate** | 100% across 6 scenarios |

**Description:** Modernized Bees Algorithm for dynamic robot path planning in environments with moving obstacles.

**Key Features:**
- Adaptive parameter tuning
- Multi-objective optimization
- Dynamic obstacle handling
- ROS/Gazebo integration

**GitHub:** [swarm-path-planning-bees](https://github.com/molhamfetnah/swarm-path-planning-bees)

---

### 3. Hybrid Inverse Kinematics Ensemble

| Field | Details |
|-------|---------|
| **Type** | Robotics & Machine Learning |
| **Status** | Submitted to IEEE RA-L |
| **Submission** | 26-2479 |
| **Success Rate** | 100% on random targets |

**Description:** Hybrid ensemble combining Damped Least Squares with neural network solver and learned uncertainty estimation.

**Key Features:**
- Multiple IK solver strategies
- Uncertainty-based weighting
- 6-DOF UR5-like manipulator
- 5ms average solve time

**Collaboration:** Luca Ricci (University of Tuscia, Italy)

---

## AI/ML Projects

### 4. Sentiment Analysis Dashboard

| Field | Details |
|-------|---------|
| **Type** | Data Engineering & AI |
| **Year** | 2024 |
| **Stack** | Python, Redis, Dask, Dash |

**Description:** Scalable hybrid dashboard combining local GPU acceleration with cloud NLP models for Arabic sentiment analysis.

**Technologies:**
- Redis for caching
- Dask for parallel processing
- Torch for model inference
- Dash/Plotly for visualization
- Arabert, pyarabic for Arabic NLP

---

### 5. Edge AI Emotion Detection

| Field | Details |
|-------|---------|
| **Type** | Edge AI & Computer Vision |
| **Year** | 2024 |
| **Deployment** | Raspberry Pi 5 |

**Description:** Real-time facial emotion recognition deployed on edge devices for robotics control.

**Technologies:**
- PyTorch, TensorFlow
- CNN models (FER-2013 dataset)
- Raspberry Pi 5 optimization
- Low-latency inference pipeline

---

### 6. Local LLM Deployment with RAG

| Field | Details |
|-------|---------|
| **Type** | Local AI & MLOps |
| **Year** | 2024 |
| **Stack** | Ollama, LLaMA, FastAPI, Docker |

**Description:** Private local chat system using Retrieval Augmented Generation for domain-specific queries.

**Features:**
- On-premise fine-tuning
- Secure Docker deployment
- FastAPI REST endpoints
- API authentication

---

## Embedded Systems Projects

### 7. P10 DMD Display Systems

| Field | Details |
|-------|---------|
| **Type** | Embedded Systems |
| **Company** | Ala'a Screens Company |
| **Period** | 2022-2023 |

**Projects Delivered:**
- Basketball stadium clock/counter system
- Interactive school bus display system

**Technologies:**
- Atmega8 microcontrollers
- C programming
- Circuit design
- PCB layout (Proteus, EasyEDA)

---

### 8. N8N Automation Server

| Field | Details |
|-------|---------|
| **Type** | DevOps & Automation |
| **Year** | 2024 |
| **Stack** | Docker, N8N, Twingate |

**Description:** Private N8N automation server with secure remote access for serverless workflows.

---

## Open Source Projects

### 9. opencode-presentations-skill

| Field | Details |
|-------|---------|
| **Type** | Open Source Tool |
| **Version** | 2.0.0 |
| **Platform** | OpenCode |

**Description:** Enhanced Marp-based presentation generator for OpenCode with 20 design styles.

**Features:**
- 20 CSS design styles
- 5 slide sizes
- AI image generation
- Live preview
- Full test suite

**GitHub:** [opencode-presentations-skill](https://github.com/molhamfetnah/opencode-presentations-skill)

---

## Project Statistics

| Category | Count |
|----------|-------|
| Research Papers | 3 |
| AI/ML Projects | 3 |
| Embedded Systems | 2 |
| Open Source Tools | 1 |
| **Total** | **9** |

---

## Technical Skills Used

- Python, C++, C
- PyTorch, TensorFlow, OpenCV
- ROS, Gazebo, ROS2
- Docker, N8N, FastAPI
- Redis, Dask, MongoDB
- KiCad, EasyEDA, Proteus
- LaTeX, Markdown