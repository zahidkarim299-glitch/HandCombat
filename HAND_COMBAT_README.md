# ⚔ HAND COMBAT — Full Control System README

> A two-player browser fighting game controlled entirely by **real-time hand gestures** captured through your webcam, powered by **MediaPipe Hand Tracking**.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Requirements & Setup](#2-requirements--setup)
3. [Player Positions](#3-player-positions)
4. [How Hand Tracking Works](#4-how-hand-tracking-works)
5. [Gesture Reference — All Controls](#5-gesture-reference--all-controls)
6. [Weapons & Abilities — Detailed Breakdown](#6-weapons--abilities--detailed-breakdown)
   - [⚔ Sword (Fist)](#-sword-fist)
   - [🛡 Shield (Open Palm)](#-shield-open-palm)
   - [🏹 Energy Arrow (Point Finger)](#-energy-arrow-point-finger)
   - [💥 Shockwave (Clap)](#-shockwave-clap)
   - [⚡ Power Blast Beam (Double Open Palm Clap)](#-power-blast-beam-double-open-palm-clap)
7. [Combat Mechanics](#7-combat-mechanics)
   - [Damage Values](#damage-values)
   - [Cooldowns](#cooldowns)
   - [Shield Blocking](#shield-blocking)
   - [Hit Detection Ranges](#hit-detection-ranges)
8. [Game States](#8-game-states)
9. [HUD (Heads-Up Display)](#9-hud-heads-up-display)
10. [Screen Effects](#10-screen-effects)
11. [Tips & Strategy](#11-tips--strategy)
12. [Troubleshooting](#12-troubleshooting)
13. [Technical Reference](#13-technical-reference)

---

## 1. Overview

**Hand Combat** is a two-player fighting game where no keyboard or gamepad is required. Each player stands in front of a shared webcam and performs hand gestures to attack, defend, and use special abilities. The game uses Google's **MediaPipe Hands** library to detect up to 4 hands simultaneously (2 per player), enabling both single-hand and two-hand combination moves.

---

## 2. Requirements & Setup

| Requirement | Details |
|---|---|
| **Browser** | Any modern browser (Chrome, Firefox, Edge, Safari) |
| **Webcam** | Required — 720p or higher recommended |
| **Camera Permission** | Must be **allowed** when prompted |
| **Internet** | Required to load MediaPipe CDN scripts |
| **Players** | 2 players standing in front of the same webcam |
| **Lighting** | Good, even lighting improves hand detection accuracy |

### Starting the Game

1. Open `hand-combat.html` in your browser.
2. Click **Allow** when the browser requests webcam access.
3. Both players position themselves in front of the webcam (see [Player Positions](#3-player-positions)).
4. Once both hands are detected, the game begins automatically.

---

## 3. Player Positions

The webcam frame is split into two halves. **Each player must stay in their designated half at all times.**

```
┌─────────────────────┬─────────────────────┐
│                     │                     │
│   PLAYER 1 (Blue)   │  PLAYER 2 (Orange)  │
│                     │                     │
│   Stand on the      │   Stand on the      │
│   LEFT side of      │   RIGHT side of     │
│   the webcam view   │   the webcam view   │
│                     │                     │
└─────────────────────┴─────────────────────┘
         ◄ Your LEFT          Your RIGHT ►
         (as seen by the camera, not mirrored)
```

> **Important:** The game reads raw camera coordinates. Player 1's zone is where the wrist X-position is less than 50% of the frame width. Player 2's zone is the other 50%. Crossing into the other player's zone may cause your hands to be mis-assigned.

**Player Colors:**
- **Player 1** — Blue (`#00c8ff`) — Left side
- **Player 2** — Orange/Red (`#ff5520`) — Right side

---

## 4. How Hand Tracking Works

The game uses **MediaPipe Hands** running locally in your browser. No video data is sent to any server.

- Up to **4 hands** are tracked simultaneously (2 per player).
- Each hand is represented by **21 landmarks** — key points on the fingers, knuckles, and wrist.
- The game reads these landmarks every camera frame (at 1280×720 resolution) to determine:
  - **Which player** the hand belongs to (based on wrist X position).
  - **What gesture** is being made (based on finger extension states).
  - **Where the hand is** in canvas space (for hit detection and weapon positioning).

The skeleton of each hand is drawn on screen in the player's color as visual feedback.

**MediaPipe Settings used by the game:**

| Setting | Value | Effect |
|---|---|---|
| `maxNumHands` | 4 | Allows 2 hands per player |
| `modelComplexity` | 0 (Lite) | Faster tracking, lower CPU |
| `minDetectionConfidence` | 0.7 | High reliability threshold |
| `minTrackingConfidence` | 0.5 | Maintains tracking mid-gesture |

---

## 5. Gesture Reference — All Controls

These are all the gestures each player can perform. Gestures are read from the **primary (first detected) hand** unless otherwise noted. Two-hand gestures require **both hands to be in your half** of the camera frame.

| Gesture | Hand Shape | Action | Type |
|---|---|---|---|
| ✊ **Fist** | All 4 fingers curled down | Summon Glowing Sword | Single-hand |
| 🖐 **Open Palm** | All 4 fingers extended up | Raise Energy Shield | Single-hand |
| ☝ **Point Finger** | Only index finger extended | Shoot Energy Arrow | Single-hand |
| 👏 **Clap** | Both hands brought close together (any gesture) | Release Shockwave | Two-hand |
| 🖐🖐 **Double Palm Clap** | Both hands open AND brought close together | Fire Power Blast Beam | Two-hand |

### Gesture Detection Logic

The game detects finger state by comparing the **tip landmark** to the **PIP (middle joint) landmark** of each finger:

| Finger | Tip Landmark | PIP Landmark |
|---|---|---|
| Index | 8 | 6 |
| Middle | 12 | 10 |
| Ring | 16 | 14 |
| Pinky | 20 | 18 |

A finger is counted as **extended (up)** if its tip Y-coordinate is **above** (lower Y value than) its PIP joint.

| Extended Fingers | Gesture Detected |
|---|---|
| 0 fingers up | `FIST` → ⚔ Sword |
| 4 fingers up | `PALM` → 🛡 Shield |
| Index only up | `POINT` → 🏹 Arrow |
| Any other combination | `NONE` → No weapon |

> **Note:** The thumb is not used in gesture detection. Only the index, middle, ring, and pinky fingers are evaluated.

---

## 6. Weapons & Abilities — Detailed Breakdown

### ⚔ Sword (Fist)

**Gesture:** Make a fist — curl all four fingers down.

**How it works:**
- A glowing energy sword appears at your wrist and extends in the natural direction of your hand.
- The blade's angle is calculated from **wrist (Landmark 0) → middle finger base (Landmark 9)**, so it always points where your hand is naturally aimed.
- The sword has a blade length of **148 pixels**.
- A motion trail of up to 12 ghost-blade samples follows the tip, creating a blur effect as you swing.

**Hitting the opponent:**
- The sword deals damage when its **blade tip** comes within **75 pixels** of the opponent's wrist.
- You must swing or extend your arm toward the opponent's hand position.
- The weapon disappears instantly if you change your gesture (e.g., opening your palm).

**Sword Visual:**
- Yellow-gold glowing blade with a bright white core.
- Cross-guard rendered at 22% along the blade length.
- Diamond sparkle at the blade tip.
- Orange motion trail along recent tip positions.

---

### 🛡 Shield (Open Palm)

**Gesture:** Open your hand flat — extend all four fingers upward.

**How it works:**
- A pulsating circular energy shield appears centered on your wrist.
- The shield radius oscillates between ~59 px and ~77 px (base radius 68 px + a sine wave pulse of ±9 px).
- While shielded, all incoming damage is **completely blocked** — arrows, shockwaves, beams, and sword strikes.
- **Either hand** showing an open palm grants the shield. If your secondary hand is open, you remain shielded even if your primary hand changes gesture.

**Shield Visual:**
- Glowing ring in the player's color with a translucent fill.
- A rotating hexagonal inner pattern.
- Two inner concentric rings at 65% and 38% of the shield radius.

> **Strategy:** You cannot attack while shielding with both hands. You must choose between offense and defense.

---

### 🏹 Energy Arrow (Point Finger)

**Gesture:** Extend only your index finger — keep middle, ring, and pinky curled.

**How it works:**
- An energy arrow is **automatically fired** from your wrist position every time the gesture is held.
- Arrows travel horizontally at **13 pixels per frame** across the screen.
  - Player 1's arrows fly **right** (positive X direction).
  - Player 2's arrows fly **left** (negative X direction).
- Each arrow leaves an energy trail of up to 18 trail samples behind it.
- Arrows are destroyed when they leave the canvas or hit their target.

**Hit detection:**
- An arrow hits the opponent if it comes within **58 pixels** of their wrist.
- A shielded opponent **blocks** the arrow (no damage, a shield ring effect plays).

**Arrow Visual:**
- Glowing orb in the player's color with a bright white core.
- Fading energy trail in the player's color behind it.

---

### 💥 Shockwave (Clap)

**Gesture:** Bring **both hands** close together (wrists within **190 pixels** of each other), with **any gesture combination** (except both open palms, which triggers the Beam instead).

**How it works:**
- A large expanding ring erupts from the midpoint between your two wrists.
- The shockwave expands outward at **11 pixels per frame** until it fills the screen (max radius: 55% of the largest screen dimension).
- It deals damage to the opponent **once**, when the expanding ring passes through the opponent's hand position (within a 30 px tolerance band).
- A shielded opponent blocks the shockwave.

**Shockwave Visual:**
- A large glowing ring in the player's color that fades as it expands.
- A thick outer glow layer plus a bright inner ring.
- Screen shake of magnitude 9 on trigger.

---

### ⚡ Power Blast Beam (Double Open Palm Clap)

**Gesture:** Open **both hands** flat (both showing open palm gesture) AND bring them close together (wrists within **190 pixels**).

**How it works:**
- A full-width horizontal energy beam fires from your side of the screen across the entire arena.
- The beam lasts for **80 frames** and deals its damage at approximately **45% through** its duration (frame 36).
- The beam width pulses: base width 20 px ± 12 px as a sine wave.
- A shielded opponent blocks the beam.

**Beam Visual:**
- A wide horizontal beam stretching from one edge of the screen to the other.
- Gradient coloring: player color → white center → player color.
- Intense screen shake (magnitude 18) and a large screen flash on trigger.

> **Note:** The Power Blast Beam has the **highest damage** and **longest cooldown** in the game. Use it sparingly.

---

## 7. Combat Mechanics

### Damage Values

| Attack | Damage |
|---|---|
| ⚔ Sword strike | 14 HP |
| 🏹 Energy Arrow | 22 HP |
| 💥 Shockwave | 28 HP |
| ⚡ Power Blast Beam | 35 HP |

All players start with **100 HP**. The game ends when one player reaches **0 HP**.

### Cooldowns

These are the minimum time gaps between uses of each ability:

| Ability | Cooldown |
|---|---|
| ⚔ Sword hit | 1,200 ms (1.2 seconds) |
| 🏹 Arrow fire rate | 900 ms (0.9 seconds) |
| 💥 Shockwave | 2,000 ms (2 seconds) |
| ⚡ Power Blast Beam | 3,500 ms (3.5 seconds) |

> Cooldowns are **per-player** and tracked independently. Gestures can be held continuously — the game handles rate limiting automatically.

### Shield Blocking

When a player has the **Shield active** (`isShield = true`):
- All incoming damage is reduced to **0**.
- A **shield ring** impact effect plays at the point of impact.
- The screen does **not** shake on a blocked hit.

Shield is active when:
- The primary hand shows `PALM` gesture, **OR**
- The secondary (second detected) hand shows `PALM` gesture.

### Hit Detection Ranges

| Source | Detection Method | Range |
|---|---|---|
| Sword blade tip vs. opponent wrist | Euclidean distance | 75 px |
| Arrow projectile vs. opponent wrist | Euclidean distance | 58 px |
| Shockwave ring vs. opponent wrist | Distance from ring center vs. opponent, compared to ring radius | ±30 px band |
| Power Beam vs. opponent | Automatic at frame 36 of beam duration | N/A (full-screen) |

---

## 8. Game States

The game cycles through three states:

### WAITING
- Displayed on startup or after a match.
- Shows the title screen, gesture guide, and per-player ready status.
- The game transitions to `PLAYING` **automatically** as soon as **both** players' hands are detected in their respective halves.

### PLAYING
- Active combat. All weapons, projectiles, and effects are live.
- The HUD (health bars, gesture icons, VS badge) is displayed.
- Gesture hints (`⚔ SWORD`, `🛡 SHIELD`, `↗ ARROW`) appear near each player's hand.

### GAMEOVER
- Displayed when one player's HP drops to 0.
- The winner's color fills the screen with a large "PLAYER X WINS!" message.
- **To restart:** The **winning player** makes a **FIST** gesture. The game resets immediately.
  - Both players' HP resets to 100.
  - All projectiles, effects, and cooldowns are cleared.

---

## 9. HUD (Heads-Up Display)

Two health bars are shown at the top of the screen during play.

| Element | Description |
|---|---|
| **HP Bar** | Fills left-to-right (P1) or right-to-left (P2). Color changes green → yellow → red as HP drops. |
| **HP Number** | Shows current/max HP (e.g., `86 / 100`) centered on the bar. |
| **Player Label** | `PLAYER 1` or `PLAYER 2` in the player's color below the bar. |
| **Gesture Icon** | Small icon on the bar showing the current gesture: `⚔` Fist, `🛡` Palm, `↗` Point. |
| **VS Badge** | A dim `VS` label centered between the two bars. |

---

## 10. Screen Effects

| Effect | Trigger | Behavior |
|---|---|---|
| **Screen Shake** | Any successful hit | Canvas translates randomly by ±shake value each frame; decays by ×0.78 per frame |
| **Hit Flash** | Any successful hit | A semi-transparent color overlay fades over the screen; decays by ×0.82 per frame |
| **Hit Ring** | Weapon impact | An expanding circle ring at the point of impact (different style for blocked vs. unblocked hits) |
| **Motion Trail** | Sword swings, Arrow movement | Ghost samples of previous positions drawn with decreasing opacity |

Shake magnitudes by attack type:
- Sword: **13**
- Shockwave trigger: **9**
- Arrow impact: via `hitPlayer` → **13**
- Beam trigger: **18**

---

## 11. Tips & Strategy

- **Mix offense and defense.** The Shield blocks everything but leaves you unable to attack. Drop it to counter-attack when the opponent commits to a move.
- **Arrows are reliable.** With a 0.9-second fire rate, pointing your finger is a safe way to chip damage from a distance.
- **Save your Beam.** The Power Blast Beam does the most damage (35 HP) but has a 3.5-second cooldown. Use it as a finisher.
- **Shockwaves punish static opponents.** If your opponent holds still, a shockwave will always connect. But a moving shield blocks it.
- **Sword range matters.** Your sword tip needs to be within 75 px of the opponent's wrist — lunge your arm toward them while making a fist.
- **Two-hand specials require commitment.** Bringing both hands together means neither hand is available for defense during the gesture. Time it carefully.
- **Crossing the center divider** may cause hand mis-assignment. Stay in your half.

---

## 12. Troubleshooting

| Problem | Solution |
|---|---|
| Webcam not detected | Ensure you clicked "Allow" on the browser permission prompt. Try reloading the page. |
| Hand not recognized | Ensure good lighting. Keep your hand clearly visible and avoid fast, blurry movement. |
| Wrong player detected | Make sure each player keeps their hand fully within their half of the camera frame. |
| Gestures not registering | Extend fingers clearly and deliberately. Avoid in-between positions between fist and palm. |
| Game won't start | Both players' hands must be visible simultaneously. The waiting screen shows who is ready. |
| Lag or slow tracking | The game uses the Lite MediaPipe model for speed. Close other browser tabs or apps using the webcam. |

---

## 13. Technical Reference

### Landmark Indices Used

| Landmark | Index | Used For |
|---|---|---|
| Wrist | 0 | Player position, sword origin, shield center, two-hand distance |
| Index tip / PIP | 8 / 6 | Gesture detection |
| Middle tip / PIP | 12 / 10 | Gesture detection |
| Ring tip / PIP | 16 / 14 | Gesture detection |
| Pinky tip / PIP | 20 / 18 | Gesture detection |
| Middle finger base | 9 | Sword angle calculation |

### Key Constants (`CFG` object)

| Constant | Value | Meaning |
|---|---|---|
| `START_HP` | 100 | Starting health points |
| `SWORD_DAMAGE` | 14 | Sword damage per hit |
| `SWORD_CD` | 1200 ms | Sword hit cooldown |
| `SWORD_RANGE` | 75 px | Blade tip hit proximity |
| `ARROW_DAMAGE` | 22 | Arrow damage on hit |
| `ARROW_SPEED` | 13 px/frame | Arrow travel speed |
| `ARROW_FIRE_RATE` | 900 ms | Arrow fire rate cooldown |
| `SHOCK_DAMAGE` | 28 | Shockwave damage |
| `SHOCK_CD` | 2000 ms | Shockwave cooldown |
| `BEAM_DAMAGE` | 35 | Power Beam damage |
| `BEAM_CD` | 3500 ms | Power Beam cooldown |
| `BEAM_FRAMES` | 80 frames | Power Beam duration |
| `SHIELD_RADIUS` | 68 px | Base shield radius |
| `CLAP_DIST` | 190 px | Max wrist distance to count as clap |

### Technology Stack

| Component | Technology |
|---|---|
| Hand Tracking | [MediaPipe Hands](https://cdn.jsdelivr.net/npm/@mediapipe/hands/) |
| Camera Input | [MediaPipe Camera Utils](https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/) |
| Rendering | HTML5 Canvas 2D API |
| Game Loop | `requestAnimationFrame` (~60 FPS) |
| Video Input | 1280×720 webcam feed (hidden `<video>` element) |

---

*No data is collected. No video is sent to any server. All hand tracking runs entirely in your browser.*
