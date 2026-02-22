# Ethrex Branch-Scoped Rules (`feat/ops-agent` only)

## 0) Scope

이 문서의 규칙은 **`feat/ops-agent` 브랜치 작업 컨텍스트에서만** 적용한다.  
`main` 또는 다른 일반 기능 브랜치의 전역 규칙으로 간주하지 않는다.

## 1) Implementation Purpose (Do Not Drift)

이 프로젝트에서 우선순위가 가장 높은 목적은 다음과 같다.

1. `ethrex` L1/L2 운영 이슈를 자동으로 탐지/판단/조치하는 **자율 운영 AI agent**를 구축한다.
2. 운영 자동화는 속도보다 안전을 우선하며, **저위험 자동조치 + 고위험 승인 기반 조치**를 기본 모델로 한다.
3. 장기적으로는 기능별 agent가 상호작용하는 **멀티-agent 준자율 운영**으로 확장한다.

모든 코드/문서 변경은 위 목적과의 연관성을 명확히 가져야 한다.

## 2) Source of Truth

아래 문서는 자율 운영 agent 관련 공식 기준 문서다.

1. `docs/ops-agent/implementation-plan.md`
2. `docs/ops-agent/monitoring-integration.md`
3. `docs/ops-agent/roadmap-6months.md`

구현 방향 충돌 시 위 문서를 먼저 수정해 기준을 정렬한 뒤 코드 변경을 진행한다.

## 3) Branch Change Rules

`feat/ops-agent` 브랜치 내 변경(PR/커밋)은 아래 규칙을 따른다.

1. **Goal Alignment**
- 변경 사항이 자율 운영 목적에 어떻게 기여하는지 설명한다.
- 직접 연관이 없으면, 최소한 운영 안전성/관측성/복구성 개선과 연결되어야 한다.

2. **Safety First**
- 파괴적 조치(데이터 삭제, DB 직접 변경, 무승인 고위험 액션)는 기본 금지.
- 자동조치 코드에는 precheck/postcheck/rollback 또는 escalation 경로를 포함한다.

3. **Observability Required**
- 운영 관련 변경은 메트릭/로그/상태 이벤트 중 최소 1개 이상에서 추적 가능해야 한다.
- 장애 원인 추적에 필요한 컨텍스트(incident/action/audit)가 누락되면 merge하지 않는다.

4. **Approval Policy**
- 고위험 조치 경로는 승인 게이트를 통과해야 한다.
- 현재 기본 운영 채널은 텔레그램 봇/디스코드로 간주한다.

5. **Docs Sync**
- 정책/인터페이스/운영 흐름 변경 시 `docs/ops-agent/*.md`를 함께 갱신한다.
- 변경 후 `docs/todo.md`, `docs/lessons.md`에 작업/학습 기록을 남긴다.

## 4) Required PR Checklist

PR 설명에 아래 항목을 반드시 포함한다.

1. 이번 변경이 자율 운영 목적에 기여하는 방식
2. 영향 범위: 탐지/판단/조치/승인/감사 중 어디가 바뀌는지
3. 안전성 영향: 고위험 조치 여부, 승인/롤백 영향
4. 관측성 영향: 새 메트릭/로그/이벤트 여부
5. 문서 반영 여부: `docs/ops-agent/*`, `docs/todo.md`, `docs/lessons.md`

## 5) 6-Month Execution Guardrail

중장기 구현은 `docs/ops-agent/roadmap-6months.md`를 기준으로 한다.

1. Month 1-3: 기반 플랫폼 + telemetry/diagnosis/planning/execution
2. Month 4-6: approval/governance + 멀티-agent 협업 고도화

새 작업은 가능하면 월별 마일스톤 중 어디에 속하는지 매핑한다.

## 6) Conflict Resolution Rule

요구사항이 충돌할 때 우선순위는 아래와 같다.

1. 운영 안전성
2. 데이터 정합성
3. 관측 가능성
4. 자동화 속도

자동화 편의보다 안전성과 복구 가능성을 우선한다.
