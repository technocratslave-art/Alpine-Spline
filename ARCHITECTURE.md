flowchart TD
    A[Boot ROM / Firmware] --> B[Bootloader]
    B --> C[Alpine Spline Initramfs + SquashFS]

    C -->|mount RO| D[Immutable Root - Spline Core]
    C -->|mount RW| E[/persist - State and Data]

    D --> F[bay0 PID 1 Control Plane]

    F -->|select and verify| G[Tenant Image]
    G -->|boot| H[Tenant System Linux / Android / VM]

    F -->|pause kill resume| H
    H -->|exit or panic| F
