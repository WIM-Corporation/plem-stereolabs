---
description: "ZED camera usage guide — topic verification, RViz visualization, SVO recording/playback, Object Detection demo"
paths:
  - "**/*zed*"
  - "**/*camera*"
  - "**/*depth*"
  - "**/*stereo*"
---

# ZED Camera Usage Guide

## Verify Camera Topics

After launching the ZED node, confirm topics are publishing:

```bash
# List all ZED topics
ros2 topic list | grep zed

# Check image publish rate
ros2 topic hz /zed/zed_node/rgb/color/rect/image

# Check depth info
ros2 topic echo /zed/zed_node/depth/depth_info --once
```

Key topics:

| Topic | Type | Description |
|-------|------|-------------|
| `rgb/color/rect/image` | `sensor_msgs/Image` | Rectified color image |
| `depth/depth_registered` | `sensor_msgs/Image` | Registered depth map |
| `point_cloud/cloud_registered` | `sensor_msgs/PointCloud2` | 3D point cloud |
| `imu/data` | `sensor_msgs/Imu` | IMU measurements |
| `odom` | `nav_msgs/Odometry` | Visual-inertial odometry |

All topics are under `/{camera_name}/{node_name}/` namespace (default: `/zed/zed_node/`).

## RViz2 Visualization

```bash
# Use the built-in RViz config for your camera model
rviz2 -d $(ros2 pkg prefix zed_wrapper)/share/zed_wrapper/config/rviz2/<camera_model>.rviz
```

For advanced visualization (Point Cloud coloring, Object Detection overlays):
- [zed-ros2-examples/zed_display_rviz2](https://github.com/stereolabs/zed-ros2-examples/tree/master/zed_display_rviz2)

## SVO Recording / Playback

SVO is ZED's raw sensor data format. Useful for offline testing without a physical camera.

```bash
# Start recording
ros2 service call /zed/zed_node/start_svo_rec zed_msgs/srv/StartSvoRec \
    "{bitrate: 0, compression_mode: 1, target_framerate: 0, svo_filename: '/tmp/record.svo2'}"

# Stop recording
ros2 service call /zed/zed_node/stop_svo_rec std_srvs/srv/Trigger

# Playback (no camera needed)
ros2 launch zed_wrapper zed_camera.launch.py camera_model:=zedxm \
    svo_path:=/tmp/record.svo2
```

SVO playback publishes the same topics as a live camera — downstream nodes work without changes.

## Object Detection Demo

```bash
# Launch with Object Detection enabled
ros2 launch zed_wrapper zed_camera.launch.py camera_model:=zedxm \
    object_detection.od_enabled:=true

# Runtime toggle
ros2 service call /zed/zed_node/enable_obj_det std_srvs/srv/SetBool "{data: true}"

# Body Tracking
ros2 service call /zed/zed_node/enable_body_trk std_srvs/srv/SetBool "{data: true}"
```

Detection results publish to `/zed/zed_node/obj_det/objects` (`zed_msgs/msg/ObjectsStamped`).

## Launch Arguments Quick Reference

| Argument | Default | Description |
|----------|---------|-------------|
| `camera_model` | *required* | Camera model (e.g. `zedxm`, `zed2i`) |
| `camera_name` | `"zed"` | Topic namespace prefix |
| `serial_number` | `0` | Specific camera (for multi-camera) |
| `svo_path` | `""` | SVO file path (empty = live camera) |
| `publish_tf` | `true` | Publish TF transforms |
| `publish_map_tf` | `true` | Publish map→odom TF |
| `ros_params_override_path` | `""` | Custom parameter YAML path |
