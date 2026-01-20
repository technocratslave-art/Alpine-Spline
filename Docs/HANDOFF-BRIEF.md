The HANDOFF.md is a clinically precise and structurally sound. It successfully transforms the complex conceptual requirements of the Spine into a set of engineering constraints that leave no room for "helpful" improvisation.
To ensure the developer internalizes the physical and logical boundaries of the system they are about to build, I have provided the matching docs/ARCHITECTURE.md. This is designed to be the "Anti-Drift" anchor—the permanent record of why these constraints exist.
📜 docs/ARCHITECTURE.md: The Sovereign Invariants

1. The Principle of Cellular Isolation
The Spine is not a multi-user OS; it is a Cellular Organism. The Spine (Host) is the non-thinking substrate that provides power, cooling, and scheduling. The Personas are the independent cells.
 * Host (Spine): Immutable, network-less, and sterile. Its only job is to run bay0.
 * Tenant (AI): A constrained system organ. Shared intelligence with zero autonomy.
 * Personas (User): Isolated namespaces where all human activity occurs.

2. The Metabolic Invariant (Resource Control)
Responsive computing is a result of metabolic discipline. bay0 governs the "heart rate" of the system.
 * Focus-Pause: Background Personas are frozen at the kernel level via the cgroup freezer. They do not "wait"; they stop.
 * Priority Weighting: bay0 and the Compositor always have CPU/IO priority. AI is sacrificial.

3. The Durability Invariant (Reconciliation)
Stability is not the absence of errors; it is the presence of a recovery plan. The Spine does not "remember" the last session; it reconciles the current reality against the stored intent.
 * Intent: The SQLite DB is the source of truth for what should be.
 * Reality: The Linux kernel/container state is the reality of what is.
 * Convergence: On every boot, bay0 forces Reality to match Intent.

4. The Umbilical (Secured Intelligence)
AI is powerful but dangerous. It must be accessed via a narrow, audited "Umbilical."
 * Unix Sockets: No network stacks. AI communication is a file-based privilege.
 * SO_PEERCRED: Intelligence is only provided to verified, authorized processes.
