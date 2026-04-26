# Interactive State Flow

[한국어](README.md) | [English](README.en.md)

## 요약

`interactive-state-flow`는 인터랙티브 애플리케이션에서 **지체 없이 기록되어야 하는 기준 상태**와 **비용이 큰 표현·계산·비동기 작업**을 분리하는 스킬입니다.

사용자 의도와 source-of-truth 상태는 지체 없이 기록하고, 비싼 파생 작업은 지연·취소·우선순위화·가상화·백그라운드 실행으로 분리하며, 비동기 결과는 여전히 유효하고 유용할 때만 커밋하게 합니다.

`스펙:` Agent Skills / SKILL.md | `라이선스:` MIT | `에이전트:` Codex, ChatGPT, Agent Skills 호환 도구

## 빠른 시작

**설치**

```bash
npx skills add perhapsspy/interactive-state-flow
```

혹은 `skills/interactive-state-flow` 폴더를 에이전트 스킬 디렉터리에 직접 복사합니다.

**바로 사용**

```text
$interactive-state-flow 를 사용해서 이 검색/필터 UI의 입력 상태, 파생 렌더링, 비동기 결과 반영 흐름을 정리해줘
```

## 이런 때 사용

- 입력, 선택, 라우트, 스크롤 같은 상호작용 피드백이 비싼 작업 때문에 늦을 때
- 실제 input 상태를 debounce해서 source state가 늦게 기록되는 구조가 있을 때
- 오래된 비동기 결과가 최신 상태나 화면을 덮어쓸 위험이 있을 때
- 큰 리스트, 차트, 프리뷰, 파싱, 검증, 레이아웃 준비가 상호작용 경로를 막을 때
- worker, isolate, background queue 같은 실행 경계는 있지만 결과 소유권과 freshness가 불명확할 때

## 프로젝트 가치 보존 스킬

이 스킬들은 오래 운영되는 프로젝트가 나중에도 이해되고 수정될 수 있는 상태를 유지하도록 돕습니다.

프로젝트의 가치는 코드가 늘어나서만 떨어지는 것이 아닙니다. 코드 흐름이 읽히지 않고, 결정 이유가 사라지고, 작업 맥락을 복원할 수 없고, 테스트가 실제 동작 계약을 보호하지 못하고, 인터랙티브 경로에 즉시 상태와 비싼 작업이 뒤섞일 때 가치가 떨어집니다.

- [`structure-first`](https://github.com/perhapsspy/structure-first): 코드 흐름, 책임 경계, 수정 가능성, 계약 중심 테스트를 보존합니다.
- [`project-context`](https://github.com/perhapsspy/project-context): 저장소 안의 일반 문서로 프로젝트 기억, 결정 이유, 현재 작업 상태, 작업 연속성을 보존합니다.
- `interactive-state-flow`: 지체 없이 기록되어야 하는 기준 상태와 비용이 큰 표현·비동기 작업·실행 경로를 분리해 사용자 반응성과 유지보수성을 보존합니다.

각 스킬은 독립적으로 설치하고 사용할 수 있습니다. 이 README는 같은 철학의 스킬들을 소개할 뿐, 개별 `SKILL.md`가 서로를 호출하거나 의존하게 만들지 않습니다.

## 더 보기

- 스킬 상세 규칙: [English](skills/interactive-state-flow/SKILL.md) | [한국어](skills/interactive-state-flow/SKILL.ko.md)
- 스킬 원점 맥락: [docs/reference/origin.md](docs/reference/origin.md)

## 지원

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://www.buymeacoffee.com/perhapsspy)

## 라이선스

[MIT](LICENSE)
