# plem-stereolabs

> Stereolabs ZED Camera ROS 2 Description

[한국어](../README.md)

## Overview

`stereolabs_description` is a ROS 2 package providing URDF/xacro models and 3D meshes for Stereolabs ZED cameras.

As a **Description Layer (robot-vendor-independent)** package, it has no dependencies on any specific robot driver or platform. It can be referenced by any robot vendor such as Neuromeka, Doosan, etc.

This package is a vendored copy of `zed_macro.urdf.xacro` from the official [zed-ros2-wrapper](https://github.com/stereolabs/zed-ros2-wrapper). It provides URDF models independently without requiring the ZED SDK. See [VENDOR_SOURCE.md](../VENDOR_SOURCE.md) for source tracking and sync history.

## Supported Camera Models

Full list of models supported by the official macro:

| Model | `model` value | Type | Connection |
|-------|--------------|------|------------|
| ZED | `zed` | Stereo | USB 3.0 |
| ZED Mini | `zedm` | Stereo | USB 3.0 |
| ZED 2 | `zed2` | Stereo | USB 3.0 |
| ZED 2i | `zed2i` | Stereo | USB 3.0 |
| ZED X | `zedx` | Stereo | GMSL2 |
| ZED X HDR | `zedxhdr` | Stereo | GMSL2 |
| ZED X Mini | `zedxm` | Stereo | GMSL2 |
| ZED X HDR Mini | `zedxhdrmini` | Stereo | GMSL2 |
| ZED X HDR Max | `zedxhdrmax` | Stereo | GMSL2 |
| Virtual | `virtual` | Stereo | - |
| ZED X One GS | `zedxonegs` | Mono | GMSL2 |
| ZED X One 4K | `zedxone4k` | Mono | GMSL2 |
| ZED X One HDR | `zedxonehdr` | Mono | GMSL2 |

Mono models (ZED X One series) generate a single camera frame instead of left/right camera frames.

## Installation

```bash
# Clone into workspace
cd ~/ros2_ws/src
git clone https://github.com/WIM-Corporation/plem-stereolabs.git

# Build
cd ~/ros2_ws
colcon build --packages-select stereolabs_description
source install/setup.bash
```

## Usage

### Standalone xacro execution

```bash
# Verify URDF generation
xacro $(ros2 pkg prefix stereolabs_description)/share/stereolabs_description/urdf/zed_macro.urdf.xacro \
    name:=cam model:=zedxm
```

### Calling the macro from another xacro

```xml
<xacro:include filename="$(find stereolabs_description)/urdf/zed_macro.urdf.xacro"/>
<xacro:zed_camera name="cam" model="zedxm">
    <origin xyz="0 0 0" rpy="0 0 0"/>  <!-- gnss_origin (required block, use dummy value if GNSS not used) -->
</xacro:zed_camera>
```

### Macro Parameters

Macro signature: `zed_camera(name, model, enable_gnss, custom_baseline, *gnss_origin)`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | string | `zed` | Namespace prefix applied to all link/joint names |
| `model` | string | `zed` | Camera model (see supported models table above) |
| `enable_gnss` | bool | `false` | Whether to enable the GNSS link |
| `custom_baseline` | float | `0` | Custom baseline for the `virtual` model (m) |
| `*gnss_origin` | block | (required) | GNSS antenna offset `<origin>` block |

### TF Frames Generated

**Stereo models** (zed, zedm, zed2, zed2i, zedx, zedxhdr, zedxm, zedxhdrmini, zedxhdrmax, virtual):

| Frame | Description |
|-------|-------------|
| `${name}_camera_link` | Camera mount reference point (screw hole) |
| `${name}_camera_center` | Camera housing center |
| `${name}_left_camera_frame` | Left camera frame |
| `${name}_left_camera_frame_optical` | Left optical frame (ROS convention: z-forward, x-right, y-down) |
| `${name}_right_camera_frame` | Right camera frame |
| `${name}_right_camera_frame_optical` | Right optical frame |

**Mono models** (zedxonegs, zedxone4k, zedxonehdr):

| Frame | Description |
|-------|-------------|
| `${name}_camera_link` | Camera mount reference point (screw hole) |
| `${name}_camera_center` | Camera housing center |
| `${name}_camera_frame` | Camera frame |
| `${name}_camera_frame_optical` | Optical frame (ROS convention: z-forward, x-right, y-down) |

**Additional frames** (conditional):

| Frame | Condition | Description |
|-------|-----------|-------------|
| `${name}_gnss_link` | `enable_gnss=true` | GNSS antenna position |
| `${name}_mag_link` | `model=zed2` | Magnetometer |
| `${name}_baro_link` | `model=zed2` | Barometer |
| `${name}_temp_left_link` | `model=zed2` | Left temperature sensor |
| `${name}_temp_right_link` | `model=zed2` | Right temperature sensor |

## Robot Integration

This package provides the camera model independently. Integration with a specific robot is handled by **Integration Layer** packages (e.g., `neuromeka_integrations`).

Refer to the respective integrations package documentation for integration examples.

## v1.0.0 Migration

In v1.0.0, the custom `stereolabs_zedxm` macro was replaced with the official `zed_camera` macro (Breaking Change).

Key changes:

- Macro: `stereolabs_zedxm(prefix, parent_link, mount_config_file)` -> `zed_camera(name, model, ...)`
- Xacro file: `urdf/zedxm.xacro` -> `urdf/zed_macro.urdf.xacro`
- TF frames: now match the official ZED ROS 2 driver naming exactly
- Hand-Eye calibration YAML `z` value: subtract 0.016m (mount reference point changed)

See [CHANGELOG.md](../CHANGELOG.md) for the full change history.

## License

Apache-2.0
