---
layout: default
title: ""
---


<br>
<h1 align="center">Clearpath Husky Setup Notes</h1>
<br>

<div style="display: flex; flex-wrap: wrap; justify-content: center; gap: 20px;">
  <div style="text-align: center;">
    <img src="husky_in_snow_gif.gif" style="width: 100%; min-width: 250px;">
    <p><em></em></p>
  </div>

  
</div>


## Reference Manual
Clearpath Husky Manual provided by Clearpath:  
https://uofi.box.com/s/fs2cpd2apfchoqz74t4s30reo3n4084b

---

## Robot Hardware Overview

The Clearpath Husky robot comes with **two onboard computers**:

1. **Primary Computer (CPU)**
2. **Secondary Computer (NVIDIA Jetson GPU)**

Both computers are **pre-configured with ROS2 Humble** and common robotics packages.  
The operating system installed on both machines is **Ubuntu 22.04**.

---

## 1. Setting Up the Remote Computer

The first step was setting up a **remote computer**, which is useful for running simulations and offboard applications.

The steps followed are similar to those described in:

- **Manual:** Page 8, Section 12  
- **Clearpath Documentation:**  
  https://docs.clearpathrobotics.com/docs/ros/installation/offboard_pc

Note:  
These instructions were originally written for **ROS2 Humble**. Clearpath has now updated their documentation for **ROS2 Jazzy**, but the steps remain mostly the same.

---

## 2. Running Simulation

The Husky robot can be simulated in a **Gazebo environment** to perform experiments and testing.

Simulation instructions are provided here:  
https://docs.clearpathrobotics.com/docs/ros/tutorials/simulator/overview

You can test several functionalities in simulation:

### SLAM and Map Building
https://docs.clearpathrobotics.com/docs/ros/tutorials/navigation_demos/slam

### Localization
<https://docs.clearpathrobotics.com/docs/ros/tutorials/navigation_demos/localization>

### Navigation
<https://docs.clearpathrobotics.com/docs/ros/tutorials/navigation_demos/actions>

More tutorials and examples are available on the Clearpath documentation website.

---

## 3. Communication Setup

The Husky robot supports multiple networking configurations.

**Option 1** — Robot as Access Point
The robot can act as a **WiFi access point**, creating its own local wireless network.  
Your remote computer can connect directly to this network.

**Option 2** — Shared Network
Both the robot and the remote computer can connect to the **same external WiFi network**.

Detailed instructions are available in:

- **Manual:** Section 14–15 (Pages 10–11)
- **Documentation:**  
[https://docs.clearpathrobotics.com/docs/ros/networking/overview](https://docs.clearpathrobotics.com/docs/ros/networking/overview)


### SSH Access 
Once the remote computer is connected to the robot network, you can access the onboard computer using **SSH**. Example:
```bash
ssh administrator@husky-ip-address
```

This allows you to run applications directly on the robot.

**Important Tip:** \
SSH sessions can sometimes become unstable or freeze during long runs.
To avoid losing running processes, it is recommended to use tmux.

Example :
```bash 
tmux new -s nav2
ros2 launch ...
```
This ensures that programs continue running in the background even if the SSH connection drops.

---


<br>
<h1 align="center">My Work and Applications with Husky</h1>
<br>



### Work 1: Localization with SLAM and navigation with Nav2

As th information provided in the Tutorials in the Clearpath website, we use uilize those packages for running the SLAM and Nav2. You can run it on both remote computer or Husky onboard computer (primary computer is preffered)

The SLAM and Nav2 packages work fine until we faced the issue of `queue is full`. Below is my detailed take on this. 

### Issue 1: Data Drop, Queue Full, and Transform Data Too Old

**Issue Summary:** \
While running SLAM and navigation, the following errors appeared:

- **Error:** `Message Filter dropping message ... queue is full`
- **Error:** `Transform data too old when converting from odom to map`

These errors typically occur when sensor messages are arriving faster than they are processed, or when the transform data between frames becomes outdated.


**Solutions I Found:**

**Option 1 — Run everything on the primary computer**

Running SLAM and navigation directly on the **primary CPU computer of the Husky** reduced communication delays and prevented message drops.


**Option 2 — Modify packages built from source**

I installed **SLAM Toolbox** and **Nav2** from source (GitHub) and increased queue sizes in specific files.

Changes made:

- **SLAM Toolbox**

  File: `slam_toolbox/common.cpp`  
  Increased the message queue size: 10 from 1

-  **Nav2**

   File: `nav2_costmap_2d/obstacle_layer.cpp`
   The queue size increased from 50 to 5000 and transform_tolerance is 5 sec
   



After fixing this issue, SLAM and Nav2 packages work very fine in inddor. You can use slam for localization and Map building. But issue arises in outdoor, due to less features the localization is errorneous. I took alternate approach of GPS based localization and the Nav2 for navigation. My detailed take is below.


</br>
</br>

### Work 2: GPS-Based Localization

I used the ROS2 `robot_localization` package ([link](https://docs.ros.org/en/noetic/api/robot_localization/html/index.html)) to localize the robot in an outdoor environment. Briefly, this package uses an Extended Kalman Filter (EKF) to fuse local odometry data (wheel odometry and IMU) with GPS measurements for outdoor localization. A similar concept is used indoors when performing SLAM, where multiple sensor sources are fused to estimate the robot’s pose.

By default, the robot starts with an EKF-based localization system, but GPS is not integrated into this default configuration.

- First, I stopped the Clearpath default EKF localization.

- Second, I installed the `robot_localization` package from source and modified the configuration. I implemented a **dual EKF setup** to improve outdoor localization performance.

Make sure that both the default EKF and the dual EKF are **not running simultaneously**. As mentioned earlier, the default EKF must be stopped before starting the dual EKF.

- Third, once the dual EKF is running, the **TF tree becomes complete**, forming the standard transform chain: `map → odom → base_link`


- Finally, with this localization pipeline running correctly, Nav2-based navigation can be executed.

</br>

**A good guide to for this setup:** [https://docs.nav2.org/tutorials/docs/navigation2_with_gps.html](https://docs.nav2.org/tutorials/docs/navigation2_with_gps.html)