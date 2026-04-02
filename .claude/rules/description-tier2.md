---
description: "Tier 2 description rules for Stereolabs peripherals — vendor-agnostic URDF, no plem/neuromeka dependencies, {vendor}_{model} macro naming"
paths:
  - "**/*_description/**"
---
# Description Tier 2 Rules — stereolabs_description

스코프: 이 저장소 전체

## Tier 2 핵심 원칙

**로봇 벤더 비종속**: 이 패키지는 특정 로봇 드라이버나 플랫폼에 의존하지 않는다.

- `package.xml`에 `plem_*`, `neuromeka_*`, `doosan_*` 의존성을 추가하지 않는다.
- 허용 의존성: ROS2 기본 패키지(`urdf`, `xacro`, `ament_cmake`)만.

## 매크로 인터페이스

### 네이밍 규칙

- 매크로 이름: `{vendor}_{model}` (예: `stereolabs_zed_x_mini`, `stereolabs_zed_2i`)
- 파일명: `{model}.urdf.xacro` (예: `zed_x_mini.urdf.xacro`)

### 필수 파라미터

모든 매크로는 다음 파라미터를 가진다:

```xml
<xacro:macro name="stereolabs_zed_x_mini" params="prefix:=''">
  <!-- prefix: 로봇 네임스페이스 prefix, 기본값 빈 문자열 -->
</xacro:macro>
```

- `prefix`는 모든 링크/조인트 이름에 적용: `${prefix}zed_x_mini_base_link`
- `prefix`에 기본값 `''`을 명시한다.

## meshes 구조

```
meshes/{model}/
├── visual/
│   └── {part}.dae        # 시각화용 (컬러 있음)
└── collision/
    └── {part}.stl         # 충돌 검사용 (단순화)
```

## 테스트 요구사항

모든 xacro 파일에 대해:
- `xacro {file}` 파싱 성공 테스트 필수
- `prefix:=test_` 파라미터로 네임스페이스 테스트 필수

## 버전 관리

- SemVer를 따른다. 새 모델 추가: patch 범프. 매크로 인터페이스 변경: minor 범프.
- URDF 스펙 상세는 wim_control Rule 13(urdf-standards)을 따른다.

## 패키지 구조 (최소 구성)

Tier 2 패키지는 빌드 로직을 최소화한다:

### CMakeLists.txt 템플릿

```cmake
cmake_minimum_required(VERSION 3.8)
project(stereolabs_description)

find_package(ament_cmake REQUIRED)

install(DIRECTORY meshes urdf
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
```

### package.xml 최소 의존성

```xml
<package format="3">
  <name>stereolabs_description</name>
  <version>0.1.0</version>
  <description>Stereolabs ZED camera URDF and meshes</description>
  <maintainer email="dev@wimcorp.kr">WIM Corporation</maintainer>
  <license>Proprietary</license>

  <buildtool_depend>ament_cmake</buildtool_depend>
  <exec_depend>urdf</exec_depend>
  <exec_depend>xacro</exec_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

`plem_*`, `neuromeka_*` 의존성은 절대 추가하지 않는다.

## mesh 파일 규격

```
meshes/{model}/
├── visual/
│   └── {part}.dae    # 시각화용. 컬러 텍스처 포함 가능.
└── collision/
    └── {part}.stl    # 충돌 검사용. < 1000 faces로 단순화.
```

- mesh URI는 반드시 `package://stereolabs_description/meshes/...` 형식 사용
- DAE 파일의 단위는 미터(REP-103)
- STL은 ASCII 포맷 권장 (바이너리도 허용)
