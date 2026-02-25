---
title: ""
layout: default
---


<br>

<h1 align="center">PX4-SITL Instructions (Simulation)</h1>
<br>
<br>





---

**Helpful Links:**

 https://docs.px4.io/main/en/ros2/user_guide.html

‣ 

---

**PX4** is an **open-source flight control software** used to operate drones (UAVs), VTOLs, fixed-wing aircraft, and even ground/rover robots.

It runs on flight controller hardware (like Pixhawk) or in simulation (SITL).

### **1. Install PX4**

```python
cd
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
bash ./PX4-Autopilot/Tools/setup/ubuntu.sh --no-sim-tools
cd PX4-Autopilot/
make px4_sitl
```

        Comments:
        1. --no-sim-tools basically keeps your existing simulator (e.g., Gazebo Harmonic).

        2. Error might happen because of empy package version incompatibility , uninstall it and reinstall:

```python
pip3 uninstall empy
pip3 install --user empy==3.3.4
```

### **2. Setup Micro XRCE-DDS Agent & Client**

**Micro XRCE-DDS** is a lightweight communication system used to connect **resource-constrained devices** (like PX4 flight controllers) to the **ROS 2 DDS network**.

Official document says to use `git clone -b v2.4.2 [https://github.com/eProsima/Micro-XRCE-DDS-Agent.git](https://github.com/eProsima/Micro-XRCE-DDS-Agent.git)` , but this is not stable,  use:

```python
git clone [https://github.com/eProsima/Micro-XRCE-DDS-Agent.git](https://github.com/eProsima/Micro-XRCE-DDS-Agent.git)
cd Micro-XRCE-DDS-Agent
mkdir build
cd build
cmake ..
make
sudo make install
sudo ldconfig /usr/local/lib/
```

### **3. Install QGC:**

**QGroundControl (QGC)** is a **ground control station (GCS)** application used to monitor and control drones running **PX4** (and also ArduPilot).

follow https://docs.qgroundcontrol.com/en/getting_started/download_and_install.html

---

### **Running a PX4 SITL (Software-in-the-loop):**

It runs PX4 on a simulated drone.

 i)  Launch PX4 Drone

```python
cd ~/PX4-Autopilot
make px4_sitl gz_x500
```

ii)  Run the XRCEAgent:

```python
MicroXRCEAgent udp4 -p 8888
```

Extras: 

How to change world in PX4 ?

There are worlds, models folder in the PX4 package. We can load from there:

```python
make px4_sitl_default gz_<vehicle_model>__<world_name>
```

example:

```python
make px4_sitl gz_x500_lawn
```