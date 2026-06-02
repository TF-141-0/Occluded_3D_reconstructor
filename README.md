## Rough FLowchart


# Occlusion-Aware 3D Reconstruction & Navigation System

> Occlusion-aware 3D scene reconstruction from sparse RGB-D data, with semantic classification and safe autonomous navigation using ROS2 and deep learning.

---

# Table of Contents
- [Problem Statement](#problem-statement)
- [Key Deliverables](#key-deliverables)
- [KPI Targets](#kpi-targets)
- [Project Workflow](#project-workflow)
- [System Execution](#system-execution)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Datasets & Models](#datasets--models)

---

# Problem Statement

Modern autonomous systems rely on sensor data to build 3D scene models for navigation. Real-world environments are **inherently partially observable**, leading to:

- **Occluded obstacles** — objects hidden behind furniture create blind spots
- **Incomplete maps** — limited viewpoints cause gaps in 3D reconstruction
- **Navigation risks** — missing geometry can cause collisions with unseen hazards
- **Mesh holes** — sparse sensor data produces fragmented 3D representations

Traditional mapping systems assume full visibility and **fail to infer occluded regions**. This system addresses that gap.

---
# Deliverables

| # | Deliverable | Description |
|---|---|---|
| 1 | **Occlusion-Aware Reconstruction** | Complete 3D mesh from ~500px sparse depth + RGB |
| 2 | **Scene Classification** | Free / Occupied / Occluded space + semantic labels |
| 3 | **Hole-Filling Algorithm** | Geometrically plausible mesh completion |
| 4 | **Nav2 Integration** | Safe path planning using reconstructed scene |

---
# KPI Targets

| Metric | Target | Baseline |
|---|---|---|
| F1 score — filled 3D mesh | **> 0.95** | 0.85 |
| Semantic classification accuracy | **> 90%** | 80% |
| Reconstruction accuracy | **< 2 cm** | 5 cm |

---
# Project Workflow

```mermaid
flowchart TD
    PW[Project Workflow]:::proc

    PW --> S1[Phase 1: Offline Data Pipeline]
    S1 --> S1A[Download ScanNet Dataset]
    S1A --> S1B[Preprocess RGB-D frames\n+ camera poses]
    S1B --> S1C[Visualize point clouds\nin Open3D]
    S1C --> S1D[Validate sparse depth\nsimulation — 500px per image]

    S1D --> S2[Phase 2: Model Training]
    S2 --> S2A[Depth Completion\nCompletionFormer]
    S2 --> S2B[Semantic Segmentation\nSegFormer — ScanNet classes]
    S2 --> S2C[Multi-view Geometry\nVGGT inference]
    S2A & S2B & S2C --> S2D[3D Reconstruction\nAtlas TSDF volume]
    S2D --> S2E[Hole Filling\nPatchComplete + Poisson fallback]
    S2E --> S2F[Offline KPI Evaluation\nF1 · Semantic acc · Recon error]

    S2F --> S3[Phase 3: Export & Deploy]
    S3 --> S3A[Export models to ONNX\ntorch.onnx.export — FP32]
    S3A --> S3B[Validate ONNX inference\nONNXRuntime + CUDA provider]
    S3B --> S3C[Build ROS2 workspace\nOC1_ML_ws]

    S3C --> S4[Phase 4: ROS2 Node Integration]
    S4 --> N1[depth_completion_node\nSparse → Dense depth]
    S4 --> N2[reconstruction_node\nRGB-D → TSDF mesh]
    S4 --> N3[semantic_node\nFloor · Wall · Ceiling · Platform · Other]
    S4 --> N4[hole_filling_node\nFill mesh gaps]
    S4 --> N5[occupancy_node\nFree · Occupied · Occluded voxels]
    N1 & N2 & N3 & N4 & N5 --> N6[nav2_interface_node\nPublish costmap to Nav2]

    N6 --> S5[Phase 5: Navigation Integration]
    S5 --> S5A[Configure Nav2\nBehaviorTree + costmap params]
    S5A --> S5B[Run on ROS2 bag\nRecorded sensor data]
    S5B --> S5C[Visualize in RViz2\nMesh · Occupancy · Semantics]
    S5C --> S5D[Tune latency\nTarget ≥ 5Hz costmap update]

    S5D --> S6[Phase 6: KPI Validation]
    S6 --> S6A{KPIs met?}
    S6A -->|F1 > 0.95\nSemantic > 90%\nError < 2cm| S6B[System Online]
    S6A -->|Below target| S6C[Fine-tune models\nWeights & Biases logging]
    S6C --> S2F

classDef proc fill:none,stroke:none
```

---
# System Execution

```mermaid
flowchart TD
    SE[System Execution]:::proc

    IN[RGB-D Sensor\nRGB frames + 500px sparse depth]
    IN -->|sensor_msgs/Image\n+ PointCloud2| DC

    DC[depth_completion_node\nCompletionFormer — ONNX]
    DC -->|Dense depth map| VG

    VG[VGGT inference\nCamera poses + point maps]
    VG -->|Posed RGB + depth| RC

    RC[reconstruction_node\nAtlas — TSDF volume]
    RC -->|Raw TSDF mesh| HF

    HF[hole_filling_node\nPoisson + PatchComplete — ONNX]
    HF -->|Complete mesh| SM

    SM[semantic_node\nSegFormer — ONNX]
    SM -->|Labelled mesh\nfloor · wall · ceiling · platform| OC

    OC[occupancy_node\nFree · Occupied · Occluded]
    OC -->|3D occupancy grid| NV

    NV[nav2_interface_node\nCostmap2D publisher]
    NV -->|nav2_msgs/Costmap| NP

    NP[Nav2 Stack\nPath planning + obstacle avoidance]
    NP -->|cmd_vel| RB[Robot / Simulation]

    RC -->|Mesh + labels| RV[RViz2\nLive visualization]
    SM --> RV
    OC --> RV
    NP --> RV

classDef proc fill:none,stroke:none
```

---
