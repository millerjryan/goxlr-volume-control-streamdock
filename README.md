# GoXLR Volume Control — Stream Dock Plugin

A plugin for the **VSDinside N1 Stream Dock** that lets you control any channel volume in the [GoXLR Utility](https://github.com/GoXLR-on-Linux/goxlr-utility) using the N1's hardware knob encoder.

![Plugin version](https://img.shields.io/badge/version-1.4.0-blue)
![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS-lightgrey)
![License](https://img.shields.io/badge/license-MIT-green)

---

## Features

- **Rotate** the N1 knob to raise or lower the selected GoXLR channel volume
- **Press** the knob to toggle mute on a configurable fader (Muted to All or Muted to X)
- **Live display** on the N1 screen shows the channel name, current volume %, and a level bar
- **Real-time sync** via GoXLR Utility WebSocket — display updates instantly from any source (hardware faders, GoXLR UI, etc.)
- **Configurable** channel, knob step size, target fader, and mute route — all set in the Property Inspector
- **Auto-reconnects** if the GoXLR Utility daemon stops and restarts

---

## Screenshots

```
┌─────────────┐
│ Headphones  │  ← channel name
│             │
│    75%      │  ← large volume display
│             │
│ ▓▓▓▓▓▓▓░░░ │  ← level bar
└─────────────┘
```

When muted:
```
┌─────────────┐
│    Mic      │
│             │
│    60%      │  ← volume stays visible
│             │
│ ▓▓▓▓▓▓░░░░ │  ← bar turns red
│ FADER A MUTED│
└─────────────┘
```

---

## Requirements

| Requirement | Details |
|---|---|
| Hardware | VSDinside Stream Dock N1 |
| Software | VSD Craft (Stream Dock app) |
| GoXLR app | **GoXLR Utility** (open-source) — NOT the official TC-Helicon app |
| GoXLR Utility version | v1.0.0+ (HTTP API on port 14564) |
| OS | Windows 10+ or macOS 10.14+ |

> ⚠️ **Important:** This plugin requires the [GoXLR Utility](https://github.com/GoXLR-on-Linux/goxlr-utility) daemon, not the official TC-Helicon app. The two applications cannot run at the same time.

---

## Installation

### Option A — Install from release zip

1. Download the latest `com.jomi.goxlrvolumecontrol.sdPlugin.zip` from the [Releases](../../releases) page
2. Extract and place the `com.jomi.goxlrvolumecontrol.sdPlugin` folder in your Stream Dock plugins directory:
   - **Windows:** `%APPDATA%\HotSpot\StreamDock\plugins\`
   - **macOS:** `~/Library/Application Support/HotSpot/StreamDock/plugins/`
3. Restart VSD Craft

### Option B — Build from source

```bash
git clone https://github.com/YOUR_USERNAME/goxlr-volume-control-streamdock.git
```

Then copy the `com.jomi.goxlrvolumecontrol.sdPlugin` folder directly into your plugins directory — no build step needed, the plugin is pure HTML/JavaScript.

---

## Setup

1. Make sure **GoXLR Utility** is running (look for its icon in the system tray)
2. In VSD Craft, assign a **Knob** slot to the **GoXLR Volume** action (found under the *GoXLR* category)
3. Click the action to open its **Property Inspector** and configure:

| Setting | Description |
|---|---|
| GoXLR Channel | The channel whose volume the knob controls (e.g. Headphones, Mic, Game…) |
| Knob Step Size | How many % each tick of the knob changes the volume (1–20%) |
| Enable knob press mute | Turn on/off the press-to-mute feature |
| Fader to Mute | Which GoXLR fader (A/B/C/D) is toggled when you press the knob |
| Mute Route | **Muted to All** — full silence; **Muted to X** — stream mute only |

---

## How It Works

### Architecture

```
N1 Knob → VSD Craft → Stream Dock WebSocket → Plugin (index.html)
                                                    ↕
                                          GoXLR Utility Daemon
                                          HTTP  localhost:14564/api/command
                                          WS    localhost:14564/api/websocket
```

The plugin runs in a hidden Chromium window inside VSD Craft (the standard Stream Dock JavaScript SDK pattern). It communicates with two endpoints simultaneously:

- **HTTP POST** to `/api/command` — sends `SetVolume` and `SetFaderMuteState` commands
- **WebSocket** at `/api/websocket` — receives live JSON Patch events for instant display updates

Volume values in the GoXLR Utility API are integers **0–255**, converted to percentages for display.

### Key API Commands

**Set volume:**
```json
{ "Command": ["<mixer_serial>", { "SetVolume": ["Headphones", 200] }] }
```

**Toggle fader mute:**
```json
{ "Command": ["<mixer_serial>", { "SetFaderMuteState": ["A", "MutedToAll"] }] }
```

**Mute state values:** `Unmuted` | `MutedToAll` | `MutedToX`

### Stream Dock Events Used

| Event | Purpose |
|---|---|
| `dialRotate` | Knob turned — adjusts volume by `ticks × stepSize` |
| `dialDown` | Knob pressed — toggles fader mute state |
| `willAppear` | Action placed on canvas — initialises instance, fetches volume |
| `willDisappear` | Action removed — cleans up instance |
| `didReceiveSettings` | Settings saved — applies channel/step/mute changes |
| `setImage` | Pushes rendered SVG to the N1 knob screen |

---

## Project Structure

```
goxlr-volume-control-streamdock/
│
├── com.jomi.goxlrvolumecontrol.sdPlugin/   ← installable plugin folder
│   ├── manifest.json                        ← plugin metadata & action definitions
│   ├── index.html                           ← main plugin logic (JS, browser context)
│   ├── inspector.html                       ← Property Inspector settings UI
│   └── images/
│       ├── plugin_icon.svg                  ← 128×128 plugin icon
│       ├── action_icon.svg                  ← 40×40 action list icon
│       └── category_icon.svg                ← 48×48 category icon
│
├── README.md
├── CHANGELOG.md
├── LICENSE
└── .gitignore
```

---

## Supported GoXLR Channels

| Channel | Description |
|---|---|
| Mic | Microphone input level |
| LineIn | Line In input level |
| Console | Console/gaming input level |
| System | System audio level |
| Game | Game audio level |
| Chat | Chat/voice input level |
| Sample | Sampler channel level |
| Music | Music channel level |
| Headphones | Headphone output level |
| MicMonitor | Mic monitoring level |
| LineOut | Line Out level |

---

## Troubleshooting

**Display shows `---` after placing the action**
- Ensure GoXLR Utility is running — the plugin polls every 5 seconds, so it should update quickly
- Open `http://localhost:14564` in a browser to confirm the daemon is accessible

**Volume changes in the display but not on the GoXLR**
- Check the correct channel is selected in the Property Inspector
- Confirm your GoXLR device is recognised in the GoXLR Utility UI

**Knob press does nothing**
- Make sure "Enable knob press mute" is toggled on in the Property Inspector
- Verify the selected fader (A/B/C/D) exists on your device (GoXLR Mini only has A–D as well)

**Display goes grey / offline**
- The GoXLR Utility daemon may have stopped — the plugin auto-reconnects every 10 seconds

---

## Contributing

Pull requests are welcome. For significant changes, please open an issue first to discuss what you'd like to change.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Commit your changes (`git commit -m 'Add my feature'`)
4. Push and open a Pull Request

---

## Acknowledgements

- [GoXLR Utility](https://github.com/GoXLR-on-Linux/goxlr-utility) — the open-source GoXLR daemon this plugin talks to
- [VSDinside Plugin SDK](https://github.com/VSDinside/VSDinside-Plugin-SDK) — Stream Dock plugin development framework
- [Space Plugin SDK Docs](https://sdk.key123.vip/en/) — official SDK documentation

---

## License

MIT — see [LICENSE](LICENSE) for details.
