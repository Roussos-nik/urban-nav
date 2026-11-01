# Progress log

Simulated urban sidewalk delivery robot: ROS2 Jazzy, Gazebo Harmonic, Nav2, robot_localization, and a custom LTV-MPC controller benchmarked against Regulated Pure Pursuit and MPPI.

## Status

| Phase | Status |
| --- | --- |
| 0 Environment and repo | Done (2026-11-01) |
| 1 ROS2 concepts sprint | Next |
| 2 Robot model and urban world | Not started |
| 3 Localisation with GPS dropouts | Not started |
| 4 Nav2 baseline | Not started |
| 5 MPC controller plugin | Not started |
| 6 Benchmarking | Not started |
| 7 Engineering polish | Not started |
| 8 Presentation | Not started |

## Environment

- Laptop: HP Victus 16-d1xxx, NVIDIA GeForce RTX 3060 Laptop GPU (6 GB), NVIDIA driver 595.99.02
- OS: Ubuntu 24.04.5 LTS, dual-boot with Windows 11 (120 GB ext4 partition)
- ROS2 Jazzy (deb packages), Gazebo Harmonic via the ros_gz vendor packages
- Extra packages: navigation2, nav2_bringup, nav2_minimal_tb*, robot_localization, ros_gz, gz_ros2_control, ros2_controllers, slam_toolbox, xacro
- Workspace: `~/urban_nav_ws`, with this repo at `~/urban_nav_ws/src/urban-nav`

## Phase 0: Environment and repo (done 2026-11-01)

### Done

- Installed Ubuntu 24.04 natively (dual-boot) and the NVIDIA driver via `ubuntu-drivers`
- Installed ROS2 Jazzy; the talker/listener demo works
- Ran the Nav2 TurtleBot3 demo: the robot localised with AMCL and reached goals set in RViz
- Created the workspace and GitHub repo (SSH authentication)
- Created eight package skeletons (seven ament_cmake, one ament_python); `colcon build` succeeds
- Added GitHub Actions CI: builds and tests the workspace in the `ros:jazzy-ros-base` container on every push; green
- Added an Apache-2.0 LICENSE, `.gitignore` and a ROS2-style `.clang-format`

### Key decisions

- **Ubuntu 24.04 + Jazzy + Harmonic rather than 26.04 + Lyrical.** Jazzy has two years of bug fixes, binaries for every package this project needs, and the most tutorials. Lyrical's Nav2 binaries only arrived in August 2026, and Lyrical requires C++20.
- **Native install, not WSL2 or a VM**, so Gazebo gets full GPU rendering.
- **120 GB for Ubuntu.** Enough as long as benchmark bags record only the topics the analysis needs.
- **`mpc_core` uses ament_cmake only as build glue.** It contains no ROS headers and depends on no ROS packages, so it can be unit-tested without ROS and reused on hardware.
- **Dependencies go into `package.xml` only when code needs them.** CI's `rosdep install` relies on `package.xml` being accurate.
- **CI runs inside the official `ros:jazzy-ros-base` image**, so it uses the same ROS distro as my laptop, and `colcon test-result` makes the job fail on any failing test.
- **clang-format for C++ formatting**, using the ROS2-style config from `ament_clang_format`, because it integrates with editors (format on save).

### Problems and fixes

1. **`FileNotFoundError: 'gz'` when launching the demo.** The terminal had been opened before ros_gz was installed, so its PATH didn't include the Gazebo tools (`/opt/ros/jazzy/opt/gz_tools_vendor/bin`). Fix: open a new terminal. Lesson: open a new terminal (or re-source) after installing ROS packages.
2. **Nav2 stuck at "Navigation: inactive"; goals rejected.** The planner server's global costmap timed out waiting for the `base_link -> map` transform. AMCL doesn't publish `map -> odom` until it receives an initial pose, so `lifecycle_manager_navigation` aborted bringup. Fix: set the 2D Pose Estimate straight after launch. Lesson: lifecycle activation depends on TF being available. On my own robot, set the initial pose in config or derive it from GPS.
3. **RViz's "Startup" button made it worse.** It re-runs bringup from the configure transition, which fails for nodes that are already active ("Transition is not registered"). Fix: stop everything and relaunch cleanly.
4. **Pasting a `sudo` command together with other commands.** The lines after it were consumed as password attempts. Lesson: run `sudo` commands on their own.

### Open issues

- **Formatter conflict to resolve with the first C++ file.** The ament_cmake package template's tests also run `ament_uncrustify`, which disagrees with clang-format. When a package gets its first C++ source, add `set(ament_cmake_uncrustify_FOUND TRUE)` to its test block so CI checks formatting one way only.
- RViz's RobotModel display shows "Status: Error" in the TurtleBot demo (cosmetic; not investigated)
- The laptop has hybrid Intel/NVIDIA graphics; if Gazebo is slow, check `prime-select query`

### Next

- Phase 1: ROS2 concepts sprint (five C++ exercises in a scratch package)
