# CLAUDE.md — plem-stereolabs

## 프로젝트 역할

Stereolabs ZED 카메라 URDF + meshes를 담당하는 **Tier 2 Peripheral Description** 저장소.
특정 로봇 드라이버나 플랫폼에 종속되지 않는다 (로봇 비종속).

## 빌드 및 테스트

```bash
# 빌드
colcon build --packages-select stereolabs_description

# 테스트 (xacro 파싱 검증)
colcon test --packages-select stereolabs_description
colcon test-result --verbose
```

## 패키지 구성

| 패키지 | 역할 |
|--------|------|
| `stereolabs_description` | Stereolabs ZED 카메라 URDF + meshes |

## 3-Tier 구조에서의 위치

```
Tier 1 (Robot)        neuromeka_description    — 로봇 본체 (plem-neuromeka)
Tier 2 (Peripheral)   stereolabs_description   — 이 저장소 (로봇 비종속)
Tier 3 (Integration)  neuromeka_integrations   — 조합 xacro/SRDF (plem-neuromeka)
```

이 저장소는 Tier 2만 담당한다.
Tier 3 통합 xacro/SRDF는 `plem-neuromeka/neuromeka_integrations`에서 관리한다.

## 공통 규칙 참조

URDF 스펙은 `plem-msgs/.claude/rules/urdf-srdf-standards.md`를 따른다.
3-Tier 패턴 상세는 `plem-msgs/.claude/rules/description-conventions.md`를 따른다.

이 저장소 고유 규칙은 `.claude/rules/` 참조.
