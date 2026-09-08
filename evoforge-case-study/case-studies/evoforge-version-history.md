
---
**MainbyteLabs Portfolio Sample**
Service: Technical Documentation / Setup Guide
Scenario: Full documentation sprint for EvoForge — setup guide, user docs, and technical reference.
Disclaimer: Client project. Published with permission. Identifying details may be omitted.
---

# Case Study: Documenting an AI\-Iterated Codebase — EvoForge

**What this is:** a worked example of tracking and explaining changes across
three AI\-assisted iterations of a real Python project, the way I'd document
version history for a client's evolving codebase.

## Background

EvoForge is a Python artificial\-life simulator: a persistent 2D world where
neural\-network\-driven agents move, eat, fight, and reproduce, with mutated
genomes passed to offspring. It's built on `pygame` for rendering and `numpy`
for the agent neural nets, structured as a standard `src/` package with a
CLI, test suite, and telemetry export.

I directed an AI coding assistant through three iterations of this codebase
over about five weeks (Aug 1 → Sep 6). Below is what changed at each step,
based on a direct diff of the source — not a changelog the AI wrote about
itself, but my own read of what the code actually did before and after.

## v1 → v2.1: from simulation to spectacle

v1 was a working simulation with no narration: agents lived, died, and
reproduced, but nothing on screen explained why the population looked the
way it did.

v2.1 added a presentation layer on top of the same simulation core:

- **Event logging** — `world.py` grew a `log_event()` method and a bounded
  `deque` of recent events (new lineages, kill\-streak milestones), plus a
  `flashes` queue so births and kills could be rendered as short\-lived
  visual pings instead of disappearing silently.
- **Procedural naming** — `genome.py` gained \~60 lines implementing
  `species_name()`, a deterministic name generator that reads an agent's
  traits (diet, speed, size, aggression, vision) and produces a label like
  "Savage Crimson Reaper" — same traits always produce the same name, with
  no extra state to store or keep in sync.
- **A "documentary" auto\-camera** — `app.py` and `world.py` added
  `pick_notable_agent()`, which picks whichever living agent is most
  interesting right now (oldest, deadliest, most prolific, etc.) and smoothly
  pans the camera to it, turning idle\-watching into something closer to a
  nature documentary.
- **Renderer** (`render.py`) grew from 263 to 411 lines to support trails,
  captions, and the new event/flash data feeding the screen.

Net effect: the simulation's *internal* logic didn't change — the same
agents, same rules — but it became legible and watchable instead of being a
black box you'd only inspect via saved JSON.

## v2.1 → v2.2: cutting scope, hardening what stayed

v2.2 is a refinement pass, not a growth pass. Two things happened at once:

**Removed:** the procedural naming system (`species_name()`, `hue_word()`,
and the noun/adjective pools in `genome.py`) was deleted outright —
`genome.py` returned to its exact v1 length and content. The "documentary"
mode in `app.py` (`K_d` toggle, `doc_caption`, timer\-based smoothing) was
also removed in its v2.1 form.

**Replaced with something narrower and more robust:** rather than naming
agents, v2.2 tracks numeric **records** (`max_generation`, `max_kills`,
`max_age`, `max_radius`) and only logs an event when a record is actually
broken — fewer, more meaningful log lines instead of a name generator's
flavor text. The camera feature came back as **`X` — spotlight mode**, a
simpler rotate\-through\-criteria auto\-follow (kills → age → radius → children
by fixed time window) with camera snapping straight to the target instead of
lerping. The event pipeline in `world.py` was renamed and simplified —
`log_event`/`flash` collapsed into `log()` and a single `frame_events` list
drained once per frame via `drain_events()` — and `render.py` picked up a
matching `_ingest_events()` / particle\-list renderer plus real day/night
background blending.

`app.py` also gained procedural sound: `_make_tone()` generates short sine\-
wave chimes with `numpy` at runtime for births and kills, so there's no
audio asset to ship, with a mute toggle (`M`) wired to the new event
pipeline.

Net effect: v2.2 is a **simplification and consolidation** release. It
removed a cosmetic feature (name generation) that added surface area without
changing simulation behavior, and rebuilt the event/camera/audio systems on
a single unified event queue instead of three separate ad hoc mechanisms.

## What this demonstrates

Documenting an AI\-iterated codebase isn't about restating commit messages —
it's reading the actual diff, telling growth apart from refactoring apart
from deliberate removal, and explaining *why* a change was worth making, not
just that it happened. That's the same skill this applies to a client's
codebase: version history, README updates, and change documentation that a
non\-author can actually trust, because it's grounded in what the code does,
not what a tool claims it did.
