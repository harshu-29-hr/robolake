# 02 — Exploring a raw MCAP robot log

**Dataset:** `RobotisSW/latched_test4` — 8.5 s ROS 2 (Jazzy) recording from a ROBOTIS two-armed mobile robot, teleoperated by a human with leader controllers.

## Overview
- 10,714 messages across 16 topics in 8.47 s
- 85.83 MB, chunks **not compressed** → ~10 MB/s → ~36 GB per robot-hour
- Timestamps are integer nanoseconds since the Unix epoch (wall-clock time)

## Topic map
| Role | Topics | Rate |
|---|---|---|
| Cameras | ZED left (head); camera_left & camera_right (wrists), compressed images | head ~15 Hz, wrists ~30 Hz |
| Calibration | 5 × `camera_info` | ~15 Hz |
| State | `/joint_states` (25 joints: 2 arms + mobile base wheels) | ~101 Hz real |
| Actions | `/leader/joint_trajectory_command_broadcaster_left/right`, `/leader/joystick_controller_left/right` | ~202 Hz recorded |
| Geometry | `/tf`, `/tf_static` | ~202 Hz / static |
| Static | `/robot_description` (URDF) | once |

## Rates
- Joints (~101 Hz) and commands (~202 Hz) run 3–13× faster than the cameras, so aligning them means choosing or interpolating readings per camera frame.
- Cameras on the same robot run at different rates (wrists 30.4 Hz, head 15.2 Hz), so cameras must be aligned to each other too, not just to the joints.
- Each wrist camera's calibration topic runs at 15 Hz while its images run at 30 Hz.
- My first rate metric (median gap) reported ~950 Hz for `/joint_states`, but it was wrong for bursty data. Count ÷ duration is the honest rate.
- Cameras are not perfectly steady: the head camera's max gap was ~101 ms vs a 66.7 ms period (a late or dropped frame).

## Two clocks
- Each message has `header.stamp` (when measured) and `log_time` (when recorded).
- `log_time − header.stamp`: median ~1.9 ms, max 99 ms. The 99 ms was the very first message, at startup.
- `log_time` arrives in bursts (gaps of 0–15.8 ms); `header.stamp` is regular (10 ms ± 0.24 ms), with no out-of-order stamps.
- **Decision:** align on `header.stamp`; flag messages with large delays.

## Duplicates
- Every `/joint_states` timestamp appears twice (856 pairs + 1 single), so the real joint rate is ~101 Hz, not the 202 Hz recorded.
- I first guessed the pairs were left/right arm halves; the data disproved it: all messages contain all 25 joints.
- Pairs were **identical copies**: 856 duplicates, 0 conflicting values. Safe to de-duplicate by (topic, timestamp).

## Units
- ROS joint positions are in radians (SI convention). Other formats use degrees or normalized ranges, so units must be recorded per joint.

## MCAP (raw) vs LeRobot (clean)
| | MCAP log | LeRobot dataset |
|---|---|---|
| Timing | each topic at its own rate, bursty | one shared 30 FPS timeline |
| Clock | int nanoseconds, two clocks | float32 seconds from episode start |
| Cameras | different rates per camera (15 / 30 Hz) | all cameras on the same timeline |
| Duplicates | yes | none |
| Structure | messages per topic | one row per frame |
| Compression | none in this file | AV1 video + Parquet |

## What robolake must handle
- A user-provided **topic map** (cameras / state / actions), since every robot names topics differently.
- **Align on sensor time** (`header.stamp`), not recorder time.
- **Align cameras to each other**, not just joints to cameras.
- **De-duplicate** messages by (topic, timestamp).
- **Resample** fast streams (100–200 Hz) onto the camera or training timeline.
- **Flag** dropped camera frames, startup delays and large clock gaps.
- **Record units** per joint and **compress** raw data on storage.