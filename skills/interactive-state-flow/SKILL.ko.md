# Skill: Interactive State Flow (Korean Pair)

> 영문 기본 문서: `SKILL.md`

## Purpose

기준 상태의 빠른 기록과 비용이 큰 표현·비동기 작업을 분리해 반응성과 유지보수성을 보존합니다.

이 스킬은 특정 프레임워크 성능 튜닝이 아닙니다. 사용자 의도를 최신으로 유지하고, 비싼 작업을 긴급한 상호작용 경로에서 분리하며, 현재 상호작용 맥락에 여전히 속하는 결과만 커밋하게 합니다.

## Core Thesis

사용자 의도와 기준 상태는 지체 없이 기록합니다.

비용이 큰 표현, 파생 계산, IO, 비동기 후속 작업은 비용과 실행 경로가 정당할 때만 실행합니다.

결과는 owner가 fresh하고 유용하다고 받아들인 뒤에만 커밋합니다. Owner는 결과가 현재 사용자 의도, 기준 상태, 화면 맥락, 표현 정책에 아직 속하는지 판단할 수 있는 가장 작은 경계입니다.

기준 상태를 빠르게 기록한다는 말은 동기 렌더링을 강제한다는 뜻이 아닙니다. 사용자 의도를 비싼 표현 작업을 줄이기 위해 지연하지 말라는 뜻입니다.

## Use / Do Not Use

이런 때 사용합니다:

- 입력, 터치, 스크롤, 포커스, 제스처, navigation, route, 선택 피드백이 늦다.
- 비용이 큰 파생 표현이 기준 상태를 너무 즉시 따라간다.
- 비동기 결과가 순서 없이 도착해 최신 상태를 덮어쓸 수 있다.
- 하나의 컴포넌트, 화면, 핸들러, 흐름 안에 intent, IO, derivation, scheduling, presentation mutation이 섞여 있다.
- 큰 리스트, 차트, 프리뷰, 파싱, 검증, 레이아웃 준비, 변환이 상호작용을 막는다.
- 백그라운드 실행은 있지만 소유권, 취소, 순서, 실패 동작, freshness가 불명확하다.
- 사람이나 에이전트가 큰 인터랙티브 흐름을 적은 맥락으로 추적해야 한다.

이런 때 사용하지 않습니다:

- 변경이 정적이거나 비인터랙티브다.
- 현재 구현이 작고 명확하며 반응성이 좋고, 비용이 큰 파생 작업이나 비동기 순서 위험이 없다.
- 문제의 본질이 인터랙티브 반응성이 아니라 백엔드 처리량이다.
- 추가 스케줄링, 백그라운드 실행, 상태 경계의 비용이 작업 자체보다 크다.

## Common Agent Cases

Codex 작업이 다음을 언급하면 이 스킬을 사용합니다:

- 검색, typeahead, command palette, filter, date input에서 source value가 debounce되거나, 늦게 기록되거나, 오래된 결과에 덮어써진다.
- preview, diff, file viewer, log viewer, map/canvas, chart 렌더링이 선택, 입력, 스크롤, tab 변경, mode 변경을 막는다.
- streaming, tool output, upload, parse, fetch, realtime, SSE, socket, refresh, background 결과가 잘못된 run, route, tab, screen, session, selected item에 반영될 수 있다.
- URL, route, focus, selection, remount, cache, fallback, quiet refresh 동작에서 화면은 current처럼 보이지만 source state, freshness, ownership은 그렇지 않다.

코드에 async work, effect, worker, memoization, caching이 있다는 이유만으로 이 스킬을 사용하지 않습니다. 인터랙티브 지연, 오래된 결과, 책임 혼합, 사용자에게 보이는 freshness 위험이 있어야 합니다.

## Failure Smells

- 실제 input, selection, route, intent 상태를 렌더링 비용을 줄이려고 debounce한다.
- 비동기 또는 백그라운드 완료가 현재 intent, screen, operation 확인 없이 UI 상태를 바꾼다.
- 렌더링, 프리뷰 생성, 파싱, 무거운 파생 작업이 사용자 의도를 기록하는 경로에서 같이 실행된다.
- 비용이 큰 파생 데이터를 기준 상태처럼 다룬다.
- 백그라운드 작업에 취소, 순서, 실패 처리, 최종 커밋 owner가 없다.
- 보이는 범위, cached view, progressive preview만 필요해도 전체 기준 상태의 표현을 만든다.
- 하나의 불투명한 핸들러가 입력 캡처, IO 시작, 데이터 파생, 작업 스케줄링, 표현 변경을 모두 한다.

## Primary Flow

1. 사용자에게 보이는 동작 계약을 이름 붙인다. 어떤 intent/source state가 지체 없이 갱신되는지, 어떤 표현이 늦게 따라와도 되는지, 어떤 owner가 freshness를 승인하는지, 사용자가 current/stale/pending/cached/progressive 중 무엇을 볼 수 있는지 확인한다.
2. 사용자 의도를 지체 없이 기록한다.
3. 기준 상태를 source owner를 통해 커밋한다.
4. 후속 작업을 긴급도, 비용, 가시성, freshness 위험으로 분류한다.
5. 상호작용 경로를 가볍게 유지한다.
6. 비용이 큰 작업을 지연, 취소, 배치, 우선순위화, 가상화, 생략, 캐시하거나 정당한 경우에만 실행 경로를 옮긴다.
7. 비동기 또는 표현 결과는 여전히 fresh하고 유용할 때만 owner를 통해 커밋한다.
8. 스케줄러 내부가 아니라 동작 계약을 테스트한다.

## Rules

### 1. 기준 상태와 표현 상태를 분리한다

기준 상태는 현재 input, 최신 intent, 선택, 활성 route/screen/context, operation id, source-of-truth model 같은 현재의 진실을 기록합니다.

표현 상태는 visible rows, chart model, preview output, rendered range, pending/stale/cached display, progressive output처럼 그 진실이 어떻게 보이는지를 설명합니다.

기준 상태는 최신이어야 합니다. 표현은 반응성을 보존하고 사용자를 오도하지 않을 때만 늦게 따라올 수 있습니다. 렌더링을 줄이기 위해 기준 상태를 지연하지 말고, 비용이 큰 파생 작업을 지연합니다.

### 2. 상호작용 경로를 가볍게 유지한다

input, navigation, selection, immediate feedback을 처리하는 경로는 작고 예측 가능해야 합니다.

피할 흐름: `input -> update source -> filter/sort/build/render everything -> feedback appears`.

선호할 흐름: `input -> record intent -> commit source -> show feedback -> schedule expensive work -> discard stale work -> commit fresh presentation`.

### 3. 도구를 고르기 전에 분류한다

debounce, memoization, virtualization, transition, idle work, background execution, batching, cancellation부터 시작하지 않습니다.

먼저 묻습니다: intent/source state인가? derived work인가? presentation인가? async IO인가? expensive computation인가? stale해질 수 있는가? 지금 보이거나 유용한가? 취소 가능한가? 별도 실행 경로가 필요한가?

그다음 가장 가벼운 대응을 선택합니다:

- intent 또는 source state -> source owner를 통해 지체 없이 기록
- 비싸지만 보이는 표현 -> 지연, 가상화, 캐시, progressive output
- 비싸지만 보이지 않는 표현 -> 생략, idle 처리, 유용할 가능성이 높을 때만 precompute
- stale 위험이 있는 async 또는 IO -> operation identity, cancellation, freshness gate
- 무거운 CPU 작업 -> 전달 비용, 소유권, 취소, 순서, 실패 동작이 명확할 때만 실행 경로 이동

### 4. 비동기 결과는 freshness gate를 통과해야 한다

완료된 결과가 항상 유효한 결과는 아닙니다. 커밋 전 최신 intent, operation/generation, 취소 상태, screen/route/session/context, 유용성, 사용자에게 보이는 의미를 확인합니다.

필요할 때 request id, generation id, operation token, cancellation token, abort handle, input snapshot, screen key, session key 같은 명시적 identity를 사용합니다.

stale captured value가 아니라 owner가 현재 보유한 identity와 비교합니다.

### 5. 백그라운드 실행은 경계로 다룬다

상호작용 경로를 보호해야 하고 경계의 입력, 출력, 소유권, 취소, 순서, 실패 동작이 명확할 때만 작업을 다른 실행 경로로 옮깁니다.

좋은 후보에는 parsing, sorting, filtering, aggregation, validation, indexing, preview generation, layout model prep, large IO, expensive transformation이 포함됩니다.

작업이 작거나, UI 객체에 직접 의존하거나, 전달 비용이 계산 비용보다 크거나, 결과 소유권/커밋 순서가 불명확하거나, 안전하게 취소할 수 없거나, 코드가 더 이해하기 어려워지면 옮기지 않습니다.

### 6. 표현은 owner를 통해 커밋한다

비동기 또는 백그라운드 작업은 UI를 직접 변경하지 말고 owner에게 결과를 반환해야 합니다.

Owner는 결과가 fresh한지, 유용한지, 화면 맥락이 현재인지, source state에 안전하게 묶을 수 있는지, UI가 current/stale/pending/cached/skeleton/progressive 중 무엇을 보여야 하는지 판단합니다.

이 질문에 답할 수 있는 가장 작은 owner를 사용합니다. 답할 owner가 없다면 아직 비동기 또는 백그라운드 경계를 추가하지 않습니다.

### 7. 화면 맥락을 존중한다

상태가 있다고 해서 모든 표현을 만들지 않습니다. 지금 보이는지, 곧 보일 가능성이 있는지, 상호작용 중 필요한지, stale content로 보여도 안전한지, progressive output이 적합한지, 보이는 범위만으로 충분한지, 결과가 아직 fresh한지 확인합니다.

표현은 완전하기만 해서는 안 되고 유용해야 합니다. Stale output은 계속 보일 수 있지만, 구분이 중요할 때 stale output을 current처럼 조용히 보여주면 안 됩니다.

navigation, reload, unmount, screen replacement가 freshness boundary라면 다음 활성 화면 전에 버려질 표현 작업은 생략합니다.

## Hard Rules

- 표현 비용을 줄이려고 사용자 의도나 기준 상태를 지연하지 않는다.
- 비싼 파생 작업만 지연하면 될 때 실제 intent 상태를 debounce하지 않는다.
- freshness 확인 없이 비동기 결과를 커밋하지 않는다.
- 비싸다는 이유만으로 다른 실행 경로로 옮기지 않는다.
- intent, IO, derived data, scheduling, presentation mutation을 하나의 불투명한 흐름에 섞지 않는다.
- 책임을 분류하기 전에 성능 도구부터 고르지 않는다.
- 백그라운드 작업이 UI mutation을 직접 소유하게 하지 않는다.
- 작고 명확하며 반응성 있는 코드를 스케줄링 경계 도입만을 위해 더 어렵게 만들지 않는다.
- 구분이 중요할 때 stale output을 current처럼 조용히 보여주지 않는다.
- 사용자에게 보이는 stale, laggy, race-prone 상호작용을 고쳤다고 refactor 모양만으로 선언하지 않는다. 프로젝트에서 가장 작은 적절한 근거로 동작 계약을 확인한다.
- 이 스킬을 일반 UI 디버깅, 제품 정책, 브라우저 검증 절차의 owner로 사용하지 않는다. stale, cached, pending, 검증 정책이 정의되지 않았다면 이 스킬 안에서 정하지 말고 결정이 필요하다고 드러낸다.

## Contract Tests

스케줄러 내부가 아니라 동작 계약을 테스트합니다:

- Immediate state: intent와 source state가 지체 없이 기록되고, freshness가 중요할 때 operation identity가 보인다.
- Freshness: stale, cancelled, inactive-screen/session, obsolete-generation 결과가 최신 상태나 표현을 덮어쓸 수 없다. request A가 B보다 먼저 시작했지만 B보다 늦게 끝나면 A가 B를 덮어쓰면 안 된다.
- Interaction: 긴급한 피드백이 비싼 작업에 막히지 않고, 지연된 비긴급 작업이 기준 상태를 오염시키지 않는다.
- Presentation: 비싼 표현은 안전한 경우에만 늦게 따라오고, 필요할 때 stale/pending 표현이 명시되며, 유용하거나 보이는 표현만 만들어진다.
- Execution boundary: 백그라운드 작업의 입력/출력이 명확하고, owner를 통해 커밋하며, 실패를 안전하게 처리하고, 취소되거나 오래된 결과를 커밋할 수 없다.

## Platform Mapping

플랫폼의 로컬 interaction path와 execution boundary에 해당하는 것을 사용합니다. 특정 API를 설계 중심에 두지 않습니다.

예에는 web worker 또는 deferred rendering, React Native UI/JS paths, Flutter isolates, Android coroutine dispatchers, iOS background queues, desktop worker pools, game job systems가 포함됩니다.

플랫폼 이름은 달라도 원칙은 같습니다. 상호작용 경로를 보호하고, 현재 상호작용 맥락을 이해하는 owner를 통해 fresh하고 유용한 결과만 커밋합니다.

## Final Checklist

마치기 전에 확인합니다:

- 사용자에게 보이는 동작과 freshness 계약을 이름 붙였고, 프로젝트에서 가장 작은 적절한 근거로 확인했다.
- 사용자 의도와 기준 상태가 지체 없이 기록된다.
- 비용이 큰 파생 작업이 긴급 상호작용 경로에 강제로 올라오지 않는다.
- 표현은 안전하고 이해 가능한 경우에만 늦게 따라온다.
- 비동기 결과에 freshness identity와 owner가 있다.
- 취소되거나 오래된 작업이 상태나 표현을 바꿀 수 없다.
- 실행 경계의 입력, 출력, 소유권, 취소, 순서, 실패 동작이 명확하다.
- 테스트가 immediate state, freshness, interaction, presentation, execution-boundary 계약을 다룬다.
