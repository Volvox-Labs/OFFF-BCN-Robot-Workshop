# Robot Previz — Current State

**Volvox Labs · Intro to Robotic and Kinetic Art Workshop**

A single-file, browser-based tool for designing and animating a 6-axis robot arm and a connected screen output — no install, no build step.

---

## How to Run

```bash
# Option 1 — open directly
open robot-previz/index.html

# Option 2 — serve locally (avoids any CORS edge cases)
npx serve robot-previz
```

Also deployable as-is on **GitHub Pages** (static file, no backend required).

---

## Feature Overview

### 1. 3D Robot Arm (Forward Kinematics)

Models the **uFactory xArm 6** using Three.js procedural geometry (no external model files). Six joints in a parented hierarchy:

| Joint | Name            | Axis | Range         |
|-------|-----------------|------|---------------|
| J1    | Base Rotation   | Y    | −360° → +360° |
| J2    | Shoulder        | Z    | −118° → +120° |
| J3    | Elbow           | Z    | −225° → +11°  |
| J4    | Wrist Roll      | X    | −360° → +360° |
| J5    | Wrist Pitch     | Z    | −97°  → +180° |
| J6    | End Effector    | X    | −360° → +360° |

The active/selected joint glows cyan. Camera uses OrbitControls (orbit, zoom, pan).

---

### 2. Joint Control Panel (Sidebar)

Six sliders — one per joint — on the right sidebar. Dragging a slider immediately updates the 3D arm. Displays degree value next to each label.

When a keyframe is selected on the timeline, the sliders show that keyframe's pose. Editing a slider while a keyframe is selected **updates that keyframe** in place.

---

### 3. Keyframe Timeline

Bottom panel. The core animation workflow:

- **Playhead** — draggable vertical cursor showing current time
- **Keyframe markers** — diamond shapes on the track, draggable to retime
- **Add Keyframe** — captures current joint angles at the current playhead position
- **Select / Delete** — click a marker to select it; DELETE button appears
- **Duration** — adjustable total duration (top bar, default 5 seconds)
- **Play / Pause / Stop / Loop** — full transport controls

Playback interpolates between keyframes using **smoothstep** (ease-in/out) for fluid motion.

---

### 4. Screen Object

A physical 2D screen represented as a `THREE.PlaneGeometry` in the 3D scene. Toggled via the **SCREEN** button in the top bar.

**Two placement modes** (sidebar, SCREEN PLACEMENT section):

- **Fixed** — screen sits at a configurable world position (X/Y/Z) and Y rotation; moves independently of the robot
- **Mounted** — screen is parented to J6 (end effector), so it moves with the robot arm

**Size controls:** width, height, scale sliders. Optional bezel frame toggle.

The screen renders a live `CanvasTexture` (1280×720 offscreen canvas) and updates every animation frame.

---

### 5. Screen Content Editor

Full-screen overlay (EDIT SCREEN CONTENT button). Composites a stack of layers drawn onto the screen texture.

**Layer types:**

| Icon | Type       | Description                           |
|------|------------|---------------------------------------|
| ▬    | Background | Flat HSL color fill for the screen    |
| ○    | Circle     | Filled circle                         |
| ■    | Rectangle  | Filled square/rectangle               |
| ▲    | Triangle   | Filled triangle                       |
| ╱    | Line       | Stroked line                          |
| ◻    | Cube (3D)  | Three.js BoxGeometry in screen space  |

**2D layer properties:** Hue, Saturation, Lightness, Size, Pos X/Y (normalized 0–1), Opacity, Rotation°

**3D Cube layer properties:** Color (HSL), Position X/Y/Z (screen-local metres), Rotation X/Y/Z (degrees), Scale, Emissive glow, Wireframe toggle

The 3D cube lives in the coordinate space of the screen object — it is parented to `screenGroup`, so it moves with the screen in both Fixed and Mounted modes. Position X/Y maps across the screen face (±0.8m, ±0.45m); Z is depth popping forward from the screen surface.

---

### 6. Parameter Mapping (Signal Flow)

Each layer (2D or 3D) can have any number of **mappings** that drive a layer property from a joint angle in real time.

Each mapping defines:
- **Joint** (J1–J6) — source signal
- **Property** — target (e.g. Hue, Size, Pos X, Rot Y)
- **Output range** — min/max value the property is driven to

The joint angle is normalized to its full range, then remapped to the output range. Mappings are additive and override the layer's base value at runtime.

**Signal Flow overlay** (`≈` button, top bar) — draws a patch-bay diagram showing all active joint→property connections as animated bezier curves. Line brightness and thickness reflect the current joint value live.

---

### 7. PiP Preview

A small picture-in-picture overlay in the bottom-left corner of the 3D viewport (480×270px) showing the current screen texture output. Toggled with the **PiP** button in the top bar.

---

### 8. Preset System

**PRESETS** button (top bar) opens an overlay with two sections:

**Built-in presets:**

| Name         | Description                                 |
|--------------|---------------------------------------------|
| Blank Canvas | Default home pose, screen off               |
| Pendulum     | J1 sweeps ±60°, circle hue follows          |
| Breath       | J2/J3 rise and fall, circle pulses in size  |
| Scan         | J1 full sweep, line scans across X          |
| Wave         | All joints animated, multi-shape composition|

**User presets:** Type a name and click SAVE. Stored in `localStorage` under key `rvp-user-presets`. Persists across browser sessions.

Applying a preset replaces all joints, keyframes, screen state, and screen content in one action.

---

## UI Layout

```
┌──────────────────────────────────────────────────────┐
│  VOLVOX LABS  Robot Previz  [SCREEN] [PiP] [≈] [PRESETS] [Duration] │
├───────────────────────────────────────┬──────────────┤
│                                       │ SCREEN       │
│         THREE.JS VIEWPORT             │  PLACEMENT   │
│         (OrbitControls)               │  (when on)   │
│         [PiP overlay]                 ├──────────────┤
│                                       │ JOINTS       │
│                                       │  J1 ──●──    │
│                                       │  J2 ──●──    │
│                                       │  J3 ──●──    │
│                                       │  J4 ──●──    │
│                                       │  J5 ──●──    │
│                                       │  J6 ──●──    │
├───────────────────────────────────────┴──────────────┤
│  [▶] [‖] [■] [↺]  [+ KEYFRAME]  ──◆──────◆──  0.00s │
└──────────────────────────────────────────────────────┘
```

**Overlays** (float over the viewport, closeable):
- Screen Content Editor — full-width, triggered from sidebar
- Signal Flow — patch bay canvas diagram
- Presets — panel with built-in + user presets

---

## Technical Notes

- **Single file:** `robot-previz/index.html` — all CSS, JS, and HTML inline
- **Dependencies (CDN only):**
  - Three.js r128 via cdnjs
  - OrbitControls via jsdelivr
  - JetBrains Mono + DM Sans via Google Fonts
- **No build tools, no backend, no file I/O**
- **State** lives in a single `state` object (in-memory, resets on page reload)
- **User presets** persist via `localStorage`
- Responsive to window resize via `ResizeObserver`

---

## What's Not Implemented (Out of Scope)

- Real robot communication (no WebSocket, no xArm SDK)
- Save / export / import session data
- Inverse kinematics
- End effector / gripper simulation
- Multiple robot arms
- Collision detection
