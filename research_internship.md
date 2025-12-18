# Research Internship — CUARL (CRAWLR)

## Overview
From **May 2025 to Dec 2025**, I worked at **Concordia University Aerospace Robotics Lab (CUARL)** on **CRAWLR — Concordia Robotic Articulated Wheel-Legged Rover**, an experimental space-rover platform for research into **push–pull locomotion ("inchworming")** in high-sinkage granular terrain and robust mobility/control for unstructured planetary environments.

My work spanned **ROS2 system integration**, **CAN-based motor control**, **Gazebo/RViz simulation**, **teleoperation**, and **perception-based position tracking** for closed-loop control and experiment data collection.

- **Full-time:** May 2025 – Aug 2025
- **Part-time extension:** Sep 2025 – Dec 2025

## Project Context: What is CRAWLR?
CRAWLR is a hybrid **wheel–leg articulated rover** designed to explore locomotion modes beyond standard rolling, especially in scenarios where a rover is immobilized or the terrain is too steep/unstable. The platform is intended for research into:

- Push–pull locomotion strategies for high sinkage granular terrain
- Robust control and system safety for a one-of-a-kind experimental rover
- Simulation + field testing workflows that reduce risk to expensive hardware

The rover is actuated via multiple joint + wheel actuators (e.g., articulated arms with shoulder/elbow/wrist + wheel) and requires careful hardware/software integration for safe and reliable operation.

## Stack & Tools
**Robotics / Simulation**
- ROS2 (Humble)
- ros2_control / ros2_controllers
- MoveIt2
- RViz / Gazebo
- Nav2 (exploration/next steps)

**Hardware I/O**
- CAN bus + EPOS4 motor drivers
- ros2_canopen + Virtual CAN

**Perception**
- ZED2 / ZED2i cameras
- ZED SDK (visual-inertial position tracking)
- ArUco markers (OpenCV-based)

**Dev / Deployment**
- Git (including VM/remote integration + troubleshooting)
- Docker containers (including running ROS2 + ZED SDK constraints on Jetson platforms)
- Linux (system setup, debugging, stability)

---

## Phase 1 (May–Aug 2025): System Ramp-Up + Control/Simulation Foundations

### 1) Codebase & Toolchain Familiarization
The first part of the internship was focused on becoming productive inside an existing ROS2 system that had grown complex over time.

What I did:
- Set up a local development environment and mirrored key rover functionality on my own machine.
- Resolved ROS2 package issues and compatibility problems (broken/misconfigured packages).
- Fixed Git/VM workflow issues to restore reliable access to remotes and consistent development.

Challenges I ran into:
- Incomplete documentation and fragmented ownership of parts of the codebase
- ROS2 package failures due to incorrect configs / missing assumptions
- VM-specific connectivity and tooling issues

### 2) Motion Planning Exploration with MoveIt2
To ramp-up on motion planning concepts needed for articulated systems, I used the **MoveIt2 PRBT-6 demo** as a learning platform.

What I implemented/learned:
- Wrote a C++ planning script to test planning modes (Cartesian planning, joint locking, constraints, motion groups)
- Integrated MoveIt2 workflows with ros2_control setups
- Built intuition around Move Group interfaces and constraint handling

Outcome:
- Established the planning/tooling baseline needed to work with (and support) the rover’s inchworm-style motion planning development.

### 3) Motor Control over CAN + Virtual CAN Pipeline
A major focus was improving the reliability and safety of motor testing by enabling a workflow where **the same ROS2 control logic** can be validated without being physically connected to the rover.

What I contributed:
- Extended a minimal CAN setup to support multiple motor interfaces.
- Transmitted direct CAN messages to validate motor rotation and verify command paths.
- Built/validated a **Virtual CAN pipeline (ros2_canopen)** for simulated motor interfaces.

Key result:
- Achieved a workflow where the rover control stack can run against Virtual CAN to produce realistic feedback, enabling faster iteration and reducing risk to hardware.

### 4) Gazebo Simulation Integration
Goal: create a simulation environment that mirrors the hardware stack closely enough to validate mobility/control strategies before going on hardware.

What I did:
- Worked through early instability from incomplete pipelines and URDF/SDF issues.
- Debugged version/compatibility mismatches (Gazebo versions vs CAN/sim packages).
- Migrated toward a more feature-complete simulation package and adapted it for Virtual CAN.

Ongoing challenges at that stage:
- Unrealistic joint behavior and model glitches
- Unstable PID tuning and controller integration issues

### 5) Teleoperation (Xbox Controller) — First End-to-End Control Pipeline
I started implementing joystick teleoperation, connecting high-level operator input to safe rover commands:

**Xbox Controller → Joy → Twist → skid-steer solver → Float64MultiArray → ros2_control → CAN → EPOS4**

What I built:
- Message conversion layer from Twist output into ROS2 controller-friendly commands
- Tested the control chain in simulation and attempted hardware validation

Constraints:
- VM device driver friction for Xbox controller
- Limited end-to-end verification due to existing simulation glitches

---

## Phase 2 (Sep–Dec 2025): Simulation Architecture Fixes + Teleop Packaging + Perception Tracking

### 1) Resolving ROS2 Control + Gazebo Plugin Architecture Conflicts
I hit a fundamental limitation: CANopen hardware drivers and Gazebo ROS2 control plugins cannot reliably run simultaneously (it’s an **either/or** system).

What I implemented:
- **Conditional plugin loading** via Xacro + launch arguments to switch between simulation and hardware without hand-editing config files.
- Reduced a major workflow pain point: switching modes became a launch-time choice instead of a manual reconfiguration process.

### 2) Controller Architecture Migration for Reliability
A key integration blocker was reliability: an older approach used many controllers (e.g., one per wheel/leg), leading to random controller initialization failures.

Solution:
- Migrated to a proven, simpler architecture: **two consolidated controllers** (unified wheel + unified leg controller).

Result:
- Near-100% startup reliability and consistent teleop behavior.

### 3) Teleoperation Package (Production-Ready + Documented)
I built a complete teleoperation package designed to support both simulation and hardware through a single launch interface.

Features:
- Single launch architecture controlled by a `sim_gazebo` argument:
  - `sim_gazebo:=true` launches Gazebo + RViz workflow
  - `sim_gazebo:=false` runs on hardware via CAN

Validation:
- Verified smooth motion in Gazebo.
- Validated control on real rover during field trials.

Hard failure + fix:
- Control would drop after ~5–10 minutes.
- Root cause: VM power management sleeping the VM while joystick still appeared active.
- Fix: adjusted power management + added a keep-awake bash function while running teleop.

Deliverable:
- A documented teleop package with setup, calibration, troubleshooting, and parameter explanations.

### 4) ZED2 Vision-Based Position Tracking for Closed-Loop Feedback
Objective: provide accurate position feedback to close the loop for rover control and support experiment data collection.

Attempt 1 — ZED position tracking on Jetson Nano:
- Identified performance/compute limits: certain ZED tracking pipelines ran far below real-time rates (sub-Hz behavior in some configurations).

Approach — practical, testable alternatives:
- Implemented **ArUco marker tracking** as a perception-based position tracking alternative.
- Compared two efficient distance estimation methods:
  - ArUco geometric calculation (camera calibration + marker geometry)
  - Depth-based measurement using ZED stereo depth focused on marker ROI

Performance outcomes (as tested):
- Achieved real-time ArUco tracking around **20–30 Hz** (with tuning; higher rates possible depending on FPS/config).
- Observed depth method can be smoother but has different performance scaling vs ArUco.

Experiment design under constraints:
- When the primary rover became unavailable (motor failure), I adapted testing using a smaller competition robot as a moving platform and later moved the camera itself to validate measurement behavior.

Integration goal:
- Package/pipe position outputs back into ROS2 message types to enable teammate integration into a **Simulink-based control stack**, enabling the transition from open-loop to closed-loop experiments.

### 5) Research Output
This internship contributed critical infrastructure (teleop + simulation reliability + perception tracking foundations) supporting CUARL’s long-term autonomy roadmap.

I’m expected to be listed on a **journal publication** (planned around Jan 2026 timeframe) due to contributing a key component enabling closed-loop feedback and experimental progress.

---

## What I Learned
- **Safety-first robotics**: for expensive one-off platforms, reliability and safe defaults matter as much as “making it move.”
- **System integration is the job**: the hardest problems were often interfaces—controllers, plugins, versions, assumptions.
- **Build test infrastructure early**: Virtual CAN and simulation workflows massively increase iteration speed and reduce hardware risk.
- **Adaptability wins**: motor failures and tooling limits forced creative experiments and practical alternatives.

## Links / References
- CRAWLR background (push–pull locomotion): https://users.encs.concordia.ca/~kskoniec/publication/creager2015push/
- MoveIt2 docs: https://moveit.picknik.ai/
- ZED SDK docs: https://www.stereolabs.com/docs/
- OpenCV ArUco: https://docs.opencv.org/4.x/d5/dae/tutorial_aruco_detection.html
