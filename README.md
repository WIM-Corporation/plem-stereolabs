# plem-stereolabs

> Stereolabs ZED 카메라 ROS 2 Description

[English](docs/README.en.md)

## 개요

`stereolabs_description`은 Stereolabs ZED 카메라의 URDF/xacro 모델과 3D 메시를 제공하는 ROS 2 패키지이다.

**Description Layer (로봇 벤더 비종속)** 패키지로, 특정 로봇 드라이버나 플랫폼에 의존하지 않는다. Neuromeka, Doosan 등 어떤 로봇 벤더에서도 동일하게 참조할 수 있다.

이 패키지는 공식 [zed-ros2-wrapper](https://github.com/stereolabs/zed-ros2-wrapper)의 `zed_macro.urdf.xacro`를 벤더링한 복사본이다. ZED SDK 의존 없이 URDF 모델만 독립 제공한다. 벤더링 출처와 동기화 이력은 [VENDOR_SOURCE.md](VENDOR_SOURCE.md)를 참조한다.

## 지원 카메라 모델

공식 매크로가 지원하는 전체 모델 목록:

| 모델 | `model` 값 | 타입 | 연결 |
|------|-----------|------|------|
| ZED | `zed` | 스테레오 | USB 3.0 |
| ZED Mini | `zedm` | 스테레오 | USB 3.0 |
| ZED 2 | `zed2` | 스테레오 | USB 3.0 |
| ZED 2i | `zed2i` | 스테레오 | USB 3.0 |
| ZED X | `zedx` | 스테레오 | GMSL2 |
| ZED X HDR | `zedxhdr` | 스테레오 | GMSL2 |
| ZED X Mini | `zedxm` | 스테레오 | GMSL2 |
| ZED X HDR Mini | `zedxhdrmini` | 스테레오 | GMSL2 |
| ZED X HDR Max | `zedxhdrmax` | 스테레오 | GMSL2 |
| Virtual | `virtual` | 스테레오 | - |
| ZED X One GS | `zedxonegs` | 단안 | GMSL2 |
| ZED X One 4K | `zedxone4k` | 단안 | GMSL2 |
| ZED X One HDR | `zedxonehdr` | 단안 | GMSL2 |

단안 모델(ZED X One 시리즈)은 좌/우 카메라 프레임 대신 단일 카메라 프레임만 생성된다.

## 설치

```bash
# 워크스페이스에 클론
cd ~/ros2_ws/src
git clone https://github.com/WIM-Corporation/plem-stereolabs.git

# 빌드
cd ~/ros2_ws
colcon build --packages-select stereolabs_description
source install/setup.bash
```

## 사용법

### Standalone xacro 실행

```bash
# URDF 생성 확인
xacro $(ros2 pkg prefix stereolabs_description)/share/stereolabs_description/urdf/zed_macro.urdf.xacro \
    name:=cam model:=zedxm
```

### 다른 xacro에서 매크로 호출

```xml
<xacro:include filename="$(find stereolabs_description)/urdf/zed_macro.urdf.xacro"/>
<xacro:zed_camera name="cam" model="zedxm">
    <origin xyz="0 0 0" rpy="0 0 0"/>  <!-- gnss_origin (필수 블록, GNSS 미사용 시 더미 값) -->
</xacro:zed_camera>
```

### 매크로 파라미터

매크로 시그니처: `zed_camera(name, model, enable_gnss, custom_baseline, *gnss_origin)`

| 파라미터 | 타입 | 기본값 | 설명 |
|---------|------|-------|------|
| `name` | string | `zed` | 모든 링크/조인트 이름에 적용되는 네임스페이스 접두사 |
| `model` | string | `zed` | 카메라 모델 (위 지원 모델 표 참조) |
| `enable_gnss` | bool | `false` | GNSS 링크 활성화 여부 |
| `custom_baseline` | float | `0` | `virtual` 모델에서 사용할 커스텀 베이스라인 (m) |
| `*gnss_origin` | block | (필수) | GNSS 안테나 오프셋 `<origin>` 블록 |

### 생성되는 TF 프레임

**스테레오 모델** (zed, zedm, zed2, zed2i, zedx, zedxhdr, zedxm, zedxhdrmini, zedxhdrmax, virtual):

| 프레임 | 설명 |
|--------|------|
| `${name}_camera_link` | 카메라 마운트 기준점 (나사 구멍) |
| `${name}_camera_center` | 카메라 하우징 중심 |
| `${name}_left_camera_frame` | 좌안 카메라 프레임 |
| `${name}_left_camera_frame_optical` | 좌안 광학 프레임 (ROS 규약: z-전방, x-우측, y-하방) |
| `${name}_right_camera_frame` | 우안 카메라 프레임 |
| `${name}_right_camera_frame_optical` | 우안 광학 프레임 |

**단안 모델** (zedxonegs, zedxone4k, zedxonehdr):

| 프레임 | 설명 |
|--------|------|
| `${name}_camera_link` | 카메라 마운트 기준점 (나사 구멍) |
| `${name}_camera_center` | 카메라 하우징 중심 |
| `${name}_camera_frame` | 카메라 프레임 |
| `${name}_camera_frame_optical` | 광학 프레임 (ROS 규약: z-전방, x-우측, y-하방) |

**부가 프레임** (조건부):

| 프레임 | 조건 | 설명 |
|--------|------|------|
| `${name}_gnss_link` | `enable_gnss=true` | GNSS 안테나 위치 |
| `${name}_mag_link` | `model=zed2` | 자기 센서 |
| `${name}_baro_link` | `model=zed2` | 기압 센서 |
| `${name}_temp_left_link` | `model=zed2` | 좌측 온도 센서 |
| `${name}_temp_right_link` | `model=zed2` | 우측 온도 센서 |

## 로봇 통합

이 패키지는 독립적으로 카메라 모델만 제공한다. 특정 로봇에 부착하는 통합 설정은 **Integration Layer** 패키지(`neuromeka_integrations` 등)에서 처리한다.

통합 예시는 해당 integrations 패키지의 문서를 참조한다.

## v1.0.0 마이그레이션

v1.0.0에서 커스텀 `stereolabs_zedxm` 매크로가 공식 `zed_camera` 매크로로 교체되었다 (Breaking Change).

주요 변경사항:

- 매크로: `stereolabs_zedxm(prefix, parent_link, mount_config_file)` -> `zed_camera(name, model, ...)`
- xacro 파일: `urdf/zedxm.xacro` -> `urdf/zed_macro.urdf.xacro`
- TF 프레임: 공식 ZED ROS 2 드라이버와 동일한 이름 사용
- Hand-Eye 캘리브레이션 YAML의 `z` 값: 0.016m 감산 필요 (마운트 기준점 변경)

상세 변경 이력은 [CHANGELOG.md](CHANGELOG.md)를 참조한다.

## 라이선스

Apache-2.0
