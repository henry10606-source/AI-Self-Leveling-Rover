# AI Self-Leveling Rover Proposal
## AMB82-MINI Four-Wheel Mobile Platform with Two-Axis Auto-Leveling

---

## 1. Design Concept

This project proposes an **AI/IoT-enabled four-wheel mobile platform** based on the **AMB82-MINI** controller.

The core idea is:

> Even when the rover climbs uphill, drives downhill, or tilts sideways, the upper platform automatically compensates for the body angle and remains approximately horizontal.

The robot is divided into two functional layers:

- **Mobility layer:** four-wheel rover motion.
- **Stabilization layer:** a two-axis platform that compensates for **Pitch** and **Roll**.

The AMB82-MINI is the main controller. An external IMU measures vehicle attitude, and two high-torque servo motors adjust the platform in real time.

The project also adds **IoT monitoring** and an **AI-assisted terrain-state function**, so the system combines embedded control, sensing, networking, and AI-assisted decision making.

---

## 2. Project Goals

### Goal 1 — Four-Wheel Mobile Rover

The rover can:

- move forward,
- move backward,
- turn left,
- turn right,
- stop.

A motor driver is used between the AMB82-MINI and the DC geared motors.

### Goal 2 — Two-Axis Auto-Leveling Platform

The upper platform automatically compensates for:

- **Pitch** — uphill / downhill tilt,
- **Roll** — left / right tilt.

The target platform attitude is:

```text
Target Pitch = 0°
Target Roll  = 0°
```

Example:

```text
Rover body Pitch = +15°
Platform compensation ≈ -15°
Final platform angle ≈ 0°
```

### Goal 3 — Real-Time IMU Feedback

An external IMU continuously measures:

```text
Acceleration
Angular velocity
Pitch
Roll
```

The AMB82-MINI uses this feedback to calculate the platform correction angle.

### Goal 4 — IoT Monitoring and Control

The AMB82-MINI provides a simple Wi-Fi control / monitoring interface.

The user can view:

- rover status,
- Pitch angle,
- Roll angle,
- platform correction angle,
- driving mode,
- stabilization status.

The user can send high-level commands such as:

```text
forward
backward
left
right
stop
auto_level_on
auto_level_off
```

### Goal 5 — AI-Assisted Terrain State Recognition

The AI function is not responsible for direct safety-critical servo control.

Instead, IMU data is used to classify the rover state into terrain conditions such as:

```text
FLAT
UPHILL
DOWNHILL
LEFT_TILT
RIGHT_TILT
UNSTABLE
```

The classification result can be used to:

- reduce driving speed on large slopes,
- display terrain status on the IoT dashboard,
- warn the user when the tilt is too large,
- switch between normal and cautious driving modes.

The real-time balancing loop remains local on the AMB82-MINI.

---

# 3. System Architecture

```text
                         USER
                           │
                    Wi-Fi / Web UI
                           │
                           ▼
                 ┌───────────────────┐
                 │    AMB82-MINI     │
                 │                   │
                 │ Robot Controller  │
                 │ IoT Interface     │
                 │ Terrain AI Logic  │
                 └─────────┬─────────┘
                           │
              ┌────────────┼─────────────┐
              │            │             │
              ▼            ▼             ▼
          External IMU   Motor Driver   Servo Control
              │            │             │
              │            │         ┌───┴────┐
              │            │         │        │
              │            │      Pitch     Roll
              │            │      Servo     Servo
              │            │         │        │
              │            │         └───┬────┘
              │            │             │
              │            │      Auto-Level Platform
              │            │
              │      ┌─────┴──────────────┐
              │      │                    │
              ▼      ▼                    ▼
          Pitch/Roll  Left Wheels      Right Wheels
```

The controller has two simultaneous loops.

### Mobility Loop

```text
User / AI Command
      │
      ▼
AMB82-MINI
      │
      ▼
Motor Driver
      │
      ▼
Four-Wheel Motion
```

### Stabilization Loop

```text
IMU
 │
 ▼
Pitch / Roll Measurement
 │
 ▼
AMB82-MINI
 │
 ▼
Filtering + PID Control
 │
 ▼
Pitch Servo + Roll Servo
 │
 ▼
Platform stays horizontal
 │
 └──────────── feedback ────────────► IMU
```

---

# 4. Hardware Design

## Main Hardware

| Component | Quantity | Source | Purpose |
|---|---:|---|---|
| AMB82-MINI | 1 | Provided by instructor | Main robot controller |
| External 6-axis IMU (e.g. MPU6050-class) | 1 | Self-prepared | Measure Pitch / Roll |
| High-torque servo motor | 2 | Self-prepared | Two-axis platform correction |
| Four-wheel rover chassis | 1 | Self-prepared | Mechanical base |
| DC geared motor | 4 | Self-prepared | Wheel drive |
| Motor driver | 1 or more | Self-prepared | Drive DC motors |
| Wheels | 4 | Self-prepared | Rover movement |
| Battery pack | 1 | Self-prepared | Main power source |
| 5–6 V high-current servo supply / regulator | 1 | Self-prepared | Servo power |
| Two-axis platform frame | 1 | Self-prepared / fabricated | Mechanical leveling mechanism |
| Wires, connectors, screws, brackets | as needed | Self-prepared | Assembly |

## Mechanical Structure

```text
                ┌────────────────────┐
                │   LEVEL PLATFORM   │
                │       Payload      │
                └─────────┬──────────┘
                          │
                     Roll Axis
                     Servo Y
                          │
                     Pitch Axis
                     Servo X
                          │
                ┌────────────────────┐
                │   FOUR-WHEEL BODY  │
                │    AMB82-MINI      │
                │       IMU          │
                └────────────────────┘
                   O              O
                   O              O
```

The first servo compensates for forward / backward tilt.

The second servo compensates for left / right tilt.

This forms a simple **two-axis gimbal platform** mounted on a mobile rover.

## Power Design

The servo motors and wheel motors should not be powered directly from AMB82-MINI output pins.

Recommended power structure:

```text
Battery
  │
  ├──► Motor Driver ──► DC Motors
  │
  ├──► 5–6 V Regulator ──► Servo Motors
  │
  └──► Regulated Supply ──► AMB82-MINI

All grounds are connected together.
```

This reduces controller reset, voltage drop, servo noise, and unstable sensor readings.

---

# 5. Auto-Leveling Control Algorithm

## Step 1 — Read IMU

The controller continuously obtains:

```text
Pitch_measured
Roll_measured
```

## Step 2 — Define Target Angle

```text
Pitch_target = 0°
Roll_target  = 0°
```

## Step 3 — Calculate Error

```text
Pitch_error = Pitch_target - Pitch_measured
Roll_error  = Roll_target - Roll_measured
```

\[
e_{pitch}=0^\circ-	heta_{pitch}
\]

\[
e_{roll}=0^\circ-	heta_{roll}
\]

## Step 4 — Filter Sensor Data

Raw IMU data contains vibration and motor noise.

```text
Raw IMU Data
     │
     ▼
Noise Filter
     │
     ▼
Stable Pitch / Roll
```

A low-pass filter or complementary filter can be used.

## Step 5 — PID Compensation

The stabilization controller can use PID control:

\[
u(t)=K_Pe(t)+K_I\int e(t)\,dt+K_Drac{de(t)}{dt}
\]

where:

```text
e(t) = platform angle error
KP   = proportional gain
KI   = integral gain
KD   = derivative gain
u(t) = servo correction command
```

The first prototype can begin with proportional control and then upgrade to PID.

## Step 6 — Servo Output

Example neutral position:

```text
Pitch Servo Neutral = 90°
Roll Servo Neutral  = 90°
```

The controller calculates:

```text
Pitch Servo = 90° + Pitch correction
Roll Servo  = 90° + Roll correction
```

Software limits are added to prevent the servo from exceeding the mechanical range.

---

# 6. AI and IoT Features

## IoT Dashboard

The rover publishes status information through Wi-Fi.

Example:

```text
Robot Status: RUNNING

Pitch: +12.4°
Roll:  -2.1°

Pitch Servo: 77°
Roll Servo:  92°

Terrain: UPHILL
Auto-Level: ON
Motor Speed: 55%
```

Possible user commands:

```text
move_forward()
move_backward()
turn_left()
turn_right()
stop()

auto_level_on()
auto_level_off()

set_speed(level)
robot_status()
```

The commands remain high-level. The user does not directly control low-level PWM values.

## AI-Assisted Terrain Classification

Input:

```text
Pitch
Roll
Acceleration
Angular velocity
```

Output:

```text
FLAT
UPHILL
DOWNHILL
LEFT_TILT
RIGHT_TILT
UNSTABLE
```

Example:

```text
IMU
 │
 ▼
Feature / AI Classification
 │
 ▼
UPHILL
 │
 ├──► Dashboard displays "UPHILL"
 │
 ├──► Reduce maximum speed
 │
 └──► Increase monitoring frequency
```

This AI layer assists driving behavior but does **not** replace the real-time stabilization controller.

---

# 7. Safety Principle

The AMB82-MINI has final authority over real-time motion safety.

```text
User / AI Command
       │
       ▼
AMB82-MINI Safety Supervisor
       │
       ├── excessive tilt → STOP
       │
       ├── IMU failure → STOP
       │
       ├── communication timeout → STOP
       │
       ├── servo angle limit → LIMIT OUTPUT
       │
       ├── low voltage → WARNING / STOP
       │
       └── safe → execute command
```

Example:

```text
if |Pitch| > Safety_Limit
or |Roll| > Safety_Limit:

    stop motors
    keep platform in safe position
    send warning
```

The AI system cannot override this local safety layer.

---

# 8. Development Plan and Evaluation

## Stage 1 — IMU Test

Goal:

- connect the external IMU to AMB82-MINI,
- obtain stable Pitch and Roll values,
- verify angle direction and sensor calibration.

Success condition:

```text
Tilt forward → Pitch changes correctly
Tilt sideways → Roll changes correctly
```

## Stage 2 — One-Axis Leveling

Hardware:

```text
AMB82-MINI
+
IMU
+
1 Servo
+
Small Platform
```

Goal:

Tilt the system forward and backward. The platform should rotate in the opposite direction.

## Stage 3 — Two-Axis Leveling

Add the second servo.

Goal:

```text
Pitch
+
Roll
```

The platform should remain approximately horizontal when the base is tilted in different directions.

## Stage 4 — Four-Wheel Rover

Install the balancing platform on the rover chassis.

Test:

```text
Forward
Backward
Left
Right
Stop
```

The stabilization system continues running while the rover moves.

## Stage 5 — Ramp Test

Prepare different slopes.

Example:

```text
0°
10°
20°
30°
```

The rover drives uphill and downhill while the upper platform attempts to remain horizontal.

Possible demonstration payload:

- cup with a small amount of water,
- ball,
- smartphone,
- small object.

## Stage 6 — IoT Dashboard

Add:

- real-time Pitch / Roll display,
- terrain-state display,
- rover status,
- drive commands,
- auto-level ON / OFF.

## Stage 7 — AI Terrain Function

Collect IMU data for:

```text
Flat
Uphill
Downhill
Left tilt
Right tilt
Unstable motion
```

Use the data to build a lightweight terrain-state classifier.

## Evaluation Metrics

### Platform Level Error

Prototype target:

```text
Static platform error ≤ ±3°
```

under moderate slope conditions.

### Response Time

Measure:

```text
Base tilt
   ↓
IMU detection
   ↓
Servo correction
   ↓
Platform stabilizes
```

### Stability

Observe:

```text
overshoot
servo vibration
platform oscillation
settling time
```

### Mobility

The rover should successfully:

```text
move forward
move backward
turn left
turn right
stop
climb the test ramp
```

while maintaining platform stabilization.

---

# 9. Final Recommendation

The recommended final system is:

```text
                    USER
                      │
               Wi-Fi / Web UI
                      │
                      ▼
              ┌───────────────┐
              │   AMB82-MINI  │
              │               │
              │ IoT Interface │
              │ Terrain AI    │
              │ PID Control   │
              │ Safety Logic  │
              └───────┬───────┘
                      │
        ┌─────────────┼──────────────┐
        │             │              │
        ▼             ▼              ▼
       IMU       Motor Driver    2-Axis Servo
        │             │              │
  Pitch / Roll    4 DC Motors    Level Platform
```

The most important design principle is to separate:

```text
AI / IoT high-level intelligence
```

from:

```text
real-time stabilization and safety control
```

The AMB82-MINI performs the local control loop continuously, while the AI / IoT layer provides terrain recognition, monitoring, user interaction, and higher-level behavior.

The final prototype integrates:

- embedded control,
- IMU sensing,
- servo control,
- four-wheel mobility,
- feedback control,
- IoT communication,
- AI-assisted state recognition,
- physical system safety.

This makes the project suitable as a complete **Artificial Intelligence and IoT Device Design** final project.

---

## Project Name

**AI Self-Leveling Rover**

### Chinese Title

**AMB82-MINI 四輪 AI 自動水平載物平台車**

### English Title

**AI Self-Leveling Rover: AMB82-MINI Four-Wheel Mobile Platform with Two-Axis Auto-Leveling**
