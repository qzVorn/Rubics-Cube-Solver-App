# 🧊 3D Rubik's Cube Solver

An interactive, zero-dependency Rubik's Cube solver that runs entirely in a single HTML file. Users paint sticker colors onto a fully rotatable 3D cube, then watch a step-by-step animated solution powered by Herbert Kociemba's Two-Phase Algorithm.

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat&logo=html5&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=flat&logo=javascript&logoColor=black)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat&logo=css3&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)

---

## Table of Contents

- [Summary](#summary)
- [Features](#features)
- [Getting Started](#getting-started)
- [Architecture Overview](#architecture-overview)
- [Cube State Representation](#cube-state-representation)
  - [Facelet Indexing](#facelet-indexing)
  - [Color Mapping](#color-mapping)
  - [Dual Source-of-Truth Model](#dual-source-of-truth-model)
- [The Solver: Kociemba's Two-Phase Algorithm](#the-solver-kociembas-two-phase-algorithm)
  - [How It Works](#how-it-works)
  - [Phase 1: Reduce to G1](#phase-1-reduce-to-g1)
  - [Phase 2: Solve from G1](#phase-2-solve-from-g1)
  - [Integration via cubejs](#integration-via-cubejs)
- [3D Rendering Engine](#3d-rendering-engine)
  - [Coordinate System](#coordinate-system)
  - [Rotation Matrices](#rotation-matrices)
  - [Perspective Projection](#perspective-projection)
  - [Painter's Algorithm](#painters-algorithm)
  - [Mouse/Touch Orbit Controls](#mousetouch-orbit-controls)
- [Move System](#move-system)
  - [Move Notation](#move-notation)
  - [Applying Moves to State](#applying-moves-to-state)
  - [Animated Transitions](#animated-transitions)
- [Scramble Generation](#scramble-generation)
- [Validation Logic](#validation-logic)
- [UI Components](#ui-components)
  - [Color Palette and Painting](#color-palette-and-painting)
  - [2D Net View](#2d-net-view)
  - [Step-by-Step Playback](#step-by-step-playback)
- [Customization Guide](#customization-guide)
  - [Changing Cube Colors](#changing-cube-colors)
  - [Adjusting Scramble Length](#adjusting-scramble-length)
  - [Modifying Animation Speed](#modifying-animation-speed)
  - [Canvas and Layout Sizing](#canvas-and-layout-sizing)
  - [Adding New Move Descriptions](#adding-new-move-descriptions)
- [Troubleshooting](#troubleshooting)
- [Dependencies](#dependencies)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

---

## Summary

This project is a fully self-contained, browser-based Rubik's Cube solver. It requires no build tools, no package manager, no server, and no external CDN — everything lives in a single `index.html` file.

The user interacts with the cube in two phases:

1. **Paint Phase** — Click stickers on an interactive 3D cube (with full 360° mouse/touch rotation) or on a 2D unfolded net to assign colors. A real-time counter ensures exactly 9 stickers per color.
2. **Solve Phase** — The Kociemba Two-Phase Algorithm computes a near-optimal solution (≤ 22 moves). The solution is displayed as a numbered step list with plain-English descriptions, and each move is animated on the 3D cube with full forward/backward navigation.

The solver engine is [cubejs](https://github.com/ldez/cubejs) by Petri Lehtinen (akheron), inlined directly into the HTML file. It implements Herbert Kociemba's Two-Phase Algorithm in JavaScript, finding solutions in 0.01–0.4 seconds after an initial 4–5 second precalculation step.

---

## Features

- **Zero Dependencies** — Single HTML file, no npm, no CDN, no build step.
- **Interactive 3D Cube** — Pure Canvas 2D API with manual 3D projection; mouse drag to orbit, touch-friendly.
- **Full Sticker Editing** — All 54 stickers are paintable, including centers.
- **Real-Time Validation** — Live color counter (X/9 per color) with error detection.
- **Kociemba Solver** — Near-optimal solutions in ≤ 22 moves, computed in milliseconds.
- **Step-by-Step Playback** — Forward, backward, play/pause, speed control, progress bar.
- **Smooth Animations** — Per-layer rotation animations with configurable duration and easing.
- **Random Scramble** — Generates guaranteed-solvable random states via move sequences.
- **Responsive Layout** — Three-panel design (palette, 3D view, steps) that adapts to screen size.
- **Standard Colors** — Uses official Rubik's Cube colors: White, Yellow, Red, Orange, Blue, Green.

---

## Getting Started

```bash
# Clone the repository
git clone https://github.com/your-username/rubiks-cube-solver.git
cd rubiks-cube-solver

# Open in your browser — that's it
open index.html
# or
xdg-open index.html   # Linux
start index.html       # Windows
```

No server required. No compilation. Just open the file.

> **Note:** On first load, the solver performs a one-time precalculation (~4–5 seconds) to build lookup/pruning tables. A status indicator shows progress.

---

## Architecture Overview

The application is structured as three logical layers within a single file:

```
┌──────────────────────────────────────────────────────────┐
│                        index.html                        │
├──────────────────────────────────────────────────────────┤
│  <style>        — All CSS (layout, palette, animations)  │
├──────────────────────────────────────────────────────────┤
│  <script>       — cubejs library (cube.js + solve.js)    │
│                    Kociemba Two-Phase Algorithm           │
├──────────────────────────────────────────────────────────┤
│  <script>       — Application logic                      │
│                    ├── State management                   │
│                    ├── 3D rendering engine                │
│                    ├── UI event handlers                  │
│                    ├── Validation                         │
│                    ├── Move animation system              │
│                    └── Step playback controller           │
└──────────────────────────────────────────────────────────┘
```

---

## Cube State Representation

### Facelet Indexing

The cube state is stored as a flat array of 54 elements, indexed 0–53. Each element represents one sticker (facelet). The mapping follows the standard cubejs convention:

```
Indices 0–8:   U face (Up/Top)       — White
Indices 9–17:  R face (Right)        — Red
Indices 18–26: F face (Front)        — Green
Indices 27–35: D face (Down/Bottom)  — Yellow
Indices 36–44: L face (Left)         — Orange
Indices 45–53: B face (Back)         — Blue
```

Within each face, stickers are ordered left-to-right, top-to-bottom:

```
              ┌────────────────┐
              │  0    1    2   │
              │                │
              │  3    4    5   │
              │                │
              │  6    7    8   │
┌─────────────┼────────────────┼─────────────┬─────────────┐
│ 36  37  38  │ 18   19   20  │  9   10  11  │ 45  46  47  │
│             │               │              │             │
│ 39  40  41  │ 21   22   23  │ 12   13  14  │ 48  49  50  │
│             │               │              │             │
│ 42  43  44  │ 24   25   26  │ 15   16  17  │ 51  52  53  │
└─────────────┼────────────────┼─────────────┴─────────────┘
              │ 27   28   29  │
              │               │
              │ 30   31   32  │
              │               │
              │ 33   34   35  │
              └────────────────┘
       L            F              R              B
                    U (top)
                    D (bottom)
```

The six center stickers (which define each face's identity) are at indices **4, 13, 22, 31, 40, 49**.

### Color Mapping

The application uses two parallel representations: display colors (what the user sees) and solver characters (what cubejs expects).

```javascript
// ── Display Colors ──────────────────────────────────────────
// Customize these hex values to change the cube's appearance.
const DISPLAY_COLORS = {
  W: '#ffffff',   // White  — Up face
  Y: '#ffd700',   // Yellow — Down face
  R: '#dc3545',   // Red    — Right face
  O: '#ff8c00',   // Orange — Left face
  B: '#003da5',   // Blue   — Back face
  G: '#009b48',   // Green  — Front face
};

// ── Face-to-Color Mapping ───────────────────────────────────
// Maps cubejs face letters to display color keys.
const FACE_TO_COLOR = {
  U: 'W',  // Up    → White
  R: 'R',  // Right → Red
  F: 'G',  // Front → Green
  D: 'Y',  // Down  → Yellow
  L: 'O',  // Left  → Orange
  B: 'B',  // Back  → Blue
};
```

### Dual Source-of-Truth Model

A critical design decision is using **two separate state models** for painting and solving to prevent state drift:

```javascript
// ── Paint Mode ──────────────────────────────────────────────
// paintedColors[] is the sole source of truth while the user
// is editing stickers. It stores color keys ('W','Y','R',...).
let paintedColors = new Array(54).fill('W');

// ── Solve Mode ──────────────────────────────────────────────
// cubeObj is a native cubejs Cube object — the sole source of
// truth during solution playback. Display colors are derived
// on-the-fly from cubeObj.asString().
let cubeObj = null;

// ── Saved Checkpoint ────────────────────────────────────────
// When the user clicks "Solve", the exact cubejs state is
// saved. Navigating to any step restores this checkpoint and
// re-applies moves 0..N, guaranteeing perfect state.
let savedCubeJSON = null;

let mode = 'paint'; // 'paint' | 'solve'
```

**Why two models?** Converting back and forth between display colors and cubejs strings on every move caused cumulative rounding/mapping errors. The dual model ensures the solver's internal state is never corrupted by display logic.

```javascript
// ── Convert display colors → cubejs string ──────────────────
function colorsToCubeString(colors) {
  const centers = [4, 13, 22, 31, 40, 49];
  const faces = ['U', 'R', 'F', 'D', 'L', 'B'];
  const map = {};
  faces.forEach((f, i) => {
    map[colors[centers[i]]] = f;
  });
  return colors.map(c => map[c]).join('');
}

// ── Convert cubejs string → display colors ──────────────────
function cubeStringToColors(str) {
  const centers = [4, 13, 22, 31, 40, 49];
  const faces = ['U', 'R', 'F', 'D', 'L', 'B'];
  const map = {};
  faces.forEach((f, i) => {
    map[str[centers[i]]] = FACE_TO_COLOR[f];
  });
  return str.split('').map(ch => map[ch]);
}
```

---

## The Solver: Kociemba's Two-Phase Algorithm

### How It Works

Herbert Kociemba's Two-Phase Algorithm is one of the most efficient methods for solving a 3×3×3 Rubik's Cube. Unlike beginner layer-by-layer methods (which may produce 80+ move solutions), Kociemba finds near-optimal solutions of **≤ 22 moves** — close to the theoretical minimum of 20 moves (God's Number).

The algorithm divides the solution into two phases by exploiting the algebraic group structure of the cube:

```
All cube states (G0)  ──Phase 1──►  Restricted subgroup (G1)  ──Phase 2──►  Solved (Identity)
    ~4.3 × 10¹⁹                      ~1.95 × 10¹⁰                              1
     states                             states                                 state
```

### Phase 1: Reduce to G1

**Goal:** Transform the scrambled cube into the subgroup G1 = ⟨U, D, R², L², F², B²⟩.

A cube is in G1 when three conditions are met simultaneously:

| Condition | Description | Coordinate Range |
|-----------|-------------|-----------------|
| Corner Orientation | All 8 corners have orientation 0 | 0–2,186 |
| Edge Orientation | All 12 edges have orientation 0 | 0–2,047 |
| UD-Slice Edges | The 4 middle-layer edges (FR, FL, BL, BR) are somewhere in the middle layer | 0–494 |

Phase 1 searches through **2,187 × 2,048 × 495 ≈ 2.22 billion** states using all 18 face moves (U, U', U2, R, R', R2, F, F', F2, D, D', D2, L, L', L2, B, B', B2).

### Phase 2: Solve from G1

**Goal:** Solve the cube from the G1 state to the identity (solved) state.

| Condition | Description | Coordinate Range |
|-----------|-------------|-----------------|
| Corner Permutation | All 8 corners in correct position | 0–40,319 |
| Edge Permutation (U/D) | All 8 U/D edges in correct position | 0–40,319 |
| UD-Slice Permutation | The 4 middle-layer edges in correct order | 0–23 |

Phase 2 searches through **40,320 × 40,320 × 24 / 2 ≈ 19.5 billion** states using only the restricted move set: {U, U', U2, D, D', D2, R², L², F², B²} (10 moves). These moves preserve the G1 conditions established in Phase 1.

The algorithm uses **Iterative Deepening A\* (IDA\*)** with precomputed **pruning tables** to dramatically reduce the search space. It iteratively tries longer Phase 1 solutions hoping for shorter Phase 2 solutions, keeping the best total.

### Integration via cubejs

The [cubejs](https://github.com/ldez/cubejs) library by Petri Lehtinen provides the JavaScript implementation. It is inlined directly in the HTML file (no external fetch).

```javascript
// ── Solver Initialization ───────────────────────────────────
// Must be called once before any solve() calls.
// Builds move tables and pruning tables (~4-5 seconds).
Cube.initSolver();

// ── Solving a Cube ──────────────────────────────────────────
// Create a Cube from a 54-character facelet string and solve.
const cubeString = colorsToCubeString(paintedColors);
const cubeObj = Cube.fromString(cubeString);
const solution = cubeObj.solve();
// solution = "R U' F2 D B' L2 ..." (string of moves)

// ── Applying Moves ──────────────────────────────────────────
cubeObj.move("R U' F2");  // Apply moves to internal state
const newFacelets = cubeObj.asString(); // Get updated state

// ── State Serialization ─────────────────────────────────────
const checkpoint = cubeObj.toJSON();
// { cp: [...], co: [...], ep: [...], eo: [...] }
const restored = new Cube(checkpoint); // Perfect restoration
```

**Key cubejs API methods used in this project:**

| Method | Purpose |
|--------|---------|
| `new Cube()` | Create a solved cube instance |
| `Cube.fromString(str)` | Create cube from 54-char facelet string |
| `Cube.initSolver()` | Build lookup/pruning tables (one-time) |
| `Cube.random()` | Generate a random cube state |
| `cube.solve()` | Return solution string (≤ 22 moves) |
| `cube.move(alg)` | Apply move sequence to cube |
| `cube.asString()` | Export cube as 54-char facelet string |
| `cube.toJSON()` | Serialize state as JSON object |
| `cube.isSolved()` | Check if cube is the identity |
| `new Cube(json)` | Restore cube from serialized state |

---

## 3D Rendering Engine

The cube is rendered using the **HTML5 Canvas 2D API** with manual 3D math — no WebGL, no Three.js, no libraries. This keeps the project dependency-free and broadly compatible.

### Coordinate System

Each of the 26 visible cubies is defined by its 3D position on a grid centered at the origin. Each sticker is a quadrilateral (4 vertices) in 3D space, offset slightly from the cubie surface.

```javascript
// ── Cubie Geometry ──────────────────────────────────────────
// Each cubie occupies a 1×1×1 unit cell. The full cube spans
// from -1.5 to +1.5 on each axis. Stickers are inset by a
// small gap for the black border effect.

const CUBIE_SIZE = 1.0;       // Size of one cubie
const GAP = 0.05;             // Gap between stickers (black border)
const STICKER_INSET = 0.08;   // Inset from cubie edge

// Example: Front face sticker at grid position (row=0, col=0)
// 3D corners before rotation:
// (-1.5 + GAP, 1.5 - GAP, 1.5)   top-left
// (-0.5 - GAP, 1.5 - GAP, 1.5)   top-right
// (-0.5 - GAP, 0.5 + GAP, 1.5)   bottom-right
// (-1.5 + GAP, 0.5 + GAP, 1.5)   bottom-left
```

### Rotation Matrices

User orbiting and face-turn animations use standard 3D rotation matrices applied to every vertex before projection.

```javascript
// ── Rotation around X axis ──────────────────────────────────
function rotateX(point, angle) {
  const cos = Math.cos(angle);
  const sin = Math.sin(angle);
  return [
    point[0],
    point[1] * cos - point[2] * sin,
    point[1] * sin + point[2] * cos,
  ];
}

// ── Rotation around Y axis ──────────────────────────────────
function rotateY(point, angle) {
  const cos = Math.cos(angle);
  const sin = Math.sin(angle);
  return [
    point[0] * cos + point[2] * sin,
    point[1],
   -point[0] * sin + point[2] * cos,
  ];
}

// ── Rotation around Z axis ──────────────────────────────────
function rotateZ(point, angle) {
  const cos = Math.cos(angle);
  const sin = Math.sin(angle);
  return [
    point[0] * cos - point[1] * sin,
    point[0] * sin + point[1] * cos,
    point[2],
  ];
}
```

Orbit rotation is applied as a combined Y-then-X rotation controlled by mouse/touch drag deltas:

```javascript
// ── Apply orbit rotation to a point ─────────────────────────
function applyOrbit(point) {
  let p = rotateY(point, orbitAngleY);
  p = rotateX(p, orbitAngleX);
  return p;
}
```

### Perspective Projection

3D points are projected to 2D canvas coordinates using a simple perspective (pinhole camera) projection:

```javascript
// ── Perspective Projection ──────────────────────────────────
const CAMERA_DISTANCE = 8;  // Distance from camera to origin
const PROJECTION_SCALE = 200; // Pixels per unit at z=0

function project(point3D) {
  const [x, y, z] = point3D;
  const scale = CAMERA_DISTANCE / (CAMERA_DISTANCE - z);
  return {
    x: canvasCenterX + x * PROJECTION_SCALE * scale,
    y: canvasCenterY - y * PROJECTION_SCALE * scale, // Y is inverted
    depth: z, // Retained for sorting
  };
}
```

**Customizable parameters:**

| Variable | Default | Effect |
|----------|---------|--------|
| `CAMERA_DISTANCE` | `8` | Larger = less perspective distortion (more orthographic) |
| `PROJECTION_SCALE` | `200` | Larger = bigger cube on screen |

### Painter's Algorithm

Since we use Canvas 2D (no z-buffer), faces are drawn back-to-front using the **Painter's Algorithm**:

```javascript
// ── Sort stickers by average depth (back to front) ──────────
stickers.sort((a, b) => {
  const avgDepthA = a.projectedVerts.reduce((s, v) => s + v.depth, 0) / 4;
  const avgDepthB = b.projectedVerts.reduce((s, v) => s + v.depth, 0) / 4;
  return avgDepthA - avgDepthB; // Draw furthest first
});

// ── Draw each sticker as a filled polygon ───────────────────
stickers.forEach(sticker => {
  ctx.beginPath();
  ctx.moveTo(sticker.projectedVerts[0].x, sticker.projectedVerts[0].y);
  for (let i = 1; i < 4; i++) {
    ctx.lineTo(sticker.projectedVerts[i].x, sticker.projectedVerts[i].y);
  }
  ctx.closePath();
  ctx.fillStyle = DISPLAY_COLORS[sticker.color];
  ctx.fill();
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 2;
  ctx.stroke();
});
```

### Mouse/Touch Orbit Controls

The cube can be freely rotated by dragging. The implementation tracks pointer movement and updates orbit angles:

```javascript
// ── Mouse/Touch Orbit ───────────────────────────────────────
let isDragging = false;
let lastPointerX, lastPointerY;
let orbitAngleX = -Math.PI / 6; // Initial tilt (30° down)
let orbitAngleY = Math.PI / 5;  // Initial rotation

const ORBIT_SENSITIVITY = 0.007; // Radians per pixel of drag

canvas.addEventListener('pointerdown', (e) => {
  isDragging = true;
  lastPointerX = e.clientX;
  lastPointerY = e.clientY;
});

canvas.addEventListener('pointermove', (e) => {
  if (!isDragging) return;
  const dx = e.clientX - lastPointerX;
  const dy = e.clientY - lastPointerY;
  orbitAngleY += dx * ORBIT_SENSITIVITY;
  orbitAngleX += dy * ORBIT_SENSITIVITY;
  // Clamp vertical rotation to prevent flipping
  orbitAngleX = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, orbitAngleX));
  lastPointerX = e.clientX;
  lastPointerY = e.clientY;
  render(); // Re-draw with new angles
});

canvas.addEventListener('pointerup', () => { isDragging = false; });
```

---

## Move System

### Move Notation

Standard Rubik's Cube notation is used throughout. Each move is a face letter optionally followed by a modifier:

| Move | Description | Degrees |
|------|-------------|---------|
| `U` | Up face clockwise (90°) | 90° CW |
| `U'` | Up face counter-clockwise (90°) | 90° CCW |
| `U2` | Up face half-turn (180°) | 180° |
| `R` | Right face clockwise | 90° CW |
| `R'` | Right face counter-clockwise | 90° CCW |
| `R2` | Right face half-turn | 180° |
| `F` | Front face clockwise | 90° CW |
| `D` | Down face clockwise | 90° CW |
| `L` | Left face clockwise | 90° CW |
| `B` | Back face clockwise | 90° CW |

*(Primed `'` and double `2` variants exist for all six faces.)*

### Applying Moves to State

Moves are applied to the cubejs `Cube` object directly. The display is then derived from the updated state:

```javascript
// ── Apply a single move and update display ──────────────────
function applyMove(moveStr) {
  cubeObj.move(moveStr);
  const colors = cubeStringToColors(cubeObj.asString());
  updateDisplayFromColors(colors);
}
```

### Animated Transitions

Move animations rotate the affected layer's stickers smoothly over a configurable duration. The system identifies which of the 54 stickers belong to the turning layer, then applies an incremental rotation to only those stickers during each animation frame.

```javascript
// ── Animation Configuration ─────────────────────────────────
const ANIMATION_DURATION_MS = 400;  // Milliseconds per move
const ANIMATION_EASING = 'ease';     // CSS-style easing name

// ── Layer Sets ──────────────────────────────────────────────
// Precomputed mapping: face → set of facelet indices that
// rotate when that face turns. Built dynamically at init by
// applying a test move and diffing the before/after state.
let layerSets = {};

function buildLayerSets() {
  const faces = ['U', 'R', 'F', 'D', 'L', 'B'];
  faces.forEach(face => {
    const before = new Cube();
    const after = new Cube();
    after.move(face);
    const bStr = before.asString();
    const aStr = after.asString();
    const indices = new Set();
    for (let i = 0; i < 54; i++) {
      if (bStr[i] !== aStr[i]) indices.add(i);
    }
    layerSets[face] = indices;
  });
}

// ── Animate a Move ──────────────────────────────────────────
function applyMoveAnimated(moveStr, callback) {
  const face = moveStr[0];
  const isPrime = moveStr.includes("'");
  const isDouble = moveStr.includes("2");
  const targetAngle = isDouble ? Math.PI : Math.PI / 2;
  const direction = isPrime ? -1 : 1;
  const axis = getAxisForFace(face); // 'x', 'y', or 'z'
  const affectedIndices = layerSets[face];

  const startTime = performance.now();

  function frame(now) {
    const elapsed = now - startTime;
    const t = Math.min(elapsed / ANIMATION_DURATION_MS, 1);
    // Ease-in-out
    const eased = t < 0.5
      ? 2 * t * t
      : 1 - Math.pow(-2 * t + 2, 2) / 2;
    const angle = eased * targetAngle * direction;

    renderWithLayerRotation(affectedIndices, axis, angle);

    if (t < 1) {
      requestAnimationFrame(frame);
    } else {
      cubeObj.move(moveStr); // Commit move to state
      render();              // Clean render from new state
      if (callback) callback();
    }
  }

  requestAnimationFrame(frame);
}
```

---

## Scramble Generation

Random scrambles are generated by applying a sequence of random moves to a solved cube, rather than using `Cube.random()`. This guarantees that center stickers remain in their standard positions, avoiding the center-permutation issues that caused "invalid sticker count" errors.

```javascript
// ── Scramble Configuration ──────────────────────────────────
const SCRAMBLE_MOVE_COUNT = 25; // Number of random moves

const MOVE_FACES = ['U', 'R', 'F', 'D', 'L', 'B'];
const MOVE_SUFFIXES = ['', "'", '2'];

function randomizeCube() {
  // Start from solved state
  const cube = new Cube();
  let lastFace = '';
  const moves = [];

  for (let i = 0; i < SCRAMBLE_MOVE_COUNT; i++) {
    let face;
    do {
      face = MOVE_FACES[Math.floor(Math.random() * 6)];
    } while (face === lastFace); // Avoid consecutive same-face moves

    const suffix = MOVE_SUFFIXES[Math.floor(Math.random() * 3)];
    const move = face + suffix;
    cube.move(move);
    moves.push(move);
    lastFace = face;
  }

  // Extract display colors from the scrambled state
  paintedColors = cubeStringToColors(cube.asString());
  mode = 'paint';
  updateUI();
}
```

**Why not `Cube.random()`?** The `Cube.random()` method randomizes corner/edge permutation and orientation but can create states where — when converted to a facelet string — the center stickers don't match the expected U/R/F/D/L/B positions. This caused the solver to reject the cube. Generating scrambles via move sequences preserves center identity.

---

## Validation Logic

Before solving, the cube state is validated to catch impossible or incorrectly painted configurations:

```javascript
function validateCube(colors) {
  const errors = [];

  // 1. Check that each color appears exactly 9 times
  const counts = {};
  Object.keys(DISPLAY_COLORS).forEach(c => { counts[c] = 0; });
  colors.forEach(c => { counts[c] = (counts[c] || 0) + 1; });

  for (const [color, count] of Object.entries(counts)) {
    if (count !== 9) {
      errors.push(`${color} has ${count} stickers (need exactly 9)`);
    }
  }

  // 2. Check that all 6 center stickers are unique colors
  const centers = [4, 13, 22, 31, 40, 49];
  const centerColors = new Set(centers.map(i => colors[i]));
  if (centerColors.size !== 6) {
    errors.push('All 6 center stickers must be different colors');
  }

  return errors; // Empty array = valid
}
```

The solver also catches **impossible permutations** (e.g., a single flipped edge or twisted corner) by trapping the exception from `cubeObj.solve()` and displaying a helpful error message.

---

## UI Components

### Color Palette and Painting

The left panel displays clickable color swatches and a live sticker counter:

```javascript
// ── Build Color Swatches ────────────────────────────────────
const PALETTE_ORDER = ['W', 'Y', 'R', 'O', 'B', 'G'];

function buildPalette(container) {
  PALETTE_ORDER.forEach(colorKey => {
    const swatch = document.createElement('div');
    swatch.className = 'color-swatch';
    swatch.style.backgroundColor = DISPLAY_COLORS[colorKey];
    swatch.dataset.color = colorKey;
    swatch.addEventListener('click', () => {
      selectedColor = colorKey;
      highlightActiveSwatch();
    });
    container.appendChild(swatch);
  });
}

// ── Live Color Counter ──────────────────────────────────────
function updateColorCounts() {
  const counts = {};
  PALETTE_ORDER.forEach(c => { counts[c] = 0; });
  paintedColors.forEach(c => { counts[c]++; });

  PALETTE_ORDER.forEach(c => {
    const el = document.getElementById(`count-${c}`);
    el.textContent = `${counts[c]}/9`;
    el.classList.toggle('complete', counts[c] === 9);
    el.classList.toggle('over', counts[c] > 9);
  });
}
```

### 2D Net View

A flattened cross-shaped net lets users paint stickers without rotating the 3D cube:

```
              ┌───┐
              │ U │
         ┌────┼───┼────┬────┐
         │ L  │ F │ R  │ B  │
         └────┼───┼────┴────┘
              │ D │
              └───┘
```

Each cell in the net is a clickable `<div>` that updates `paintedColors[index]` when clicked with the selected color.

### Step-by-Step Playback

The right panel shows the solution as a numbered list with navigation controls:

```javascript
// ── Parse Solution into Steps ───────────────────────────────
function parseSolution(solutionStr) {
  return solutionStr.trim().split(/\s+/).filter(Boolean);
}

// ── Generate Human-Readable Description ─────────────────────
function describeMove(move) {
  const faceNames = {
    U: 'Top', D: 'Bottom', R: 'Right',
    L: 'Left', F: 'Front', B: 'Back'
  };
  const face = move[0];
  const name = faceNames[face] || face;

  if (move.includes("'")) return `Rotate ${name} face 90° counter-clockwise`;
  if (move.includes('2'))  return `Rotate ${name} face 180°`;
  return `Rotate ${name} face 90° clockwise`;
}

// ── Step Navigation ─────────────────────────────────────────
function goToStep(targetStep) {
  // Restore cube to initial state
  cubeObj = new Cube(savedCubeJSON);

  // Re-apply moves 0 through targetStep
  for (let i = 0; i <= targetStep; i++) {
    cubeObj.move(solveSteps[i]);
  }

  currentStep = targetStep;
  const colors = cubeStringToColors(cubeObj.asString());
  updateDisplayFromColors(colors);
  highlightCurrentStep();
  updateProgressBar();
}
```

**Playback controls:**

| Control | Function |
|---------|----------|
| **Previous** | Navigate to the prior step (rebuilds state from checkpoint) |
| **Next** | Animate and apply the next move |
| **Play / Pause** | Auto-advance through remaining steps |
| **Speed Slider** | Adjust animation duration (100ms – 2000ms) |
| **Progress Bar** | Visual indicator of current position in solution |

---

## Customization Guide

### Changing Cube Colors

Modify `DISPLAY_COLORS` to change the visual appearance. The keys (`W`, `Y`, `R`, `O`, `B`, `G`) are internal identifiers — only the hex values matter for rendering:

```javascript
// Example: Pastel color scheme
const DISPLAY_COLORS = {
  W: '#f0f0f0',   // Light grey instead of pure white
  Y: '#ffe066',   // Soft yellow
  R: '#ff6b6b',   // Soft red
  O: '#ffa94d',   // Soft orange
  B: '#4dabf7',   // Light blue
  G: '#51cf66',   // Light green
};
```

### Adjusting Scramble Length

```javascript
// Fewer moves = easier scramble, more moves = more random
const SCRAMBLE_MOVE_COUNT = 25; // Default: 25 (range: 1–100)
```

For a truly random-looking scramble, 20–30 moves is sufficient. The cube reaches a statistically uniform random state after ~20 moves.

### Modifying Animation Speed

```javascript
// Milliseconds per move animation
const ANIMATION_DURATION_MS = 400; // Default: 400

// Slower (good for learning):
const ANIMATION_DURATION_MS = 800;

// Faster (good for demonstrations):
const ANIMATION_DURATION_MS = 150;
```

The speed slider in the UI overrides this default at runtime.

### Canvas and Layout Sizing

```javascript
// Canvas resolution (also set via HTML attributes)
const CANVAS_WIDTH = 500;
const CANVAS_HEIGHT = 500;

// 3D projection parameters
const CAMERA_DISTANCE = 8;    // Increase for less perspective
const PROJECTION_SCALE = 200; // Increase for larger cube
```

```css
/* Panel widths — adjust for different screen layouts */
.left-panel  { width: 220px; }
.center-panel { flex: 1; min-width: 400px; }
.right-panel { width: 280px; }
```

### Adding New Move Descriptions

Extend `describeMove()` to add context-specific hints:

```javascript
function describeMove(move) {
  const faceNames = {
    U: 'Top (White)',
    D: 'Bottom (Yellow)',
    R: 'Right (Red)',
    L: 'Left (Orange)',
    F: 'Front (Green)',
    B: 'Back (Blue)',
  };
  // ... rest of function
}
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Invalid: X has N stickers (need 9)" | Incorrect number of a color painted | Check the color counter — each color must show 9/9 |
| "Center stickers must be unique" | Two faces have the same center color | Ensure all 6 center stickers (index 4,13,22,31,40,49) are different |
| "Could not solve — impossible state" | Invalid corner twist, edge flip, or permutation parity | On a real cube, only states reachable by legal moves are solvable. Reassemble or re-paint |
| Solver takes > 5 seconds on init | Normal — pruning tables are being computed | Wait for the status indicator to show "Ready" |
| Cube appears black / no stickers | Canvas size is 0 or CSS is hiding it | Check that the canvas element has non-zero width/height |
| Steps don't match the 3D view | State drift from old dual-state bug | Ensure you're using the latest version with the dual source-of-truth model |

---

## Dependencies

**None.** The entire application — including the cubejs solver library — is inlined in a single HTML file.

For reference, the inlined library is:

| Library | Version | Author | License | Purpose |
|---------|---------|--------|---------|---------|
| [cubejs](https://github.com/ldez/cubejs) | 1.3.x | Petri Lehtinen (akheron) | MIT | Kociemba Two-Phase Algorithm solver |

---

## Project Structure

```
rubiks-cube-solver/
├── index.html          # Everything — HTML, CSS, JS, solver
├── README.md           # This file
└── LICENSE             # MIT License
```

That's it. One file. No `node_modules`, no `package.json`, no `webpack.config.js`.

---

## Contributing

Contributions are welcome. Some ideas for improvement:

- **WebGL rendering** — Replace Canvas 2D with WebGL or Three.js for smoother visuals and proper lighting.
- **Mobile optimization** — Improve touch handling and responsive layout for small screens.
- **Solution phases** — Group moves by solving phase (cross, F2L, OLL, PLL) with explanatory labels.
- **Undo/redo for painting** — Add a history stack for the paint mode.
- **Import/export** — Allow pasting a facelet string or scramble algorithm.
- **Accessibility** — Add keyboard navigation and screen reader support.
- **Internationalization** — Translate move descriptions and UI labels.

To contribute:

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add your feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request.

---

## License

This project is licensed under the **MIT License**. See [LICENSE](LICENSE) for details.

The inlined cubejs library is also MIT-licensed. See [cubejs LICENSE](https://github.com/ldez/cubejs/blob/master/LICENSE).

---

## Acknowledgments

- **Herbert Kociemba** — Creator of the [Two-Phase Algorithm](http://kociemba.org/cube.htm) that makes sub-second solving possible.
- **Petri Lehtinen (akheron)** — Author of the original [cube.js](https://github.com/akheron/cubejs) CoffeeScript implementation.
- **ldez** — Maintainer of the [cubejs fork](https://github.com/ldez/cubejs) used in this project.
- **Erno Rubik** — For inventing the puzzle that launched a thousand algorithms.
```

---
