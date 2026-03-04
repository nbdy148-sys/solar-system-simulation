# 3D Solar System Simulation

A real-time, GPU-accelerated 3D simulation of the solar system built entirely from scratch in Python. Every planet's motion is driven by real gravitational physics — not pre-programmed circular paths.

## How It Works

The simulation implements Newton's Law of Universal Gravitation:

**F = Gm₁m₂ / r²**

At every timestep, gravitational force is calculated between every unique pair of bodies (Sun + 8 planets) simultaneously — this is known as an N-body simulation. That force is split into x, y, and z components, converted into acceleration, and used to update each planet's velocity and position. This chain runs 24 times per frame, with each step representing one hour of real time.

## Features

- Full N-body physics engine written from scratch using NumPy
- Real astronomical data: actual planetary masses, orbital distances, and velocities from NASA
- GPU-accelerated 3D rendering via Vispy and PyOpenGL
- Interactive turntable camera — rotate, zoom, and pan freely with the mouse
- Orbital trails showing each planet's path through 3D space
- 2,000 background stars for spatial depth

## Tech Stack

- **Python** — core language
- **NumPy** — vector math and physics calculations
- **Vispy** — GPU-accelerated 3D rendering
- **PyOpenGL** — OpenGL backend
- **PyQt6** — window management

## Controls

| Input | Action |
|---|---|
| Left click + drag | Rotate view |
| Scroll wheel | Zoom in/out |
| Right click + drag | Pan |

## What I Learned

This project started as a 2D Matplotlib simulation in Google Colab and evolved into a full 3D GPU-rendered application. Along the way I debugged 9 real errors — from scope issues in Colab to missing graphics backends in VS Code — documenting each one with its root cause and fix. The full error log is included in this repository.
