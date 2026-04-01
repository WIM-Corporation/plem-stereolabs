# plem-stereolabs

> Stereolabs ZED Camera ROS 2 Description

[한국어](../README.md)

## Overview

`stereolabs_description` is a ROS 2 package providing URDF/xacro models and 3D meshes for Stereolabs ZED cameras.

As a **Tier 2 (robot-vendor-independent)** package, it has no dependencies on any specific robot driver or platform. It can be referenced by any robot vendor such as Neuromeka, Doosan, etc.

## Included Models

| Model | Macro Name | Dimensions | Weight | Interface |
|-------|-----------|------------|--------|-----------|
| **ZED X Mini** | `stereolabs_zedxm` | 94 x 32 x 37 mm | 150 g | GMSL2 (ZED Link Duo) |

The ZED X Mini is a stereo depth camera with a 50 mm baseline, working range of 100 mm -- 8 m (2.2 mm lens), and IP67 rating.

## Installation

```bash
# Clone into workspace
cd ~/ros2_ws/src
git clone <repo-url> stereolabs_description

# Build
cd ~/ros2_ws
colcon build --packages-select stereolabs_description
source install/setup.bash
```

## Usage

### Standalone xacro execution

```bash
# Verify URDF generation (mount_config_file required)
xacro urdf/zedxm.xacro prefix:=cam_ parent_link:=tool0 mount_config_file:=/path/to/camera_mount.yaml
```

### Calling the macro from another xacro

```xml
<xacro:include filename="$(find stereolabs_description)/urdf/zedxm.xacro"/>
<xacro:stereolabs_zedxm prefix="cam_"
                         parent_link="tool0"
                         mount_config_file="$(find my_config)/config/camera_mount.yaml"/>
```

### Macro Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `prefix` | string | (required) | Namespace prefix applied to all link/joint names |
| `parent_link` | string | (required) | Parent link to attach the camera to |
| `mount_config_file` | string | (required) | Path to Hand-Eye calibration YAML file |

### mount_config_file Format

```yaml
camera_mount:
  x: 0.0
  y: 0.0
  z: 0.05
  roll: 0.0
  pitch: 0.0
  yaw: 0.0
```

### Key Frames Generated

- `${prefix}zedx_link` -- Camera housing center (mount reference point)
- `${prefix}zedx_left_optical_frame` -- Left eye optical center (Hand-Eye calibration reference)
- `${prefix}zedx_depth_optical_frame` -- Depth optical frame (ROS convention: z-forward, x-right, y-down)

## Robot Integration

This package provides the camera model independently. Integration with a specific robot is handled by **Tier 3 integrations** packages (e.g., `neuromeka_integrations`).

Refer to the respective integrations package documentation for integration examples.

## Adding a New Model

1. Create `urdf/{model}.xacro` -- the macro name must follow the `stereolabs_{model}` convention.
2. Place STL/DAE meshes under `meshes/{model}/`.
3. Add an xacro parsing test to `test/test_xacro.py`.
4. Bump the `package.xml` version (SemVer patch).

## License

Apache-2.0
