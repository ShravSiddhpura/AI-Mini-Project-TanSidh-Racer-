# AI-Mini-Project-TanSidh-Racer-

Here is a complete document for your "Artificial Intelligence" subject, based on the game code you provided.

-----

# AI Project Document: Cyber Drift: Rival Race

**Subject:** Artificial Intelligence
**Project:** Cyber Drift: Rival Race (A 3D Racing Game with a Predictive AI)

-----

## 1\. The Game: Introduction

**Cyber Drift: Rival Race** is a real-time, 3D "endless runner" style racing game developed using **Three.js (JavaScript)**. The player controls a cyan-colored car in a futuristic, neon-drenched environment.

The primary objective is to drive as far as possible, accumulating distance and score, while avoiding static (yellow) obstacles. The key feature of this project is the inclusion of a single, AI-controlled "Rival" car (pink-red) that races alongside the player. This Rival uses predictive AI to navigate the same environment, avoid obstacles, and maintain a competitive pace with the player.

-----

## 2\. Game Rules & Objectives

The rules of the game are simple:

  * **Controls:**
      * **W:** Accelerate
      * **S:** Brake / Reverse
      * **A / D:** Steer Left / Steer Right
      * **R:** Restart the game
      * **Esc:** Pause the game
  * **Objective:** Survive for as long as possible. The game ends if the player collides with a yellow obstacle or the Rival car.
  * **Shield:** The player and the Rival both have a short-term shield at the beginning of the game, allowing them to pass through obstacles or each other without penalty.
  * **Scoring:** The player's score increases by **150 points** each time they successfully pass the Rival. The total distance traveled is also tracked.
  * **Game Over:** A "Game Over" screen is displayed upon collision (e.g., "CRASHED" or "HIT BY RIVAL").

-----

## 3\. Technology Used

  * **Core Language:** JavaScript (ES6)
  * **Rendering Engine:** **Three.js** (v0.160.0). This library is used for all 3D operations, including:
      * Scene setup, camera (PerspectiveCamera), and lighting (HemisphereLight, DirectionalLight).
      * Creating and managing 3D objects (Geometries like `BoxGeometry`, `PlaneGeometry`, `CylinderGeometry`).
      * Applying materials and colors (`MeshStandardMaterial`, `MeshBasicMaterial`).
      * Running the main `animate` render loop.
  * **Platform:** HTML5 Canvas (via WebGL renderer) and CSS3 for the HUD.
  * **World Generation:** The game uses a **procedural looping** (or "circular buffer") system to create the endless road. As segments, stripes, and roadside structures move past the player, they are recycled and moved back to the front, creating the illusion of infinite travel.

-----

## 4\. AI Algorithm Analysis

The core of this project is the `Rival` class. This AI agent's behavior is governed by a set of rules and a predictive algorithm designed to make it a competent and realistic-looking co-racer.

### 4.1. PEAS Representation

This AI can be described using the **PEAS (Performance, Environment, Actuators, Sensors)** framework:

  * **Performance Measure:**
    1.  **Survival:** Do not crash into obstacles.
    2.  **Pacing:** Stay close to the player's speed (not too fast, not too slow).
    3.  **Proximity:** Remain within the player's view (managed by "tethering" logic).
    4.  **Collision Avoidance:** Do not crash into the player.
  * **Environment:**
      * **Dynamic:** The player and obstacles are constantly moving relative to the AI.
      * **Partially Observable:** The AI "sees" the entire obstacle list but only *acts* on obstacles within a finite `lookAhead` distance.
      * **Continuous:** The game world uses continuous 3D coordinates (position, velocity).
      * **Multi-agent:** The environment contains the AI agent and the human player.
  * **Actuators:**
      * `this.speed`: The AI's target forward velocity.
      * `this.lane`: The AI's target X-axis position.
  * **Sensors:**
      * `player.velocity.z` (Player's speed)
      * `player.pos` (Player's position)
      * `obstacles` (A list of all obstacle objects and their positions)
      * `this.pos` (Its own position)

### 4.2. Agent Type: Model-Based Reflex Agent

This AI is a **Model-Based Reflex Agent**.

  * It is *not* a simple reflex agent because it doesn't just react to the *current* state.
  * It is **model-based** because it uses its `lookAhead` distance to predict a *future state*—namely, "Will my current lane be blocked in the next `X` meters?" It models the world to make a decision.

### 4.3. Core Algorithm: Predictive Look-Ahead

The AI's decision-making logic resides in the `Rival.update()` function and runs on every frame.

**1. Speed Matching (Reactive Behavior):**
The Rival first matches the player's speed.

```javascript
// compute player "forward speed"
const playerSpeed = Math.abs(player.velocity.z);

// map to rival speed units (slightly faster to feel competitive)
const desired = playerSpeed * 1.0 + 0.6; 
this.speed = THREE.MathUtils.lerp(this.speed, desired, dt * 1.8);
```

This is a reactive behavior that keeps the Rival engaged with the player.

**2. Obstacle Avoidance (Predictive Behavior):**
This is the main AI algorithm.

  * **Step 1: Define Look-Ahead:** The AI calculates a "danger zone" in front of it. This zone *grows* as the player's (and thus its own) speed increases, which is a key feature.

    ```javascript
    const lookAhead = 28 + playerSpeed*8; 
    ```

  * **Step 2: Check Current Lane:** The AI iterates through all obstacles to see if any are inside its danger zone and in its current lane.

    ```javascript
    let blocked = false;
    for(const o of obstacles){
      const delta = this.pos.z - o.position.z; // (z decreases, so positive delta is "ahead")
      if(delta > 0 && delta < lookAhead){
        if(Math.abs(o.position.x - this.pos.x) < 1.5){
          blocked = true;
          break;
        }
      }
    }
    ```

  * **Step 3: Lane Selection (If Blocked):** If `blocked` is true, the AI must find a new lane. This follows a **Greedy, Best-First Search** strategy, where the "heuristic" is a hardcoded preference list.

    ```javascript
    if(blocked){
      // prefer center lane, then left/right
      const preference = [0, -4, 4]; 
      const options = preference.filter(l => Math.abs(l - this.pos.x) > 0.1);
      
      let chosen = this.lane; // default stay
      for(const opt of options){
        // check if 'opt' is free in lookAhead zone
        let optBlocked = false;
        // ... (inner loop, same as Step 2 but for 'opt') ...
        
        if(!optBlocked){
          chosen = opt; // Choose the *first* available preferred lane
          break;
        }
      }
      this.lane = chosen;
    }
    ```

**3. Player Avoidance & Tethering (Utility Behaviors):**

  * **Player Avoidance:** A simple reflex rule nudges the AI away if it gets too close to the player, preventing non-game-over collisions.
  * **Tethering:** The AI is "tethered" (or "rubber-banded") to the player's Z-position. This is a common game design trick to ensure the Rival is always visible and part of the action, rather than speeding off into the distance or falling infinitely behind.

### 4.4. Connection to AI Syllabus

  * **Agents and Environments:** This project is a perfect implementation of a **Model-Based Agent** in a **Dynamic, Continuous, Multi-Agent Environment**. The PEAS representation is clearly defined.
  * **Search Methods (Informed Search):** The lane-selection logic (Step 3) is a form of **Greedy Best-First Search**.
      * **State:** The AI's current lane.
      * **Goal:** Be in a lane that is not blocked.
      * **Heuristic:** The `preference = [0, -4, 4]` array. The AI *greedily* assumes the center lane (`0`) is the "best" choice. If that's blocked, it tries its next-best, and so on. It picks the first available option from its preferred list without looking any further ahead.
  * **Local Search:** This can also be seen as a form of **Hill Climbing** (a type of Local Search). The "height" is "safety." If the current lane is "unsafe" (blocked), it moves to the first neighbor state (preferred lane) that has a higher "safety" (is not blocked).
  * **Adversarial Search:** This project **does not** use adversarial search (like Minimax or Alpha-Beta Pruning). The Rival is not "against" the player; it is not trying to block or crash the player. Its goal is *survival* and *pacing*.

-----

## 5\. Algorithm Complexity

The efficiency of the AI algorithm is critical for a real-time game that must run every frame (ideally 60 times per second).

Let **`N`** = the total number of obstacles in the `obstacles` array.
Let **`L`** = the number of alternative lanes to check (a constant, `L=2` in this code).

The `Rival.update()` function runs in **$O(N)$** time complexity per frame.

**Analysis:**

1.  **Speed Matching:** $O(1)$ (constant time).
2.  **Check Current Lane (Step 2):** The AI loops through all obstacles. In the worst case, it checks all `N` obstacles. This is $O(N)$.
3.  **Lane Selection (Step 3):** This only runs if the lane is blocked.
      * It loops `L` times (for the `options` array).
      * Inside *each* of those loops, it *again* loops through all `N` obstacles to check if the new lane is clear.
      * The complexity for this step is $O(L \times N)$.
4.  **Total Complexity:** $O(1) + O(N) + O(L \times N)$.

Since `L` is a small constant (2), it is not a variable. Therefore, the complexity $O(L \times N)$ simplifies to $O(N)$.

The total time complexity of the AI's decision-making process per frame is **$O(N)$**. This is extremely efficient and perfectly suitable for real-time applications, as the AI's "thinking" time scales linearly with the number of obstacles on screen.

-----

## 6\. Sample Game Output

Below are descriptions of the game in action, illustrating the AI's behavior.

**Screenshot 1: The Race Begins**

*Description: The player and Rival start the race. The Rival's shield is active (bright glow).*

**Screenshot 2: Predictive Avoidance in Action**
[Image from behind the player, looking ahead. A yellow obstacle is in the center lane. The pink Rival, which was in the center, is now smoothly moving to the right lane (lane `4`) well before reaching the obstacle.]
*Description: The Rival's AI detects the yellow obstacle in its `lookAhead` zone. It has initiated a lane change to its preferred alternative (the right lane) to avoid a collision.*

**Screenshot 3: Game Over**

*Description: The player failed to avoid an obstacle, resulting in a "Game Over" state. Pressing 'R' will restart the game.*
