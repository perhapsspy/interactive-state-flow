# Origin

## 목적

이 문서는 이 스킬을 만들게 된 ChatGPT 대화의 핵심 맥락을 저장한다.

원문 대화를 그대로 보관하지 않고, 나중에 스킬을 리뷰하거나 수정할 때 필요한 문제 정의, 설계 결정, 보존해야 할 의도를 정리한다. 진행 이력은 task logs에 두고, 이 문서는 현재 믿고 참고할 수 있는 배경 맥락만 담는다.

## 출발 문제

처음 문제는 프론트엔드나 인터랙티브 UI 코드에서 상태 업데이트, 렌더링, 비동기 후속 작업, Web Worker 같은 실행 경로가 섞이는 데서 출발했다.

핵심 위험은 다음이다.

- 사용자 입력이나 상호작용은 즉시 반응해야 한다.
- 사용자 의도와 기준 상태는 즉시 기록되어야 한다.
- 렌더링, 표현 모델 생성, 파생 계산, IO, 비동기 후속 작업은 비용이 클 수 있다.
- 비싼 표현 작업을 상태 업데이트와 같은 경로에서 처리하면 UI가 버벅이고 코드 흐름이 불투명해진다.
- 오래된 비동기 결과가 최신 상태를 덮어쓸 수 있다.
- 큰 프로젝트에서는 사람과 AI 에이전트 모두 상태 흐름과 표현 비용이 섞인 코드를 추적하기 어렵다.

초기 표현은 "프론트엔드 비동기 상태와 Web Worker 사용"에 가까웠지만, 대화 중 이 표현은 너무 웹 중심적이고 Web Worker에 편향되어 있다고 판단했다.

## 최종 문제 정의

최종 정의는 특정 API나 프레임워크가 아니라 다음 구조 문제다.

```txt
interaction path와 expensive work path를 분리한다.
```

더 구체적으로는 다음 문장이 스킬의 중심이다.

```txt
사용자 의도와 기준 상태는 즉시 기록하고,
비용이 큰 표현·계산·비동기 작업은 적절히 지연·분리·취소하며,
결과는 여전히 유효하고 유용할 때만 반영한다.
```

영문 핵심 문장은 다음이다.

```txt
Record user intent and source-of-truth state immediately.
Let expensive presentation and async work follow only when fresh, useful, and affordable.
```

짧은 구호는 다음이다.

```txt
State now, presentation when useful, async results only when fresh.
```

## 기존 스킬과의 관계

대화에서는 이 스킬을 `structure-first`에 녹일지, 독립 스킬로 둘지 검토했다.

최종 결정은 독립 스킬이다.

- `structure-first`는 코드의 읽히는 성공 흐름, 책임 경계, 수정 가능성, 계약 중심 테스트를 보존한다.
- `project-context`는 프로젝트 기억, 결정 이유, 작업 이력, 세션 연속성을 보존한다.
- `interactive-state-flow`는 인터랙티브 앱에서 즉시 상태와 비싼 표현·비동기 작업의 경계를 분리해 반응성, 상태 정확성, 비동기 결과 안정성을 보존한다.

세 스킬은 같은 상위 철학을 공유한다.

```txt
Project Value Preservation Skills
```

한국어로는 다음처럼 본다.

```txt
프로젝트 가치 보존 스킬
```

의미는 다음이다.

```txt
프로젝트가 오래 지나도 사람과 AI 에이전트가 다시 이해하고,
안전하게 수정하고,
효율적으로 이어서 작업할 수 있게 만드는 스킬들.
```

## 독립성 결정

각 `SKILL.md` 내부에서는 서로를 직접 연결하지 않는다.

넣지 말아야 할 문구의 예:

```txt
Use structure-first first.
Use project-context together.
Refer to structure-first.
Apply this after structure-first.
```

이유는 다음이다.

- 스킬 간 결합도가 생긴다.
- 독립 설치 가능성이 떨어진다.
- 자동 활성화 기준이 흐려진다.
- 불필요한 문맥 로딩 가능성이 생긴다.
- 한 스킬의 변경이 다른 스킬에 영향을 주기 쉬워진다.

따라서 README나 이 origin 문서 같은 repo reference에서만 같은 철학의 독립 스킬로 묶어 설명한다.

## 이름 결정

최종 이름은 다음으로 정했다.

```txt
interactive-state-flow
```

이름 판단:

- `interactive-state-flow`: 최종 선택. 웹, 모바일, 데스크톱, 네이티브 UI 등 인터랙티브 시스템 전반에 맞고, 사용자 상호작용과 상태 흐름을 함께 드러낸다.
- `responsive-state-flow`: 괜찮지만 responsive layout과 혼동될 수 있다.
- `state-presentation-flow`: source state와 presentation state 구분은 잘 드러나지만 사용자 반응성 의미가 약하다.
- `interaction-safe-flow`: UX 반응성은 강하지만 상태 구조 의미가 약하다.
- `async-presentation-flow`: 비동기 표현에는 맞지만 source state 즉시성이 약하다.

## 핵심 구조 개념

스킬은 다음 개념을 중심으로 한다.

- `interaction path`: 사용자 입력, 터치, 스크롤, 포커스, 제스처, 선택, 즉시 피드백이 처리되는 긴급 경로.
- `source-of-truth state`: 현재 input, 선택, route, operation id, source model처럼 즉시 최신이어야 하는 기준 상태.
- `presentation state`: visible rows, chart model, preview output, rendered range, stale/pending/progressive display처럼 기준 상태를 보여주는 방식.
- `derived work`: 기준 상태에서 만들어지는 계산, 필터링, 정렬, aggregation, validation, layout model, preview generation.
- `freshness gate`: 비동기 결과가 최신 사용자 의도와 현재 화면 맥락에 여전히 맞는지 확인하는 반영 조건.
- `execution context`: UI thread, event loop, worker, isolate, coroutine dispatcher, background queue, job system 같은 실행 경로.
- `owning boundary`: 결과를 반영해도 되는지 판단하고 presentation commit을 소유하는 경계.

## 반드시 포함해야 하는 실행 지침

대화에서 `SKILL.md`는 철학 문서가 아니라 작업 중 판단 가능한 지침이어야 한다고 정리했다.

따라서 다음 형식을 반드시 유지한다.

- failure smells
- primary flow 또는 decision flow
- rules
- do-not rules
- contract tests
- platform mapping
- final checklist

피해야 할 추상 문장:

```txt
Make UI architecture better.
Use good async structure.
Optimize rendering.
Separate concerns.
```

좋은 문장:

```txt
Record source state immediately; defer only expensive derived presentation.
Async result must pass a freshness gate before it mutates state or presentation.
Do not move work to another execution context unless input/output ownership is clear.
```

## 테스트 계약

스킬은 구현 세부가 아니라 동작 계약을 보호해야 한다.

- Immediate State Contract: 사용자 의도와 source state가 presentation throttling 때문에 늦어지지 않는다.
- Freshness Contract: stale/cancelled/inactive-context 결과가 최신 상태나 표현을 덮어쓰지 않는다.
- Interaction Contract: 긴급한 상호작용 피드백이 비용 큰 작업에 막히지 않는다.
- Presentation Contract: 비싼 표현은 안전한 경우에만 늦게 따라오며, stale/pending/progressive 상태가 필요할 때 명확하다.
- Execution Boundary Contract: background work의 input/output, failure, cancellation, commit owner가 명확하다.

## 플랫폼 범위

이 스킬은 웹 전용이 아니다.

적용 대상:

- Web, React, Vue, Svelte
- React Native
- Flutter
- Android native
- iOS/macOS
- Desktop UI
- Game UI
- 기타 interactive application code

플랫폼별 API는 예시일 뿐이다.

- Web: Web Worker, transition, deferred update, idle task, abort signal.
- React Native: UI thread, JavaScript thread, native animation, background task.
- Flutter: main isolate, background isolate.
- Android: main/UI thread, coroutine dispatcher, worker thread.
- iOS/macOS: main actor, main queue, background queue, async task.
- Desktop UI: event dispatch thread, render loop, dispatcher, worker pool.
- Game UI: input/render frame, job system, async loading.

핵심은 특정 API가 아니라 다음 원칙이다.

```txt
Protect the interaction path and commit only fresh, useful results.
```

## 스킬의 차별점

기존 공개 스킬이나 일반 최적화 문서는 보통 다음 문제를 다룬다.

- React performance
- Web Worker usage
- Core Web Vitals
- virtualization
- debounce/throttle
- framework best practices
- API migration

그 흐름은 대개 다음이다.

```txt
느려졌다 -> 측정한다 -> 병목을 찾는다 -> 최적화한다
```

이 스킬은 다음을 목표로 한다.

```txt
상태, 표현, 비동기 결과, 실행 경로의 책임을 처음부터 나눈다
-> 반응성과 유지보수성이 함께 무너지지 않게 한다
```

따라서 이 스킬의 더 정확한 분류는 다음이다.

```txt
interactive application structure skill
```

또는:

```txt
long-term maintainability skill for interactive state and async presentation flow
```

## 초기 작업 브리프의 구현 제약

대화에서 Codex에 넘길 작업 브리프는 다음 제약을 요구했다.

- 새 스킬은 독립 스킬로 만든다.
- 기존 스킬 안에 병합하지 않는다.
- `SKILL.md` 내부에서 다른 스킬을 직접 참조하지 않는다.
- README에서만 같은 철학의 독립 스킬들을 묶어 소개한다.
- Web Worker, React, debounce 같은 특정 도구를 중심에 두지 않는다.
- 플랫폼 중립 용어를 중심으로 쓴다: `interaction path`, `source-of-truth state`, `presentation boundary`, `derived work`, `async result`, `freshness gate`, `execution context`, `owning boundary`.
- `SKILL.md`는 `Purpose`, `Core Thesis`, `Use / Do Not Use`, `Failure Smells`, `Primary Flow`, `Rules`, `Hard Rules`, `Contract Tests`, `Platform Mapping`, `Final Checklist`를 포함한다.
- frontmatter description은 사용 시점이 자동으로 드러나게 쓴다: 즉시 기록되어야 하는 user intent/source state와 지연·취소·우선순위화·별도 실행 경로가 필요한 expensive presentation/async work를 함께 설명한다.
- 테스트는 scheduler나 framework 내부 구현이 아니라 immediate state, freshness, interaction, presentation, execution boundary 계약을 보호한다.

## 리뷰 시 보존할 의도

향후 문구를 다듬더라도 다음 의도는 보존한다.

- source state는 늦추지 않는다.
- 늦출 대상은 source state가 아니라 expensive derived presentation이다.
- completed async result와 valid async result를 구분한다.
- background execution은 목표가 아니라 경계다.
- presentation commit에는 owner가 있어야 한다.
- 화면 맥락과 사용자가 실제로 볼 표현 비용을 판단한다.
- contract tests는 scheduler 세부 구현이 아니라 상태 즉시성, freshness, 상호작용, 표현, 실행 경계를 보호한다.
