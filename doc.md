# Relativistic Wave Propagation Simulation Documentation

This document explains all functions in the Relativistic Wave Propagation Simulation, their interactions with other functions, and their dependencies.

## Table of Contents
- [Global Configuration](#global-configuration)
- [DOM Elements](#dom-elements)
- [Main Components](#main-components)
- [Core Functions](#core-functions)
- [Event Handlers](#event-handlers)
- [Utility Functions](#utility-functions)
- [Simulation Flow](#simulation-flow)

## Global Configuration

The simulation uses a global `config` object that stores all simulation parameters:

```javascript
let config = {
    circleInterval: 50,    // ms between circle generations
    circleSpeed: 2,        // growth speed of circles (analogous to speed of light)
    objectSpeed: 1.5,      // speed of moving object
    observerSpeed: 0,      // speed of observer (0 means stationary)
    isRunning: false,      // simulation state (starts paused)
    fadeTime: 4000,        // time for circles to fade (ms)
    isStarted: false,      // tracks if simulation has been started
    hueSpeed: 0.5,         // speed of hue cycling
    currentHue: 0          // current hue value (0-360)
};
```

## DOM Elements

Key DOM elements referenced throughout the application:

- `container`: Main container div for the simulation
- Control elements: `startBtn`, `resetBtn`, `pauseBtn`
- Slider inputs: `circleIntervalInput`, `circleSpeedInput`, `objectSpeedInput`, `fadeTimeInput`, `hueSpeedInput`, `observerSpeedInput`
- Value displays: `circleIntervalValue`, `circleSpeedValue`, `objectSpeedValue`, `fadeTimeValue`, `hueSpeedValue`, `observerSpeedValue`
- Debug display: `debugInfo`
- Analog controls: `objectControl`, `objectControlStick`, `objectControlLine`, `observerControl`, `observerControlStick`, `observerControlLine`

## Main Components

### Movable Objects

The simulation has two main movable objects:

```javascript
const movingObject = {
    x: 0,
    y: 0,
    radius: 5,
    color: '#ff0000',
    vx: 1,
    vy: 1,
    element: document.createElement('div')
};

const observer = {
    x: 0,
    y: 0,
    radius: 5,
    color: '#0ff',
    vx: 0, // Initial velocity
    vy: 0,
    element: document.createElement('div')
};
```

### Data Storage

The simulation uses two main arrays to store simulation elements:

```javascript
let circles = []; // Stores all light wave circles
let ghosts = [];  // Stores all observer perception points
```

## Core Functions

### `initPoints()`
- **Purpose**: Initializes the movable points (source and observer)
- **Dependencies**: `movingObject`, `observer`, `container`
- **Called by**: `startBtn` click handler when simulation is first started
- **Calls**: `initializePositions()`
- **Description**: Creates and styles the DOM elements for the moving object (light source) and the observer, then adds them to the container.

### `initializePositions()`
- **Purpose**: Sets random positions and velocities for the moving object and observer
- **Dependencies**: `movingObject`, `observer`, `container`
- **Called by**: `initPoints()`, `startBtn` click handler, `resetBtn` click handler
- **Calls**: `updatePoints()`
- **Description**: Assigns random positions within the container and random velocity directions to both the movable object and observer.

### `updatePoints()`
- **Purpose**: Updates the visual positions of the moving object and observer
- **Dependencies**: `movingObject`, `observer`
- **Called by**: `initializePositions()`, `update()`, window resize handler
- **Description**: Updates the CSS positions of the movable object and observer elements to match their data coordinates.

### `createCircle(x, y)`
- **Purpose**: Creates a new light wave circle at the given position
- **Parameters**: Position coordinates `x` and `y`
- **Dependencies**: `config`, `circles`, `container`
- **Called by**: `circleGenerator()`
- **Description**: Creates a new DOM element for a light wave circle, styles it with the current hue, and adds it to the circles array.

### `createGhost(x, y, hue)`
- **Purpose**: Creates a ghost image (observation point) when a wave intersects with the observer
- **Parameters**: Position coordinates `x` and `y`, and the hue value
- **Dependencies**: `ghosts`, `container`
- **Called by**: `update()` when circle-observer collision is detected
- **Description**: Creates a colored dot representing the observer's perception of the light source at a particular moment.

### `update()`
- **Purpose**: Main animation loop function that updates all simulation elements
- **Dependencies**: `config`, `movingObject`, `observer`, `circles`, `ghosts`, `container`
- **Called by**: `startBtn` click handler, recursive self-call via `requestAnimationFrame`
- **Calls**: `updatePoints()`, `getMaxRadius()`, `createGhost()`, `updateDebugInfo()`, `updateAnalogStickPosition()`
- **Description**: Updates positions of the moving object and observer, grows circles, checks for collisions, updates opacities, removes old elements, and updates UI.

### `circleGenerator()`
- **Purpose**: Periodically creates new circles at the light source position
- **Dependencies**: `config`, `movingObject`
- **Called by**: `startBtn` click handler, recursive self-call via `setTimeout`
- **Calls**: `createCircle()`
- **Description**: Creates a new light wave circle at the current position of the moving object, then schedules itself to run again after the configured interval.

### `resetSimulation()`
- **Purpose**: Resets the simulation to its initial state
- **Dependencies**: `circles`, `ghosts`
- **Called by**: `resetBtn` click handler, `startBtn` click handler when restarting
- **Calls**: `updateDebugInfo()`
- **Description**: Removes all circles and ghosts from the DOM and resets the corresponding arrays.

## Event Handlers

### UI Control Event Handlers
- **Sliders**: Each slider input has an event listener that updates the corresponding configuration value and display
- **Buttons**:
  - `startBtn`: Starts or restarts the simulation
  - `pauseBtn`: Toggles pause/resume
  - `resetBtn`: Resets the simulation

### Analog Control Event Handlers
- `setupAnalogControl()`: Sets up mouse and touch event handlers for the analog control sticks
- Events handled include: mousedown, mousemove, mouseup, touchstart, touchmove, touchend

## Utility Functions

### `getMaxRadius()`
- **Purpose**: Calculates the maximum radius threshold for circle deletion
- **Dependencies**: `container`
- **Called by**: `update()`
- **Returns**: Maximum radius value based on container dimensions
- **Description**: Calculates a threshold (2x the larger dimension of the container) beyond which circles are removed.

### `updateDebugInfo(circlesDeleted = 0)`
- **Purpose**: Updates the debug information display
- **Parameters**: Optional count of circles deleted in the current frame
- **Dependencies**: `debugInfo`, `circles`, `ghosts`, `container`, `config`
- **Called by**: `update()`, `resetSimulation()`
- **Description**: Updates the text content of the debug info element with current simulation stats.

### `updateAnalogStickPosition(stick, line, vx, vy)`
- **Purpose**: Updates the position of an analog stick based on a velocity vector
- **Parameters**: stick DOM element, line DOM element, x-velocity, y-velocity
- **Called by**: `setupAnalogControl()`, `update()`, `updateObserverControlsVisibility()`, start button handler
- **Description**: Positions the analog stick and direction line to visually represent a given velocity vector.

### `updateObserverControlsVisibility()`
- **Purpose**: Updates the visibility and state of observer controls based on speed
- **Dependencies**: `config`, `observerControl`, `observerControlStick`, `observerControlLine`, `observer`
- **Called by**: Observer speed input event handler
- **Calls**: `updateAnalogStickPosition()`
- **Description**: Enables or disables the observer control based on whether the observer speed is greater than zero.

## Simulation Flow

1. **Initialization**:
   - When the page loads, DOM elements are accessed and stored
   - Event listeners are attached to UI controls
   - Analog controls are set up

2. **Starting the Simulation**:
   - User clicks the Start button
   - `initializePositions()` sets random positions
   - `initPoints()` creates visual elements
   - `update()` animation loop begins
   - `circleGenerator()` starts creating circles periodically

3. **Runtime**:
   - `update()` is called every animation frame and:
     - Updates object positions
     - Grows existing circles
     - Checks for collisions between circles and observer
     - Creates "ghost" perceptions when collisions occur
     - Removes circles that have grown too large
     - Updates UI elements

4. **User Interaction**:
   - Sliders adjust simulation parameters in real-time
   - Analog sticks allow directional control of the source and observer
   - Pause/Resume can stop/start the animation
   - Reset clears all elements and returns to initial state

5. **Cleanup**:
   - Circles are removed when they exceed maximum size
   - Ghosts fade out and are removed after they become transparent
   - Window resize handler ensures objects stay within boundaries 