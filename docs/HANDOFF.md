Alpine-Spline v0.1

One-Time Developer Handoff (Immutable Contract)

Read this once. Follow it literally. Do not improvise.

This repository defines a closed, sovereign OS substrate called Alpine-Spline.
It is not a distro, not a framework, and not a playground.

Your task as a developer is to materialize the contract, not reinterpret it.

If something feels missing, ambiguous, or inconvenient, that is intentional.
Ask before changing anything structural.


0. What You Are Building (Plain Language)

Alpine-Spline is an immutable Alpine Linux Spine that boots directly into a single orchestrator binary called bay0.

bay0 is PID 1

bay0 is the only authority

Everything else is expendable


The system is designed to:

survive crashes, power loss, partial updates, and hardware failure

reconcile its own state on every boot

prioritize user focus and responsiveness over background work

treat AI as a shared, constrained system organ, not an app


If the system fails, it must fail loud, explain why, and recover.

Silence is failure.


1. Non-Negotiable v0 Rules

These rules are not suggestions.

1. Path A only
Alpine kernel + initramfs.
No host kernel passthrough committed to the repo.


2. bay0 is PID 1
No systemd. No supervisord. No “just for now” init hacks.


3. Single deployable artifact
bay0 must be deployable as a single file into initramfs
(static musl preferred; minimal shared libs acceptable only if justified).


4. Observability > Speed
A 5-second boot that explains itself beats a 2-second silent boot.


5. /persist is the only durable state
Everything else is disposable.


6. Reconciliation on boot is mandatory
bay0 must converge reality to intent every boot.


7. No blocking on external tools
podman, sway, mounts, IO must all be bounded by timeouts.


8. Grammar-based namespaces
No string prefix hacks. User and system namespaces must be unambiguous.


9. Hermetic builds only
mkspine.sh must build inside a pinned container image by digest.


10. If a dev asks “where is the log,” the delivery failed



2. Canonical Repository Structure

This structure is authoritative.
Do not add top-level directories without explicit approval.

/Alpine-Spline
├── /crates
│   └── /bay0
│       └── /src
│           ├── main.rs         # Entrypoint / CLI dispatch
│           ├── init.rs         # bay0::init(), early boot, mounts
│           ├── validate.rs     # bay0 --validate (health feelers)
│           ├── reconcile.rs    # Desired vs actual convergence
│           ├── persona.rs      # Persona lifecycle & naming grammar
│           ├── umbilical.rs    # AI unix socket + SO_PEERCRED checks
│           ├── pid1.rs         # SIGCHLD reaper, signal handling
│           └── state/
│               ├── mod.rs      # Exclusive /persist boundary
│               ├── db.rs       # SQLite access + WAL + migrations
│               ├── schema.rs   # Schema + versions
│               ├── nack.rs     # Failed-upgrade memory
│               ├── time.rs     # Last-known-good time
│               └── entropy.rs  # Entropy seed read/write (bounded)
├── /scripts
│   ├── mkspine.sh              # Hermetic build (pinned digest)
│   ├── boot-qemu.sh            # Serial-first VM boot
│   └── mkpersist.sh            # Persist disk creation
└── /docs
    ├── HANDOFF.md              # This document
    └── ARCHITECTURE.md         # Short, non-poetic overview

If code touches /persist outside state/, it is wrong.


3. Developer Environment (Host)

You are expected to work on Linux (native or VM).

Required tools:

podman or docker

qemu-system-x86_64

sgdisk, mkfs.ext4

bash, curl, tar, gzip, cpio

Rust toolchain (cargo, rustc)


If something is missing, install it.
Do not work around missing tools by changing architecture.


4. Hermetic Build: mkspine.sh

mkspine.sh is the factory. If it is compromised, everything is compromised.

Mandatory properties

Runs inside a container (podman/docker)

Uses pinned Alpine image by SHA256 digest

Never pulls latest

Produces exactly:

./vmlinuz

./build/spine.img



Ghost Root requirement

The initramfs must include:

/etc/passwd

/etc/group

/etc/nsswitch.conf


Even if Alpine is “static,” UID 0 must resolve cleanly.

bay0 injection

bay0 must be present in initramfs

/sbin/init must resolve to bay0

Build must fail if bay0 is missing


5. Serial-First Boot Invariant

boot-qemu.sh must boot with:

console=ttyS0,115200

bay0 logs to /dev/console.

If serial output is broken, nothing else is trusted.


6. Persist Disk (VM)

mkpersist.sh creates persist.img (ext4, e.g. 10G).

In QEMU:

attached as virtio-blk

appears as /dev/vda

mounted by bay0 at /persist


All “survival reads” from /persist must:

have hard timeouts (≈200ms)

fail open (“dirty boot”) rather than hang



7. bay0 Responsibilities (PID 1)

7.1 Signal handling (mandatory)

SIGCHLD: reap all orphans (no zombie leakage)

SIGTERM/SIGINT: bounded shutdown

flush entropy seed

write last-known-good time

checkpoint DB

exit


bay0 must set:

oom_score_adj = -1000

PID 1 must not be killed before the system panics.

7.2 State Bridge (mandatory)

Only state::* may touch /persist.

The API must be typed and narrow:

open DB

migrate schema

list personas

set persona state

read/write NACK registry

read/write monotonic time

read/write entropy seed

append genesis log


SQLite:

WAL mode

synchronous=NORMAL

schema versioned via PRAGMA user_version

migrations run before reconcile


8. Reconciliation on Boot (mandatory)

On every boot:

1. Read desired state from DB


2. Query actual runtime (podman)


3. Converge deterministically


Rules:

starting → container may be partial → rm -f → stopped

running but missing → stopped

ambiguity → converge to stopped and log why

The system must never trust half-states.


9. Persona Naming Grammar (mandatory)

No simple prefixes.

Use semantic grammar:

system tenants: sys://ai

user personas: p://work


Internal mapping (example):

sys__ai

p__work


User input must be validated:

alphanumeric only

no empty segments

grammar enforced before DB write


10. bay0 --validate (Health Feelers)

This command is mandatory.

It must be:

fast

non-destructive

decisive


Minimum checks
1. Freezer test

start test container

confirm CPU burn

pause → CPU drops to zero

unpause → CPU resumes


2. Umbilical test

attempt unix socket connect

verify peer credentials (SO_PEERCRED)

fail if socket is fake or stale


3. Persist test

write/read marker in /persist

enforce timeout


4. Log test

append to genesis/validate log


Output contract

Single summary: GREEN / YELLOW / RED

Each failure prints:

component

reason

next action (if applicable)


11. VM Bring-Up Checklist (Do Not Skip)

1. cargo build --release


2. scripts/mkspine.sh


3. scripts/mkpersist.sh


4. scripts/boot-qemu.sh


5. Inside VM: bay0 --validate


If validation is not GREEN (or justified YELLOW), stop.



12. What You Must NOT Do

Do not introduce systemd

Do not add networking to Spine

Do not let personas touch /persist directly

Do not optimize before logging

Do not add new dependencies casually

Do not “improve” architecture without approval



13. Definition of Done (v0.1)

The work is complete when:

bay0 boots as PID 1

/persist mounts or times out cleanly

reconciliation clears ghosts after hard reboot

bay0 --validate runs and explains failures

no zombie processes accumulate

logs explain every failure path


If something breaks now, it is an implementation defect, not a design gap.


This document is the contract.
Follow it. Commit it. Then let the machine speak.

