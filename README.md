# NRL JARVIS VISION — NATURAL TWO-HAND VIRTUAL STEERING COCKPIT

A modular desktop robotics control cockpit built for the National Robotics League (NRL). Featuring a dark graphite theme, electric cyan highlights, technical typography, pure control mathematics, **natural two-hand virtual steering gesture control**, differential drive mixing, and a 2D kinematic robot simulator.

---

## 1. Architecture Overview

```
main.py                       # Application Entry Point (Tkinter Desktop Engine)
requirements.txt              # Pinned Project Dependencies
README.md                     # Comprehensive Setup & Architecture Guide
models/
  └── hand_landmarker.task    # Official Google MediaPipe HandLandmarker Model Bundle
config/
  └── settings.json           # Application Configuration
src/
  ├── camera.py               # Camera capture lifecycle (DirectShow + fallback)
  ├── hand_tracker.py         # Multi-hand tracking engine (MediaPipe Tasks + OpenCV fallback)
  ├── tracking_types.py       # NormalizedPoint, SingleHandData, and HandTrackingResult types
  ├── opencv_hand_detector.py # OpenCV multi-hand contour & skin-color vision fallback
  ├── steering.py             # Steering processor (Two-hand angle estimator, EMA filter, deadband, calibration)
  ├── control_math.py         # Pure mathematical functions (chord angles, wrap_angle, differential mixing)
  ├── safety.py               # 8-state explicit state machine, E-stop, staleness checks
  ├── modes.py                # Exclusive operating mode manager
  ├── simulator.py            # Simulated 2D kinematic differential drive vehicle
  ├── telemetry.py            # Rolling FPS counters & performance timers
  ├── communications.py       # Abstract future hardware transport interface (disabled by default)
  └── ui/
      └── cockpit_ui.py       # JARVIS Cockpit Desktop UI (Tkinter Engine with Realistic Virtual Wheel)
tests/
  ├── test_control_math.py    # 72 mathematical unit tests (including two-hand chord & angle wrapping)
  ├── test_steering.py        # 16 steering processor, calibration, and zeroing tests
  ├── test_safety.py          # 20 state transition, arming precondition, and E-stop tests
  └── test_modes.py           # 6 operating mode exclusivity tests
```

---

## 2. Requirements & Installation

- **OS**: Windows Desktop
- **Python**: 3.12 (inside `.venv`)

### Quick Setup

```powershell
# Activate your virtual environment
.\.venv\Scripts\activate

# Install dependencies
python -m pip install -r requirements.txt
```

---

## 3. Running Automated Tests

Run the complete 140-test suite via `pytest`:

```powershell
python -m pytest tests/ -v
```

All 140 unit tests pass deterministically without requiring a webcam or physical hardware.

---

## 4. Launching the JARVIS Cockpit

Launch the cockpit desktop application:

```powershell
python main.py
```

---

## 5. Natural Two-Hand Virtual Steering Workflow

### Driving Instructions
1. **Extend Both Hands**: Sit comfortably in front of your webcam and hold both hands out as if gripping an invisible steering wheel at 9 and 3 o'clock.
2. **Observe Tracking**:
   - The vision feed will detect both hands.
   - Left hand is rendered in **Electric Cyan**; Right hand is rendered in **Electric Coral/Magenta**.
   - An indicator tag `L: OK | R: OK` confirms dual-hand lock.
   - A virtual steering wheel chord connects the palm centers of your hands on the camera feed.
3. **Calibrate Neutral Driving Pose**:
   - Hold your hands in your natural, level resting posture.
   - Click **`🎯 CALIBRATE NEUTRAL POSE`**.
   - Hold steady for 2.0 seconds while the system collects outlier-resistant median angle samples.
   - Status will update to `CALIBRATED (0° = X.X°)`.
4. **Arm the Cockpit**:
   - Press **`A`** or click **`ARM SYSTEM`**.
   - Top status transitions from `DISARMED` $\rightarrow$ `READY` $\rightarrow$ `RUNNING`.
5. **Steer Naturally**:
   - **Turn Right**: Lower your right hand and raise your left hand (rotating clockwise). The virtual wheel turns clockwise, and the vehicle steers right.
   - **Turn Left**: Lower your left hand and raise your right hand (rotating counter-clockwise). The virtual wheel turns counter-clockwise, and the vehicle steers left.
   - **Center**: Level your hands to return steering to `0.00`.
6. **Safety & Emergency Stop**:
   - If either hand leaves the camera frame, simulated commands **immediately zero** (`TRACKING LOST`).
   - Press **`SPACEBAR`** at any time for instant emergency stop.

---

## 6. Two-Hand Steering Mathematics

### Hand Orientation Angle
Given left hand palm center $(x_L, y_L)$ and right hand palm center $(x_R, y_R)$ in normalized image space ($x$ right, $y$ down):
$$dx = x_R - x_L, \quad dy = y_R - y_L$$
$$\theta = \text{atan2}(dy, dx)$$

- Level hands: $dy = 0 \implies \theta = 0^\circ$
- Turning right: Left hand goes UP ($y_L$ decreases), Right hand goes DOWN ($y_R$ increases) $\implies dy > 0 \implies \theta > 0$
- Turning left: Left hand goes DOWN ($y_L$ increases), Right hand goes UP ($y_R$ decreases) $\implies dy < 0 \implies \theta < 0$

### Calibrated Relative Steering
Given calibrated neutral angle $\theta_{\text{neutral}}$ and maximum steering range $\theta_{\text{max}} = 45^\circ$:
$$\Delta\theta = \text{wrap\_angle}(\theta - \theta_{\text{neutral}})$$
$$s_{\text{raw}} = \text{clamp}\left(\frac{\Delta\theta}{\theta_{\text{max}}}, -1.0, +1.0\right)$$

### Differential-Drive Motor Mixing
$$v = \text{throttle setpoint}, \quad s = \text{filtered steering}$$
$$\text{left}_{\text{raw}} = v - s, \quad \text{right}_{\text{raw}} = v + s$$
$$m = \max(1.0, |\text{left}_{\text{raw}}|, |\text{right}_{\text{raw}}|)$$
$$\text{left} = \frac{\text{left}_{\text{raw}}}{m}, \quad \text{right} = \frac{\text{right}_{\text{raw}}}{m}$$

---

## 7. Camera Overlay vs. Cockpit Wheel

There are two distinct visual representations:
1. **Camera Overlay**: A dynamic chord and center circle drawn in the camera image coordinates connecting the two detected palm anchors, scaling with physical hand separation.
2. **Cockpit Virtual Steering Wheel**: A stylized, circular rim with rotating 3-spoke hub and 12-o'clock alignment marker situated in the center cockpit dashboard. It rotates strictly according to calibrated controller angle $\Delta\theta$.

---

## 8. Hardware Warning & NRL Integration Note

> [!WARNING]
> The communications module (`src/communications.py`) is **disabled by default** and uses a `MockTransport`. The physical NRL Hexa Command Hub uses the NRL C++ SDK. No physical motor commands or unverified serial protocols should be transmitted until exact packet framing, pin mappings, and timeout safety are physically bench-tested.
