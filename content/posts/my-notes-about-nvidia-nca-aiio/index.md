---
title: 'My notes about NVIDIA NCA-AIIO'
date: 2026-11-27T08:43:32-03:00
draft: true
tags:
   - NVIDIA
   - AI Infrastructure
showTableOfContents: true
summary: "My study notes for NVIDIA NCA-AIIO, covering GPU communication, the port math behind a DGX BasePOD, and the tools used to operate GPU infrastructure."
---

# Introduction

These are my personal study notes for the **NVIDIA-Certified Associate: AI Infrastructure and Operations (NCA-AIIO)** exam. I started with NVIDIA's fundamentals course, which gives a good overview. But when I got to the infrastructure section, I wanted something more concrete. For example, a QM9700 has 32 physical sockets. Why does the H100 BasePOD design stop at 16 nodes?

Coming from networking, I found it easier to start with the workload and follow the traffic. What are the GPUs exchanging? Where does the traffic go when I add another server? Which links carry it? That made it easier to understand where each NVIDIA technology fits.

> **_IMPORTANT:_** These are working notes, not a complete exam guide. The [official exam blueprint](https://www.nvidia.com/en-us/learn/certification/ai-infrastructure-operations-associate/#exam-preparation) is the checklist for what to study.

------

# Training, inference, and GPU communication

## Training versus inference

During **training**, the model makes a prediction, computes a loss, and uses backpropagation to calculate gradients. The optimizer then uses those gradients to update the weights. GPU memory has to hold more than the weights: activations, gradients, and optimizer state can all contribute to the memory footprint.

During **inference**, the model uses its existing weights to produce a result. There is no backpropagation or optimizer update. For a serving workload, increasing batch size can improve throughput, but waiting to assemble a batch can increase request latency. For an LLM, memory isn't just about whether the weights fit. The **KV cache** also grows with the tokens being processed and the number of active sequences.

**Key Concept:** If the problem is slow inference on a trained model, **TensorRT** is an optimization tool. If a distributed training job spends too long synchronizing gradients, look at the communication path instead.

## Why GPUs need to talk to each other

In a simple **data-parallel** training setup, each GPU has a copy of the model and works on a different portion of a batch. Each computes local gradients. The GPUs then combine those gradients so they can apply the same update. An **all-reduce** combines the values from all participants and gives the result back to each one. **NCCL** is NVIDIA's library for GPU collectives such as all-reduce and all-gather.

| Component | What it does |
|---|---|
| **NVLink / NVSwitch** | NVLink connects GPUs; NVSwitch switches NVLink traffic across a scale-up domain. In a DGX H100, that domain is inside the server. |
| **InfiniBand / Ethernet with RoCE** | These are ways to build a scale-out fabric between servers. |
| **GPUDirect RDMA** | Lets a compatible NIC read or write GPU memory directly, without staging the payload in host memory. It doesn't replace the fabric. |
| **NCCL** | Provides the collective operations the application calls. It's a library, not an interconnect. |

If a job scales well inside one DGX H100 but slows down once I add servers, and the profiler shows more time in all-reduce, I'd start with the **scale-out path**. I'd check which interfaces and transport NCCL selected before assuming the fabric itself is congested.

------

# DGX BasePOD: why 16 H100 nodes?

This example comes from **§3.3 and Table 6** of NVIDIA's March 2023 DGX BasePOD reference architecture. It's a two-switch **NDR200** design. The 200 Gbps figure is the speed of a server-facing link, not the speed limit of a QM9700 socket.

## Count sockets first, then links

The QM9700 has **32 physical OSFP sockets**. Each socket can carry **two 400 Gbps NDR links**. With the server-facing breakout optics and cabling, that same socket can provide **four 200 Gbps NDR200 links**.

| Allocation on **each** QM9700 | Physical sockets | Links |
|---|---:|---:|
| To the other switch | 16 | 32 × 400 Gbps |
| To the DGX H100 nodes | 16 | 64 × 200 Gbps |
| **Total** | **32** | |

The inter-switch cables are **twin-port OSFP DACs**. One DAC takes up one physical socket **on each switch** and carries two 400 Gbps links. That's why 16 DACs consume 16 sockets per switch.

Each DGX H100 has **eight 200 Gbps compute links** in this design: **four to the first switch and four to the second**. That gives us:

```text
Per switch:
  32 physical sockets - 16 inter-switch sockets = 16 server-facing sockets
  16 sockets × 4 links per socket                = 64 server-facing links
  64 links ÷ 4 links per DGX H100                 = 16 DGX H100 nodes
```

The two switches connect to the **same** 16 nodes. The design also has equal aggregate bandwidth on the server-facing and inter-switch sides of each switch: **12.8 Tbps** on each side.

**Key Concept:** You could use all 32 sockets for servers, but then you'd have no sockets left for the inter-switch cables. The 16-node answer comes from the **specific topology and link allocation**, not from a general limit of the QM9700.

------

# Operating GPU infrastructure

## MIG versus time-slicing

**Multi-Instance GPU (MIG)** partitions a supported physical GPU into instances with dedicated compute and memory resources. If two services fit into separate instances and need hardware isolation, MIG is the feature to look for.

**Time-slicing** allows multiple workloads to share access to a GPU over time. It doesn't give them the memory or fault isolation that MIG provides. And **GPUDirect RDMA** has nothing to do with partitioning: it changes how data reaches GPU memory.

## Kubernetes, GPU Operator, and DCGM

| Question | Component |
|---|---|
| What advertises `nvidia.com/gpu` so a pod can request it? | The **NVIDIA Kubernetes device plugin**. |
| What manages the lifecycle of NVIDIA GPU software components in a Kubernetes cluster? | The **NVIDIA GPU Operator**. It can deploy and manage components including drivers, the device plugin, and telemetry. |
| What provides GPU health checks, diagnostics, and metrics? | **DCGM**. **DCGM Exporter** exposes GPU metrics for Prometheus to scrape. |

A useful distinction when troubleshooting: if a pod can't request a GPU, check **resource advertisement and scheduling**. If the pod has a GPU but utilization is low, look at what the application is waiting on. A training job waiting for the next batch points to the **input pipeline**: data loading, preprocessing, or storage throughput. Low utilization alone doesn't tell you which one.

------

# References

- [NVIDIA NCA-AIIO exam blueprint and practice questions](https://www.nvidia.com/en-us/learn/certification/ai-infrastructure-operations-associate/)
- [Machine Learning Systems: Hardware Acceleration](https://mlsysbook.ai/vol1/11_hw_acceleration/11_hw_acceleration.html) and [Collective Communication](https://mlsysbook.ai/vol2/05_collective_communication/05_collective_communication.html)
- [NVIDIA NCCL: Collective Operations](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/collectives.html) and [GPUDirect RDMA](https://docs.nvidia.com/cuda/gpudirect-rdma/index.html)
- [NVIDIA DGX BasePOD Reference Architecture, March 2023, §3.3, Figure 17 and Table 6](https://www.pny.com/en-eu/File%20Library/Professional/DATASHEET/DGX/nvidia-dgx-basepod-reference-architecture.pdf)
- [NVIDIA QM97XX Switch Manual](https://docs.nvidia.com/networking/display/qm97xx-1u-ndr-400gbps-infiniband-switch-systems-user-manual.pdf) and [MCP4Y10-Nxxx Twin-Port DAC Specifications](https://docs.nvidia.com/networking/display/mcp4y10-nxxx-twin-port-2x400gb-s-osfp-to-2x400gb-s-osfp-passive-dac-product-specifications.pdf)
- [NVIDIA TensorRT](https://developer.nvidia.com/tensorrt), [GPU Operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/overview.html), [GPU Sharing](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html), and [DCGM](https://docs.nvidia.com/datacenter/dcgm/latest/user-guide/feature-overview.html)
