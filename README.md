# FluxGuard 🛡️🛑

![Rust](https://img.shields.io/badge/Language-Rust-orange)
![Consistency](https://img.shields.io/badge/Consistency-Eventual%20(CRDT)-purple)
![Integration](https://img.shields.io/badge/Proxy-Envoy%20WASM-green)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

**FluxGuard** is a global rate-limiting system designed for the Edge.

Traditional rate limiters (like Redis Token Buckets) require a central counter. If your users are in Tokyo and your Redis is in Virginia, the latency ruins the request time. FluxGuard solves this by pushing the counter logic to the Edge using **CRDTs (Conflict-free Replicated Data Types)**.

## 🏗️ Architecture: Global Sync, Local Speed

### 1. The PN-Counter CRDT
FluxGuard implements a Positive-Negative Counter where increments are always commutative and associative.
* **Local Action:** An Edge Node (e.g., in Frankfurt) accepts a request and increments its local counter immediately (0ms latency penalty).
* **Gossip Protocol:** Asynchronously, the node broadcasts its counter delta to other regions (Tokyo, US-East).
* **Merge:** Nodes merge the incoming deltas. The global count is mathematically guaranteed to converge to the correct value eventually.

### 2. Envoy WASM Filter
The enforcement logic is compiled to **WebAssembly (WASM)** using Rust and injected directly into **Envoy Proxy**. This allows the rate limiter to run inside the data plane sidecar, eliminating network hops for the check itself.

## 🛠️ Tech Stack

* **Language:** Rust (compiled to `wasm32-unknown-unknown`)
* **Algorithm:** State-based CRDTs (G-Counter / PN-Counter)
* **Protocol:** UDP Gossip for inter-region sync
* **Integration:** Envoy Proxy / Istio

## 🧪 Simulation

FluxGuard includes a visual simulator to demonstrate convergence:

1.  **Region A** receives 100 requests.
2.  **Region B** receives 50 requests.
3.  *Network Partition occurs.*
4.  Region A sees 100, Region B sees 50.
5.  *Partition heals.*
6.  Both regions converge to **150** automatically without conflict.
