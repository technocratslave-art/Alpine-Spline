Canonical layout for v0.1, with v0.2 lanes clearly separated so nobody “accidentally” drifts the core.

Alpine-Spine/
├── README.md
├── LICENSE
├── .editorconfig
├── .gitignore
├── Cargo.lock
├── Cargo.toml                      # workspace root
│
├── crates/
│   └── bay0/
│       ├── Cargo.toml
│       └── src/
│           ├── main.rs             # arg parsing, dispatch, top-level error policy
│           ├── init.rs             # bay0::init() – PID1 boot path
│           ├── pid1.rs             # SIGCHLD reaper, subreaper, child supervision
│           ├── reconcile.rs        # intent vs reality convergence loop
│           ├── validate.rs         # bay0 --validate feelers + health checks
│           ├── persona.rs          # persona grammar, lifecycle, freezer hooks
│           ├── umbilical.rs        # unix socket, SO_PEERCRED handshake, gates
│           ├── watchdog.rs         # v0.1 metabolic watchdog ladder (0-3)
│           ├── cgroup.rs           # cgroupv2 helpers (cpu.max, io.max, freezer)
│           ├── mount.rs            # mount namespace setup, tmpfs overlays, bind rules
│           ├── net.rs              # v0.1 “no global net” enforcement + persona net opt-in
│           ├── log.rs              # deterministic, minimal, serial-first logging
│           └── state/
│               ├── mod.rs          # state:: API surface (exclusive /persist owner)
│               ├── db.rs           # state.db read/write + checksum + rollback to .bak
│               ├── lkg.rs          # last-known-good hash + boot attestation records
│               └── persist.rs      # mount, timeout rules (200ms), dirty-boot handling
│
├── scripts/
│   ├── mkspine.sh                  # one-shot hermetic build (pinned digest)
│   ├── mkpersist.sh                # /persist image creation + format + labels
│   ├── boot-qemu.sh                # serial-first VM boot harness
│   ├── install-usb.sh              # write image to USB + safety checks (no auto-magic)
│   └── doctor.sh                   # optional: host-side helper (prints hashes, sanity)
│
├── images/
│   ├── spine.sqsh                  # build artifact (optional committed? usually not)
│   ├── spine.uki                   # build artifact (optional committed? usually not)
│   └── hashes/
│       ├── spine.sha256            # canonical hash card source (printable)
│       └── spine.sig               # detached signature (optional in v0.1)
│
├── docs/
│   ├── HANDOFF.md                  # constitution + “what devs will try to add”
│   ├── THREAT_MODEL.md             # threat reality matrix + what we don’t promise
│   ├── INVARIANTS.md               # five Spine laws, exact definitions
│   ├── PERSONAS.md                 # persona types: work/web/ai/burner + defaults
│   ├── UMBILICAL.md                # push/pull model, no shared folders, credential checks
│   ├── WATCHDOG.md                 # metabolic signals + integer ladder + safe focus
│   ├── BUILD.md                    # hermetic build steps, pinned digests, reproducibility
│   ├── INSTALL.md                  # USB/VM install, BIOS/F12 sticker instructions
│   └── ROADMAP_v0_2.md             # electrical spine (WDT/GPIO) – explicitly out of v0.1
│
├── configs/
│   ├── personas/
│   │   ├── easy.toml               # optional (if you keep “easy” as a persona template)
│   │   ├── work.toml
│   │   ├── web.toml
│   │   ├── ai.toml
│   │   └── burner.toml
│   └── bay0.toml                   # minimal governor config (paths, defaults, policy)
│
├── third_party/
│   └── NOTICE                      # if you vendor anything (try not to)
│
└── v0_2/
    ├── hardware/
    │   ├── wdt/                    # schematics, BOMs, microcontroller firmware
    │   ├── gpio/                   # pin maps per platform
    │   └── platform_matrix.md      # what can actually support heartbeat sovereignty
    └── docs/
        └── ELECTRICAL_SPINE.md     # “tamper=brownout=death” spec, definitions, limits

Notes that matter (so nobody “improves” you into drift):

crates/bay0/src/state/ is the only place allowed to touch /persist. No exceptions.

v0_2/ is physically separated. If someone wants to talk about watchdog hardware, they do it there, not in bay0 v0.1.
