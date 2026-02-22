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

## 2026-02-22 - Fork 워크플로우 문서화

### Observed patterns

- 포크 기반 협업 문서는 “리모트 명명 규칙(origin/upstream)”을 먼저 고정해야 혼선을 줄일 수 있다.
- push 전 자동 동기화는 pre-push 훅보다 명시적 alias 커맨드가 실패 원인 파악이 쉽다.
- `main` 보호 규칙(`ff-only`, no force push)을 문서에 포함해야 팀 전체 히스토리 안정성이 올라간다.

### Self-rules

1. 포크 워크플로우 문서에는 반드시 `origin/upstream 역할`을 명시한다.
2. 자동화 명령 제안 시, 실패 시 복구 방법(충돌/force-with-lease 조건)까지 함께 적는다.
3. 브랜치 보호 원칙(main no force push, feature branch rebase)을 체크리스트에 포함한다.

## 2026-02-23 - 운영 채널 변경 문서화

### Observed patterns

- 운영 채널 변경은 정책 섹션만 수정하면 누락이 발생하고, 테스트/리스크/단계별 계획 섹션까지 일관 변경이 필요하다.
- 채널 변경 후 잔존 키워드 검색(`rg`)으로 검증하면 문서 불일치를 빠르게 줄일 수 있다.

### Self-rules

1. 운영 채널 변경 시 `정책`, `구현 단계`, `테스트`, `리스크` 네 섹션을 함께 점검한다.
2. 용어 치환 작업 후에는 반드시 키워드 검색으로 잔존 문구를 확인한다.

## 2026-02-23 - 전역 규칙(AGENTS.md) 도입

### Observed patterns

- 자율 운영 같은 장기 목표는 문서가 있어도 전역 규칙이 없으면 일상 코드 변경에서 쉽게 희석된다.
- 구현 목적과 안전 정책을 PR 체크리스트 수준으로 강제해야 목적 드리프트를 줄일 수 있다.
- 기여 문서와 전역 규칙 문서가 분리되면 발견성이 떨어지므로 상호 링크가 필요하다.

### Self-rules

1. 장기 전략 목표가 있는 프로젝트는 루트 전역 규칙 파일로 목적/금지사항/검증 항목을 명문화한다.
2. 전역 규칙 추가 시 반드시 `CONTRIBUTING.md`에 참조 링크를 추가한다.
3. 전역 규칙에는 최소한 `목적`, `안전 규칙`, `문서 동기화 규칙`, `PR 체크리스트`를 포함한다.

## 2026-02-23 - 규칙 스코프 교정(전역 -> 브랜치 한정)

### Observed patterns

- 실험성/특정 기능 브랜치 정책을 전역 규칙으로 표현하면 협업자 오해를 유발한다.
- 규칙 문서 도입 후에는 적용 범위(scope)를 제목과 본문에 모두 명시해야 한다.

### Self-rules

1. 브랜치 전용 정책은 제목에서 `Branch-Scoped`를 명시한다.
2. `CONTRIBUTING.md`에는 정책 적용 대상을 조건부로 적는다.

## 2026-02-23 - 브랜치명 표기 정합성

### Observed patterns

- 브랜치 스코프 규칙 문서에서 브랜치명이 실제 브랜치와 다르면 규칙 신뢰도가 떨어진다.

### Self-rules

1. 브랜치 한정 규칙 문서 작성 시 실제 `git branch --show-current` 값과 문구를 맞춘다.
