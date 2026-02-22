# Ethrex 아이디어 기반 평가 (RaaS / Native L2 / Latency Routing)

일자: 2026-02-22

## 평가 범위

이 문서는 다음 3가지 제품 아이디어 관점에서 현재 `ethrex`를 평가합니다.

1. Rollup-as-a-Service Client
2. Native L2 Integration Client
3. Latency-Optimized Routing Client

분석은 저장소 정적 점검(코드/문서/CLI 옵션) 기반이며, 성능 벤치마크와 프로덕션 트래픽 실측은 포함하지 않습니다.

## 한눈에 보는 결론

| 아이디어 | 현재 적합도 (5점) | 상태 판단 |
| --- | --- | --- |
| Rollup-as-a-Service Client | 4.0 | 핵심 토대는 이미 강함. 운영 템플릿/자동화 계층이 필요 |
| Native L2 Integration Client | 4.5 | 사실상 구현되어 있음. 운영 분리/보안 가드 보강 필요 |
| Latency-Optimized Routing Client | 2.0 | 기본 peer scoring은 있으나 latency-최적화 체계는 미구현 |

## 1) Rollup-as-a-Service Client 평가

### 현재 강점

- 빠른 기동 UX가 이미 존재합니다.
  - `ethrex l2 --dev` 한 번으로 로컬 L2를 띄우는 경로가 문서화되어 있습니다: `docs/getting-started/README.md:77`.
  - L2 소개 문서에서도 "just a command" 형태로 빠른 시작을 강조합니다: `docs/l2/introduction.md:9`.
- 실제 dev 경로도 L1 초기화 -> 컨트랙트 배포 -> L2 초기화를 자동으로 수행합니다: `cmd/ethrex/l2/command.rs:78`.
- 배포 모드 스펙트럼(centralized/validium/based)이 이미 준비되어 있습니다.
  - 배포 개요: `docs/l2/deployment/overview.md:3`
  - vanilla 배포: `docs/l2/deployment/vanilla.md:11`
  - validium 배포: `docs/l2/deployment/validium.md:11`
  - based contracts / shared bridge router 옵션: `cmd/ethrex/l2/deployer.rs:316`, `cmd/ethrex/l2/deployer.rs:351`
- 단일 바이너리 + feature 게이트 구조로 공통 코드 재사용이 큽니다: `cmd/ethrex/Cargo.toml:82`.
- 정적 LOC 기준 러프 측정으로도 non-L2 비중이 높습니다.
  - 측정값: `TOTAL_RS_LOC=133485`, `L2_RS_LOC(=crates/l2 + cmd/ethrex/l2)=20011`
  - 해석: 해당 분류 기준으로 non-L2가 약 85.0%로, "대부분 코드 공유" 방향성과 일치

### 현재 갭

- "클릭 몇 번" 수준의 완전 관리형 RaaS UX는 아직 부족합니다.
  - 현재는 CLI 중심이며, 테넌트/환경별 프로파일 템플릿과 정책 가드 자동화가 약합니다.
- 배포 후 운영 자동화(모니터링, 키관리, failover, 업그레이드 워크플로우)를 제품화한 계층이 없습니다.
- 멀티테넌트 관점의 표준 구성 산출물(`config bundle`, `ops runbook`, `SLO`)이 없습니다.

### 권고 우선순위

1. `ethrex l2 deploy` 상단에 `--profile <saas-dev|saas-staging|saas-prod>` 계층을 추가
2. 템플릿 생성기(`genesis`, signer, watcher, prover 조합)를 고정된 스키마로 제공
3. 배포 산출물을 JSON manifest로 표준화해 UI/컨트롤 플레인이 호출하기 쉽게 정리

## 2) Native L2 Integration Client 평가

### 현재 강점

- L1/L2가 단일 클라이언트 표면에 통합되어 있습니다.
  - `ethrex` 바이너리에 `l2` 서브커맨드가 내장: `cmd/ethrex/cli.rs:471`
  - L2 관련 feature 묶음이 동일 바이너리에 연결: `cmd/ethrex/Cargo.toml:111`
- L2 운영 필수 요소(bridge/watcher/committer/proof coordinator)가 같은 옵션 표면에서 제어됩니다.
  - watcher/bridge/router/l2-rpcs 옵션: `cmd/ethrex/l2/options.rs:384`
  - committer/proof coordinator signer 옵션: `cmd/ethrex/l2/options.rs:569`, `cmd/ethrex/l2/options.rs:699`
- sequencer 런타임이 통합 orchestration 형태로 구성되어 있습니다.
  - `L1Watcher`, `L1Committer`, `ProofCoordinator`, `L1ProofSender`, `BlockProducer`, `Admin API`를 함께 기동: `crates/l2/sequencer/mod.rs:111`, `crates/l2/sequencer/mod.rs:221`
- 운영 제어/상태 관찰용 Admin API가 이미 제공됩니다: `docs/l2/admin.md:1`.
- Shared Bridge 문서 기준으로 다중 L2 상호작용을 watcher가 흡수하는 설계가 명시됩니다: `docs/l2/fundamentals/shared_bridge.md:21`, `docs/l2/fundamentals/shared_bridge.md:36`.

### 현재 갭

- 통합은 강하지만, 운영 분리(deploy unit 분해) 관점은 아직 제한적입니다.
  - 구성요소가 단일 프로세스 내에서 강결합되어 장애 격리가 어렵습니다.
- 운영 안전 기본값이 충분히 엄격하지 않은 구간이 있습니다.
  - 예: dev 편의 기본 signer 경로가 존재하며, 운영 환경 가드 강화를 더 해야 합니다.
- 옵션 표면이 크고 복합적이라 운영자 학습곡선이 높습니다.

### 권고 우선순위

1. `integrated` / `split-services` 실행 프로파일을 공식화
2. non-dev 실행 시 signer/bridge 핵심 옵션 fail-fast validation 강화
3. `l2` 옵션을 역할별(`watcher`, `committer`, `prover`) 설정 파일로 분리 가능하게 설계

## 3) Latency-Optimized Routing Client 평가

### 현재 강점

- peer 선택의 기본 점수화 메커니즘은 이미 존재합니다.
  - `score`와 `inflight requests`를 가중해 best peer 선택: `crates/networking/p2p/discv4/peer_table.rs:633`, `crates/networking/p2p/discv4/peer_table.rs:646`
  - discv5도 동일한 구조: `crates/networking/p2p/discv5/peer_table.rs:694`, `crates/networking/p2p/discv5/peer_table.rs:707`
- sync 경로에서 성공/실패 기반 peer score 보정이 들어갑니다: `crates/networking/p2p/sync/snap_sync.rs:643`, `crates/networking/p2p/sync/snap_sync.rs:666`.

### 핵심 한계

- "latency-aware routing"의 필수인 RTT 기반 선택이 없습니다.
  - lookup/random 선택은 랜덤 중심: `crates/networking/p2p/discv4/peer_table.rs:704`, `crates/networking/p2p/discv4/peer_table.rs:824`
  - discv5도 동일: `crates/networking/p2p/discv5/peer_table.rs:765`, `crates/networking/p2p/discv5/peer_table.rs:936`
- metrics에 ping count/rate는 있지만 latency 분포(예: RTT histogram) 노출이 없습니다: `crates/networking/p2p/metrics.rs:618`.
- 네트워킹 문서도 discovery/startup/revalidation 개선 여지가 크다고 명시합니다: `docs/l1/fundamentals/networking.md:12`, `docs/l1/fundamentals/networking.md:47`, `docs/l1/fundamentals/networking.md:72`.

### 권고 우선순위

1. RTT/EWMA latency를 peer state에 추가하고 selection weight에 반영
2. 지역 편향 완화를 위해 "low-latency + diversity(ASN/region)" 하이브리드 정책 도입
3. `p50/p95 latency`, `cross-region fairness` 메트릭을 수집해 회귀 추적

## 제안 실행 로드맵

### Phase 1 (2-4주)

1. RaaS 프로파일(`saas-dev/staging/prod`)과 배포 manifest 스키마 확정
2. non-dev 보안 가드(fail-fast signer/bridge validation) 강화
3. P2P latency metric(RTT) 수집 추가

### Phase 2 (4-8주)

1. Native L2 split-services 실행 모드 도입
2. 템플릿 생성기와 운영 기본 runbook 제공
3. latency-aware peer selection 알고리즘 실험(A/B)

### Phase 3 (8주+)

1. 컨트롤 플레인/API 기반 RaaS 자동화
2. 글로벌 라우팅 정책(지역 균등화 포함) 고도화
3. 성능/가용성 SLO 기반 릴리즈 게이트 정착

## 최종 판단

- 아이디어 1(RaaS)과 2(Native L2 통합)는 `ethrex`가 이미 강한 기반을 가지고 있어 제품화 단계로 전환할 수 있습니다.
- 아이디어 3(Latency 최적 라우팅)는 현재 상태가 "기본 peer scoring" 수준이므로, 계측/정책/검증 체계를 먼저 구축해야 합니다.
- 따라서 단기 사업화 관점에서는 1, 2를 우선하고 3은 병행 R&D 트랙으로 두는 전략이 현실적입니다.

## 가정 및 한계

- 평가 시점은 2026-02-22 저장소 기준입니다.
- 실행/지연 성능 수치는 실측 벤치마크가 아니라 코드/문서 구조 기반 추론입니다.
