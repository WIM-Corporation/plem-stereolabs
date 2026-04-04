# Vendor Source Tracking

## zed_macro.urdf.xacro

- **Source**: https://github.com/stereolabs/zed-ros2-wrapper
- **File**: `zed_wrapper/urdf/zed_macro.urdf.xacro`
- **Branch**: master
- **License**: Apache 2.0
- **Date vendored**: 2026-04-04

### Local modifications

1. Mesh path: `$(find zed_msgs)` → `$(find stereolabs_description)` (6 occurrences)

No other modifications. All frame names, physical specs, and TF structure are identical to the official source.

## Meshes

- **Source**: https://github.com/stereolabs/zed-ros2-wrapper (`zed_msgs/meshes/`)
- **Vendored models**: zedxm.stl

Additional model meshes can be added from the same source as needed.

## Sync Procedure

1. Check `zed-ros2-wrapper` releases for changes to `zed_macro.urdf.xacro`
2. If changed: download, re-apply mesh path patch, test
3. If new model added: copy the new STL mesh
4. Update this file with new commit hash and date
5. Bump patch version in `package.xml`
