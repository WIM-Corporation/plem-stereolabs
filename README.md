# plem-stereolabs

> Stereolabs ZED 카메라 ROS 2 Description

[English](docs/README.en.md)

## 개요

`stereolabs_description`은 Stereolabs ZED 카메라의 URDF/xacro 모델과 3D 메시를 제공하는 ROS 2 패키지이다.

**Tier 2 (로봇 벤더 비종속)** 패키지로, 특정 로봇 드라이버나 플랫폼에 의존하지 않는다. Neuromeka, Doosan 등 어떤 로봇 벤더에서도 동일하게 참조할 수 있다.

## 포함 모델

| 모델 | 매크로 이름 | 크기 | 무게 | 인터페이스 |
|------|-----------|------|------|-----------|
| **ZED X Mini** | `stereolabs_zedxm` | 94 x 32 x 37 mm | 150 g | GMSL2 (ZED Link Duo) |

ZED X Mini는 스테레오 뎁스 카메라로, 베이스라인 50 mm, 작동 범위 100 mm -- 8 m (2.2 mm 렌즈), IP67 등급이다.

## 설치

```bash
# 워크스페이스에 클론
cd ~/ros2_ws/src
git clone <repo-url> stereolabs_description

# 빌드
cd ~/ros2_ws
colcon build --packages-select stereolabs_description
source install/setup.bash
```

## 사용법

### Standalone xacro 실행

```bash
# URDF 생성 확인 (mount_config_file 필수)
xacro urdf/zedxm.xacro prefix:=cam_ parent_link:=tool0 mount_config_file:=/path/to/camera_mount.yaml
```

### 다른 xacro에서 매크로 호출

```xml
<xacro:include filename="$(find stereolabs_description)/urdf/zedxm.xacro"/>
<xacro:stereolabs_zedxm prefix="cam_"
                         parent_link="tool0"
                         mount_config_file="$(find my_config)/config/camera_mount.yaml"/>
```

### 매크로 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|---------|------|-------|------|
| `prefix` | string | (필수) | 모든 링크/조인트 이름에 적용되는 네임스페이스 접두사 |
| `parent_link` | string | (필수) | 카메라를 부착할 부모 링크 |
| `mount_config_file` | string | (필수) | Hand-Eye 캘리브레이션 YAML 파일 경로 |

### mount_config_file 형식

```yaml
camera_mount:
  x: 0.0
  y: 0.0
  z: 0.05
  roll: 0.0
  pitch: 0.0
  yaw: 0.0
```

### 생성되는 주요 프레임

- `${prefix}zedx_link` -- 카메라 하우징 중심 (마운트 기준점)
- `${prefix}zedx_left_optical_frame` -- 좌안 광학 중심 (Hand-Eye 캘리브레이션 기준)
- `${prefix}zedx_depth_optical_frame` -- 뎁스 광학 프레임 (ROS 규약: z-전방, x-우측, y-하방)

## 로봇 통합

이 패키지는 독립적으로 카메라 모델만 제공한다. 특정 로봇에 부착하는 통합 설정은 **Tier 3 integrations** 패키지(`neuromeka_integrations` 등)에서 처리한다.

통합 예시는 해당 integrations 패키지의 문서를 참조한다.

## 새 모델 추가

1. `urdf/{model}.xacro` 작성 -- 매크로 이름은 `stereolabs_{model}` 규칙을 따른다.
2. `meshes/{model}/`에 STL/DAE 메시를 배치한다.
3. `test/test_xacro.py`에 xacro 파싱 테스트를 추가한다.
4. `package.xml` 버전을 SemVer patch 범프한다.

## 라이선스

Apache-2.0
