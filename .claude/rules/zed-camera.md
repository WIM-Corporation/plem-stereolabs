---
description: "ZED Camera ROS 2 — QoS, topic naming, TF alignment, headless NITROS, param_overrides, common mistakes"
---

# ZED Camera ROS 2 — 코딩 규칙

zed-ros2-wrapper 사용 시 필수 규칙. 전체 파라미터/서비스는 `.claude/references/zed-ros2-api-reference.md` 참조.

## 카메라 모델 (`camera_model` launch 값)

| 모델 | `camera_model` | 연결 |
|------|----------------|------|
| ZED / ZED Mini | `zed` / `zedm` | USB 3.0 |
| ZED 2 / 2i | `zed2` / `zed2i` | USB 3.0 |
| ZED X / X Mini | `zedx` / `zedxm` | GMSL2 |
| ZED X HDR / Mini / Max | `zedxhdr` / `zedxhdrmini` / `zedxhdrmax` | GMSL2 |
| ZED X One GS / 4K / HDR | `zedxonegs` / `zedxone4k` / `zedxonehdr` | GMSL2 (단안) |

GMSL2 카메라는 ZED Link + Jetson 필수. ZED X One은 단안 — 위치 추적/매핑 미지원.

## QoS — Silent Failure 원인 1위

드라이버 기본 QoS: **RELIABLE + VOLATILE**. 구독자 QoS 불일치 시 데이터가 오지 않고 에러도 없음.

```python
# 올바른 구독 예
qos = QoSProfile(reliability=ReliabilityPolicy.BEST_EFFORT,
                 history=HistoryPolicy.KEEP_LAST, depth=1)
self.create_subscription(Image, topic, cb, qos)
```

TRANSIENT_LOCAL 사용 시 연결 자체가 안 됨. 반드시 VOLATILE 사용.

## 토픽명 (v5.1+ 네이밍)

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

## Namespace — 다중 카메라

`camera_name` + `serial_number`로 카메라별 namespace 분리:
```bash
ros2 launch zed_wrapper zed_camera.launch.py \
    camera_model:=zedxm camera_name:=front serial_number:=12345678
# → /front/zed_node/...
```
시리얼 확인: `ZED_Explorer -a`

## TF 프레임 정합 (plem URDF 사용 시)

- 공식 벤더링 매크로 사용: 프레임명이 ZED 드라이버와 100% 일치
- 카메라 이름 규칙: `{prefix}cam` (예: `cam`, 멀티카메라 시 `front_cam`)
- 주요 프레임: `{cam_name}_camera_link`, `{cam_name}_left_camera_frame_optical`
- plem URDF와 함께 사용 시 드라이버 TF 비활성화 필수 (충돌 방지):

```yaml
pos_tracking:
  publish_tf: false
  publish_map_tf: false
```

Hand-Eye 캘리브레이션: `neuromeka_integrations/urdf/sensors/config/zedxm_mount.yaml`
TF 통합 상세: `.claude/references/zed-robot-integration.md` 참조.

## Headless (SSH) — NITROS SIGSEGV

`DISPLAY` 미설정 환경에서 NITROS EGL 초기화 실패 → SIGSEGV.

```
nvbufsurftransform: Could not get EGL display connection
exit code -11 (SIGSEGV)
```

해결: `param_overrides:="debug.disable_nitros:=true"` 추가.

## `param_overrides` Launch Argument

YAML 수정 없이 파라미터 오버라이드. **최고 우선순위**. 세미콜론 구분:

```bash
ros2 launch zed_wrapper zed_camera.launch.py camera_model:=zedxm \
    param_overrides:="object_detection.od_enabled:=true;debug.disable_nitros:=true"
```

> **핵심**: `od_enabled:=true`를 launch arg로 직접 전달하면 효과 없음.
> `zed_camera.launch.py`에 미선언된 arg이므로 ROS 2가 경고 없이 무시. 반드시 `param_overrides` 사용.

## 성능 이슈 시

CPU 과부하: `grab_compute_capping_fps`, `point_cloud_freq` 동적 조절. 해상도: `pub_downscale_factor: 2.0`. 상세: `zed-optimization.md`

## Lazy Publishing

ZED는 구독자가 있을 때만 publish. 데이터 미수신 시 구독자 수 확인: `ros2 topic info <topic>`

## 첫 실행 AI 모델 최적화

Object Detection / Body Tracking 첫 실행 시 TensorRT 최적화 수 분 소요.
캐시: `/usr/local/zed/resources/` (이후 즉시 로드)

## `zed_msgs` 핵심 메시지

**Object** (`obj_det/objects`): `label`, `label_id`, `confidence`(1-99), `position[3]`(m), `velocity[3]`(m/s), `tracking_state`(0=OFF,1=OK,2=SEARCHING,3=TERMINATE), `bounding_box_2d`(4corners), `bounding_box_3d`(8corners), `dimensions_3d[w,h,l]`(m)

**HealthStatusStamped** (`health_status`): `low_image_quality`, `low_lighting`, `low_depth_reliability`, `low_motion_sensors_reliability` (bool)

**PosTrackStatus** (`pose/status`): `odometry_status`(0=OK,1=UNAVAILABLE), `spatial_memory_status`(0=OK,2=SEARCHING,3=OFF)

## 참조

| 문서 | 내용 |
|------|------|
| `zed-ros2-api-reference.md` | 전체 토픽/서비스/파라미터 표 |
| `zed-usage-guide.md` | RViz, SVO, OD 데모, param_overrides 상세 |
| `zed-yolo-config.md` | YOLO ONNX config, class 정의 |
| `zed-dds-network.md` | DDS/커널/MTU 네트워크 튜닝 |
| `zed-optimization.md` | Frequency/latency/ROI 성능 최적화 |
| `zed-robot-integration.md` | Manipulator TF, multi-camera, streaming |
| `zed-recording.md` | SVO/rosbag 녹화·재생·벤치마크 |
