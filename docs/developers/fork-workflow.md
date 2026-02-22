# Fork 기반 작업 워크플로우

이 문서는 아래 구조로 작업할 때의 표준 흐름을 정의한다.

- `origin`: `https://github.com/theo-learner/ethrex.git` (내 포크)
- `upstream`: `https://github.com/lambdaclass/ethrex.git` (원본)

## 1. 최초 1회 설정

이미 클론한 저장소에서 다음 순서로 리모트를 정리한다.

```bash
git remote rename origin upstream
git remote add origin https://github.com/theo-learner/ethrex.git
git fetch --all
git branch --set-upstream-to=origin/main main
git config remote.pushDefault origin
```

검증:

```bash
git remote -v
```

예상 결과:

- `origin` -> `theo-learner/ethrex`
- `upstream` -> `lambdaclass/ethrex`

## 2. 일상 작업 흐름

```bash
git switch main
git pull --ff-only origin main
git switch -c feat/<topic>
# 작업
git add -A
git commit -m "docs(l1): ..."
```

이후 푸시는 `git push` 대신 아래 3장의 `git push-theo`를 사용한다.

## 3. push 시 upstream 최신 반영 자동화 (권장)

`git push-theo` alias를 등록하면, push 직전에 자동으로 `upstream/main`을 반영한다.

```bash
git config alias.push-theo '!f(){ \
set -e; \
branch=$(git branch --show-current); \
if [ -z "$branch" ]; then echo "Detached HEAD 상태입니다."; exit 1; fi; \
git fetch upstream main; \
if [ "$branch" = "main" ]; then \
  git merge --ff-only upstream/main; \
else \
  git rebase upstream/main; \
fi; \
git push -u origin "$branch"; \
}; f'
```

사용:

```bash
git push-theo
```

동작 요약:

1. `upstream/main` fetch
2. 현재 브랜치가 `main`이면 `ff-only merge`
3. feature 브랜치면 `rebase upstream/main`
4. `origin`(theo 포크)로 push

## 4. 보호 가드 (선택)

실수로 `git push`를 쓰지 않게 하려면 shell alias를 추가한다.

```bash
alias git-push='echo "Use git push-theo"'
```

또는 팀 단위로 `.githooks/pre-push`를 운영해 `upstream/main` 미반영 브랜치 push를 차단할 수 있다.

## 5. 원본 동기화(main 유지)

포크의 `main`을 주기적으로 원본과 동기화한다.

```bash
git switch main
git fetch upstream
git merge --ff-only upstream/main
git push origin main
```

## 6. 충돌 시 처리 원칙

1. feature 브랜치 충돌:
- `git rebase upstream/main` 중 충돌 해결 후 `git rebase --continue`

2. 이미 원격에 올린 브랜치 rebase 후 push:
- `git push --force-with-lease`

3. `main`은 force push 금지:
- 항상 `--ff-only` 기반으로 유지

## 7. 추천 체크리스트

push 전:

1. `git status`가 clean인지 확인
2. `git push-theo` 사용
3. PR 생성 전 CI/lint 결과 확인

이 규칙을 지키면 `theo` 포크 작업 속도와 `upstream` 동기 정확도를 동시에 유지할 수 있다.
