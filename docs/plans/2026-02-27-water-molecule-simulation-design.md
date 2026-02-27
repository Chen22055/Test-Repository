# Water Molecule Simulation Design

**Date:** 2026-02-27

## Overview

Classical molecular dynamics simulation of water molecules (H2O) running in a browser. Single HTML file with canvas-based 2D visualization.

## Visual Design

- **Canvas:** 800x600 pixels, dark blue background (#0a1628)
- **Molecules:** Ball-and-stick model
  - Oxygen: Red circle (#e74c3c), radius 12px
  - Hydrogen: White circle (#ecf0f1), radius 8px
  - Bonds: White lines connecting atoms
- **Water angle:** H-O-H at ~104.5° (real water bond angle)

## Physics Model

- **Translation:** Position (x, y), velocity (vx, vy)
- **Rotation:** Angular position (theta), angular velocity (omega)
- **Intermolecular forces:** Soft repulsion when molecules overlap
- **Wall collisions:** Bounce with slight energy loss (restitution = 0.95)
- **Momentum transfer:** Angular velocity changes on collision

## Controls

| Control | Type | Range | Default |
|---------|------|-------|---------|
| Temperature | Slider | 0.1 - 5.0 | 1.0 |
| Molecule Count | Slider | 5 - 50 | 20 |
| Play/Pause | Button | - | Playing |
| Reset | Button | - | - |

## Interaction

- Click canvas to add molecule at cursor position
- Hover shows molecule info

## Acceptance Criteria

1. Molecules render correctly with O and H atoms
2. Molecules bounce off walls
3. Molecules repel each other on overlap
4. Temperature slider affects molecule velocities
5. Play/Pause stops/starts simulation
6. Single HTML file works in browser
