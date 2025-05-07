# `kinect_ros2`

## Interface

### Overview
Basic Kinect-v1 (for the Xbox 360) node, with IPC support, based on [libfreenect](https://github.com/OpenKinect/libfreenect).  
For now, it only supports a single Kinect device. If multiple devices are connected, the first one listed by `freenect_num_devices` will be selected.

### Published topics
* `image_raw` - RGB image (`rgb8`) ([sensor_msgs/Image](http://docs.ros.org/api/sensor_msgs/html/msg/Image.html))
* `camera_info` - RGB camera_info ([sensor_msgs/CameraInfo](http://docs.ros.org/api/sensor_msgs/html/msg/CameraInfo.html))
* `depth/image_raw` - Depth image (`mono16`) ([sensor_msgs/Image](http://docs.ros.org/api/sensor_msgs/html/msg/Image.html))
* `depth/camera_info` - Depth camera_info ([sensor_msgs/CameraInfo](http://docs.ros.org/api/sensor_msgs/html/msg/CameraInfo.html))

---

## Installation

### 1. Install libfreenect
This package was tested using a manual build of [libfreenect](https://github.com/OpenKinect/libfreenect).  
Some Kinect firmware versions may require enabling `BUILD_OPENNI2_DRIVER` or other flags.

```bash
sudo apt install libfreenect-dev libusb-1.0-0-dev
```
#### Make sure Cython is installed (required by cv_bridge and other packages)
```bash
pip install cython
```

### 2. Clone this repository
```bash
cd ~/ros2_ws/src
git clone https://github.com/YOUR_USERNAME/kinect_ros2.git](https://github.com/Foodpal-Tec/kinect_ros2.git
```

### 3. Install missing ROS 2 dependencies
```bash
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
```

### 4. Build your workspace
```bash
colcon build
. install/setup.bash
```

---

## 🛠 Modifications added for RTAB-Map Compatibility

To use this Kinect node with `rtabmap_ros`, several modifications were made:

- ✅ **Synchronized timestamps**: The RGB and depth images now share a common timestamp, which is critical for the `approximate_time` sync policies in `rtabmap_sync` and `rgbd_odometry`.
- ✅ **Fixed header stamps**: Previously, images were published with `stamp: 0`, causing synchronization failures. The `now()` time is now applied properly to both image and `camera_info` headers.
- ✅ **Frame IDs** are set to `"kinect_rgb"` and `"kinect_depth"` respectively, and are consistent across image and camera info messages.
- ✅ **Integration with `depth_image_proc`** via `PointCloudXyzNode` is optionally included in the main node for visualization or point cloud generation.

---

## Running the Node

You can launch the Kinect node and `depth_image_proc` together using a minimal main:

```bash
ros2 run kinect_ros2 kinect_ros2_main
```

Alternatively, load it as a component via:

```bash
ros2 component load /ComponentManager kinect_ros2 kinect_ros2::KinectRosComponent
```

---

## RTAB-Map Integration (Basic)

Install RTAB-Map from apt:

```bash
sudo apt install ros-<your-distro>-rtabmap-ros
```

Then launch RTAB-Map with topics remapped to match the Kinect node:

```bash
ros2 launch rtabmap_launch rtabmap.launch.py \
  rgb_topic:=/image_raw \
  depth_topic:=/depth/image_raw \
  camera_info_topic:=/camera_info \
  approx_sync:=true \
  qos_scan:=2 qos_camera_info:=2 qos_image:=2
```

Make sure your node is running before launching RTAB-Map.

---

## Devices tested
* Kinect Model 1473 (Xbox 360 original)

---

## License

MIT
