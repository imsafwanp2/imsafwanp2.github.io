---
title: ""
layout: default
---

<h1 align="center">Tello drone AprilTag tracking</h1>
<br>
<br>

**Objective**

This lab demo demonstrates **vision-based closed-loop control** using a **DJI Tello drone**.

The drone detects an **AprilTag (tag36h11 family)** and autonomously follows it:

- Rotates (yaw control) to center the tag
- Moves forward to maintain a desired distance
- Stops when it reaches the target distance

This experiment showcases **real-time perception → control loop integration**.

---

**Hardware & Software Setup**

Hardware

- **DJI Tello** drone
- Printed AprilTag (tag36h11)
- Laptop for programming (WiFi communication)

Software

- Python 3.10
- `djitellopy` (Tello SDK wrapper)
- OpenCV (`cv2`)
- `pupil_apriltags` detector
- NumPy

---

**Demo instructions**

Code: https://github.com/imsafwan/April_tag_follow

- Disable the firewall on the host computer.
- Run the `apriltag_track.py` script.
- The script will start communicating with the drone, make it take off, start the camera, and begin image processing for tracking.
- Keep `emergency.py` ready in case you need to immediately stop the drone.
- Once you're done, turn the firewall back on for the host computer.

---

**Code Logical flow:**

- Imports required libraries: `djitellopy` for controlling the **DJI Tello**, `cv2` for image processing, `numpy` for numerical operations, and `pupil_apriltags` for AprilTag detection.
- Defines frame size, PID gains for yaw and forward motion, and initializes previous error terms for closed-loop control.
- Sets camera intrinsic matrix and distortion coefficients for pose estimation using `solvePnP()`.
- Initializes the Tello drone, connects via WiFi, resets velocities, and starts video streaming.
- Continuously captures frames from the drone camera and resizes them.
- Detects AprilTags in the grayscale image and draws bounding boxes around detected tags.
- Estimates the tag pose using known tag size and camera calibration to obtain translation (distance).
- Computes horizontal (yaw) error from image center and distance error from desired stopping distance (0.5 m).
- Applies PID control to calculate yaw velocity and forward velocity.
- Sends RC control commands to the drone for real-time closed-loop tracking.
- Stops and lands safely when the user presses the ‘q’ key.

---

**Codes & Resources**

Github: [https://github.com/imsafwan/April_tag_follow](https://github.com/imsafwan/April_tag_follow)

Video [https://www.youtube.com/watch?v=fKat43Ljl54](https://www.youtube.com/watch?v=fKat43Ljl54)

Apriltag generation: [https://chaitanyantr.github.io/apriltag.html](https://chaitanyantr.github.io/apriltag.html)

---