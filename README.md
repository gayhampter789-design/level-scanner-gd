# Level Scanner — Geode Mod

A Geometry Dash mod that:
- **Records your inputs** while you play any non-default level and exports them as a `.gdr` or `.gdr2` replay file
- **Dumps level data** (objects, triggers, speed portals, song info) to a JSON file
- **Runs an autoplay pathfinder** that simulates GD physics and beam-searches for a valid path, then exports the solution as a replay and/or plays it back live as a bot

---

## Requirements

| Requirement | Version |
|---|---|
| Geometry Dash | 2.2074 |
| Geode SDK | 4.0.0+ |
| CMake | 3.21+ |
| C++ Compiler | MSVC 2022 / Clang 14+ / GCC 12+ |

---

## Building

### 1. Install Geode SDK

Follow the [official Geode guide](https://docs.geode-sdk.org/getting-started/):

```bash
# Set environment variable pointing to your Geode SDK clone
export GEODE_SDK=/path/to/geode-sdk   # Linux/macOS
set GEODE_SDK=C:\path\to\geode-sdk    # Windows CMD
```

### 2. Configure & Build

```bash
cd LevelScanner
cmake -B build
cmake --build build --config Release
```

The compiled `.geode` file will be at:
```
build/yourname.level-scanner.geode
```

### 3. Install

Copy the `.geode` file into your GD mods folder:

| Platform | Path |
|---|---|
| Windows | `%LOCALAPPDATA%\GeometryDash\geode\mods\` |
| macOS | `~/Library/Application Support/GeometryDash/geode/mods/` |
| Android | `/sdcard/Android/data/com.robtopx.geometryjump/files/geode/mods/` |

---

## Usage

### Recording Inputs
1. Open any **non-default** level's info page
2. Tap the green **Scan Level** button (bottom-left area)
3. Choose your format (`.gdr` or `.gdr2`) and press **Start Recording**
4. The mod enters the level — play normally
5. On death or completion, the replay is saved automatically

### Autoplay / Pathfinder
1. Open the **Scan Level** popup
2. (Optional) Enable **Live bot after autoplay** to watch it play in real-time
3. Press **Run Autoplay**
4. The mod parses the level, runs the beam-search pathfinder in the background, and either:
   - **Exports the replay** + shows a success notification, or
   - **Shows an error** explaining how far it got (e.g. *"Stuck at 47%"*)

---

## Output Files

All files are saved to:
```
Documents/GeometryDash/LevelScanner/
├── LevelName_20240101_120000.gdr       ← xdBot binary replay
├── LevelName_20240101_120000.gdr2      ← JSON extended replay
└── LevelName_scan.json                 ← Level data dump
```

### `.gdr` Format (xdBot binary)
```
[4 bytes] magic    = 0x72426478 ("xdBr")
[4 bytes] version  = 1
[4 bytes] fps      (float32 LE)
[4 bytes] input count
Per input:
  [4 bytes] frame  (uint32 LE)
  [1 byte]  flags  bit0=player2  bit1=down
```

### `.gdr2` Format (JSON)
```json
{
  "meta": { "level": "...", "id": 0, "fps": 240, "attempt": 1, "seed": 0 },
  "inputs": [
    { "frame": 142, "p2": false, "down": true },
    { "frame": 143, "p2": false, "down": false }
  ]
}
```

---

## Mod Settings

| Setting | Default | Description |
|---|---|---|
| `replay-format` | `gdr2` | Default export format |
| `bot-fps` | `240` | Physics simulation & replay FPS (60–360) |
| `autoplay-live` | `false` | Auto-start live bot after pathfinding |

---

## Architecture

```
src/
├── main.cpp              # All Geode hooks (LevelInfoLayer, PlayLayer, GJBaseGameLayer)
├── ScanPopup.cpp/.hpp    # In-game UI popup
├── InputRecorder.cpp/.hpp# Captures inputs during human play
├── LevelParser.cpp/.hpp  # Parses GJGameLevel objects into structured data
├── PhysicsSimulator.cpp/.hpp  # Frame-by-frame GD physics engine
├── Pathfinder.cpp/.hpp   # Beam-search pathfinder over simulated frames
├── BotPlayer.cpp/.hpp    # Injects replay inputs during live bot mode
└── ReplayExporter.cpp/.hpp    # Writes .gdr / .gdr2 files
```

### Pathfinder Algorithm
The autoplay uses a **beam search** over simulated game states:
- Each frame, every live branch forks into two: *jump* and *no jump*
- Dead branches (player collides with obstacle or falls) are pruned
- The beam is trimmed to the **64 furthest-progressed** branches each frame
- If any branch reaches the end of the level → solution found
- If all branches die → reports the furthest X% reached

> **Note:** The simulator models the cube gamemode physics. Slopes, ships, UFOs, balls, and wave are not yet fully simulated — the pathfinder will report failure on sections it cannot model.

---

## Known Limitations

- Pathfinder currently models **cube gamemode only** — other gamemodes will likely fail the solver
- Very large levels (10,000+ objects) may take several seconds to parse
- The `.gdr` format targets xdBot compatibility — other bot tools may use slightly different headers

---

## License

MIT — do whatever you want with it.
