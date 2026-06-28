# WFCA: Adaptive Routing for High-Radix Interconnects

This repository contains a cycle-accurate architectural profile of a Weighted Fault and Congestion Aware (WFCA) routing algorithm. The implementation is mapped onto a Fat-Tree Network-on-Chip (NoC) to manage the asymmetric tensor dataflows inherent to spatial AI accelerators.

## Problem Statement
Spatial reduction architectures (e.g., the MAERI Augmented Reduction Tree) induce highly asymmetric, many-to-one communication patterns. 

* **Baseline Limitations:** The standard Oblivious Nearest Common Ancestor (NCA) routing algorithm selects redundant upward links uniformly at random. It does not account for network congestion, resulting in routing decisions that saturate local queues and cause structural deadlock under high injection rates.
* **Deterministic Routing Limitations:** Attempting to optimise routing by deterministically selecting the single least-congested port results in synchronised routing decisions across the network. This correlation causes oscillatory congestion patterns and premature network saturation.

## Proposed Architecture: WFCA
To eliminate synchronisation and balance the asymmetric load, this repository implements a Soft-Weighted Probabilistic Routing (WFCA) framework.

* **Dynamic Probability Distribution:** The algorithm translates hardware credit counters (queue depths) into an inverse weight probability distribution.
* **Proportional Load Balancing:** Ports with lower congestion are assigned a mathematically higher probability of selection, while heavily loaded ports retain a non-zero probability to prevent complete starvation. 
* **Stochastic Routing:** By utilising a weighted stochastic policy instead of a deterministic greedy choice, WFCA reduces correlation across routers and mitigates oscillatory congestion.

## Performance Evaluation
The routing logic was evaluated using BookSim. Downward routing was delegated to the native deterministic implementation to preserve physical wiring integrity, while the custom WFCA logic governed the upward redundant phase.

* **Latency Reduction:** The WFCA framework achieved an 8% reduction in average packet latency (decreasing from 469.5 cycles to 432.9 cycles) compared to the baseline NCA under a 0.70 injection rate.
* **Deadlock Prevention:** The stochastic policy successfully prevented the synchronised deadlocks observed in both the baseline and deterministic approaches.
* **Hardware Optimisation:** To address the silicon area and active power overhead of per-cycle floating-point probability computation, the final implementation incorporates a Power-of-Two Choices (P2C) hardware comparator. This optimised selection engine improved the latency reduction to 11% and sustained stable operation at a physical saturation limit of 0.707.

## Simulation Configuration
* **Topology:** Fat-Tree (`k=4`, `n=3`)
* **Traffic Model:** Asymmetric Tensor Dataflow (Shuffle)
* **Simulator:** BookSim (Cycle-Accurate)
