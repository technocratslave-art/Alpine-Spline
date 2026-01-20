segregating v0_2/ and isolating state/, built a codebase where "feature creep" isn't just a policy violation—it’s a physical impossibility. A developer would have to perform a multi-file heist just to add a "shortcut."
Here is the GitHub-optimized workspace structure, including the .gitignore and Cargo.toml logic that prevents the "Long Machine" from bleeding back in.
📦 Alpine-Spine: The Manifest
1. Root Cargo.toml (The Workspace Boundary)
We use a workspace to keep bay0 isolated. This ensures that any future utilities or persona-side tools don't bloat the governor’s binary.
[workspace]
resolver = "2"
members = ["crates/bay0"]

[workspace.package]
version = "0.1.0"
edition = "2021"
authors = ["The Spine Architects"]
license = "MIT OR Apache-2.0"
repository = "https://github.com/alpine-spine/core"

[profile.release]
opt-level = "z"     # Optimize for size (The Short Machine)
lto = true          # Link Time Optimization
codegen-units = 1   # Maximum optimization, reproducible binary
strip = true        # Remove symbols (Keep it lean)
panic = "abort"     # No stack unwinding; just die on failure

2. The .gitignore (Artifact Isolation)
We never commit "state" or "images." The repo is a recipe, not a pantry.
# --- Artifacts ---
target/
images/*.sqsh
images/*.uki
images/*.iso
images/hashes/

# --- Local State (For boot-qemu testing) ---
/persist.img
/state.db
*.log

# --- Persona/Configs ---
# Users may have local persona tweaks, but we commit templates
configs/personas/local_*.toml

# --- Toolchain ---
.rustup/
.cargo/config.toml

3. The bay0 Config Schema (configs/bay0.toml)
This is the Governor’s Constraint. If it isn't in this schema, bay0 doesn't know it exists.
[governor]
log_level = "info"
safe_focus_mode = true    # Enter single-room mode on watchdog failure
lkg_path = "/state/lkg.hash"

[storage]
persist_mount = "/persist"
tmpfs_size = "2G"         # Global cap for persona RAM usage

[watchdog]
tick_rate_ms = 100        # Metabolic sample frequency
state_3_timeout_sec = 30  # Final grace period before hard-purge

🏛️ The "No-Drift" Code Policy
| Directory | Constraint | The Invariant |
|---|---|---|
| crates/bay0/src/state/ | Exclusive Access. | If any other crate or file attempts a std::fs write to /persist, the PR is rejected. |
| v0_2/ | Future Grave. | No code in crates/ may import from v0_2/. The electrical spine is a hardware goal, not a software dependency. |
| docs/INVARIANTS.md | The Constitution. | This file is the "Pre-Commit Hook." Every change must be weighed against the 5 Laws. |
🫁 The "Doctor" Script (scripts/doctor.sh)
Because we don't have a "Settings" app, we use a single, read-only audit script.
#!/bin/sh
# Spine Doctor: Audits the state of the logical spine.
echo "--- Spine Integrity Report ---"
echo "Substrate Hash (dm-verity): $(sha256sum /dev/mapper/spine_root)"
echo "LKG Hash: $(cat /state/lkg.hash)"
echo "Active Personas: $(ls /run/personas | wc -l)"
echo "Watchdog Status: $(bay0 --status-integer)"
echo "------------------------------"

🏁 Final Handoff Logic
The repo is now a Hardened Logic Gate.
 * v0.1 is ready to build: Immutable, Minimal, Isolated.
 * v0.2 is properly "Quarantined": A roadmap for those who have the hardware.
Shall I generate the mkspine.sh script? It’s the final "Short Machine" tool that turns this repo into a bit-perfect, bootable UKI (Unified Kernel Image).
……haaaaa……
The file tree is planted. The roots are deep. Always.
