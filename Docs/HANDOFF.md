Scope: v0.1 (Logical Spine) + v0.2 Roadmap (Electrical Spine)
Audience: Core developers, reviewers, hardware partners
Non-Goal: Popularity, convenience, feature parity

0. Why This Document Exists

This project does not fail because of bugs.
It fails if someone “helps.”

Most operating systems collapse because well-meaning developers slide convenience into places where authority used to be. That slide is subtle, incremental, and usually justified as “just one exception.”

This document exists to explain why we do things differently, and to permanently block the instincts that will try to undo the architecture.

If a change violates the principles below, it is not Alpine-Spine, even if it compiles.


1. The Core Problem We Are Solving (Not the One People Assume)

We are not trying to:

Detect malware better
Patch faster
Predict attacks
Out-AI attackers
Build a “secure desktop”


Those are Long Machine goals.

The real problem:

Persistence + time + shared state = eventual compromise
Every modern exploit chain depends on at least one of:
Surviving a reboot
Sharing a kernel namespace
Accumulating privileges over time
Waiting quietly


The Spine deletes all four assumptions.


2. Architectural Philosophy: The Short Machine

Long Machine (Industry Default)

Infinite features
Infinite patches
Infinite trust
Infinite patience


Short Machine (Spine)

Minimal surface
Minimal time
Minimal trust
No patience


We do not try to survive attacks.
We refuse to host them.

3. v0.1: The Logical Spine (What Exists Today)

v0.1 is deliberately portable, boring, and brutal.

3.1 Immutable Substrate

The OS is a single SquashFS image
Read-only by design
Verified by hash on boot
No mutable root
No package manager
No background services
No cron
No update daemon


Why this matters:
Most malware persistence relies on writing somewhere.
We removed “somewhere.”

> If the image hash changes, the system does not “repair.”
It dies and reverts.

3.2 bay0 (PID 1) — The Governor

bay0 is not an init system.
It is a governor.

It:

Owns lifecycle
Owns persona creation
Owns freezing/thawing
Owns I/O delegation
Owns death


It does not:

Run apps
Parse content
Inspect files
Interpret intent
Make “smart” decisions


Why this matters:
Once PID 1 becomes clever, it becomes corruptible.

bay0 reacts to physics, not meaning.

3.3 Personas (Execution Islands)

Each Persona is:
A kernel namespace silo
Its own PID tree
Its own mounts
Its own cgroups
Its own tmpfs overlay


No Persona:

Shares memory
Shares IPC
Shares network
Shares filesystem state
Has root on the substrate


Why this matters:
Most exploits assume lateral movement.
There is nowhere to move to.

3.4 tmpfs Overlays (Amnesia by Default)
Personas write only to RAM unless explicitly allowed.

On:
Freeze
Exit
Reboot
Power loss


Everything disappears.

Why this matters:
Malware’s business model is survival.
We made survival optional and temporary.


3.5 /persist — The State Gate

/persist exists, but it is not ambient.
No Persona mounts it directly
All access goes through bay0’s state:: API
Commits are explicit
Narrow
Logged


Why this matters:
State is authority.
Authority must be rare, visible, and intentional.


4. The Umbilical (Controlled Reality Transfer)
There is exactly one way Personas communicate:

Persona → bay0 → Persona

Unix socket

Credential-verified

Explicit push / pull
No shared folders
No shared clipboard
No drag-and-drop


Why this matters:
Shared convenience is how infections propagate.
The Umbilical forces a conscious boundary crossing.

Yes, it costs one click.
That click is the wall.

5. The Metabolic Watchdog (v0.1)
The Watchdog is not security AI.
It is a thermometer.

5.1 What It Sees
Only coarse system physics:
CPU jitter
Context switch rate
Memory pressure (PSI)
I/O cadence


No:

Files
Text
Network payloads
User data
Intent


5.2 What It Can Say

Exactly one integer:

0 = Normal

1 = Warn

2 = Throttle

3 = Critical


5.3 What bay0 Does

bay0 does not debate.
State 2 → clamp CPU/I/O
State 3 → SIGSTOP → purge


Why this matters:
Content-aware security can be tricked.
Physics cannot be argued with.

6. The Principle of Invisible Safety (Linda Test)

If Linda notices security, we failed.

Correct behavior:
Browser thrashes
Cursor pauses briefly

Screen goes black
Desktop returns
Files intact

No dialog
No sermon
No fear


She blames her finger.
Not the machine.

Security should feel like speed.

7. What We Explicitly Do NOT Promise (Audit-Critical)
We do not guarantee:

Availability

> The machine chooses death over compromise.


Continuous uptime

> Reboots are a feature.

Firmware purity

> We assume hostile silicon and remove persistence.

Protection from stolen hardware

> Sovereignty is physical as well as logical.


These are not omissions.
They are design choices.


8. v0.2 Roadmap: The Electrical Spine (Why Software Is Not Enough)

v0.1 defeats remote, logical, and automated threats.

v0.2 defeats time-based and physical ghosts.

8.1 The Problem

Any software governor can be delayed.
Any delay can be exploited.

8.2 The Fix

Move final authority out of code and into voltage.


8.3 Hardware Watchdog (Dead-Man)

bay0 must toggle a GPIO every 500 ms

External WDT monitors pulse

Missed pulse → power rail cut

RAM loses refresh

State evaporates

No logs.
No grace.
No debate.

Why this matters:
You cannot negotiate with a capacitor.

8.4 v0.2 Scope Reality
    v0.2 is:

Device-specific
Hardware-dependent

Not portable
Not universal
And that is correct.

Electrical sovereignty cannot be abstracted.

9. The Five Spine Laws (Non-Negotiable)

1. No persistence without explicit commit
2. No shared global namespace
3. No anomaly without purge
4. No heartbeat without voltage (v0.2)
5. No trust in software alone

Break one → not Spine.


10. Guidance for Future Developers (Read This Twice)

If you feel the urge to:

Add auto-update

Add background helpers

Add convenience IPC

Add “just one” exception

Add recovery without reboot

Add silent persistence


Stop.

You are rebuilding the Long Machine.

The Spine is not missing features.
It is missing assumptions.


11. Final Word

Most systems try to survive attacks.
The Spine refuses to host them.

Most systems optimize for uptime.
The Spine optimizes for truth.

Most systems trust vendors.
The Spine trusts physics.

If this makes you uncomfortable, good.
That discomfort is the wall holding.
