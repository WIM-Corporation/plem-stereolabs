# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-04-03

### Fixed
- **[BREAKING]** Left camera Y offset sign: `-baseline_half` → `+baseline_half` (ROS convention: left = +Y)
- Added `optical_offset_x` (-0.01m) to left/right eye joints, aligning with official zed_macro.urdf.xacro

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
- Tier 3 `zedxm_mount.yaml` Y-value must be recalculated: `y_new = y_old - 2 × baseline_half`
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
