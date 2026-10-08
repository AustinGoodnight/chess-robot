# CheckM8 — Autonomous Chess Board

A chess board that plays itself. A Raspberry Pi runs Stockfish and moves the pieces from underneath with an electromagnet on an XY gantry, dragging them along the lanes *between* squares so they never knock over their neighbors.

Built by a six-person team for GEEN 1400 at CU Boulder, fall 2025, for $192 against a $600 budget. Write-up, photos and CAD: **[austingoodnight.dev/projects/auto-chess-robot](https://austingoodnight.dev/projects/auto-chess-robot/)**

## How it works

- **Game logic.** [python-chess](https://python-chess.readthedocs.io/) holds the board state, validates moves and detects captures and game end. Stockfish runs as a UCI engine with a 0.5 s limit per move.
- **Motion.** Each UCI move (e.g. `e2e4`) is converted into gantry coordinates. Stepper motors are driven straight from the Pi's GPIO with step/direction pulses, microstepped to 3,200 steps per revolution. Every move starts and ends at a fixed home corner, so positions are tracked open-loop from a known origin.
- **Collision-free paths.** To move a piece, the gantry drives under it, switches the magnet on, offsets half a square into the lane between squares, travels along the lanes, then steps into the center of the target square.
- **Captures.** The captured piece is removed first and carried along the lanes to a storage area around the edge of the board. Storage is a 10 × 9 grid around the 8 × 8 board, filled row 0 first, then the side columns. The attacking piece then moves normally.

## Hardware

| Part | Notes |
|---|---|
| Raspberry Pi | Runs everything; drives the motors and magnet over GPIO |
| 3 × NEMA 17 steppers | Two drive the X axis together, one drives Y; belts and pulleys on metal rods |
| Stepper drivers | Step/direction inputs |
| Electromagnet | On the Y carriage, switched by a transistor from a GPIO pin |
| Frame | 3D-printed parts, designed in Onshape |

### GPIO pins (BCM)

| Signal | Pin |
|---|---|
| X step / direction | 20 / 26 |
| Y step / direction | 16 / 19 |
| Electromagnet | 6 |

Calibration constants live at the top of `game`: `stepsPerRotation`, `rotationLength` (mm of travel per motor revolution) and `squareSize` (mm).

## Running it

On the Raspberry Pi:

```bash
sudo apt install stockfish
pip install chess gpiozero numpy
python3 game
```

Start with the magnet carriage at the home corner. As committed, the board plays Stockfish against itself; `get_user_move()` in `game` takes moves typed in UCI format for human-vs-engine games.

## Files

| File | What it is |
|---|---|
| `game` | The main program: game loop, path planning, captures and motor control |
| `motorController.py` | Standalone motor and magnet control with the pin layout, used for bring-up |
| `keyboardControl.py` | Jog the gantry from the keyboard, used to test motion before the game logic existed |
| `chessCode.py`, `unsure.py`, `locator.py` | Early experiments with board-state tracking and move location |
