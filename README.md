# Flight Control Actuator Unit

STM32로 항공기 조종면 구동기를 축소 모사하는 **실물 제어 학습 프로젝트**입니다. 명령 위치를 받아 모터를 구동하고, 위치 피드백으로 목표 위치를 추종하며, 센서·통신·전원 이상에서 출력을 제한하는 과정을 다룹니다.

> **Status: planned.** 저장소 생성 단계입니다. 아래 구성과 검증 항목은 목표이며, 구현·실측 결과가 아닙니다. 실제 보드, 모터, 구동 회로 및 전원 사양은 착수 시 확정합니다. 이 장치는 실제 항공기용 제어기나 비행 인증 장비가 아닙니다.

## Why this project

[radar-track-fcc-hil](https://github.com/SungJaeYu/radar-track-fcc-hil)에서 STM32 + Zephyr로 센서 입력, UART, 인터럽트 및 추적 처리를 학습하고 있습니다. 이 프로젝트는 그 다음에 **MCU 레지스터 → 타이머/PWM → 모터 드라이버 → 구동기 → 위치 피드백**을 실물로 연결해 보는 작은 후속 과제입니다. 동일한 HIL의 추적 기능을 다시 만들지 않고, 구동 출력과 하드웨어 통합에 집중합니다.

## Intended system

```text
PC command / test tool
        │ serial command + telemetry
        ▼
STM32 (CMSIS device headers, register-level peripheral setup)
  ├─ command validation and timeout
  ├─ periodic position-control loop
  ├─ timer / PWM output
  ├─ position feedback input
  └─ current/fault monitoring where hardware supports it
        │
        ▼
low-voltage motor driver → small actuator / motor
        ▲                     │
        └── position feedback ┘
```

첫 구현은 검증된 저전압 모터 드라이버 모듈과 저출력 모터를 사용합니다. 3상 인버터나 항공기 수준의 조종면 구동은 범위에 넣지 않습니다. PCB는 배선·전원·보호회로와 측정 요구가 정리되고 브레드보드/모듈 기반 구동이 안정화된 후 선택적으로 설계합니다.

## Scope

- STM32의 clock, GPIO, timer/PWM, interrupt, UART 및 필요한 feedback 입력을 레지스터 수준에서 설정합니다. CMSIS/device header는 사용하며 STM32 HAL과 RTOS는 핵심 구동 경로에서 사용하지 않는 것을 목표로 합니다.
- 정해진 주기로 명령 위치와 측정 위치를 비교하고, 제한된 출력 범위에서 제어 명령을 갱신합니다. 위치 센서는 모터/기구 선택 후 encoder 또는 potentiometer 중 적절한 것을 고릅니다.
- 명령 범위 확인, 명령 timeout, 센서값 이상, 과전류 감지 가능 시 과전류 대응, 출력 제한 및 fault 기록을 구현합니다. watchdog 적용 여부와 재시동 동작은 실제 회로에 맞춰 결정합니다.
- PC 시험 도구는 명령과 측정값, 제어 출력, 상태 및 fault를 기록합니다. 실제 위치 단위는 기구·센서 보정 이후 정합니다.
- 목표 위치 step 응답, 정상 추종, 명령 중단, 센서 이상, 물리적 구동 범위 및 주기 흔들림을 재현 가능한 조건에서 측정합니다.

## Milestones

- [ ] **M0 — Hardware and interface:** 보드/모터/드라이버/센서/전원을 확정하고 핀, 전압, 전류, 기계적 제한과 시험 절차를 기록한다.
- [ ] **M1 — Peripheral bring-up:** UART, 주기 timer, PWM, feedback 입력을 각각 확인하고 파형과 측정 로그를 남긴다.
- [ ] **M2 — Closed-loop motion:** 안전한 출력 제한에서 위치 제어를 수행하고 목표/실제 위치 및 step 응답을 기록한다.
- [ ] **M3 — Fault handling:** 명령 timeout과 센서 오류 시 출력을 안전한 상태로 전환하고 복구 조건을 문서화한다.
- [ ] **M4 — Reproducibility:** 배선도·빌드/플래시 절차·시험 시나리오·측정 결과와 한계를 공개 가능한 범위에서 정리한다.
- [ ] **Optional — PCB:** 모듈 기반 원형 검증 후 필요가 확인되면 회로도/PCB 설계와 bring-up을 진행한다.

## Evidence to keep

보드와 부품 모델, 클럭 및 타이머 계산, PWM 주파수/분해능, 제어 주기와 jitter, 명령 대비 실제 위치 그래프, step 응답의 overshoot/settling time, timeout·센서 이상 시 상태 전이, 소스 코드와 시험 로그를 남깁니다. 목표 수치는 측정 전에 정하며, 측정 결과와 목표를 구분합니다.

## Relationship to the portfolio

이 저장소는 P1 센서 추적 HIL 다음에 **레지스터와 실물 actuator의 연결**을 학습하는 제한된 범위의 후속 프로젝트입니다. 기존 P2/P3 항공 AI 대표작의 완료나 우선순위를 대신하지 않습니다. 구현 전에는 이력서의 완료 실적으로 쓰지 않습니다.
