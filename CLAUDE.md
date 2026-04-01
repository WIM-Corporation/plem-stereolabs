# CLAUDE.md — stereolabs_description

## 프로젝트 역할

Stereolabs ZED 카메라 URDF + meshes. **로봇 벤더에 비종속**인 Tier 2 주변기기 description.
다른 로봇 벤더(Neuromeka, Doosan 등)에서 동일 패키지를 참조할 수 있다.

## 빌드 및 테스트

```bash
colcon build --packages-select stereolabs_description
colcon test --packages-select stereolabs_description
colcon test-result --verbose
```

## 새 모델 추가 절차

1. `urdf/{model}.urdf.xacro` 작성 — `stereolabs_{model}` 매크로 이름 규칙 준수
2. `meshes/{model}/` 아래 STL/DAE 배치
3. `test/test_xacro.py`에 xacro 파싱 테스트 추가
4. `package.xml` 버전 범프 (SemVer patch)

## 공통 규칙 참조

URDF 스펙은 wim_control Rule 13(urdf-standards)을 따른다.
이 저장소 고유 규칙은 `.claude/rules/` 참조.
