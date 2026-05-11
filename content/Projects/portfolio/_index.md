---
title: "Projects Portfolio"
description: "Technical projects in AI/ML, Robotics, Embedded Systems, and Open Source."
showDate: false
showAuthor: false
showTableOfContents: true
---

# Projects Portfolio

> Comprehensive showcase of technical projects spanning research, AI/ML, embedded systems, and open source development.

---

## Quick Stats

| Category | Count | Status |
|----------|-------|--------|
| **Research Papers** | 3 | Submitted (2026) |
| **AI/ML Projects** | 3 | Completed |
| **Embedded Systems** | 2 | Deployed |
| **Open Source Tools** | 1 | Published |
| **Kaggle Competitions** | 2 | Active |
| **Total Projects** | **11** | — |

---

## 🔬 Research Projects

### Distributed Model Predictive Control for Multi-UAV Formation

| Attribute | Details |
|-----------|---------|
| **Domain** | Robotics & Control Systems |
| **Journal** | Robotics and Autonomous Systems (Elsevier) |
| **Manuscript** | ROBOT-D-26-01147 |
| **Status** | 📤 Submitted (May 10, 2026) |
| **Benchmark** | ✅ 100% (7/7 scenarios) |

**Overview:** Distributed MPC for multi-UAV quadrotor formation with consensus-based coordination and geometric SO(3) attitude control.

**Key Contributions:**
- Local MPC solver with formation constraints
- Consensus protocols (ring/mesh/star topology)
- Quaternion-based geometric controller
- Formation planner (grid, line, circle, wedge)

**Tech Stack:** Python, NumPy, ROS/Gazebo

---

### Modernized Bees Algorithm for Dynamic Path Planning

| Attribute | Details |
|-----------|---------|
| **Domain** | Swarm Intelligence & Optimization |
| **Journal** | Applied Soft Computing (Elsevier) |
| **Manuscript** | ASOC-D-26-06746 |
| **Status** | 📤 Submitted (May 6, 2026) |
| **Success Rate** | ✅ 100% (6 scenarios, 0.35s avg) |

**Overview:** Modernized Bees Algorithm for robot path planning in dynamic environments with moving obstacles.

**Key Contributions:**
- Adaptive parameter tuning
- Multi-objective optimization
- Dynamic obstacle handling
- ROS/Gazebo integration

**Code:** [GitHub](https://github.com/molhamfetnah/swarm-path-planning-bees)

---

### Hybrid Inverse Kinematics Ensemble with Uncertainty Estimation

| Attribute | Details |
|-----------|---------|
| **Domain** | Robotics & Machine Learning |
| **Journal** | IEEE Robotics and Automation Letters |
| **Submission** | 26-2479 |
| **Status** | 📤 Submitted (May 9, 2026) |
| **Success Rate** | ✅ 100% (random targets), 86.7% (overall) |

**Overview:** Hybrid ensemble combining Damped Least Squares with neural network solver and learned uncertainty estimation.

**Key Contributions:**
- Multiple IK solver strategies
- Uncertainty-based solver weighting
- 6-DOF UR5-like manipulator
- 5ms average solve time

**Collaboration:** Luca Ricci (University of Tuscia, Italy)

---

## 🤖 AI/ML Projects

### Sentiment Analysis Dashboard

| Attribute | Details |
|-----------|---------|
| **Type** | Data Engineering & AI |
| **Year** | 2024 |
| **Stack** | Python, Redis, Dask, Dash |

**Description:** Scalable hybrid dashboard combining local GPU acceleration with cloud NLP models for Arabic sentiment analysis.

**Architecture:**
- Redis for caching
- Dask for parallel processing
- Torch for model inference
- Dash/Plotly for visualization
- Arabert, pyarabic for Arabic NLP

---

### Edge AI Emotion Detection

| Attribute | Details |
|-----------|---------|
| **Type** | Edge AI & Computer Vision |
| **Year** | 2024 |
| **Deployment** | Raspberry Pi 5 |

**Description:** Real-time facial emotion recognition deployed on edge devices for robotics control.

**Stack:** PyTorch, TensorFlow, CNN (FER-2013 dataset), Raspberry Pi 5 optimization

---

### Local LLM Deployment with RAG

| Attribute | Details |
|-----------|---------|
| **Type** | Local AI & MLOps |
| **Year** | 2024 |
| **Stack** | Ollama, LLaMA, FastAPI, Docker |

**Description:** Private local chat system using Retrieval Augmented Generation for domain-specific queries.

**Features:** On-premise fine-tuning, Docker deployment, FastAPI REST endpoints, API authentication

---

## ⚡ Embedded Systems Projects

### P10 DMD Display Systems

| Attribute | Details |
|-----------|---------|
| **Company** | Ala'a Screens Company |
| **Period** | 2022-2023 |
| **Role** | Embedded Systems Designer |

**Deliverables:**
- 🏀 Basketball stadium clock/counter system
- 🚌 Interactive school bus display system

**Tech Stack:** Atmega8 microcontrollers, C programming, Circuit design, PCB layout (Proteus, EasyEDA)

---

### N8N Automation Server

| Attribute | Details |
|-----------|---------|
| **Type** | DevOps & Automation |
| **Year** | 2024 |
| **Stack** | Docker, N8N, Twingate |

**Description:** Private N8N automation server with secure remote access for serverless workflows.

---

## 🛠️ Open Source Projects

### opencode-presentations-skill

| Attribute | Details |
|-----------|---------|
| **Type** | Open Source Tool |
| **Version** | 2.0.0 |
| **Platform** | OpenCode |
| **License** | MIT |

**Description:** Enhanced Marp-based presentation generator for OpenCode with 20 design styles.

**Features:**
- 20 CSS design styles (Modern, Professional, Creative, Tech)
- 5 slide sizes (16:9, 4:3, 1:1, 9:16, 21:9)
- AI image generation (Pollinations.ai)
- Live preview with hot reload
- Full test suite (20 tests, 100% pass)

**Code:** [GitHub](https://github.com/molhamfetnah/opencode-presentations-skill)

---

## 🏆 Kaggle Competition Projects

### Kaggle Playground Series S6E4

| Attribute | Details |
|-----------|---------|
| **Type** | Competition Solution |
| **Status** | Model developed & deployed as API |

{{< github repo="molhamfetnah/kaggle-playground-series-s6e4" >}}

---

### AI Cup 2026 Bird Classification

| Attribute | Details |
|-----------|---------|
| **Type** | Competition Solution |
| **Status** | Active development |

{{< github repo="molhamfetnah/ai-cup-2026-bird-classification" >}}

---

## Technologies Used Across Projects

| Category | Technologies |
|----------|--------------|
| **Languages** | Python, C++, C |
| **AI/ML** | PyTorch, TensorFlow, OpenCV, YOLO, Ultralytics |
| **Robotics** | ROS, ROS2, Gazebo, Control Systems |
| **Embedded** | Arduino, STM32, ESP32, KiCad, EasyEDA |
| **Data** | Redis, Dask, Pandas, NumPy |
| **DevOps** | Docker, N8N, FastAPI, Linux |
| **Documentation** | LaTeX, Markdown |