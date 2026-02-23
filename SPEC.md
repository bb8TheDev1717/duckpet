# DuckPet – Project Spec

## Concept
Desktop pet (Duck or Dino) that lives on the desktop/taskbar, reacts to user behavior,
detects running music apps and vibes along, does chaotic prank events, and has a settings panel.
Built as a joke app for a friend – fully consensual prank.

## Tech Stack
- **Language:** C# + Godot Engine
- **Why Godot:** Native 3D model support, C# scripting, exports to Windows + Linux without Wine/JAR
- **Target:** Windows 11 x64 (primary), Arch Linux x64 (secondary, same codebase)
- **Output:** `DuckPet.exe` (Windows), `DuckPet` binary (Linux)
- **Assets hosted on:** GitHub Repo (sprites, sounds, models)

## Cross-Platform Asset Download
- On first launch: detect OS via `RuntimeInformation.IsOSPlatform`
- Download assets from GitHub releases via `HttpClient`
- Cache to `%APPDATA%/DuckPet/` (Windows) or `~/.local/share/duckpet/` (Linux)

## Autostart
- **Windows:** Copy exe to Startup folder (`%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup\`)
- **Linux:** Write `.desktop` file to `~/.config/autostart/`

## Window
- Transparent, borderless, always-on-top
- Click-through on empty areas, clickable on duck sprite only
- Sits on desktop layer (below other windows except when interacting)

---

## Skins
| Skin | Description |
|---|---|
| Duck (default) | Yellow duck, sunglasses, 3D model |
| Dino | Green dino reskin, same states + animations |

Switchable in Settings panel.

---

## Music Detection (HIGH PRIORITY FEATURE)
- Poll running processes every 5 seconds
- Detected apps: `Spotify`, `Amazon Music`, `iTunes`, `Apple Music`, `Tidal`, `Deezer`, `YouTube Music`
- **If music app is detected as active/playing:**
  - Duck teleports to sit on top of that app's taskbar button (toolbar)
  - Duck puts on sunglasses
  - Duck dances – 3-move animation loop (wiggle left, wiggle right, spin) until song/app stops
  - When music stops → duck returns to previous state
- **How to detect window position:** `FindWindow` + `GetWindowRect` (Windows), `xdotool`/`wmctrl` (Linux)

---

## Duck States & Behaviors

### Idle
- Duck stands on desktop, wiggles, occasional blink
- Sunglasses on by default
- Idle sounds (optional quack)

### Sitting on Taskbar / App Toolbar
- See Music Detection above
- Can also randomly hop onto any open window's title bar

### Sleeping
- Duck lies down, snore animation, Zzz speech bubble
- Triggered after configurable timeout (default: 10 min no interaction)
- **Click while sleeping → PC shuts down immediately**
  - Windows: `shutdown /s /t 0`
  - Linux: `shutdown now`

### Running from Cursor
- Duck detects cursor within ~150px radius
- Runs away, bounces off screen edges
- Panic animation + speed lines

### Crashout Mode
- Triggered randomly (rare) or via right-click → "Make him crashout"
- Duck shakes, eyes go crazy, brief scream animation
- **Nuke:** Duck pulls out nuke, throws it → full-screen bright white flash overlay, stays 10 seconds, fades out

### Capybara Event (random, rare ~every 30-60 min)
- Capybara sprite runs across the screen left→right
- Duck pulls out hunting rifle, aims, shoots
- Blood splatter VFX appears on screen
- Blood slowly drips down over 15 seconds, then fades

### Snake Mini-Game
- Triggers after configurable idle timeout (cursor not moved)
- Small overlay grid appears, duck becomes snake head
- Duck auto-plays or user can take control (arrow keys)
- Exits immediately when cursor moves

---

## Settings Panel
**Settings is first priority to implement.**
Open via: right-click on duck → "Settings"

| Setting | Type | Options |
|---|---|---|
| Skin | Dropdown | Duck / Dino |
| RGB Mode | Toggle | Duck pulses rainbow colors |
| Crashout Style | – | Nuke only |
| Sleep Timeout | Slider | 1–60 min |
| Snake Idle Timeout | Slider | 1–30 min |
| Autostart | Toggle | Enable/disable |
| Volume | Slider | Duck sound effects |

---

## VFX Summary
| Effect | Trigger | Duration |
|---|---|---|
| Nuke white flash | Crashout | 10s |
| Blood splatter + drip | Capybara event | 15s |
| RGB pulse | Settings toggle | continuous |
| Speed lines | Running from cursor | while running |
| Zzz bubble | Sleeping | while sleeping |

---

## 3D Models / Animations
- Models: 3D (`.glb`/`.gltf`) hosted on GitHub, downloaded on first launch
- If 3D not available → fallback to 2D sprite sheets
- Required animations per character:
  - idle_stand, idle_blink
  - dance_wiggle_left, dance_wiggle_right, dance_spin (3-move music loop)
  - sleep_loop, sleep_snore
  - run_left, run_right
  - crashout_shake
  - shoot_rifle
  - sunglasses_on (transition)

---

## Project Structure (Godot + C#)
```
DuckPet/
├── project.godot
├── Scripts/
│   ├── Main.cs               # Entry point, window setup
│   ├── Duck.cs               # Duck entity, state machine
│   ├── MusicDetector.cs      # Process polling, window position
│   ├── CursorTracker.cs      # Mouse position, proximity detection
│   ├── IdleTracker.cs        # Idle timeout logic
│   ├── SettingsPanel.cs      # Settings UI
│   └── Platform/
│       ├── Autostart.cs      # OS-specific autostart
│       └── AssetDownloader.cs
├── States/
│   ├── IdleState.cs
│   ├── SleepingState.cs
│   ├── MusicDanceState.cs
│   ├── RunningState.cs
│   ├── CrashoutState.cs
│   └── SnakeState.cs
├── VFX/
│   ├── NukeEffect.cs
│   └── BloodSplatter.cs
├── Events/
│   └── CapybaraEvent.cs
└── Assets/                   # Downloaded at runtime from GitHub
    ├── models/duck/
    ├── models/dino/
    ├── sfx/
    └── music/
```

---

## Implementation Order
1. **Settings Panel** (UI, all toggles/sliders, skin switch) ← START HERE
2. Godot project setup, transparent always-on-top window
3. Duck 3D model rendering + idle animation
4. State machine skeleton
5. Music detection + dance state
6. Cursor proximity → running state
7. Sleep state + shutdown on click
8. Crashout VFX (nuke flash)
9. Capybara event
10. Snake mini-game
11. Autostart logic (Win + Linux)
12. Asset downloader (GitHub)
13. Linux build + test

---

## Build Commands
```bash
# Windows
dotnet build -r win-x64

# Linux
dotnet build -r linux-x64

# Or via Godot export presets (recommended)
```

## GitHub Repo Structure (suggested)
```
/                    ← Godot project source
/releases/           ← Built binaries
/assets/             ← Hosted asset packs (models, sfx)
  /duck-3d/
  /dino-3d/
  /sfx/
SPEC.md              ← This file
```
