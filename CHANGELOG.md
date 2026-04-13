# Changelog

All notable changes to this project are documented here.

---

## [1.4.0] — Display Redesign for N1 Screen

### Changed
- Completely redesigned the knob display SVG for readability on the small N1 screen
- Volume percentage is now rendered at **30px** (was 15px) — dominant, readable from a distance
- Channel name rendered at **14px bold** (was 11px) at the top with a separator line
- Replaced the decorative arc dial with a **horizontal level bar** at the bottom, keeping full level information without sacrificing space for the number
- Mute indicator is now a small text line at the very bottom edge rather than a badge overlapping the volume

---

## [1.3.0] — Display and Sync Fixes

### Fixed
- **Caution triangle (⚠) always showing** — the GoXLR Utility WebSocket wraps responses in a `{ id, data }` envelope; the plugin was reading `msg.Status` directly which never matched. Fixed by unwrapping `msg.data` before processing
- **WS GetStatus request** now correctly includes the required `id` field (`{ id: 1, data: "GetStatus" }`)
- **Volume replaced by "MUTED" text** — the display now always shows the actual `XX%` value even when a fader is muted. Mute state is communicated via arc/bar colour and a small badge, not by hiding the number
- **Arc/bar collapsed when muted** — `fillFrac` was forced to `0` when muted; the level indicator now always reflects the true volume regardless of mute state

### Added
- **5-second HTTP polling fallback** — polls `GetStatus` every 5 seconds so the display always shows a real value even if WebSocket patch events are delayed or missed

---

## [1.2.0] — Knob Press Mute + New Plugin ID

### Added
- **Knob press → mute toggle** — pressing the N1 knob calls `SetFaderMuteState` on the GoXLR Utility API, toggling between the configured mute route and `Unmuted`
- **Mute configuration** in the Property Inspector:
  - Enable/disable knob press mute (toggle switch)
  - Select which fader to mute (A / B / C / D)
  - Choose mute route: **Muted to All** (full silence) or **Muted to X** (stream mute only)
- **Mute state indicator** on the display: arc/bar turns red, volume stays visible, small fader badge appears
- **Live mute sync** via WebSocket patch events — display updates instantly if fader is muted from any source

### Changed
- Plugin ID changed from `com.goxlr.volumecontrol` to **`com.jomi.goxlrvolumecontrol`**
- Author set to `Jomi`

---

## [1.1.0] — Runtime Fix

### Fixed
- **Plugin was silently non-functional** — `CodePath` was set to `plugin.js` with a `Nodejs` block, but the plugin was written for the browser-context JavaScript SDK (which expects `connectElgatoStreamDeckSocket` on `window`). Switched `CodePath` to `index.html` so Stream Dock loads the plugin in a hidden Chromium window where `WebSocket`, `fetch`, and `window.*` all work correctly
- **Display never rendered** — consequence of the above; `setImage` was never sent because registration never succeeded
- **Incorrect plugin install path** in README — corrected to `%APPDATA%\HotSpot\StreamDock\plugins\`

---

## [1.0.0] — Initial Release

### Added
- Knob rotate → adjust GoXLR channel volume via `SetVolume` HTTP command
- Channel selectable from all 11 GoXLR channels in the Property Inspector
- Configurable knob step size (1–20% per tick)
- SVG display rendered to N1 knob screen showing channel name and volume %
- Real-time volume sync via GoXLR Utility WebSocket JSON Patch events
- Auto-reconnect to GoXLR Utility daemon on disconnect
- Monitors `goxlr-daemon.exe` / `goxlr-daemon` process for launch/terminate events
