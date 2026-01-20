🏛️ Alpine-Spine v0.1: Truth-Checked Invariants
| Feature | Status | Engineering Reality | The v0.1 Guarantee |
|---|---|---|---|
| Immutable Root | ✅ v0.1 | SquashFS is read-only by design. | Bit-identical boot every time. |
| Metabolic Focus | ✅ v0.1 | SIGSTOP on background cgroups. | 0% CPU for non-active Personas. |
| Zero-Leak AI | ✅ v0.1 | Umbilical socket with SO_PEERCRED. | No AI RAM/Cycles when paused. |
| Dark Suspend | ⚠️ v0.1* | Depends on platform ACPI/S3/S0ix. | Software-level total silence. |
| Hot-Plug Personas | ✅ v0.1 | Fast mounting of OCI/SquashFS. | New OS environments in <1s. |
| Spine Revert | ✅ v0.1 | Atomic SHA256 check on boot. | Automatic rollback to LKG image. |
| Hardware Kill | ❌ Platform | Only on custom GPIO/Physical wiring. | Not a universal software invariant. |
| Time Travel | 🟡 v0.2 | Requires journaling/diffing filesystem. | LKG is system-wide, not persona. |
| No-Root Model | 🟡 v0.2 | bay0 has elevated caps in v0.1. | Reduced surface, not pure object. |
| Determ. Replay | ❌ Expt. | I/O races and timing make this rare. | Event logging only; not bit-perfect. |

🔍 Strategic Realignment: The "One-Sentence" Rules
To keep the documentation from becoming "marketing theater," we apply these three filters to every claim:
 * The LKG Rule: If we can't prove the state is bit-identical using a single SHA256 hash, it isn't an invariant; it's a feature.
 * The Silicon Rule: If the claim depends on a specific motherboard, battery controller, or GPIO pin (e.g., "Hardware Kill Switch"), it is Platform-Specific, not a Spine guarantee.
 * The Memory-Safe Rule: If a capability requires bay0 to trust a Persona (e.g., "Automatic Update"), it is a Risk Layer to be added in v0.2, not part of the v0.1 core.

🧱 The v0.1 Reality Check
By tightening the table, we protect the Sovereign Substrate. We aren't promising Linda a "magic phone"; we are promising her a machine that doesn't rot.
 * v0.1 is the Substrate: Immutable, isolated, and metabolic.
 * v0.2 is the Defensibility: Audits, cryptography, and time-travel.
