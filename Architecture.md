# Alpine Spline — Architecture Overview

Alpine Spline is a minimal Alpine Linux–based **control plane**, not a general-purpose OS.  
It boots first, briefly owns the hardware, enforces invariants, and then yields execution to tenant systems.

---

## High-Level Architecture

```mermaid
flowchart TD
    A[Boot ROM / Firmware] --> B[Bootloader]
    B --> C[Alpine Spline<br/>Initramfs + SquashFS]

    C -->|mount RO| D[Immutable Root<br/>Spline Core]
    C -->|mount RW| E[/persist<br/>State & Data]

    D --> F[bay0<br/>PID 1 Control Plane]

    F -->|select / verify| G[Tenant Image]
    G -->|boot| H[Tenant System<br/>(Linux / Android / VM)]

    F -->|pause / kill / resume| H
    H -->|exit / panic| F
