# ═══════════════════════════════════════════════════════
#   SOLAR SYSTEM 3D SIMULATION
#   Physics engine: N-body gravitational simulation
#   Renderer: Vispy (GPU accelerated)
# ═══════════════════════════════════════════════════════

# ── Section 1: Imports & Constants
import numpy as np
from vispy import app, scene
from vispy.scene import visuals

# Gravitational constant
G = 6.674e-11

# Astronomical Unit — our ruler (Earth to Sun distance in meters)
AU = 1.496e11

# Time step — 1 hour in seconds
dt = 3600

# How many physics steps per frame (controls simulation speed)
STEPS_PER_FRAME = 24

print("Section 1 ready ✓")



# ── Section 2: Planet Data
# Each planet: name, mass (kg), distance from sun, orbital velocity, color, size
# Color is RGB format — values between 0 and 1

PLANETS_DATA = [
    {
        "name": "Sun",
        "mass": 1.989e30,
        "x": 0, "y": 0, "z": 0,
        "vx": 0, "vy": 0, "vz": 0,
        "color": (1.0, 0.85, 0.0),   # golden yellow
        "size": 20
    },
    {
        "name": "Mercury",
        "mass": 3.285e23,
        "x": 0.387*AU, "y": 0, "z": 0.0,
        "vx": 0, "vy": 47400, "vz": 0,
        "color": (0.9, 0.9, 0.9),    # white-gray
        "size": 5
    },
    {
        "name": "Venus",
        "mass": 4.867e24,
        "x": 0.723*AU, "y": 0, "z": 0.0,
        "vx": 0, "vy": 35020, "vz": 0,
        "color": (0.9, 0.8, 0.6),    # pale orange
        "size": 7
    },
    {
        "name": "Earth",
        "mass": 5.972e24,
        "x": 1.000*AU, "y": 0, "z": 0.0,
        "vx": 0, "vy": 29780, "vz": 0,
        "color": (0.3, 0.6, 1.0),    # blue
        "size": 7
    },
    {
        "name": "Mars",
        "mass": 6.390e23,
        "x": 1.524*AU, "y": 0, "z": 0.0,
        "vx": 0, "vy": 24070, "vz": 0,
        "color": (0.8, 0.3, 0.1),    # red
        "size": 6
    },
    {
        "name": "Jupiter",
        "mass": 1.898e27,
        "x": 5.203*AU, "y": 0, "z": 0.0,
        "vx": 0, "vy": 13070, "vz": 0,
        "color": (0.8, 0.6, 0.4),    # tan
        "size": 14
    },
    {
        "name": "Saturn",
        "mass": 5.683e26,
        "x": 9.537*AU, "y": 0, "z": 0.0,
        "vx": 0, "vy": 9690, "vz": 0,
        "color": (0.9, 0.85, 0.6),   # pale gold
        "size": 12
    },
    {
        "name": "Uranus",
        "mass": 8.681e25,
        "x": 19.19*AU, "y": 0, "z": 0.0,
        "vx": 0, "vy": 6800, "vz": 0,
        "color": (0.5, 0.9, 0.9),    # cyan
        "size": 10
    },
    {
        "name": "Neptune",
        "mass": 1.024e26,
        "x": 30.07*AU, "y": 0, "z": 0.0,
        "vx": 0, "vy": 5430, "vz": 0,
        "color": (0.2, 0.4, 1.0),    # deep blue
        "size": 10
    },
]

print("Section 2 ready ✓")


# ── Section 3: Physics Engine

class Planet:
    def __init__(self, data):
        self.name  = data["name"]
        self.mass  = data["mass"]
        self.color = data["color"]
        self.size  = data["size"]

        # Position (meters)
        self.x  = data["x"]
        self.y  = data["y"]
        self.z  = data["z"]

        # Velocity (meters/second)
        self.vx = data["vx"]
        self.vy = data["vy"]
        self.vz = data["vz"]

        # Acceleration (computed each step)
        self.ax = 0.0
        self.ay = 0.0
        self.az = 0.0

        # Trail — stores last 300 positions in 3D
        self.trail = []

    def update_position(self, dt):
        # Velocity updated by acceleration
        self.vx += self.ax * dt
        self.vy += self.ay * dt
        self.vz += self.az * dt

        # Position updated by velocity
        self.x += self.vx * dt
        self.y += self.vy * dt
        self.z += self.vz * dt

        # Save to trail
        self.trail.append((self.x, self.y, self.z))
        if len(self.trail) > 300:
            self.trail.pop(0)


def compute_gravity(planets):
    # Reset accelerations
    for p in planets:
        p.ax = 0.0
        p.ay = 0.0
        p.az = 0.0

    # Every unique pair
    for i in range(len(planets)):
        for j in range(i + 1, len(planets)):
            p1 = planets[i]
            p2 = planets[j]

            # Distance in 3D — same as 2D but with z added
            dx = p2.x - p1.x
            dy = p2.y - p1.y
            dz = p2.z - p1.z
            r  = np.sqrt(dx**2 + dy**2 + dz**2)

            # Gravitational force magnitude
            F = (G * p1.mass * p2.mass) / r**2

            # Split into x, y, z components
            fx = F * (dx / r)
            fy = F * (dy / r)
            fz = F * (dz / r)

            # Apply to both planets — Newton's 3rd law
            p1.ax += fx / p1.mass
            p1.ay += fy / p1.mass
            p1.az += fz / p1.mass

            p2.ax -= fx / p2.mass
            p2.ay -= fy / p2.mass
            p2.az -= fz / p2.mass


def simulation_step(planets, dt):
    compute_gravity(planets)
    for planet in planets:
        planet.update_position(dt)


# Create planet objects
planets = [Planet(d) for d in PLANETS_DATA]

print("Section 3 ready ✓")


# ── Section 4: 3D Canvas

# ── Scale factor: converts real meters to visual units
# Real distances are too huge to display directly
SCALE = 1.0 / AU

# ── Create the main window
canvas = scene.SceneCanvas(
    title='Solar System 3D Simulation',
    size=(1200, 900),
    bgcolor='black',
    show=True
)

# ── Create a 3D viewport inside the window
view = canvas.central_widget.add_view()
view.camera = 'turntable'          # mouse drag to rotate
view.camera.fov = 45               # field of view
view.camera.distance = 15         # starting zoom distance
view.camera.elevation = 30        # starting vertical angle
view.camera.azimuth = 45          # starting horizontal angle

# ── Create a visual marker for each planet
planet_visuals = []
for planet in planets:
    marker = visuals.Markers()
    marker.set_data(
        pos=np.array([[0.0, 0.0, 0.0]]),
        face_color=planet.color,
        size=planet.size,
        edge_width=0
    )
    view.add(marker)
    planet_visuals.append(marker)

# ── Create a trail line for each planet
trail_visuals = []
for planet in planets:
    line = visuals.Line(
        color=planet.color,
        width=1,
        method='gl'
    )
    view.add(line)
    trail_visuals.append(line)

# ── Add planet name labels
label_visuals = []
for planet in planets:
    label = visuals.Text(
        text=planet.name,
        color='white',
        font_size=8,
        anchor_x='left'
    )
    view.add(label)
    label_visuals.append(label)

# ── Add stars in the background
num_stars = 2000
star_positions = (np.random.rand(num_stars, 3) - 0.5) * 200
star_markers = visuals.Markers()
star_markers.set_data(
    pos=star_positions,
    face_color=(1, 1, 1, 0.6),
    size=1.5,
    edge_width=0
)
view.add(star_markers)

print("Section 4 ready ✓")

 
"""What Each Part Does
```
canvas      → the actual window that opens on your screen
view        → the 3D viewport inside the window
camera      → turntable mode = click drag to rotate freely

planet_visuals → GPU rendered dot for each planet
trail_visuals  → glowing line tracing each orbit
label_visuals  → planet name floating in 3D space
star_markers   → 2000 random stars filling the background
```

## Mouse Controls You Get For Free
```
Left click + drag   → rotate the entire solar system
Scroll wheel        → zoom in and out
Right click + drag  → pan the view """

# ── Section 5: Animation Loop

def update(event):
    # ── Run physics steps
    for _ in range(STEPS_PER_FRAME):
        simulation_step(planets, dt)

    # ── Update visuals for each planet
    for i, planet in enumerate(planets):
        # Convert real position to visual scale
        x = planet.x * SCALE
        y = planet.y * SCALE
        z = planet.z * SCALE

        # Move the planet dot
        planet_visuals[i].set_data(
            pos=np.array([[x, y, z]]),
            face_color=planet.color,
            size=planet.size,
            edge_width=0
        )

        # Draw the trail
        if len(planet.trail) > 2:
            trail_array = np.array(planet.trail) * SCALE
            trail_visuals[i].set_data(
                pos=trail_array,
                color=planet.color,
                width=1
            )

        # Move the label
        label_visuals[i].pos = (x + 0.3, y + 0.3, z)

    # ── Refresh the canvas
    canvas.update()


# ── Section 6: Run

# Connect the timer to the update function
timer = app.Timer(interval=1/60, connect=update, start=True)

print("Section 6 ready ✓")
print("Window open — drag to rotate, scroll to zoom!")
print("Close the window to exit.")

# ── This line keeps the window alive
app.run()

## What This Does
"""
app.Timer    → calls update() 60 times per second
update()     → runs physics + redraws all planets
app.run()    → keeps the window alive until you close it'''"""