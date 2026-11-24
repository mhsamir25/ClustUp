# ClustUp: Fault-Tolerant Peer-to-Peer LLM Inference

> **Turn everyday laptops and PCs on the same Wi-Fi into a private, fast, and resilient AI powerhouse — no expensive cloud bills, no data leaving your room, and no crashes when someone closes their laptop.**

---

## What is ClustUp?

Modern AI assistants like ChatGPT are powered by **Large Language Models (LLMs)**. Right now, if you want to use a really smart AI, you usually have two choices:

1. **Use a cloud service:** You send your private notes, code, or documents to a company's servers and pay recurring subscription or usage fees.
2. **Run it on your own computer:** Your data stays 100% private and free *except* the smartest open-source AI models are massive. They require far more memory (RAM/VRAM) than a regular student laptop or home PC has, forcing you to settle for smaller, less capable models.

**ClustUp gives you a third option.**  
Instead of buying a supercomputer, ClustUp lets you **pool together the laptops and desktops already sitting in your dorm, lab, or home**. It splits one large, powerful AI model across multiple nearby devices over regular Wi-Fi so they work together as a single brain.

Best of all, it is designed for **real life** where Wi-Fi isn't always fast and friends randomly close their laptops mid-conversation.

---

## The Problem We Are Solving

Splitting an AI model across multiple computers isn't a brand-new idea, but existing peer-to-peer (P2P) setups break down in real homes and university campuses because of **two major roadblocks**:

### 1. The Latency Wall (Why normal P2P AI feels painfully slow)
Think of a standard distributed AI setup like an **assembly line**. Each laptop holds one section of the AI's "brain." Normally, AI generates text **one word (token) at a time**, and every new word depends on the previous one.
* That means for *every single word*, data has to travel over Wi-Fi from Laptop A → Laptop B → Laptop C → and back.
* Over a data center's fiber-optic cables, that's fine. Over normal home or campus Wi-Fi, those constant back-and-forth trips pile up quickly, making the AI type at a glacial pace.

### 2. Network Fragility (Why one disconnect ruins everything)
Everyday laptops are not data-center servers. Batteries die, Wi-Fi drops, lids get closed, or someone opens a heavy game.
* In a standard P2P chain, **every device is a single point of failure**.
* If even *one* friend's laptop disconnects halfway through a long answer, the entire chain breaks, the response crashes, and you lose minutes of work.

---

## How ClustUp Works

Rather than trying to magically make home Wi-Fi faster, **ClustUp changes how the AI uses the network** through three core innovations:

### 1. Distributed Speculative Decoding *(Solving the Speed Problem)*
> **Think of it like a fast junior writer working with a panel of senior editors.**

* Instead of sending one word across the Wi-Fi at a time, your own laptop runs a **tiny, super-fast "draft" AI** (around 1 billion parameters).
* This small model quickly guesses the next **5 to 10 words** in a burst and sends the whole batch across the network in **one single trip**.
* The group of devices holding the **big "target" AI** checks all of those guessed words **at the exact same time** in parallel.
* Any words the big AI agrees with are accepted immediately. If it disagrees with a word, it simply replaces it with the right one and drafting continues from there.
* **The Result:** You get the **exact same quality and text** as the giant AI model, with drastically fewer Wi-Fi round-trips — trading cheap local computing for expensive network time.

```text
[ Your Laptop (Small Draft Model) ]
   └─ Rapidly guesses 5–10 words locally
   └─ Sends the whole batch in ONE Wi-Fi trip ──►
                                                 [ Peer A ] ──► [ Peer B ] ──► [ Peer C ]
                                                 (Big Model verifies all words in parallel)
   ◄── Returns accepted words (same quality as the big model, way fewer trips!)
```

### 2. State Caching & Overlapping Shards *(Solving the Disconnect Problem)*
> **Think of it like a relay race with backup runners and saved checkpoints.**

ClustUp expects devices to drop out and plans ahead for it:
* **Overlapping Layer Assignment (Standby Backups):** Instead of giving each piece of the AI to only one computer, ClustUp assigns overlapping pieces so every part of the model has a **standby backup** ready on another device.
* **Heartbeat Monitoring:** Devices quietly send lightweight "I'm still alive" signals (heartbeats) to each other.
* **Activation Caching & Acknowledgements:** When Device A sends intermediate data (activation tensors) to Device B, it keeps a copy in memory until Device B confirms, *"Got it!"* If Device B suddenly disconnects, Device A doesn't panic or start over — it just forwards its saved copy to the backup device and generation continues seamlessly.
* **Conversation Checkpointing (KV-State):** Devices periodically save snapshots of the active conversation memory. If a replacement device steps in, it can pick up right where things left off without re-reading your entire prompt.

### 3. Zero-Configuration Hardware Pooling *(Making It Effortless)*
> **No config files. No IP addresses. Just install and connect.**

* ClustUp automatically discovers nearby devices on the same network.
* It inspects each computer's available memory (RAM) and processing power.
* **Smart Splitting:** Stronger machines automatically take on more layers of the AI, while weaker laptops take fewer.
* **Dynamic Rebalancing:** If a new friend joins the Wi-Fi or someone leaves, ClustUp automatically recalculates the plan and rebalances the workload.

---

## Key Features

| Feature | What It Means for You |
| :--- | :--- |
| ** Plug-and-Play Pooling** | Drag or select a model in the app. ClustUp finds nearby devices, downloads the right pieces automatically from Hugging Face, and sets up the swarm with zero manual configuration. |
| ** Accelerated Local Drafting** | Responses stream smoothly and noticeably faster than standard P2P pipelines, without sacrificing a single bit of output quality. |
| ** Seamless Failover** | If a device loses Wi-Fi mid-answer, a standby peer takes over automatically. Even if no backup is available, ClustUp re-plans around the remaining devices (or shows a clear message if there isn't enough memory left — never a silent crash). |
| ** Live Monitoring Dashboard** | See every connected device, which layers of the AI it holds, live health status, generation speed (tokens/sec), and how often the draft model's guesses are accepted. |

---

## Who Is ClustUp Built For?

ClustUp sits at the intersection of **Distributed Systems**, **Edge AI**, and **Peer-to-Peer Networking**. It is built for anyone who cares about **privacy, zero recurring costs, or offline access**:
* ** University Labs & Students:** Pool dorm or lab laptops to run coding and research assistants locally.
* ** Households:** Combine family desktops and laptops into a private home AI.
* ** Small Teams:** Collaborate with capable AI models on sensitive documents without uploading them to third-party clouds.

---

## Tech Stack & Architecture

| Area | Technology | Role in ClustUp |
| :--- | :--- | :--- |
| **Orchestration & Backend** | Python, FastAPI, `asyncio` | Runs the background service, manages peers, routing, sessions, and serves the REST API for the UI. |
| **AI Inference Engine** | `llama.cpp` (Python bindings) | Loads and runs quantized AI models (GGUF) across CPUs and GPUs (CUDA, Apple Silicon) with speculative decoding support. |
| **Peer Communication** | gRPC (Protocol Buffers) *(evaluating WebRTC)* | Fast, structured transfer of intermediate data (tensors), token batches, heartbeats, and acknowledgements. |
| **Model Sourcing** | Hugging Face Hub API | Automatically downloads and caches the exact model files needed by each device. |
| **Peer Discovery** | mDNS / Zeroconf | Automatically finds other ClustUp devices on the local network without manual IP setup. |
| **User Interface** | React (Web) / Lightweight Desktop Wrapper | Model selector, chat interface, swarm status, and performance monitoring dashboard. |
| **DevOps & Testing** | Git/GitHub, GitHub Actions (CI), Docker, `pytest` | Version control, automated testing, reproducible builds, and continuous integration. |
| **Benchmarking** | Python scripts, Matplotlib | Measures generation speed (tokens/sec), network latency, and failure recovery time. |

---

## Project Roadmap

ClustUp is being developed over **5 phases (November 2026 – February 2027)**:

- **Phase 1 — Architecture & Networking Core (Weeks 1–4):** System architecture, FastAPI + gRPC P2P layer, automatic peer discovery, heartbeat failure detection, `llama.cpp` integration, and a two-device sequential baseline.
- **Phase 2 — Progress Demonstration (Week 5):** Live demo of peer discovery, heartbeat monitoring, and the two-device baseline pipeline.
- **Phase 3 — Intelligent Routing & Speculative Decoding (Weeks 6–9):** Memory-aware automatic layer splitting across 3+ devices, Hugging Face auto-downloads, local draft model setup, batched parallel verification, and initial speed benchmarks.
- **Phase 4 — Fault Tolerance (Weeks 10–11):** Activation caching with acknowledgements, overlapping layer assignments, KV-state checkpointing, and live disconnect/failover stress testing.
- **Phase 5 — Evaluation, Deployment & Documentation (Weeks 12–13):** Cross-hardware benchmarking (e.g., Mac & PC) comparing speed and recovery time against the baseline, finalizing the UI/dashboard, and full deployment.

---

## Honest Engineering & Evaluation

We aim to measure ClustUp's real-world performance honestly — including edge cases where the speedup is smaller (such as when the small draft model's guesses are frequently rejected). Key engineering challenges we are actively addressing include:
* **Draft Acceptance Tuning:** Empirically testing different draft models and batch lengths (5–10 tokens) to maximize speedups.
* **Mixed Hardware Profiles:** Accounting for different operating systems and compute speeds when assigning model layers.
* **Realistic Network Stress Testing:** Using tools like Linux Traffic Control (`tc`) to simulate unstable Wi-Fi delays and sudden disconnects.

---

## Team & Academic Context

Developed as part of **Design Project-I (SWE 4506)** at the **Department of Computer Science and Engineering, Islamic University of Technology (IUT)**.

**Team 5:**
* **Mahmudul Hossain Samir** (`230042101`)
* **Ifham Bin Hossain** (`230042112`)
* **Shaikh Muhaiminul Hasan** (`230042113`)

**Academic Supervisor:**
* **Dr. Mohayeminul Islam** — Assistant Professor, Department of CSE, IUT