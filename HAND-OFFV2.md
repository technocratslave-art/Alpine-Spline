──────────────────────────────────────── One-time dev handoff: Alpine-Spline v0.1 ────────────────────────────────────────

Purpose of this repo Alpine-Spline is an immutable Spine OS image that boots directly into bay0 (PID 1). bay0 owns:

1. boot-time reconciliation,


2. the compositor lifecycle,


3. podman-based personas,


4. the AI “system tenant” behind a local unix socket,


5. resource governors (CPU/mem/thermal/PSI),


6. durability rules (timeouts, crash recovery, anti-loop memory).



Everything else is optional and must be treated as replaceable “tenants” or “personas.”

Non-negotiable v0 rules (do not debate; implement)

1. Path A only: Alpine kernel + initramfs. No host kernel passthrough except as a temporary local debug hack on a dev machine, never committed as “the way.”


2. bay0 is PID 1 and must be a single deployable artifact suitable for initramfs (prefer static musl build).


3. Observability beats speed. Every important path logs. If a dev asks “where’s the log,” the delivery has failed.


4. /persist is the only writable continuity. Everything else is disposable.


5. Personas are user land. System tenants (AI, compositor helpers) are spine-owned.


6. State reconciliation on boot is mandatory. The DB is intent; runtime is reality. bay0 forces runtime to match intent.


7. All survival reads have hard timeouts (boot cannot hang on dead storage).


8. bay0 must never block on podman/sway operations. Use timeouts + reconciliation thread.


9. Namespaces are grammar-separated, not “best effort prefixes.” Prevent collisions.


10. Builds are hermetic. mkspine.sh must build inside a pinned container image by digest.



Repository layout (canonical) This is the structure the dev must implement and keep stable:

/Alpine-Spline /crates /bay0 /src main.rs init.rs                # bay0::init() entry; PID1 responsibilities validate.rs            # bay0 --validate reconcile.rs           # desired vs actual convergence persona.rs             # create/start/stop/reset, prefixing, crash-safe state machine focus.rs               # pause/unpause + sway mark focus + GPU detach before pause umbilical.rs           # AI unix socket, peercred verification, drain logic service_ai.rs          # AI system tenant launch + warm boot + envelope thermal.rs             # thermal governor + PSI weighting state/ mod.rs               # only module allowed to touch /persist db.rs                # sqlite access + migrations + WAL settings schema.rs            # schema + versions nack.rs              # failed-upgrade registry time.rs              # last-known-good time storage/restore entropy.rs           # entropy seed read/write with timeouts log.rs                 # structured log helpers + genesis log append pid1.rs                # SIGCHLD reaper + signal handling net.rs                 # explicit “no network in spine” assertions /scripts mkspine.sh                 # builds rootfs + packages + bay0 injection + initramfs boot-qemu.sh               # boots vmlinuz + initramfs + persist disk in QEMU mkpersist.sh               # creates persist.img (ext4) for VM genesis-flash.sh           # destructive disk initializer (USB / bare metal), optional for later package-efi.sh             # UKI packaging into spine.efi (later) check-firmware.sh          # verifies firmware presence in rootfs /docs HANDOFF.md                 # this document (derived from this handoff) ARCHITECTURE.md            # brief; no prose bloat THREATMODEL.md             # concise threat vectors + mitigations checklist OPERATIONS.md              # how to run validate/doctor and interpret output /personas                    # optional OCI/Dockerfiles for sample personas /services                    # optional service images, ai-core etc.

Do not create new top-level directories unless they are docs or scripts that support the above.

Build + boot: the only supported VM workflow You will develop and validate on a Linux VM (or Linux host running QEMU). The first milestone is a serial console boot into bay0 with /persist mounted.

Host prerequisites (developer machine) Required commands:

podman (or docker) to run the hermetic build container

qemu-system-x86_64

sgdisk, mkfs.vfat, mkfs.ext4 (for persist and genesis tools)

rust toolchain for bay0 (cargo)

basic utils: bash, curl, tar, gzip, cpio, coreutils


If you are missing tools, do not “work around” by editing architecture. Install the tools.

Hermetic build requirement (mkspine.sh) mkspine.sh must:

1. run inside a container (podman/docker) unless explicitly told otherwise,


2. use a pinned image digest, never “latest,”


3. produce two artifacts:

./vmlinuz

./build/spine.img



4. inject bay0 as /sbin/init (symlink or direct), and confirm it exists


5. include a minimal /etc skeleton (“Ghost Root”) in initramfs so UID 0 resolves



Pinned build container Use an Alpine container pinned by digest. Example pattern:

ALPINE_BUILDER_IMAGE="alpine@sha256:<DIGEST_HERE>"

The dev must hardcode the digest in mkspine.sh and update it only with an explicit commit message that includes the new digest and why.

mkspine.sh must be idempotent Re-running mkspine.sh should wipe build/rootfs and rebuild cleanly.

Minimal contents that MUST exist inside rootfs

/sbin/init -> bay0

/etc/passwd, /etc/group, /etc/nsswitch.conf

/dev /proc /sys /run /persist /tmp

/lib/modules/<kver>/ (kernel modules)

/lib/firmware/ (at least i915/amdgpu/intel as available; more is fine but don’t bloat blindly)

podman + helpers (fuse-overlayfs, slirp4netns) if personas are expected in v0.1 VM test

sway + swaymsg (or minimal compositor plan if you’re staging; but Week-1 can boot to console before compositor)


Serial-first boot invariant boot-qemu.sh must always include: console=ttyS0,115200 as primary output bay0 logs to /dev/console and prints “genesis lines” early.

If serial isn’t working, nothing else is trusted.

VM persist disk mkpersist.sh creates persist.img (ext4, e.g., 10G). QEMU attaches it as virtio-blk (/dev/vda). bay0 mounts it to /persist.

First milestone (“Gate A”): boot to bay0 shell This is the Day-5 equivalent. The dev is not allowed to chase UI until this gate is green.

Pass condition:

QEMU boots

bay0 runs as PID 1

/persist mounts successfully (or times out cleanly and continues “dirty boot”)

bay0 prints a banner and drops to a shell (or provides a bay0 CLI prompt)


Required log lines (approximate; content matters, exact wording doesn’t)

bay0: starting (version)

bay0: mounted proc/sys/dev

bay0: persist mount attempt (device detection)

bay0: persist mounted OR persist timeout -> dirty boot

bay0: state db open/migrate

bay0: init complete


bay0: core behavior contracts PID 1 responsibilities (must implement)

1. SIGCHLD reaping (“Zombie Reaper”) If bay0 doesn’t reap, the VM will eventually die under podman process churn. Implement a SIGCHLD handler or a dedicated reap loop using waitpid(-1, WNOHANG) until empty.


2. Signal handling



SIGTERM/SIGINT: graceful shutdown path that flushes critical state (entropy seed, last-known-good time, db checkpoint) within bounded time, then exits.

Panic path: if something is unrecoverable, print, sync minimal state if possible, and reboot or halt based on policy.


3. Bounded IO for “survival reads” Any read from /persist required to boot must have a hard timeout (example: 200ms). If /persist is wedged, bay0 must continue and mark “dirty boot,” not hang.



State bridge pattern (mandatory) No module outside /state may touch /persist. All other code calls typed functions:

state::open()

state::migrate_if_needed()

state::list_personas()

state::set_persona_state()

state::get_active_persona()

state::set_active_persona()

state::read_nack_registry(), write_nack_registry()

state::read_last_good_time(), write_last_good_time()

state::read_entropy_seed(), write_entropy_seed()

state::append_genesis_log()


SQLite settings (durability)

WAL mode on ext4 persist.

synchronous=NORMAL (unless you have a reason to go FULL; don’t change casually)

schema versioning via PRAGMA user_version.

migrations run before reconcile.


Reconciliation on boot (mandatory) On boot:

read DB desired states

ask podman actual states

repair mismatches deterministically:

DB says running, container missing -> set stopped + log

DB says starting -> container may be partial -> rm -f + set stopped + log

DB says stopping -> ensure stopped + set stopped + log



The rule: the machine never believes “half states.” It converges.

Persona naming grammar (mandatory) Do not rely on a simple prefix. Use a strict grammar. Example:

internal system tenants: sys://ai, sys://compositor

user personas: p://work, p://gaming


Internally you may map these to podman container names like:

sys__ai

p__work


But the public CLI and DB should store the semantic names, and the mapping must be lossless and non-ambiguous.

Validation command (mandatory): bay0 --validate bay0 must provide a single executable health check that is non-destructive.

Minimum checks:

1. Freezer test



start a tiny test container, confirm CPU burn, pause, confirm CPU stops, unpause, confirm resumes, then delete it.


2. Umbilical test



confirm unix socket connect works to the AI socket path (or a stub socket in VM phase)

if compositor socket is used, do a real connect and verify peer credentials (SO_PEERCRED) match the sway process started by bay0.


3. Persist test



write a tiny marker to /persist/spine/validate.marker and read it back.

enforce timeouts; report if persist is degraded.


4. Log test



append a line to /persist/spine/genesis.log (or validate.log)


Output format requirement:

print a one-line summary: GREEN / YELLOW / RED

print failures with a short reason and the component name No walls of text.


Red-team hardening requirements (v0.1 baseline) These are not optional “later improvements.” They are part of v0.1 repo contract.

Hermetic build pinning

mkspine.sh uses pinned container digest. Never tag pulls.


Socket hijack protection (compositor)

If bay0 checks /run/wayland-0, it must do more than “file exists.”

It must attempt connect() and verify the peer creds are the compositor process it spawned (SO_PEERCRED).

If that fails, treat as not-ready and keep waiting.


Cgroup resource envelope (AI)

AI tenant runs with: --network none no-new-privileges cpu cap, cpu shares lower than personas memory cap and swap cap oom-score-adj set so AI is sacrificial compared to bay0/compositor

bay0 must set its own oom_score_adj to -1000 early in boot.


I/O priority for state sync

critical sync routines should use elevated IO priority where possible (ionice) or at minimum avoid being starved by persona IO storms.


Suspend / lid-close safety (VM can stub) Even if not fully wired in VM:

design requires an inhibit lock during flush operations to prevent half-sync corruption.


GPU surface detach before pause (Wayland stability) Before pausing a persona, bay0 must ensure the compositor is not actively scanning out a buffer that depends on a process about to freeze. Implementation can be minimal at first but must exist as a call site with logging. The contract is “no GPU deadlock because we froze the owner of the buffer.”

Thermal + PSI governor (minimal viable) In VM this may be mostly no-op, but code must exist:

read temp from /sys/class/thermal/... if present

read PSI from /proc/pressure/cpu and /proc/pressure/memory

avoid self-reinforcing throttle/oom loops by weighing PSI against thermal events


Anti-loop memory (NACK registry) If an upgrade fails and rolls back, the system must remember that the candidate hash is bad and refuse to reinstall it automatically.

VM install plan (exact steps)

1. Clone repo and build bay0 (host)



cargo build --release

ensure resulting binary runs: ./target/release/bay0 --help


2. Build spine image (hermetic)



scripts/mkspine.sh Artifacts must appear:

./vmlinuz

./build/spine.img


3. Create persist disk



scripts/mkpersist.sh (creates persist.img)


4. Boot VM



scripts/boot-qemu.sh Expected: serial console shows bay0 logs.


5. Run validation Inside the VM:



bay0 --validate (or if bay0 is only PID1, provide a spine CLI subcommand that calls validate, but do not create a second binary unless required. The preference is “single bay0 binary with subcommands.”)


Definition of done gates (what “done” means) Gate A: PID 1 boot + persist mount behavior

pass: bay0 runs, persist mounts or times out cleanly, logs visible on serial


Gate B: Reconcile + crash recovery

pass: start a persona, kill VM hard, reboot, bay0 reconciles DB vs podman, persona returns to stopped with no ghost containers


Gate C: Validate

pass: bay0 --validate prints GREEN on freezer + persist + log tests (umbilical can be YELLOW until AI tenant is wired)


Gate D: Umbilical + AI tenant (if included in v0.1 VM target)

pass: AI container is --network none and provides unix socket; validate confirms socket connect; a persona can access it via a bind-mounted socket path


Gate E: Switcher illusion (optional in VM milestone, required before calling it “product”)

pass: two personas running, switching pauses background, background CPU drops, focus returns instantly


Failure handling rules (how to code it)

1. Every external command must have:



timeout

captured stdout/stderr (at least on failure)

log line before and after execution


2. Never block the UI path If focus() is called and podman hangs, return quickly, mark desired state, let reconciler fix it.


3. Prefer “converge to stopped” on uncertainty If you don’t know whether a persona is safe, stop it, log why.


4. Keep system tenants protected No user persona name may collide with sys tenants by construction.



What NOT to do (common dev failure modes)

Do not replace podman with another runtime “because it’s simpler.”

Do not introduce systemd into Spine to “manage services.” bay0 is PID 1 by design.

Do not add network to Spine “just for convenience.”

Do not store state outside /persist.

Do not change the boot path away from initramfs+baked kernel without explicit directive.

Do not add new dependencies that expand the trust base unless they are strictly required.


Artifacts that must be committed for handoff completeness

docs/HANDOFF.md (derived from this)

scripts/mkspine.sh, mkpersist.sh, boot-qemu.sh

bay0 --validate implemented

bay0 pid1 reaping implemented

state module boundary enforced (no direct /persist writes elsewhere)

reconcile_on_boot implemented


Quick “red team” checklist for the dev (run before asking questions)

mkspine.sh uses pinned container digest

bay0 sets oom_score_adj=-1000

all survival reads time-bounded

compositor socket readiness verified by connect + peercred

persona naming grammar enforced

reconcile clears “starting/stopping” ghosts

validate returns clear GREEN/YELLOW/RED
