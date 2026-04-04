# Changelog

## [1.0.0] - 2026-04-04

### Changed
- **BREAKING**: Replaced custom `stereolabs_zedxm` macro with vendored official `zed_camera` macro from `zed-ros2-wrapper`
- TF frame names now match official ZED ROS2 driver exactly (`{name}_camera_link`, `{name}_left_camera_frame_optical`, etc.)
- Mesh directory flattened: `meshes/zedxm/zedxm.stl` → `meshes/zedxm.stl`
- Package role changed from "custom URDF" to "vendored official URDF"

### Added
- `VENDOR_SOURCE.md` for source tracking and sync procedure
- Support for all 13+ ZED camera models via official macro parameterization

### Removed
- Custom `stereolabs_zedxm` macro (`urdf/zedxm.xacro`)
- Hardcoded IMU link (per-unit factory calibration; should not be in URDF)

### Migration
- Integration layer (`sensors/zedxm.xacro`) must use `zed_camera` macro instead of `stereolabs_zedxm`
- Hand-Eye calibration YAML `z` value: subtract 0.016m (mount reference changed from body center to camera_link screw hole)
- SRDF `sensor_collision` macro now requires `cam_name` parameter

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.1] - 2026-04-03

### Fixed
- Reverted left/right eye Y offset sign to match plem body frame convention (left = -Y)
  - plem body frame differs from official zed-ros2-wrapper due to mesh rpy(-90,-90,-90) correction
  - RViz verification confirmed pre-v0.2.0 sign was correct
- Removed `optical_offset_x` — not applicable under plem body frame axes
- Reverted Tier 3 `zedxm_mount.yaml` to calibration raw values (x: 0.0626, y: 0.0019)
- Fixed frame name references `zedx_` → `zedxm_` in README and rules documentation

### Migration
- Tier 3 `zedxm_mount.yaml` Y-value must be reverted to pre-v0.2.0 value: `y = left_eye - (-baseline_half)`
- APRR `zed_tf_bridge` frame names still need `zedxm_*` update (from v0.2.0)

## [0.2.0] - 2026-04-03

### Changed
- **[BREAKING]** Frame name prefix `zedx_` → `zedxm_` to match camera model (ZED X Mini)
  - `zedx_link` → `zedxm_link`, `zedx_left_optical_frame` → `zedxm_left_optical_frame`, etc.
  - Prevents future name collision if ZED X (non-Mini) support is added

### Added
- Right camera frame (`zedxm_right_optical_frame`) and right depth optical frame
- IMU link (`zedxm_imu_link`) with Camera-IMU offset from ZED SDK calibration data
- Comment on visual mesh rpy explaining non-ROS STL axis correction

### Migration
- All frame references `zedx_*` must be updated to `zedxm_*` in downstream projects
- APRR `zed_tf_bridge` rotation compensation and frame names need re-verification
- Hand-Eye re-calibration recommended when hardware is accessible

## [0.1.0] - 2026-04-02

### Added
- `stereolabs_description` ROS2 package with Stereolabs ZED camera URDF and mesh assets
- Xacro macro-based composition: `stereolabs_zedxm` with parameterized prefix support
- Visual (DAE) and collision (STL) mesh files for ZED X Mini camera
- `mount_config_file` parameter support for flexible mounting configuration
- CMakeLists.txt and package.xml with minimal dependencies (ROS2 Humble compatible)
- CLAUDE.md and repository-specific coding rules (.claude/rules/)
- CI workflow for GitHub Actions (Ubuntu 22.04, ROS2 Humble)
- README.md (Korean) and docs/README.en.md (English)
- Xacro parsing and namespace validation tests

### Changed
- Extracted Stereolabs ZED camera description from neuromeka_description package (Tier 2 separation)
- Updated documentation with correct GitHub repository URL

### Documentation
- Added CLAUDE.md for Tier 2 peripheral description standards
- Added repository README with setup and usage instructions
- English translation of documentation provided
