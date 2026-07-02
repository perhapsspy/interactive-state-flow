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

## 더 보기

- 스킬 상세 규칙: [English](skills/interactive-state-flow/SKILL.md) | [한국어](skills/interactive-state-flow/SKILL.ko.md)
- 스킬 원점 맥락: [docs/reference/origin.md](docs/reference/origin.md)

## 지원

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://www.buymeacoffee.com/perhapsspy)

## 라이선스

[MIT](LICENSE)
