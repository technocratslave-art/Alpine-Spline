Project: Alpine-Spine (Lego Linux)

0. One-sentence goal Ship an immutable Alpine “Spine” that owns hardware and runs multiple Linux “personas” as OCI containers, plus headless “system tenants” (AI, etc.) shared via Unix sockets, with instant user switching via a Spine-controlled switcher.

1. Non-negotiable invariants (acceptance gates)

Spine root filesystem is read-only at runtime. Only /persist is durable.

bay0 is the only orchestrator (PID 1 on Spine). No other privileged manager.

Personas are peers, not nested. No VM stacking. No “containers inside containers” model.

Hardware access is capability-based:

Wayland via socket

Audio via PipeWire socket

GPU via render node only (/dev/dri/renderD*, never DRM master)

Network via user-mode NAT (slirp) unless explicitly disabled

No raw /dev/input/* into tenants


System tenants:

--network none

no Wayland, no PipeWire

expose only Unix sockets under /run/spine/<service>/


Persona state survives rebuilds via /persist/home/<persona>/ and /persist/personas.db (or equivalent).


2. Tier model (what runs where) Tier 1: Spine (immutable)


Alpine base + kernel/firmware

Wayland compositor (wlroots-based: sway/cage to start)

Podman (or compatible OCI runtime)

bay0 (PID 1)

Switcher UI (Wayland client) + hotkey binding


Tier 2: System tenants (headless services)

ai-core (Ollama) is the reference system tenant

exports /run/spine/ai/ollama.sock

stores data at /persist/ai/*


Tier 3: User tenants (visible OS personas)

Ubuntu/Arch/etc as OCI images

persistent home bind-mounted from /persist/home/<id>/

mounts injected sockets (Wayland, PipeWire, AI) explicitly


3. Repo layout (final)


crates/bay0/ : PID 1 orchestrator + core policy enforcement

crates/spine/ : user CLI (calls bay0 over unix socket; v0 may call directly)

crates/switcher/ : Wayland overlay UI (thin client) OR scripts/config if using swaymsg initially

docs/ : architecture + contracts + threat boundaries (short, precise)

scripts/ : image build helpers, dev convenience

assets/ : icons, minimal UI art


4. v0 deliverable (must demo in under 2 minutes) Boot Spine -> bay0 starts compositor -> bay0 starts ai-core system tenant -> user launches two personas -> both personas can query the same ai-core via Unix socket -> switch between personas via hotkey overlay -> background persona is throttled/frozen.


Concrete v0 features:

spine persona add <id> <image>

spine persona start <id>

spine persona stop <id>

spine persona rm <id>

spine ps (shows running + focus)

spine focus <id> (switch focus; applies throttling)

spine service status (ai-core up/down, socket exists)


5. Process boundaries and interfaces (hard contracts)


5.1 bay0 control socket

Path: /run/spine/bay0.sock

Protocol: JSON lines (one request per line) or simple framed CBOR; keep it boring.

Commands:

persona.add, persona.start, persona.stop, persona.rm, persona.list, persona.focus

service.start ai-core, service.stop ai-core, service.status


5.2 Service socket injection

Host creates service socket in /run/spine/ai/ollama.sock

Persona mount: -v /run/spine/ai/ollama.sock:/run/ai.sock:ro

Inside persona: clients use curl --unix-socket /run/ai.sock http://localhost/...


5.3 Wayland and audio

Persona gets Wayland client socket: -v /run/wayland-0:/run/wayland-0

Optional PipeWire: -v /run/pipewire-0:/run/pipewire-0

No compositor inside persona required, but allowed (nested desktops are “just apps”).


5.4 GPU

If enabled: --device /dev/dri/renderD128 only

Never pass /dev/dri/card0


6. Implementation plan (in order, no detours)



Phase A: spine boots and runs bay0

Minimal initramfs/rootfs brings up:

mount /persist

create tmpfs /run

start compositor (sway/cage)

exec bay0 as PID 1 Acceptance: boot to launcher overlay with bay0 alive.



Phase B: persona lifecycle (no switcher yet)

bay0 wraps podman:

create container with home bind mount

mount Wayland socket

optional network slirp

start/stop/rm/list


Persist persona config in /persist/personas.db (SQLite) or /persist/personas.json + journal. Acceptance: start Ubuntu persona and see GUI on Wayland.


Phase C: system tenant ai-core (socket-only)

bay0 starts ai-core with:

--network none

mount /persist/ai/models

mount /run/spine/ai for socket output


Validate socket exists. Acceptance: curl from host namespace (optional) and from a persona.


Phase D: focus switching + throttling

Implement focus command:

Mark active persona in state

Apply background policy: cgroup freeze or CPU quota reduction

Bring target window to front via compositor control (swaymsg) in v0 Acceptance: hotkey overlay selects persona; background one stops burning CPU.


Phase E: polish (still v0)

Basic launcher list (icons, names)

spine doctor prints mounts + sockets + runtime checks

Crash-safe journal for operations (stage/commit) so “power loss mid-switch” doesn’t corrupt state.


7. Definition of done (what QA verifies)

Spine can reboot 10 times; /persist retains homes and persona registry; /run is ephemeral.

Two personas can run concurrently and render; switching is instant enough to feel native.

ai-core loads once; personas share it; no network ports exposed by ai-core; only the Unix socket.

Removing a persona deletes only its container state; /persist/home/<id> remains unless explicitly wiped.

No persona has access to raw input devices or DRM master devices.

Background persona throttling works (CPU drops near-zero or frozen).


8. Explicit non-goals for v0 (do not implement)

Verified boot chains, signature allowlists, digest pinning

SELinux/LSM policy frameworks

Complex per-file integrity checks

Custom compositor from scratch

Multi-user accounts/ACLs beyond basic filesystem permissions

Network isolation beyond slirp “good enough”


9. Naming, once and forever

Repo: Alpine-Spine

Immutable base: Spine

Orchestrator: bay0

Headless services: system tenants

Visible OS containers: personas

Switching UI: switcher
