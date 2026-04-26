# Drone MARTIN

Thesis project repository for autonomous drone navigation using ROS, Parrot Sphinx, and Bebop Autonomy.

---

## Installation

```bash
sh compiling_scripts/newInstall.sh
sh compiling_scripts/ParrotSphinxInstall.sh
```

**References:**
- [Parrot Sphinx Release Notes](https://developer.parrot.com/docs/sphinx/releasenotes.html)
- [General Setup Guide](https://forum.developer.parrot.com/t/using-bebop-autonomy-with-sphinx-on-same-machine/6726/5)
- [Bebop Commands](https://bebop-autonomy.readthedocs.io/en/latest/piloting.html)

---

## `.bashrc` Setup

Add the following to your `~/.bashrc`:

```bash
# ROS PROJECT
source /opt/ros/kinetic/setup.bash
source ~/Code/Drone_MARTIN/mapping/VINS_mono/devel/setup.bash

# Navigation
source ~/Code/Drone_MARTIN/navigation/devel/setup.sh

# Bebop Autonomy
source ~/Code/Drone_MARTIN/bebop/devel/setup.bash

# CUDA
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

---

## One-Time Setup

1. Install Parrot Sphinx from repos or manually.
2. Install ROS.
3. Download and install `bebop_autonomy` in a ROS workspace.
4. Run `iwconfig` and copy the first network interface name (commonly `eth0` or `wlp2s0`).
5. Open `/opt/parrot-sphinx/usr/share/sphinx/drones/bebop2.drone` and set `stolen_interface`:
   ```xml
   <stolen_interface>wlp2s0:eth0:192.168.0.5/24</stolen_interface>
   <!-- or -->
   <stolen_interface>eth0:eth0:192.168.0.5/24</stolen_interface>
   ```
   Keep both lines and comment out the one not in use — the interface name can change after the first Sphinx run.
6. In `bebop_node.launch`, update the IP address. Keep the original commented for WiFi simulator use.

---

## Running the Simulator

Open five terminals and run each in order:

**Terminal 1 — ROS Core**
```bash
roscore
```

**Terminal 2 — Firmware**
```bash
sudo systemctl start firmwared.service
sudo firmwared
```

**Terminal 3 — Sphinx Simulator**
```bash
sphinx --datalog \
  /opt/parrot-sphinx/usr/share/sphinx/worlds/outdoor_1.world \
  /opt/parrot-sphinx/usr/share/sphinx/drones/bebop2_local.drone
```

**Terminal 4 — Bebop ROS Node**
```bash
roslaunch bebop_driver bebop_node.launch
```

**Terminal 5 — Image Viewer**
```bash
rosrun image_view image_view image:=/bebop/image_raw
```

**Test takeoff and landing:**
```bash
rostopic pub --once bebop/takeoff std_msgs/Empty
rostopic pub --once bebop/land std_msgs/Empty
```

---

## Resetting the Simulated Battery

1. Close Bebop Autonomy, firmwared, and Sphinx.
2. Run:
   ```bash
   sudo systemctl restart firmwared.service && sudo firmwared
   ```
3. Restart Sphinx (Terminal 3 command above).
4. Restart Bebop Autonomy (Terminal 4) — retry until it connects.

---

## Raspberry Pi

**SSH:**
```bash
ssh pi@192.168.42.65
# Password: Drone_MARTIN
```

**IMU-Camera Calibration:**
```bash
rosrun kalibr kalibr_calibrate_imu_camera \
  --target dataset/bebop2/april_6x6_A1.yaml \
  --cam bebop2_camera_calib.yaml \
  --imu PiSense_imu_param.yaml \
  --bag dataset/bebop2/calib_A1_1.bag \
  --timeoffset-padding 1 \
  --verbose
```

---

## Camera Calibration

**Extract frames from ROS bag:**
```bash
rosrun image_view extract_images \
  _sec_per_frame:=0.01 \
  _filename_format:="frame%04d.png" \
  image:=/bebop/image_mono
```

**Run calibration:**
```bash
rosrun camera_model Calibration \
  -w 6 -h 7 -s 40.05 \
  --camera-model pinhole \
  --camera-name bebop-front \
  --opencv --view-results -v \
  -p frame -i img/
```
