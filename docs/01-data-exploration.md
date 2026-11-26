# 01 — Exploring a real LeRobot dataset

**Dataset:** `lerobot/svla_so101_pickplace` — real SO-101 robot arm doing pick-and-place.

## Overview
- Format: LeRobot v3.0
- 30 FPS, 50 episodes, 11,939 frames total
- 2 cameras: `up` and `side`, both 640×480, AV1-encoded MP4

## Joints
`observation.state` (measured) and `action` (commanded) each have 6 values:
`shoulder_pan.pos`, `shoulder_lift.pos`, `elbow_flex.pos`, `wrist_flex.pos`, `wrist_roll.pos`, `gripper.pos`

## Episode lengths
- Shortest: 183 frames (~6.07 s)
- Longest: 306 frames (episode 41, ~10.17 s)
- Average: 238.78 frames (~7.93 s)
- Duration is `(frames - 1) / fps`, not `frames / fps`, because the first frame is at t = 0.

## Timing quality
- All 11,889 frame-to-frame gaps were exactly 1/30 s: zero irregular frames.
- 50 null gaps = the first frame of each episode (nothing to compare against).
- Timestamps are stored as float32, which adds small rounding noise (10.1666669… instead of 10.1666667). Fine for 10-second clips, but on long recordings it would drift.

## Video ↔ data link
- v3.0 packs all 50 episodes into one MP4 per camera; episode boundaries live in `meta/episodes/`.
- Each camera video has 11,939 frames and 397.97 s duration, matching the 11,939 Parquet rows exactly.
- Video frames are matched to data rows by timestamp, so any mismatch would pair joint data with the wrong image.

## Command vs. response (episode 0, gripper)
- The robot sits idle for the first ~90 frames before moving.
- The commanded gripper value (1.62) never exactly equals the measured value (1.41).
- The gripper command first changes at frame 92; the real gripper responds at frame 99. That's a 7-frame (~0.23 s) reaction lag.

## What robolake must handle
- Validate that video frame counts match data row counts, per camera.
- Check that timestamps are evenly spaced and flag gaps.
- Store time as integer nanoseconds, not float32, to avoid precision drift.
- Detect idle frames at the start and end of episodes.
- Measure command-to-response lag so action and state stay correctly aligned.