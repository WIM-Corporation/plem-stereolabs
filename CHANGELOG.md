# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
