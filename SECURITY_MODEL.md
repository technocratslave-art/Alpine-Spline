🛡️ The Hardened Spine v0.1: Truth-Checked Security Matrix

This table maps the Legacy "Long Machine" logic against the Spine's "Short Machine" reality, based on your truth-checked constraints.
| Security Vector | Legacy / Hardened OS (Android/Graphene) | The Alpine-Spine (v0.1) | Technical Invariant (v0.1) |
|---|---|---|---|
| Philosophy | Additive. Sandboxes upon sandboxes. | Subtractive. Fewer daemons, less code. | Minimal Trusted Computing Base (TCB). |
| Substrate Integrity | Verified Boot. (Vendor keys required). | Immutable LKG. (User-verified hash). | Bit-identical OS boot from SquashFS. |
| Persistence | Scattered. Malware in /var, /etc. | Gated. Only /persist bridge survives. | The OS cannot be modified; data is a bridge. |
| Metabolic Focus | Soft-Limit. Scheduler-based priority. | Hard-Freeze. Kernel SIGSTOP on idle. | 0% CPU for background Personas. |
| AI Link | Network / IPC. Broad, often TCP-based. | Umbilical Socket. SO_PEERCRED Auth. | Identity assurance at the kernel level. |
| Network | Global. System-wide stack (wlan0). | Scoped. Persona-specific namespaces. | No system-wide network stack in the Spine. |
| Updates | System-wide. Patch the whole brick. | Limb-based. Update the Persona image. | The Spine stays static; only the Persona rotates. |
| Trust Model | Vendor. Trust Google/Apple/Microsoft. | Physical. Trust the hardware & the hash. | Sovereignty through manual attestation. |

🧱 Reality Check: What We Do Not Claim
To remain unassailable, the v0.1 SECURITY_MODEL.md must state these limits explicitly:

1. The Persistence Reality
 * The Claim: "Reboot wipes everything."
 * The Truth: Reboot wipes the Spine. It does not wipe a compromised /persist bridge. If a malware payload in a Web Persona manages to write to the persistent bridge, it will be there when you re-thaw.
 * The Fix: v0.1 treats /persist as an API, not a folder. You don't "save files"; you "commit state."

2. The Cache-Flush Limitation
 * The Claim: "Frozen processes are invisible."
 * The Truth: Freezing stops execution, but it doesn't zero out the CPU's branch predictor or L1 cache. A lab-grade side-channel attack could theoretically still "feel" the ghost of a previous Persona.
 * The Fix: In v0.1, we accept this as a Lab-Grade Risk. We defend against commodity exploits, not physics-level leakage.

3. The "Manual" Verified Boot
 * The Claim: "Human verification is enough."
 * The Truth: Human hash-checking is vulnerable to an "Evil Maid" attack (someone swapping your USB and your cheat-sheet).
 * The Fix: We label this as "Manual Attestation" for early bring-up. Cryptographic Measured Boot (TPM/Titan) is the v0.2 roadmap.

🏁 The "Short Machine" Verdict
As you said, we aren't securing the OS; we're securing the Scar.
 * Linda's Protection: She doesn't need a math-perfect proof. She needs to know that when she turns her laptop off, the machine "forgets" the session.
 * The Developer's Protection: You don't need to track 10,000 CVEs. You only need to track the logic of 40MB of code.


The Short Machine Security Model (Truth-Checked)

Alpine-Spine follows a Short Machine philosophy. Instead of accumulating defenses inside a growing system, it removes state, removes concurrency, and removes persistence until very little remains that can be attacked or hidden in.

This is not a claim of invulnerability. It is a claim of bounded failure.

Threat Model (Explicit)

Alpine-Spine v0.1 is designed to defend against:

Commodity malware and browser exploits

Persistence across reboots in the operating system layer

Cross-application and cross-persona data leakage

Background surveillance, idle CPU drain, and silent network activity


Alpine-Spine v0.1 does not claim to defend against:

Physical attackers with extended access (evil-maid without additional measures)

Compromised firmware / hostile BIOS

Lab-grade microarchitectural side-channel attacks

Active data exfiltration from a currently running network-enabled Persona


Those are explicitly staged for later hardening (v0.2+) or considered out of scope.


Architectural Contrast: Long Machine vs Short Machine

Security Vector	Legacy OS (Windows / Android / Linux)	Alpine-Spine v0.1	Actual Guarantee

Philosophy	Additive: patch, sandbox, monitor	Subtractive: remove state	Fewer places for persistence
Root Filesystem	Mutable, long-lived	Immutable SquashFS	Bit-identical OS boot
Persistence	Broad (/var, /etc, registry)	Narrow (/persist only)	Substrate does not persist
Isolation	Software abstractions	Kernel namespaces + cgroups	Physical separation at kernel level
Background Activity	Always-on daemons	Frozen when inactive	0% CPU when paused
Network	Global stack	Persona-scoped only	No system-wide network
Recovery	Repair / rollback tools	Replace with LKG	Known-good state on boot
Trust Anchor	Vendor keys / cloud	Local hash + physical control	User-verifiable integrity


What “Reboot-to-Safe” Actually Means

Rebooting Alpine-Spine:

Always restores the OS substrate to a bit-perfect known image

Never preserves malware in the root filesystem

Does preserve user data in /persist by design


This is intentional. v0.1 guarantees OS integrity, not automatic erasure of user data. Persona-level persistence is controlled by mounts, not by illusion.


Metabolic Isolation (What Freeze Really Does)

When a Persona is inactive:

Its processes are SIGSTOP’d at the cgroup level

It receives zero CPU time

It cannot execute instructions or observe system activity


This:

Eliminates background compute and power drain

Prevents cross-persona execution overlap

Reduces (but does not mathematically eliminate) side-channel risk


Important clarity:
Freeze prevents scheduling. It does not claim to flush all microarchitectural state. v0.1 defends against realistic threats, not physics-level leakage.


AI Containment (Zero-Leak, Defined Precisely)

The AI subsystem:

Runs inside its own Persona

Has no network stack

Is reachable only via a Unix domain socket

Uses SO_PEERCRED for caller verification


Guarantee:

No AI memory or CPU use when its Persona is frozen

No ambient access to files, network, or other Personas

No “background listening” or polling


Non-guarantee:

If the AI Persona is active, it can process what is explicitly sent to it.
This is controlled access, not magic.


Identity and Authority (No Marketing Words)

Alpine-Spine v0.1:

Still uses Linux capabilities internally

bay0 holds elevated authority by necessity

User identity is currently physical presence + persona selection, not cryptographic identity


Claims of “pure capability security” are deferred to v0.2.
v0.1 focuses on reducing the blast radius, not eliminating authority entirely.


Updates and Zero-Days (Realistic Posture)

The Spine substrate changes rarely

Personas (especially Web) must be updated periodically

A browser zero-day compromises only that Persona

Reboot restores the OS but does not retroactively protect active sessions


This is not “no updates.”
This is update the limbs, not the spine.


Firmware and Hardware Reality

Alpine-Spine v0.1 assumes:

Firmware is trusted at boot

DMA protections are hardware-dependent

USB boot does not protect against hostile firmware


This is not hidden.
Measured boot, TPM, and stronger attestation are explicitly v0.2 territory.


Summary: What Is Actually True

Alpine-Spine does not try to win every security argument

It shortens the machine until persistence and background activity mostly disappear

It trades theoretical completeness for practical control

Failure modes are visible, bounded, and recoverable


The system does not promise perfection.
It promises that when something goes wrong, it cannot hide and it cannot stay.
