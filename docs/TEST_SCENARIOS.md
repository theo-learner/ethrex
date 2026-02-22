---
생성일: 2026-02-22
아이디어: Geth에서 ethrex로 안전하게 마이그레이션하는 CLI MVP를 설계하고 싶습니다. 핵심은 데이터 변환, 무결성 검증, 롤백 지원입니다.
버전: 1.0
프로젝트 유형: cli
---

```markdown
# Geth → Ethrex 마이그레이션 CLI MVP 테스트 시나리오

## 1. 단위 테스트 시나리오

| ID       | 대상 기능 | 입력 | 기대 결과 | 우선순위 |
|----------|-----------|------|-----------|----------|
| TC-001   | F-001: Geth 데이터 파싱 (데이터 구조 변환) | Geth `chaindata` 디렉터리 내 정상적인 LevelDB 키-값 쌍 (예: 블록 헤더, 트랜잭션, 상태 트리) | 파서가 정상적으로 블록 헤더, 트랜잭션 목록, 상태 루트를 추출하여 Ethrex 호환 형식(`Block`, `Transaction`, `StateRoot`)으로 변환 | P0 |
| TC-002   | F-002: 무결성 검증 (해시 비교) | 변환된 Ethrex 블록과 원본 Geth 블록의 블록 해시, 상태 루트, 트랜잭션 루트 | 검증 모듈이 모든 해시 값이 동일함을 확인하고 `true` 반환 | P0 |
| TC-003   | F-003: 롤백 기능 (데이터 복원) | 마이그레이션 중 생성된 임시 Ethrex 데이터 디렉터리 + 원본 Geth 백업 디렉터리 | `rollback` 명령 실행 시, 임시 데이터가 완전히 삭제되고 원본 Geth 디렉터리가 복원됨 | P0 |
| TC-004   | F-004: 진행 상황 로깅 (ProgressLogger) | 1000개 블록 변환 중, 500번째 블록 처리 중 | 로그 파일에 `INFO: Processed 500/1000 blocks (50%)` 출력, 타임스탬프 포함, 비동기 안전 | P1 |
| TC-005   | F-005: 에러 처리 - 파싱 실패 (degraded mode) | Geth `chaindata`에 손상된 LevelDB 키 (예: 잘못된 블록 번호, 비정상적인 바이너리 형식) | 파서가 해당 블록을 건너뛰고 `WARN: Skipped corrupted block #1234 (invalid RLP)` 기록, 나머지 블록은 정상 처리 | P0 |
| TC-006   | F-005: 재시도 메커니즘 (RetryHandler) | 네트워크 오류로 인해 외부 상태 저장소(S3/MinIO)에 업로드 실패 3회 | 3회 재시도 후에도 실패 시 `ERROR: Upload failed after 3 retries` 기록, 프로세스 종료 | P1 |
| TC-007   | F-001: 파싱 실패 - 빈 디렉터리 | Geth 데이터 경로가 존재하지만 비어 있음 | `ERROR: No chaindata files found in /path/to/geth/chaindata` 출력, 프로세스 종료 | P1 |
| TC-008   | F-003: 롤백 - 백업 없음 | `--backup-path`가 지정되지 않음 | `ERROR: Backup directory not specified. Rollback impossible.` 출력, 롤백 중단 | P1 |

---

## 2. 통합 테스트 시나리오

| ID       | 대상 기능 | 입력 | 기대 결과 | 우선순위 |
|----------|-----------|------|-----------|----------|
| TC-101   | F-001 + F-002 + F-004 | Geth `chaindata` (100 블록) → 변환 → 무결성 검증 → 로그 생성 | 변환 완료 후, 무결성 검증 성공 (`✓ All hashes match`), 로그 파일에 100개 블록 처리 기록, 총 소요 시간 기록 | P0 |
| TC-102   | F-001 + F-005 + F-004 | Geth `chaindata`에 3개 손상 블록 포함 (총 100 블록) | 97개 블록 정상 변환, 3개 블록 건너뛰고 경고 로그 기록, 무결성 검증은 97개 블록 기준으로 성공 | P0 |
| TC-103   | F-003 + F-004 + F-005 | 마이그레이션 중간에 강제 종료 → `rollback` 실행 | 임시 Ethrex 데이터 삭제, Geth 원본 복원 완료, 로그에 `ROLLBACK INITIATED` 및 복원된 블록 수 기록 | P0 |
| TC-104   | F-005 + F-006 | 외부 스토리지에 접근 불가 (네트워크 차단) | 첫 번째 업로드 시도 실패 → 2회 재시도 → 3회 실패 → `ERROR: Failed to upload metadata after 3 retries` 출력, 프로세스 종료 | P1 |
| TC-105   | F-001 + F-002 + F-003 | 변환 완료 후 무결성 검증 실패 (해시 불일치) | `CRITICAL: Hash mismatch detected on block #50` 출력, 자동 롤백 실행, 원본 Geth 복원 | P0 |

---

## 3. E2E 테스트 시나리오

| ID       | 대상 기능 | 입력 | 기대 결과 | 우선순위 |
|----------|-----------|------|-----------|----------|
| TC-201   | 전체 마이그레이션 흐름 (정상) | 1. Geth 데이터 디렉터리 (1000 블록) <br> 2. `--output-dir=/tmp/ethrex` <br> 3. `--backup-dir=/tmp/geth-backup` | 1. 변환 완료 (5~10분 내) <br> 2. 무결성 검증 성공 <br> 3. 로그 파일에 전체 프로세스 기록 <br> 4. `/tmp/ethrex`에 Ethrex 형식 데이터 생성 <br> 5. 프로세스 종료 코드 0 | P0 |
| TC-202   | 전체 마이그레이션 + 에러 재시도 | 1. Geth 데이터 (1000 블록) <br> 2. 외부 스토리지 임시 장애 (30초) <br> 3. 재시도 설정: 3회, 5초 간격 | 1. 업로드 실패 → 3회 재시도 → 성공 <br> 2. 로그에 `RETRY #1/3`, `RETRY #2/3`, `Upload succeeded on retry #3` 기록 <br> 3. 전체 마이그레이션 성공 | P1 |
| TC-203   | 전체 마이그레이션 + 파싱 실패 (degraded mode) | 1. Geth 데이터 (1000 블록 중 5개 손상) <br> 2. `--degraded-mode=true` | 1. 995개 블록 변환 성공 <br> 2. 손상 블록 건너뛰고 경고 로그 기록 <br> 3. 무결성 검증: 995개 블록 기준 성공 <br> 4. 프로세스 종료 코드 0 (성공) | P0 |
| TC-204   | 전체 마이그레이션 + 롤백 | 1. Geth 데이터 (100 블록) <br> 2. 마이그레이션 중간 (50번째 블록 후) `Ctrl+C` <br> 3. `--rollback` 실행 | 1. 임시 Ethrex 데이터 완전 삭제 <br> 2. Geth 백업 복원 완료 <br> 3. 로그에 `ROLLBACK SUCCESSFUL: Restored 100 blocks` 기록 <br> 4. 원본 Geth 디렉터리 상태 복원 확인 | P0 |
| TC-205   | 전체 마이그레이션 + 무결성 검증 실패 | 1. Geth 데이터 (100 블록) <br> 2. 변환 후 임의로 Ethrex 블록 해시 조작 | 1. 무결성 검증 실패 → `CRITICAL: Hash mismatch on block #42` <br> 2. 자동 롤백 실행 <br> 3. 원본 Geth 복원 <br> 4. 프로세스 종료 코드 1 (실패) | P0 |

---

## 4. 엣지 케이스

| ID       | 대상 기능 | 입력 | 기대 결과 | 우선순위 |
|----------|-----------|------|-----------|----------|
| TC-301   | F-001: Geth 데이터가 0 블록 | 빈 `chaindata` 디렉터리 (파일 존재하지만 블록 없음) | `INFO: No blocks found in source. Nothing to migrate.` 출력, 프로세스 종료 코드 0 | P1 |
| TC-302   | F-005: 파싱 실패 - 블록 번호가 음수 | LevelDB에 `blockNumber: -1`인 블록 존재 | `WARN: Invalid block number -1 detected. Skipping.` 기록, 다음 블록으로 진행 | P1 |
| TC-303   | F-003: 롤백 대상 백업이 손상됨 | 백업 디렉터리 존재하나, 일부 LevelDB 파일이 손상 | `ERROR: Backup data corrupted. Cannot rollback safely.` 출력, 롤백 중단, 원본 유지 | P0 |
| TC-304   | F-004: 로그 파일 권한 없음 | 실행 계정이 로그 디렉터리에 쓰기 권한 없음 | `ERROR: Cannot write log file to /var/log/ethrex-mig. Permission denied.` 출력, 프로세스 종료 | P1 |
| TC-305   | F-006: 재시도 중 시스템 재부팅 | 마이그레이션 중 재부팅 | 재부팅 후 재실행 시 `--resume` 옵션 없음 → `ERROR: Previous session detected but --resume not specified.` | P1 |
| TC-306   | F-001: Geth 데이터가 100GB 이상 | 대용량 데이터 (100GB, 500만 블록) | 메모리 사용량 2GB 이하 유지, 스트리밍 방식으로 처리, 변환 시간 60분 이내, 로그는 1000블록 단위로 진행률 출력 | P0 |
| TC-307   | F-002: 무결성 검증 - 블록 수 불일치 | Geth: 1000블록, Ethrex: 999블록 | `CRITICAL: Block count mismatch: Geth=1000, Ethrex=999. Incomplete migration.` 출력, 롤백 자동 실행 | P0 |
| TC-308   | F-005: degraded mode 비활성화 + 파싱 실패 | `--degraded-mode=false` + 손상 블록 존재 | 프로세스 즉시 중단, `FATAL: Corrupted block detected. Aborting (degraded mode disabled).` 출력, 종료 코드 2 | P0 |

> ✅ **SYSTEM_BASELINE 준수 확인**:  
> 모든 `파싱 실패` 시나리오(TC-005, TC-102, TC-203, TC-302)는 **degraded mode**를 통해 **부분 성공**을 허용하며, 시스템 전체가 다운되지 않고 **최대한의 데이터를 처리**함.  
> 이는 SYSTEM_BASELINE.md의 “*degraded mode: tolerate partial data loss while maintaining operational continuity*” 요구사항을 정확히 충족.
```