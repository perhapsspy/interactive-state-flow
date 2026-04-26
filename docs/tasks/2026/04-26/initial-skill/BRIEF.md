# Initial Skill

## Goal

- `interactive-state-flow`를 독립 Agent Skill 리포로 초기 세팅하고 첫 shipped skill을 만든다.
- 첫 작업 이후 새 스레드가 이어받을 수 있도록 설계 판단, 조사 근거, 현재 상태를 repo-local 문서로 남긴다.

## Scope

- 새 리포의 공개 README, shipped skill, 한국어 리뷰 페어, repo-local 작성 원칙, project-context 작업 맥락을 세팅한다.
- 기존 `structure-first`, `project-context`와 철학은 맞추되 새 `SKILL.md` 내부에는 다른 스킬 참조나 의존성을 두지 않는다.
- 프레임워크별 최적화 문서가 아니라 플랫폼 중립적인 인터랙티브 상태 흐름 구조 스킬로 정리한다.

## Current Understanding

- 핵심 명제는 "사용자 의도와 기준 상태는 즉시 기록하고, 비용이 큰 표현·계산·비동기 작업은 지연·분리·취소하며, 결과는 여전히 유효하고 유용할 때만 반영한다"이다.
- 공개 skill 포맷은 `SKILL.md` frontmatter의 `name`/`description`과 간결한 Markdown 실행 지침이 중요하다.
- 기존 리포 관행상 README는 소개와 설치 진입점, `SKILL.md`는 실행 계약, `docs/reference/**`는 현재 믿을 기준 맥락, task logs는 판단과 작업 이력을 맡는다.

## Current State

- 리포는 `interactive-state-flow`로 새로 생성되었고, `skills/interactive-state-flow` 골격은 skill-creator 스크립트로 만들었다.
- 첫 shipped skill과 한국어 페어, README 한/영 문서, AGENTS, LICENSE, origin reference가 작성된 상태다.
- 외부 공식/준공식 문서 조사와 리포 세팅 판단은 task-local `SETUP-NOTES.md`에 있다.
- ChatGPT 대화에서 나온 문제 정의와 보존할 의도는 `docs/reference/origin.md`에 별도 요약했다.
- skill-creator quick validation, project-context runtime shape, `SKILL.md` cross-reference/미완료 표식 확인은 통과했다.
- 실제 배포 전 README의 임시 설치 문구를 공개 설치 안내로 바꾸고, 기존 공개 리포 관례에 맞춰 support 섹션을 추가했다.

## Working Boundary

- `skills/interactive-state-flow/SKILL.md`
- `skills/interactive-state-flow/SKILL.ko.md`
- `README.md`
- `README.en.md`
- `docs/reference/origin.md`
- `docs/tasks/2026/04-26/initial-skill/SETUP-NOTES.md`

## Next Step

- 최종 검증 후 GitHub 원격 리포를 만들고 `main` 브랜치를 push한다.
