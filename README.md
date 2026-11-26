# XCOM 2 TTRPG Engine

Turning **XCOM 2: War of the Chosen** into the tactical/strategy engine
for a live, GM-run tabletop campaign — four players + one GM, run over
voice chat and a shared screen, with the GM quietly controlling enemies,
spawns, and mission state from a separate device.

This is a personal project, built incrementally, documented as it's
built rather than designed up front. Fan-made; not affiliated with or
endorsed by Firaxis Games, 2K Games, or Take-Two Interactive. See
**Credits & IP** below.

## Why XCOM 2

Running a tactical skirmish TTRPG session by hand (grid, minis, dice,
stat sheets) is slow. XCOM 2 already has: cover-based tactical combat,
a strategy layer (resources, research, a geoscape, a looming Avatar
countdown), a soldier/class/ability system, and — critically — a mod
SDK and console that expose almost all of it to external control. The
bet here is that modding the game to accept a human GM's commands is
less work than building a tactics engine from scratch.

## Architecture

The design converges on a **GM-client split**, not multiplayer:

```
┌─────────────────────┐        ┌──────────────────────────┐
│   Player laptop      │        │       GM laptop            │
│                      │        │                            │
│  XCOM 2 (modded)     │◄──────►│  GM overlay / command panel│
│  + Curtain app       │  TCP   │  (console commands today,  │
│  (shown via Parsec   │        │   custom panel later)      │
│   to remote players) │        │                            │
└─────────────────────┘        └──────────────────────────┘
```

- **Players** see and control their squad on one machine, shared to
  remote players via Parsec (Exclusive Input Mode, one controller at a
  time — matches XCOM's one-unit-at-a-time turn structure).
- **The GM** runs enemies, spawns, alert states, and mission/strategy
  state from a second device, without exposing the console or a 3D
  "GM's-eye view" of hidden units to the players.
- **Curtain** (planned) is a small always-on-top helper on the player
  laptop: when closed, it shows players a placeholder image, blocks
  their injected input, and lets the GM play the real game underneath
  using a capture-excluded window Parsec's stream can't see. This
  solves the "two screens, one machine" problem without needing Parsec
  to support anything it doesn't.

Everything is staged in tiers so each piece is independently testable
before the next is built on top of it.

## Status

| Tier | What it is | Status |
|---|---|---|
| **0 — Console/keybind toolkit** | Launch option, existing Workshop mods, `XComInput.ini` keybinds mapped to vanilla/mod console commands. No custom code. | 🔶 in progress — keybinds written, launch option & mod subscription pending manual steps |
| **1 — Custom GM mod** | UnrealScript mod (built on Community Highlander) adding commands Tier 0 can't reach: per-unit AI handoff, alert toggling, reinforcement calls, dynamic objectives, multi-unit spawn/remove, forced win/loss. | ⬜ not started |
| **2 — GM overlay app** | Standalone panel (replaces manual keybinds) talking to the mod over a local socket: unit list, spawn/remove, alert toggles, reinforcement control. | ⬜ not started |
| **3 — Curtain app** | Capture-exclusion overlay on the player laptop so the GM can play the real 3D view during their turn without it reaching Parsec viewers. | ⬜ not started |
| **4 — Map generator** | Quadrant-based plot/parcel generator with lockable regions, built on the SDK's existing plot/parcel system. | ⬜ not started (hardest piece) |

### Tier 0 keybinds (current)

Set in `XComInput.ini` (user config, not tracked in this repo — see
`.gitignore`). Requires the `-allowconsole` launch option.

Bound in `[XComGame.XComTacticalInput]`, not `[Engine.PlayerInput]` —
XCOM 2 uses a separate input class for the tactical mission layer that
claims keys independently of the menu/strategy layer, and it already
claims bare `F5`-`F8` for its own internal actions. Keybinds meant to
fire mid-mission have to go in that section specifically, or they're
silently never seen. `Ctrl+Alt+<letter>` is used instead of bare F-keys
since nothing else in that section claims those combos.

| Key | Console command | Effect |
|---|---|---|
| `Ctrl+Alt+A` | `SkipAI` | toggle: enemy AI stops/resumes taking turns |
| `Ctrl+Alt+S` | `X2allowselectall 1` | take control of every unit, including enemies |
| `Ctrl+Alt+R` | `X2allowselectall 0` | release that control |
| `Ctrl+Alt+D` | `DropUnit AdvTrooperM1 1` | spawn an ADVENT trooper on the enemy team at the cursor |
| `Ctrl+Alt+X` | `KillClosestUnitToCursor` | remove a unit near the cursor, no XP/loot |
| `Ctrl+Alt+K` | `KillAllAIs` | clear all enemies |
| `Ctrl+Alt+Shift+E` | `EndBattle` | force-end the mission as a loss (extra modifier guards the one irreversible command here) |

## Repo structure

```
.
├── README.md          this file
├── CLAUDE.md           repo policy for AI-assisted changes (public repo, IP rules)
├── .gitignore          excludes game assets/binaries/saves — see below
└── (future)
    ├── mod/            UnrealScript source for the Tier 1 GM mod (Community Highlander-based)
    ├── gm-panel/       Tier 2 GM overlay app
    ├── curtain/        Tier 3 capture-exclusion helper
    └── docs/           design notes per tier
```

Nothing from the XCOM 2 install, Workshop mods, saves, or character
pools is stored in this repo. Local game config/mod edits are documented
here (e.g. the keybind table above) but live in the user's own
`...\Documents\my games\XCOM2\` folder.

## Credits & IP

- **XCOM 2** and **XCOM 2: War of the Chosen** © Firaxis Games / 2K
  Games / Take-Two Interactive. This project is an unofficial, fan-made,
  non-commercial mod/tooling layer on top of a game the author owns and
  has legitimately licensed. No game assets or copyrighted content are
  redistributed here.
- **[WOTC] Community Highlander** (X2CommunityCore) — the shared
  framework this project's future custom mod will build on.
  https://steamcommunity.com/sharedfiles/filedetails/?id=1134256495
- **[WOTC] Alien Hunters Community Highlander** (X2CommunityCore) —
  required companion plugin when the base Highlander and the Alien
  Hunters DLC are both present.
  https://steamcommunity.com/sharedfiles/filedetails/?id=2534737016
- **[WOTC] XpanD's Console Commands** (XpanD) — console commands used
  for Tier 0 soldier/mission control.
  https://steamcommunity.com/sharedfiles/filedetails/?id=3113132590
- Earlier research for this project referenced **Grimy's Console
  Commands** and **[WOTC] Additional Soldier Console Commands**
  (shiremct) as prior art; both have since been removed from the Steam
  Workshop and are not used by this project, but are credited here for
  the ideas they demonstrated were possible.

If you use or build on this project, extend the same courtesy: credit
the modding tools and SDKs you depend on.
