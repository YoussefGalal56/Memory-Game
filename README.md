## Project — Multiplayer Memory Game

![Arduino](https://img.shields.io/badge/Arduino-C%20Programming-00979D?style=flat-square&logo=arduino&logoColor=white)
![EEPROM](https://img.shields.io/badge/EEPROM-Non--volatile%20Storage-grey?style=flat-square)
![SimulIDE](https://img.shields.io/badge/SimulIDE-Circuit%20Simulation-green?style=flat-square)
![Wokwi](https://img.shields.io/badge/Wokwi-Hardware%20Testing-red?style=flat-square)

**Module:** Computer Programming (4FTC2029-0105) &nbsp;|&nbsp; **Platform:** Arduino Uno &nbsp;|&nbsp; **Language:** C

---

### Overview

A microcontroller-based multiplayer memory game built on an Arduino Uno. Players memorize and repeat LED sequences using push buttons, with increasing difficulty as rounds progress. The system features three distinct game modes, a scrollable LCD menu, EEPROM high-score storage, and a sleep mode — all implemented with non-blocking code and a state machine architecture.

---

### Game Modes

| Mode | Players | Description |
|------|---------|-------------|
| **Solo Sprint** | 1 | Repeat a randomly generated LED sequence. Ends on wrong input or timeout. |
| **Synchronous Duel** | 2 | Both players race to input the same sequence simultaneously. Ends when both fail or time out. |
| **Challenger Mode** | 2 | P1 creates a sequence; P2 must replicate it to earn a point. If P2 fails, P1 can prove it and steal the point. First to 10 points wins. |

All modes include 3 selectable difficulties:

| Difficulty | Start Speed | Min Speed | Timeout |
|------------|-------------|-----------|---------|
| Easy | 800 ms | 300 ms | 7 s |
| Medium | 500 ms | 100 ms | 6 s |
| Hard | 300 ms | 50 ms | 5 s |

---

### Hardware

| Component | Qty | Purpose |
|-----------|-----|---------|
| Arduino Uno | 1 | Main microcontroller |
| LEDs | 6 | 4 sequence LEDs + 2 player-win indicators |
| Push buttons | 8 | 4 per player for sequence input |
| I2C LCD (16×2) | 1 | Menu, scores, and game messages |
| Passive buzzer | 1 | Correct / wrong / game-over audio feedback |
| 320Ω resistors | 6 | LED current limiting |
| Switch | 1 | Power on/off |
| Battery cells | 2 | Portable power via Vin pin |

**Pin mapping:**
```
Port D  →  P1 buttons (pins 2–5) + buzzer (pin 7)
Port B  →  Sequence LEDs (pins 8–11) + player LEDs (pins 12–13)
Port C  →  P2 buttons (A0–A3) + I2C LCD (A4–A5)
```

---

### Software Architecture

The codebase is split into modular `.ino` files, each responsible for a single concern:

| File | Responsibility |
|------|---------------|
| `main.ino` | Setup, main loop, state machine driver |
| `StateMachine.ino` | States: Menu → Difficulty → Countdown → Playing → Game Over → Sleep |
| `ModeSolo.ino` | Solo Sprint gameplay logic |
| `ModeDuel.ino` | Synchronous Duel gameplay logic |
| `ModeChallenger.ino` | Challenger Mode gameplay logic |
| `LEDs.ino` | LED sequence flashing |
| `buttons.ino` | Debounced button input reading |
| `LCD.ino` | Scrollable menu + flicker-free score display |
| `Buzzer.ino` | Sound feedback |
| `flashMemory.ino` | EEPROM top-5 high-score persistence |
| `utils.ino` | Random sequence generation + timing helpers |
| `Config.h` | Global constants and difficulty settings |
| `Hardware.h` | Pin definitions |

**Key implementation details:**
- `millis()` used throughout — no `delay()` calls — keeps both players' inputs responsive simultaneously
- Debouncing logic prevents mechanical button noise from causing false triggers
- LCD updates are buffered to avoid flickering during active gameplay
- Sleep mode activates after 2 minutes of menu inactivity, turning off backlight, LEDs, and buzzer
- EEPROM stores top 5 scores persistently across power cycles; scores can be wiped from the menu

---

### Testing & Results

| Stage | Method | Outcome |
|-------|--------|---------|
| Unit testing | Each `.ino` module tested in isolation after completion | All modules passed |
| Simulation | Full run in SimulIDE after code completion | Sleep mode timing bug found and fixed |
| Hardware prototype | Breadboard assembly tested before boxing | All components verified |
| Final hardware test | Complete test on finished enclosure | Worked as intended |

**Notable bug fixed:** A game round lasting over 2 minutes caused the sleep-mode timer to trigger immediately after game over, preventing the next mode from initializing correctly. Fixed by resetting the inactivity timer on state transitions.

---

### Possible Improvements

- Save top scores per game mode, not globally; store player initials with each score
- Add time-to-complete as a tiebreaker metric in Duel mode
- Expand sound feedback with a dedicated sound module instead of a single buzzer
- Add more game modes

---
---

<div align="center">
