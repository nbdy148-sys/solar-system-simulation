# Error Log — Solar System Simulation

A complete record of every error encountered during development, including root causes, fixes, and lessons learned. Errors are split across two phases: the original 2D Colab build and the final 3D VS Code build.

---

## Phase 1 — Google Colab (2D Simulation)

---

### Error 01 — The Velocity Misconception
**Type:** Conceptual misunderstanding

| | |
|---|---|
| **Error** | No error message — planets fell straight into the Sun |
| **Cause** | Did not understand why planets needed a sideways velocity (`vy`). Without it, there is no force opposing gravity's inward pull. |
| **Fix** | Every planet must start with an orbital velocity (`vy`) perpendicular to the Sun. This sideways speed is what curves the path into an orbit instead of a straight fall. |
| **Lesson** | An orbit is a tug of war between gravity pulling inward and sideways speed pushing outward. Remove either one and the orbit collapses. |

---

### Error 02 — The Acceleration Stack
**Type:** Logic error

| | |
|---|---|
| **Error** | No error message — planets visually flew off the screen |
| **Cause** | The acceleration (`ax`, `ay`) was never reset to zero at the start of each timestep. Each frame's gravity stacked on top of all previous frames, causing forces to grow infinitely. |
| **Fix** | Added a reset loop at the top of `compute_gravity()` setting `ax = 0.0` and `ay = 0.0` for every planet before recalculating forces. |
| **Lesson** | Physics must be recalculated from scratch every timestep. Think of it as erasing a whiteboard before writing new values — never accumulate across frames. |

---

### Error 03 — The Scaling Issue (Mercury Inside the Sun)
**Type:** Visual/scaling error

| | |
|---|---|
| **Error** | No error message — Mercury appeared to sit inside the Sun |
| **Cause** | Saturn is 24x further from the Sun than Mercury. Fitting both on one screen at real scale crushes the inner planets into an unreadable cluster near the center. |
| **Fix** | Used AU (Astronomical Unit) as the display ruler and adjusted the `LIMIT` variable. Later upgraded to a dual-panel view showing both the full system and inner planets simultaneously. |
| **Lesson** | Real astronomical distances are extreme. Always normalize to AU and design your visual scale around what you want to show. |

---

### Error 04 — Animation Flickering
**Type:** Rendering limitation

| | |
|---|---|
| **Error** | No error message — severe visual flickering throughout the animation |
| **Cause** | Colab's `clear_output()` approach deleted and redrew the entire figure every frame. The transition between delete and redraw caused visible flicker. |
| **Fix** | Attempted multiple approaches: increasing sleep time, drawing every 3rd frame, moving `clear_output` inside the output widget block. Flickering could not be fully eliminated in Colab — ultimately migrated to VS Code. |
| **Lesson** | Colab was not designed for real-time animation. The `clear_output()` method is fundamentally limited — a proper graphics library in a local IDE is the right tool for this job. |

---

### Error 05 — Zoom Slider NameError
**Type:** Scope error

| | |
|---|---|
| **Error** | `NameError: name 'current_limit' is not defined` |
| **Cause** | `on_zoom_change()` was defined in a different cell from `current_limit`. In Colab, each cell has its own scope — functions cannot see variables defined in other cells. |
| **Fix** | Moved all related code (canvas, slider, animation function, zoom state) into a single cell so all variables shared the same scope. |
| **Lesson** | In Colab, scope is per-cell. Functions can only access variables defined in the same cell or previously run cells at module level. |

---

### Error 06 — Simulation Ended Instantly Without Showing
**Type:** Backend configuration error

| | |
|---|---|
| **Error** | `"Simulation ended"` printed instantly with no animation visible |
| **Cause** | `matplotlib.use('Agg')` was set as the backend. The Agg backend is a non-interactive renderer that suppresses all visual output — it renders to memory only. |
| **Fix** | Removed `matplotlib.use('Agg')` entirely and let Colab use its default inline backend. |
| **Lesson** | Never set `matplotlib.use('Agg')` when you want visible output. Agg is for server-side rendering only — it intentionally suppresses display. |

---

## Phase 2 — VS Code (3D Simulation with Vispy)

---

### Error 07 — Virtual Environment Path Error
**Type:** Environment/path error

| | |
|---|---|
| **Error** | `'/C:/Users/.../python.exe' is not recognized as a name of a cmdlet` |
| **Cause** | When the VS Code virtual environment was created, it tried to auto-reinstall packages using a Unix-style path format (`/C:/...`) which PowerShell does not recognize. |
| **Fix** | Ignored the error. The original `pip install` had already succeeded. Verified by running: `python -c "import vispy; print('All good')"` |
| **Lesson** | Always verify installation by importing the library directly rather than trusting the installer output alone. |

---

### Error 08 — Missing Vispy Backend
**Type:** Missing dependency

| | |
|---|---|
| **Error** | `RuntimeError: Could not import any of the backends. You need to install any of ['PyQt4', 'PyQt5', 'PyQt6', ...]` |
| **Cause** | Vispy handles 3D GPU rendering but cannot create a window on its own. It needs a separate window manager backend to display graphics on screen. |
| **Fix** | Installed PyQt6: `pip install PyQt6`. PyQt6 acts as the window manager that Vispy uses to create and display the application window. |
| **Lesson** | Rendering libraries (Vispy, OpenGL) are always separated from window management. Always check what backend a graphics library requires before running it. |

---

### Error 09 — Window Opens and Immediately Closes
**Type:** Missing event loop

| | |
|---|---|
| **Error** | No error message — simulation window flashed on screen for a split second then disappeared |
| **Cause** | Python executed all code top to bottom, opened the window, then reached the end of the script and exited. Without an event loop, nothing kept the process alive. |
| **Fix** | Added `app.run()` at the very end of the script. This starts Vispy's event loop, keeping the application running until the user closes the window. |
| **Lesson** | Every GUI application needs an event loop to stay alive. `app.run()` hands control to the framework's loop — without it, your script finishes and the window dies immediately. |

---

*Total errors encountered: 9 — 6 in Colab, 3 in VS Code*
