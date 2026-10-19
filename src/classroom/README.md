# src/classroom — 3D Virtual Classroom add-on

All classroom features live in this folder. Upstream Hubs files are touched only through
small hooks, and each one is logged in [`/UPSTREAM_PATCHES.md`](../../UPSTREAM_PATCHES.md).

## What we reuse from Hubs (no code)

| Feature | Hubs mechanism | Where |
| :--- | :--- | :--- |
| Lock spawn / draw / pin / fly / voice / text | Room member permissions (`hub_channel.js` `HUB_CREATOR_PERMISSIONS`) | Room settings |
| Megaphone | Audio source (podium mic, `onlyMods`) → audio targets (speakers). Only users with `amplify_audio` are picked up. | Scene (Blender add-on / Spoke) |
| Breakout pod isolation | Audio zones (`bit-systems/audio-zone-system.ts`) | Scene |
| Hand raise | Presence `hand_raised` (People sidebar, name tag) | Built in |
| PDF slides | `media-pdf` + `NetworkedPDF` (page index is networked) | Built in |
| Mute a user | `hubChannel.mute()` — Reticulum checks `mute_users` | Built in |

## Classroom message protocol (Gather All, Seat Lock, Lecture Mode)

We use the existing hub channel `message` event with `type: "classroom"`.
Reticulum (`hub_channel.ex`, `handle_in("message")`) broadcasts any type and **overwrites
`session_id` with the real sender's**, so the receiver can trust it:

```
hubChannel.sendMessage({ cmd: "gather", waypoint: "podium" }, "classroom")
hubChannel.sendMessage({ cmd: "seatLock", on: true }, "classroom")
```

Receiver rule: act only if `presence.state[session_id].metas[0].roles` has `owner` or
`creator`, or the sender has `amplify_audio` / `mute_users` permission. Otherwise ignore it.

## Developing in Codespaces

On GitHub, choose **Code → Codespaces → Create codespace on `classroom`**. This needs a 4-core machine, and dependencies install automatically.

| Command | What it gives you |
| :--- | :--- |
| `npm run storybook` | Storybook on port 6006. **Build the Teacher HUD here**, in isolation and without a server. |
| `npm run test` | Lint + unit tests |
| `npm run build` | Production build (the same as CI) |

Voice and multi-user testing need the full stack, which comes in Phase 1 on our own server. Codespaces cannot carry WebRTC voice.

## CI

`.github/workflows/classroom-build.yml` builds the client image on every push to `classroom` and publishes it as
`ghcr.io/ben010618/hubs:classroom-latest` (also tagged with `classroom-<run>`).

## Known gaps (to build)

- Teacher HUD panel (one place for all controls)
- Gather All, Seat Lock, Lecture Mode (protocol above)
- Hand-raise **queue** ordered by time (raised hands exist; ordering and the HUD list don't)
- Pre-join card: Lite mode, mic check, privacy/screenshot rule
- Filipino locale (`src/assets/locales/fil.json` — none exists upstream)
- PPT → PDF conversion (server sidecar, not in this repo)
