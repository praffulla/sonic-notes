# SONiC Notes — NOS Developer & Internals

## Introduction

### What this is

This is a learning journal that is growing into a book on how **SONiC (Software for Open
Networking in the Cloud)** works inside — not only how to configure it, but how its parts are
built and how state moves through the system, from a CLI command all the way down to the
switching chip.

It is written while learning, so each chapter tries to explain *why* things are designed the way
they are, using real-world comparisons — not just *what* the commands are.

### Who it's for

- Network engineers moving from vendor operating systems (Cisco, Juniper, Arista) to SONiC
- Software engineers who want to develop, debug or extend SONiC
- Anyone preparing to support SONiC in production

### What you should already know

- Basic networking: L2/L3, VLANs, BGP, ARP
- Basic Linux: shell, processes, `systemctl`
- Helpful but not required: Docker, Redis, C++ or Python

### How the book flows

The book follows the path that configuration and state take through a SONiC switch:

```
User config / routing protocols
        ↓
  Redis databases (shared state)
        ↓
  orchagent (decides what to program)
        ↓
  sairedis → syncd (hardware boundary)
        ↓
  SAI → ASIC (the switching chip)
```

Parts I–II build the foundation (what SONiC is, its containers, its databases). Parts III–V
follow this path from top to bottom. Part VI covers running SONiC in production.

### How to use the labs

Every hands-on chapter pairs the explanation with a lab you can run on a laptop:

- **SONiC-VPP** — SONiC with a software data plane (VPP), so you can watch packets actually
  forward, using `vppctl`.
- **sonic-vs** — SONiC's virtual switch, for features VPP cannot model (buffers, PFC, most
  hardware counters).
- **`redis-cli`** — to inspect the databases at every step and see state change live.

Lab files are in [`labs/`](labs/).

### Which SONiC version

SONiC changes from release to release. The target release branch for this book is
**still to be decided**. Where behavior differs between releases, the text says so.

### Conventions

- `code style` marks commands, file names, Redis keys and code identifiers.
- Each chapter ends with **Check yourself** questions.
- Diagrams are text-based (SVG or Mermaid), so they display on GitHub and can be edited in git.
- Corrections and open questions are tracked in
  [`notes/errata-and-open-questions.md`](notes/errata-and-open-questions.md).

---

## How this book is organized

The book has **6 parts**. Each part covers one layer of SONiC, moving from the outside of the
switch (installation) to the inside (ASIC programming), then to production operations.

| Part | Theme | What you'll learn | Chapters |
|---|---|---|---|
| **I** | Installation, System Topology & Build | What SONiC is, how to install it, how it boots and splits into containers, how it is built | 1–4 |
| **II** | Redis & swsscommon (the state bus) | How every SONiC process shares state through Redis | 5–6 |
| **III** | Configuration, Kernel Sync & Control Plane | How user config and routing protocols (FRR) enter the system | 7–9 |
| **IV** | orchagent internals | How desired state is turned into SAI objects | 10–12 |
| **V** | sairedis, syncd & SAI (the hardware boundary) | How SAI calls reach the ASIC, and how counters come back | 13–16 |
| **VI** | High availability, platform & debugging | Warm reboot, platform drivers and production troubleshooting | 17–19 |

## Table of contents

Status: ✅ drafted · 📝 notes only · ⬜ not started

### Part I — Installation, System Topology & Build

- ✅ **[Chapter 1 — Disaggregated Networking & SONiC Design Philosophy](book/part1/ch01-disaggregation.md)**
  - 1.1 Monolithic NOS vs. disaggregated white box
    - 1.1.1 The two parts of a switch
    - 1.1.2 A spectrum, not two boxes
    - 1.1.3 "Monolithic" has two meanings
    - 1.1.4 Where SONiC is better than a process-based NOS like EOS
    - 1.1.5 When a monolithic NOS is better than SONiC
    - 1.1.6 Summary: when to choose which
  - 1.2 Why Microsoft built SONiC
  - 1.3 The three pillars: SAI, containers, Redis
    - 1.3.1 The problems on the table
    - 1.3.2 Pillar 1 — Hardware abstraction (SAI)
    - 1.3.3 Pillar 2 — Microservices (Docker containers)
    - 1.3.4 Pillar 3 — Shared state (Redis)
    - 1.3.5 How the pillars work together
  - 1.4 Install path (ONIE) and boot lifecycle
    - 1.4.1 The install path: ONIE
    - 1.4.2 The boot path: from power-on to a running switch
    - 1.4.3 Who restarts what when something fails
    - 1.4.4 Seeing it on a real switch
- ⬜ Chapter 2 — Installing SONiC & the SONiC-VPP lab
- ⬜ Chapter 3 — Container & subsystem topology
- ⬜ Chapter 4 — The SONiC build system

### Part II — Redis & swsscommon

- ⬜ Chapter 5 — Redis database architecture
- ⬜ Chapter 6 — swsscommon state abstractions

### Part III — Configuration, Kernel Sync & Control Plane

- ⬜ Chapter 7 — Configuration management & YANG
- ⬜ Chapter 8 — Kernel synchronization (`*mgrd` / `*syncd`)
- ⬜ Chapter 9 — Routing stack integration (FRR & fpmsyncd)

### Part IV — orchagent internals

- ⬜ Chapter 10 — orchagent execution model
- ⬜ Chapter 11 — "Life of a…" walkthroughs
- ⬜ Chapter 12 — ACLs, QoS & buffers

### Part V — sairedis, syncd & SAI

- ⬜ Chapter 13 — SAI fundamentals
- ⬜ Chapter 14 — sairedis serialization & OID virtualization
- ⬜ Chapter 15 — syncd architecture
- 📝 [Chapter 16 — Flex counters, watermarks & telemetry](book/part5/ch16-flex-counters.md)

### Part VI — High availability, platform & debugging

- ⬜ Chapter 17 — Fast & warm reboot
- ⬜ Chapter 18 — Platform subsystem (pmon)
- ⬜ Chapter 19 — Debugging & diagnostics playbook

### Appendices

- ✅ [Appendix A — Design questions to explore](book/appendix/a-design-questions.md)
  - A.1 Why did SONiC choose Redis?
