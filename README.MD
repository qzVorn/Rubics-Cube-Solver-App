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
