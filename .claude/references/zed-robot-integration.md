---
description: "ZED robot integration — manipulator TF setup, URDF 2-Layer pattern, multi-camera, streaming bridge, diagnostics"
---

# ZED Robot Integration

## 1. TF 결정 (CRITICAL — 먼저 읽을 것)

TF 설정 오류 시 ZED driver ↔ robot_state_publisher 간 충돌 → "chaotic behaviors" (좌표계 깜빡임, 위치 불일치).

### 결정 흐름도

```
고정형 manipulator? → publish_tf: false, publish_map_tf: false
모바일 로봇?       → publish_tf: true (camera #1 only)
다중 카메라?       → camera #1만 publish_tf: true, 나머지 false
```

### 고정형 Manipulator (plem)

`base_link`가 PARENT, `camera_link`가 CHILD.
Manipulator TF 트리(robot_state_publisher / MoveIt)가 권위를 가짐.
ZED `camera_link`는 마운트 포인트 링크의 child로 부착.

ZED driver에서 **반드시** 비활성화:
```yaml
pos_tracking:
  pos_tracking_enabled: false   # 로봇 FK가 위치를 제공하므로 VIO 불필요
  publish_tf: false
  publish_map_tf: false
depth:
  depth_stabilization: 0        # 이것 없으면 카메라 멈춤 (아래 설명 참조)
general:
  publish_urdf: false           # 로봇 RSP와 이중 발행 방지
```

### `depth_stabilization` 필수 조합

`depth_stabilization`의 기본값은 1이다. 이 값이 0이 아니면 SDK 내부에서 positional tracking을
활성화하여 `base_link` TF를 기다린다. `publish_tf: false`로 TF 발행을 꺼 놓았으므로
SDK가 TF를 기다리며 **카메라가 멈춘다** (데이터 발행 중단, 에러 로그 없음).

```bash
ros2 launch zed_wrapper zed_camera.launch.py camera_model:=zedxm \
    param_overrides:="pos_tracking.pos_tracking_enabled:=false;pos_tracking.publish_tf:=false;pos_tracking.publish_map_tf:=false;depth.depth_stabilization:=0"
```

비활성화 안 하면 ZED가 독립적인 `odom → camera_link` TF를 발행하여
`base_link → camera_link` 경로와 충돌한다.

> **Eye-on-Hand 독립 테스트 시에도 `publish_tf: false` 권장.**
> Eye-on-Hand 구성에서 ZED VIO는 불안정하다 — 핸드의 움직임으로 드리프트/NaN TF가
> 발생하여 rviz OOM 크래시 가능. `publish_tf:=false` + `pos_tracking_enabled:=false`로
> 설정하고, rviz Fixed Frame을 `zed_left_camera_frame`으로 지정하면 안정적으로 시각화.

### 모바일 로봇 (참고 — plem 해당 없음)

`use_zed_localization: true` → `camera_link`가 PARENT, `base_link`가 CHILD.
TF: `map → odom → camera_link → base_link`
Origin은 **반전**: `xyz="-0.12 0.0 -0.25"` (parent-child 역전 때문)

## 2. URDF — plem 2-Layer 패턴

| Layer | 패키지 | 역할 |
|------|--------|------|
| 1 | `neuromeka_description` | 로봇 본체 URDF |
| 2 | `stereolabs_description` | ZED 카메라 URDF (이 패키지) |
| 3 | `neuromeka_integrations` | 통합 xacro / SRDF |

ZED URDF는 Integration Layer 통합 xacro를 통해 포함.
Hand-Eye 캘리브레이션: `neuromeka_integrations/urdf/sensors/config/zedxm_mount.yaml`

ZED wrapper 공식 xacro 매크로:
```xml
<xacro:include filename="$(find zed_wrapper)/urdf/zed_macro.urdf.xacro" />
<xacro:zed_camera name="$(arg camera_name)" model="$(arg camera_model)" />
```
→ `<camera_name>_camera_link` reference link 생성.

## 3. Multi-Camera

```bash
ros2 launch zed_multi_camera zed_multi_camera.launch.py \
    cam_names:='[zed_front,zed_rear]' \
    cam_models:='[zedx,zedxm]' \
    cam_serials:='[<serial1>,<serial2>]'
```

핵심 규칙:
- 모든 노드는 `zed_multi` namespace + 동일 컨테이너
- **camera #1만** `publish_tf: true`, 나머지 **반드시** `false`
- URDF: 첫 번째 카메라 joint는 parent/child 반전 (visual odometry reference)
- 시리얼 확인: `ZED_Explorer -a`

## 4. Streaming Bridge (GPU 인코딩 원격 시각화)

DDS 대신 단일 GPU 인코딩 스트림으로 원격 시각화. 서버/클라이언트 양쪽 NVIDIA GPU 필수.

**서버** (Jetson with ZED):
```yaml
stream_server:
  stream_enabled: true
  codec: 'H265'
  port: 30000           # 반드시 짝수
  bitrate: 12500        # Kbps
  adaptative_bitrate: true
```

런타임: `ros2 service call /zed/zed_node/enable_streaming std_srvs/srv/SetBool "{data: true}"`

**클라이언트** (원격 PC):
```bash
ros2 launch zed_wrapper zed_camera.launch.py \
    camera_model:=zedxm stream_address:=<jetson_ip> stream_port:=30000
```

장점: 각 클라이언트가 depth/OD 등 기능을 독립 설정. 네트워크는 압축 스트림만 통과.

## 5. Diagnostics

`/diagnostics` 토픽: grab frequency, frame drop rate, publishing rates, temperature, module statuses.

```bash
ros2 topic echo /diagnostics
```

## 체크리스트

- [ ] `publish_tf: false`, `publish_map_tf: false` (fixed-base manipulator)
- [ ] Integration Layer xacro에 ZED 매크로 포함 + mount joint 정의
- [ ] `zedxm_mount.yaml` Hand-Eye 캘리브레이션 반영
- [ ] 다중 카메라 시 camera #1만 TF 발행
- [ ] Streaming bridge: server/client 양쪽 NVIDIA GPU 확인
