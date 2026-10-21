---
name: commit-message
description: "스테이지된 변경을 바탕으로 Conventional Commits 형식의 커밋 메시지를 2인 팀(commit-msg-author·commit-msg-reviewer 서브에이전트)으로 생성한다. '커밋 메시지', 'commit message', '커밋 메시지 만들어줘' 같은 요청 시 사용한다. 이미 메시지가 정해진 `git commit -m` 실행에는 쓰지 않는다."
---

# Commit Message Skill

두 서브에이전트를 순서대로 호출해 Conventional Commits 형식의 커밋 메시지를 생성한다. reviewer는 author가 쓴 파일을 읽으므로 둘을 병렬로 호출하지 않는다.

## Workflow

1. **Precondition 체크.**
   `git diff --cached --quiet`를 실행한다.
   exit code 1(변경 있음)이면 통과, exit code 0(변경 없음)이면 "먼저 `git add`" 안내 후 종료한다.
2. **author 호출.**
   Task 도구로 `commit-msg-author` 서브에이전트를 포그라운드로 호출한다.
   출력은 `_workspace/commit-draft.md`.
3. **reviewer 호출.**
   Task 도구로 `commit-msg-reviewer` 서브에이전트를 포그라운드로 호출한다.
   출력은 `_workspace/review-report.md`.
4. **판정 분기.**
   - PASS. `_workspace/commit-draft.md` 내용을 사용자에게 제시하고 완료한다.
   - REDO. reviewer의 수정 지시를 프롬프트에 넣어 author를 다시 호출한 뒤 3단계로 돌아간다(최대 2회).
5. **루프 종료.**
   PASS 또는 재호출 2회 초과 시 종료한다.
   초과 시 "자동 승인 한계 도달 — 수동 검토 권장" 경고와 함께 마지막 draft를 반환한다.

## 서브에이전트 프롬프트

서브에이전트는 이 대화를 보지 못한다. 호출할 때 다음을 프롬프트에 넣는다.

- 작업 디렉터리(저장소 루트)와 입출력 파일 경로
- 재호출이면 몇 번째 시도인지와 reviewer 수정 지시 전문
- 최종 응답으로 작성한 파일 전문을 반환하라는 요청

## 주의

- 커밋은 실행하지 않는다. 메시지만 제시하고, 커밋 여부는 사용자가 정한다.
- `_workspace/`는 `.gitignore`에 포함되어 있다.
