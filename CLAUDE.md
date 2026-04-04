# CLAUDE.md — plem-stereolabs

## 프로젝트 역할

Stereolabs ZED 카메라 URDF + meshes를 담당하는 **Description Layer** 패키지.
공식 `zed-ros2-wrapper`의 `zed_macro.urdf.xacro`를 벤더링하여 ZED SDK 의존 없이 제공한다.

## 벤더링 패키지

이 패키지는 자체 매크로가 아닌 **공식 매크로의 벤더링 복사본**이다.
유일한 수정: 메시 경로 `$(find zed_msgs)` → `$(find stereolabs_description)`.
출처 및 동기화 이력: `VENDOR_SOURCE.md` 참조.

## 빌드 및 테스트

```bash
colcon build --packages-select stereolabs_description
colcon test --packages-select stereolabs_description
colcon test-result --verbose
```

## 2-Layer 구조에서의 위치

```
Description Layer    neuromeka_description    — 로봇 본체 (자체 작성)
                     onrobot_description      — 그리퍼 (자체 작성)
                     stereolabs_description   — 이 저장소 (공식 벤더링)
Integration Layer    neuromeka_integrations   — 조합 xacro/SRDF (plem-neuromeka)
```

## 공통 규칙 참조

URDF 스펙은 `plem-msgs/.claude/rules/urdf-srdf-standards.md`를 따른다.
Description/Integration 패턴은 `plem-msgs/.claude/rules/description-conventions.md`를 따른다.
이 저장소 고유 규칙은 `.claude/rules/` 참조.
