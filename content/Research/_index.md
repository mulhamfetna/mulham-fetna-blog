---
title: "Research & Publications"
description: "Academic research papers and publications in robotics, control systems, and AI."
showDate: false
showAuthor: false
showTableOfContents: true
---

# Research Publications

> Publications and research contributions in robotics, control systems, artificial intelligence, and machine learning.

---

## Submitted Journal Papers

### Paper 1: Distributed Model Predictive Control for Multi-UAV Formation with Consensus

| Field | Details |
|-------|---------|
| **Title** | Distributed Model Predictive Control for Multi-UAV Formation with Consensus |
| **Journal** | Robotics and Autonomous Systems (Elsevier) |
| **Manuscript ID** | ROBOT-D-26-01147 |
| **Status** | Received (May 10, 2026) |
| **Authors** | Mulham Fetna, et al. |

**Abstract:**
This paper presents a comprehensive implementation of distributed Model Predictive Control (MPC) for multi-UAV quadrotor formation control with consensus-based coordination and geometric SO(3) attitude control.

**Key Contributions:**
- Local MPC solver with formation constraints
- Consensus protocol supporting ring/mesh/star topology
- Geometric SO(3) controller using quaternion representation
- Formation planner (grid, line, circle, wedge)

**Technical Details:**
- Quadrotor mass: 1.0 kg
- Inertia: diag([0.01, 0.01, 0.02]) kg·m²
- PD controller: kp=0.4, kd=0.8
- Velocity limits: ±3.0 m/s

**Results:**
- Benchmark: 7/7 scenarios passed (100% success rate)
- Stress tests: 20/21 tests passed (95.2% pass rate)

---

### Paper 2: Modernized Bees Algorithm for Dynamic Path Planning in Robotics

| Field | Details |
|-------|---------|
| **Title** | Modernized Bees Algorithm for Dynamic Path Planning in Robotics |
| **Journal** | Applied Soft Computing (Elsevier) |
| **Manuscript ID** | ASOC-D-26-06746 |
| **Status** | Received (May 6, 2026) |
| **Authors** | Mulham Fetna, et al. |

**Abstract:**
This work presents a modernized implementation of the Bees Algorithm for robot path planning in dynamic environments with constraint handling.

**Key Contributions:**
- Adaptive parameter tuning based on convergence state
- Multi-objective optimization (path length, safety, smoothness, energy)
- Dynamic obstacle handling
- Comprehensive evaluation framework with 30-run statistical analysis
- ROS/Gazebo integration

**Technical Details:**
- Scout bees: 50
- Elite sites: 5, Best sites: 20
- Neighborhood size: 0.1
- Max iterations: 500

**Results:**
- 100% success rate across 6 scenarios
- Average planning time: 0.35 seconds
- GitHub: [swarm-path-planning-bees](https://github.com/mulhamfetna/swarm-path-planning-bees)

---

### Paper 3: Hybrid Inverse Kinematics Ensemble with Learned Uncertainty Estimation

| Field | Details |
|-------|---------|
| **Title** | Hybrid Inverse Kinematics Ensemble with Learned Uncertainty Estimation for Robotic Manipulation |
| **Journal** | IEEE Robotics and Automation Letters (RA-L) |
| **Submission Number** | 26-2479 |
| **Status** | Received (May 9, 2026) |
| **Authors** | Mulham Fetna, Luca Ricci (University of Tuscia, Italy) |

**Abstract:**
This paper presents a hybrid ensemble architecture combining Damped Least Squares with neural network solver, along with learned uncertainty-based weighting that adapts solver contributions based on confidence.

**Key Contributions:**
- Hybrid ensemble combining multiple IK solver strategies
- Learned uncertainty-based weighting
- Comprehensive benchmark across 7 scenarios
- International collaboration (Italy)

**Technical Details:**
- Robot Model: 6-DOF UR5-like manipulator
- Damping factor: λ = 0.01
- Maximum iterations: 100
- Neural Network: MLP [3 input, 128, 128, 64, 6 output]
- Training: 4640 samples, 100 epochs, learning rate 0.005
- Ensemble weights: DLS=0.45, NN=0.55 after convergence

**Results:**
- 100% success rate on random targets
- 86.7% overall success rate
- Average solve time: 5ms (random targets), 8ms (overall)

---

## Research Areas

| Area | Description |
|------|-------------|
| **Robotics** | Multi-UAV formation control, path planning, inverse kinematics |
| **Control Systems** | MPC, geometric control, PID, state estimation (EKF/UKF) |
| **AI/ML** | Neural networks, ensemble learning, uncertainty quantification |
| **Optimization** | Swarm intelligence, metaheuristics, multi-objective optimization |
| **Computer Vision** | Object detection, edge AI, real-time inference |

---

## Research Infrastructure

- **Simulation:** Python/NumPy, ROS/Gazebo, MATLAB/Simulink
- **Testing:** Custom stress test frameworks, pytest
- **Documentation:** LaTeX, Markdown, Jupyter Notebooks
- **Version Control:** Git, GitHub

---

## Collaboration

### International Collaborations
- **Luca Ricci** — University of Tuscia, Italy (Co-author on IK paper)

### Academic Platforms
- IEEE ScholarOne
- Elsevier Editorial Manager
- arXiv

---

## Research Philosophy

> "Bridging academic theory with practical engineering applications through rigorous research and open-source contributions."

---

## Contact for Research Collaboration

For research collaboration opportunities, please contact:
- **Email:** molhamfetneh@gmail.com
- **GitHub:** github.com/mulhamfetna