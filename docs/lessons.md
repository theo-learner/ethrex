# Lessons

## 2026-02-22 - Project Analysis Delivery

### Observed patterns

- Feature-related docs can drift from Cargo manifests when names/defaults evolve.
- Temporary integration workarounds (especially around signing keys and cross-crate exports) tend to persist and become architectural debt.
- Local developer validation paths can diverge from CI-required paths, reducing confidence before push.

### Self-rules

1. Treat `Cargo.toml` manifests as feature source-of-truth and verify docs against them in every architecture/reporting task.
2. Escalate any temporary secret/key workaround as a high-priority risk with explicit removal criteria.
3. Flag TODO-marked public API leaks as boundary issues, not simple refactor notes.
4. Always map local test entrypoints and required CI checks side-by-side to expose validation gaps early.

## 2026-02-22 - Idea-Based Product Evaluation

### Observed patterns

- Product-idea evaluation is more useful when each claim is tied to line-level code evidence and a maturity score.
- "Shared codebase ratio" claims need explicit measurement scope; different path boundaries can change the number materially.
- Latency optimization cannot be inferred from generic peer scoring alone; RTT-level telemetry is a hard prerequisite.

### Self-rules

1. For strategy docs, always include `current strengths`, `gaps`, and `prioritized actions` per idea.
2. When presenting percentage-style claims (e.g., code sharing), always document the counting method and scope.
3. For networking performance claims, require both selection policy evidence and metric instrumentation evidence before concluding readiness.

## 2026-02-22 - 구현 계획 문서화(Ops Agent)

### Observed patterns

- 구현 계획 문서는 아키텍처 설명보다 `실행 단위(컴포넌트/API/플레이북/검증기준)` 중심일 때 구현 이관성이 높다.
- 자동조치 시스템은 기능 목록보다 `위험도 정책`과 `금지 조치`를 먼저 고정해야 안전 요구사항이 흔들리지 않는다.
- 운영 자동화 계획은 단계별 완료 기준(Definition of Done)이 없으면 범위가 쉽게 확장된다.

### Self-rules

1. 자율 운영 계획 작성 시, 반드시 `In/Out Scope`, `Risk policy`, `Playbook`, `Acceptance criteria` 4가지를 최소 세트로 포함한다.
2. 조치 자동화 문서에는 항상 `승인 필요 조건`과 `금지 조치`를 명시한다.
3. 계획 문서의 각 Phase에는 검증 가능한 완료 기준을 한 줄 이상 명시한다.

## 2026-02-22 - 모니터링 통합 전략 문서화

### Observed patterns

- 운영 자동화 문서에서 가장 중요한 결정은 “통합 여부”보다 “배포 경계”다.
- 에이전트 도입 효용은 기능 나열보다 `현재 방식 대비 변화`로 설명해야 전달력이 높다.
- 즉시 자동화 요청이 있어도 단계적 활성화(관찰 전용 → 저위험 → 승인 기반 고위험)를 명시해야 운영 리스크를 줄일 수 있다.

### Self-rules

1. 통합 전략 문서에는 반드시 `역할 경계(시스템별 책임)` 섹션을 포함한다.
2. 자동화 효용을 설명할 때는 최소 3개 이상의 측정 가능한 KPI를 함께 제시한다.
3. 운영 자동화 제안에는 항상 `도입 가드레일`과 `금지 조치`를 함께 명시한다.

## 2026-02-22 - 멀티-Agent 6개월 로드맵 문서화

### Observed patterns

- 장기 로드맵은 목표를 “최종 상태”로만 쓰면 실행성이 떨어지고, 월별 완료 기준이 있어야 구현팀/운영팀 모두 추적 가능하다.
- 멀티-agent 전략 문서에서 핵심은 agent 목록보다 `상호작용 계약(이벤트/상태/합의 규칙)`이다.
- “완전 자율” 목표는 중간 단계(준자율) KPI 없이는 품질을 계량할 수 없다.

### Self-rules

1. 장기 로드맵 문서에는 반드시 `월별 마일스톤 + 완료 기준`을 함께 기록한다.
2. 멀티-agent 설계 문서에는 `공통 이벤트 계약`과 `결정 충돌 해소 규칙`을 포함한다.
3. 최종 비전 문서라도 중간 목표 KPI(예: MTTR, 자동해결률, 오탐률)를 명시한다.
