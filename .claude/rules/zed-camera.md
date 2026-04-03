---
description: "ZED Camera ROS 2 — topics, services, parameters, TF alignment, Jetson build flags"
---

# ZED Camera ROS 2 — 코딩 규칙

zed-ros2-wrapper를 통해 ZED 카메라를 사용하는 프로젝트용.

## 카메라 모델 (launch `camera_model` 값)

| 모델 | `camera_model` | 연결 | max_depth |
|------|---------------|------|-----------|
| ZED | `zed` | USB 3.0 | 10m |
| ZED Mini | `zedm` | USB 3.0 | 10m |
| ZED 2 | `zed2` | USB 3.0 | 15m |
| ZED 2i | `zed2i` | USB 3.0 | 15m |
| ZED X | `zedx` | GMSL2 | 15m |
| ZED X Mini | `zedxm` | GMSL2 | 10m |
| ZED X HDR | `zedxhdr` | GMSL2 | 15m |
| ZED X HDR Mini | `zedxhdrmini` | GMSL2 | 10m |
| ZED X HDR Max | `zedxhdrmax` | GMSL2 | 20m |
| ZED X One GS | `zedxonegs` | GMSL2 | AI depth |
| ZED X One 4K | `zedxone4k` | GMSL2 | AI depth |
| ZED X One HDR | `zedxonehdr` | GMSL2 | AI depth |
| Virtual | `virtual` | - | 10m |

GMSL2 카메라는 ZED Link 캡처보드 + Jetson 필수. ZED X One은 단안 (위치 추적/매핑 미지원).

## Namespace 및 다중 카메라

토픽/서비스 접두사: `/<namespace>/<node_name>/`

| Launch 인자 | 기본값 | 역할 |
|-------------|--------|------|
| `camera_name` | `zed` | 카메라 이름 (namespace 미지정 시 namespace로 사용) |
| `namespace` | `""` (=camera_name) | ROS 2 namespace |
| `node_name` | `zed_node` | 노드 이름 |
| `serial_number` | `0` | 물리 카메라 구분 (다중 카메라 시 필수) |
| `camera_id` | `-1` | serial_number 대안 |

**다중 카메라**: 카메라마다 `camera_name` + `serial_number`를 다르게 지정:
```bash
# 카메라 1
ros2 launch zed_wrapper zed_camera.launch.py \
    camera_model:=zedxm camera_name:=front serial_number:=12345678
# 카메라 2
ros2 launch zed_wrapper zed_camera.launch.py \
    camera_model:=zedxm camera_name:=rear serial_number:=87654321
# 결과: /front/zed_node/..., /rear/zed_node/...
```

**launch 파일에서 다중 카메라** (공식 권장):
```python
front = GroupAction([
    PushRosNamespace('front'),
    IncludeLaunchDescription(zed_launch, launch_arguments={
        'camera_model': 'zedxm', 'camera_name': 'zed_front',
        'serial_number': '12345678'}.items())])
```

시리얼 번호 확인: `ZED_Explorer -a`

## 토픽명 (잘못 쓰면 데이터가 안 들어오고 에러도 안 남)

접두사: `/<camera_name>/<node_name>/` (기본: `/zed/zed_node/`)

| 기능 | 토픽 | 흔한 실수 |
|------|------|----------|
| 이미지 | `rgb/color/rect/image` | ~~`rgb/image_rect_color`~~ |
| 깊이 | `depth/depth_registered` | |
| Point Cloud | `point_cloud/cloud_registered` | ~~`point_cloud/cloud`~~ |
| Disparity | `disparity/disparity_image` | ~~`disparity/disparity`~~ |
| 신뢰도 | `confidence/confidence_map` | ~~`depth/confidence_map`~~ |
| Object Detection | `obj_det/objects` | ~~`objects`~~ |
| Body Tracking | `body_trk/skeletons` | |
| Fused Cloud | `mapping/fused_cloud` | ~~`fused_cloud`~~ |
| 평면 감지 | `plane` | |
| Odometry | `odom` | |
| Pose | `pose` | |
| Pose 상태 | `pose/status` | |
| IMU | `imu/data` | |
| 자력계 | `imu/mag` | 기본 비활성 |
| 기압계 | `atm_press` | 기본 비활성 |
| 온도 | `temperature/imu`, `temperature/left`, `temperature/right` | 기본 비활성 |
| GNSS fix | `pose/fused_fix`, `pose/origin_fix` | |
| 상태 | `health_status`, `heartbeat` | |
| ROI | `roi_mask/image` | 기본 비활성 |

## 서비스명

| 기능 | 서비스 | 흔한 실수 |
|------|--------|----------|
| Depth on/off | `enable_depth` | |
| OD on/off | `enable_obj_det` | ~~`enable_obj_detection`~~ |
| BT on/off | `enable_body_trk` | ~~`enable_body_tracking`~~ |
| Mapping on/off | `enable_mapping` | |
| Streaming on/off | `enable_streaming` | |
| Odometry 리셋 | `reset_odometry` | |
| 위치 추적 리셋 | `reset_pos_tracking` | |
| 자세 설정 | `set_pose` | `zed_msgs/SetPose` |
| Area memory 저장 | `save_area_memory` | `zed_msgs/SaveAreaMemory` |
| ROI 설정/초기화 | `set_roi`, `reset_roi` | `zed_msgs/SetROI` |
| SVO 녹화 | `start_svo_rec` / `stop_svo_rec` | |
| SVO 일시정지 | `toggle_svo_pause` | |
| SVO 프레임 이동 | `set_svo_frame` | `zed_msgs/SetSvoFrame` |
| 좌표 변환 (GNSS) | `toLL`, `fromLL` | `robot_localization` 타입 |

## 파라미터 — General

| 파라미터 | 기본값 | 흔한 오해 |
|----------|--------|----------|
| `general.pub_resolution` | `CUSTOM` | ~~`MEDIUM`~~ |
| `general.pub_downscale_factor` | `2.0` | ~~`1.0`~~ |
| `general.pub_frame_rate` **[D]** | `0.0` (무제한) | |
| `general.camera_timeout_sec` | `5` | |
| `general.camera_flip` | `false` | |
| `general.publish_status` | `true` | |

## 파라미터 — Depth

| 파라미터 | 기본값 | 흔한 오해 |
|----------|--------|----------|
| `depth.depth_mode` | `NEURAL_LIGHT` | ~~`NEURAL_PLUS`~~ |
| `depth.openni_depth_mode` | `false` | false=32bit float(m), true=16bit uint(mm) |
| `depth.depth_confidence` **[D]** | `95` | |
| `depth.point_cloud_freq` **[D]** | `10.0` | |
| `depth.point_cloud_res` | `COMPACT` | COMPACT/REDUCED |
| `depth.publish_depth_map` | `true` | |
| `depth.publish_depth_info` | `false` | |
| `depth.publish_point_cloud` | `true` | |
| `depth.publish_depth_confidence` | `false` | |
| `depth.publish_disparity` | `false` | |

## 파라미터 — Video 발행 제어

| 파라미터 | 기본값 | 설명 |
|----------|--------|------|
| `video.publish_rgb` | `true` | RGB 이미지 |
| `video.publish_left_right` | `false` | 좌/우 이미지 |
| `video.publish_raw` | `false` | 원본(미보정) |
| `video.publish_gray` | `false` | 그레이스케일 |
| `video.publish_stereo` | `false` | 좌우 병합 |

## 파라미터 — Positional Tracking

| 파라미터 | 기본값 | 설명 |
|----------|--------|------|
| `pos_tracking.pos_tracking_enabled` | `true` | |
| `pos_tracking.pos_tracking_mode` | `AUTO` | AUTO/GEN_1/GEN_3 |
| `pos_tracking.imu_fusion` | `true` | |
| `pos_tracking.publish_tf` | `true` | odom→camera TF |
| `pos_tracking.publish_map_tf` | `true` | map→odom TF |
| `pos_tracking.map_frame` | `map` | |
| `pos_tracking.odometry_frame` | `odom` | |
| `pos_tracking.area_memory` | `true` | loop closure |
| `pos_tracking.two_d_mode` | `false` | 2D nav (z=0) |
| `pos_tracking.publish_odom_pose` | `true` | |
| `pos_tracking.publish_pose_cov` | `false` | |
| `pos_tracking.publish_cam_path` | `false` | |

## 파라미터 — Sensors

| 파라미터 | 기본값 | 흔한 오해 |
|----------|--------|----------|
| `sensors.sensors_pub_rate` **[D]** | `100.0` | ~~`200.0`~~ |
| `sensors.publish_imu` | `true` | |
| `sensors.publish_imu_raw` | `false` | |
| `sensors.publish_mag` | **`false`** | ~~`true`~~ |
| `sensors.publish_baro` | **`false`** | ~~`true`~~ |
| `sensors.publish_temp` | **`false`** | ~~`true`~~ |

## 파라미터 — Mapping

| 파라미터 | 기본값 | 설명 |
|----------|--------|------|
| `mapping.mapping_enabled` | `false` | |
| `mapping.resolution` | `0.05` | m (0.01-0.2) |
| `mapping.max_mapping_range` | `5.0` | m (-1=자동) |
| `mapping.fused_pointcloud_freq` | `1.0` | Hz |

## 파라미터 — GNSS Fusion

| 파라미터 | 기본값 | 흔한 오해 |
|----------|--------|----------|
| `gnss_fusion.gnss_fusion_enabled` | `false` | |
| `gnss_fusion.gnss_fix_topic` | `/fix` | ~~`/gps/fix`~~ |
| `gnss_fusion.publish_utm_tf` | `true` | |
| `gnss_fusion.enable_rolling_calibration` | `true` | |

## 파라미터 — Streaming Server

| 파라미터 | 기본값 | 설명 |
|----------|--------|------|
| `stream_server.stream_enabled` | `false` | |
| `stream_server.codec` | `H265` | H264/H265 |
| `stream_server.port` | `30000` | 짝수만 |
| `stream_server.bitrate` | `12500` | Kbps |

## 파라미터 — ROI

| 파라미터 | 기본값 | 설명 |
|----------|--------|------|
| `region_of_interest.automatic_roi` | `false` | 로봇 부위 자동 제거 |
| `region_of_interest.publish_roi_mask` | `false` | |

## 파라미터 — SVO

| 파라미터 | 기본값 | 설명 |
|----------|--------|------|
| `svo.svo_loop` | `false` | 루프 재생 |
| `svo.svo_realtime` | `false` | 실시간 재생 |
| `svo.use_svo_timestamps` | `true` | |
| `svo.replay_rate` | `1.0` | 0.1-5.0 |

## TF 프레임 정합 (plem URDF ↔ 드라이버)

카메라 URDF의 TF 프레임과 드라이버가 publish하는 TF 프레임이 일치해야 한다.
ZED X Mini의 경우:
- URDF 프레임: `{prefix}zedxm_left_optical_frame`
- 드라이버 프레임: zed-ros2-wrapper의 `camera_model` 파라미터에 따라 결정

plem URDF와 함께 사용 시 드라이버의 TF publish를 비활성화하여 프레임 충돌을 방지한다:
```yaml
pos_tracking:
  publish_tf: false
  publish_map_tf: false
```

Hand-Eye 캘리브레이션 결과는 Tier 3 Integration 패키지에 저장된다 (예: `neuromeka_integrations/urdf/sensors/config/zedxm_mount.yaml`).

## Headless (SSH) 환경 주의

> **SIGSEGV 크래시 위험**: SSH 등 headless 환경(`DISPLAY` 미설정)에서 NITROS가 EGL 초기화에 실패하여 카메라 열기 단계에서 SIGSEGV가 발생한다.

증상:
```
nvbufsurftransform: Could not get EGL display connection
(Argus) Error BadParameter: ...createBuffer()...
exit code -11 (SIGSEGV)
```

해결 (택일):
- `param_overrides:="debug.disable_nitros:=true"` 추가
- `DISPLAY=:N`이 설정된 환경(로컬 또는 Xvfb)에서 실행

## `param_overrides` Launch Argument

CLI에서 YAML을 수정하지 않고 파라미터를 오버라이드하는 launch argument. 최고 우선순위.

```bash
# 세미콜론으로 복수 파라미터 전달
ros2 launch zed_wrapper zed_camera.launch.py camera_model:=zedxm \
    param_overrides:="object_detection.od_enabled:=true;debug.disable_nitros:=true"
```

> **주의**: `object_detection.od_enabled:=true`를 launch argument로 직접 전달하면 효과 없음. `zed_camera.launch.py`에 선언된 launch argument가 아니므로 ROS 2 launch가 경고 없이 무시한다. 반드시 `param_overrides`를 통해 전달할 것.

## Jetson 빌드

colcon build 시 Jetson 전용 cmake 플래그 필수:
```
-DCMAKE_LIBRARY_PATH=/usr/local/cuda/lib64/stubs
-DCMAKE_CXX_FLAGS=-Wl,--allow-shlib-undefined
```
누락 시 silent link failure 발생.

## zed_msgs 핵심 메시지

**Object** (`obj_det/objects` 토픽):
```
label, label_id, confidence(1-99)
position[3] (m), velocity[3] (m/s)
tracking_state: 0=OFF, 1=OK, 2=SEARCHING, 3=TERMINATE
bounding_box_2d (4 corners), bounding_box_3d (8 corners)
dimensions_3d [w,h,l] (m)
```

**HealthStatusStamped** (`health_status` 토픽):
```
low_image_quality, low_lighting, low_depth_reliability, low_motion_sensors_reliability (bool)
```

**PosTrackStatus** (`pose/status` 토픽):
```
odometry_status: 0=OK, 1=UNAVAILABLE
spatial_memory_status: 0=OK, 2=SEARCHING, 3=OFF
```
