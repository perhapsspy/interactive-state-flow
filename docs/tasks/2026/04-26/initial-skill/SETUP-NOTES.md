# Initial Skill Setup Notes

## 목적

이 문서는 `interactive-state-flow` 리포를 처음 세팅할 때 참고한 외부 문서, 기존 리포 관행, 추가 판단을 기록한다.

현재 믿고 반복 사용할 기준 맥락은 `docs/reference/**`에 두고, 이 문서처럼 특정 세팅 작업의 조사와 판단은 task-local 문서로 둔다.

## 포지셔닝

`interactive-state-flow`는 인터랙티브 애플리케이션에서 사용자 의도와 기준 상태를 즉시 기록하고, 비용이 큰 표현·계산·IO·비동기 결과 반영을 별도 실행 흐름과 소유 경계로 분리하게 하는 구조 스킬이다.

이 스킬은 특정 프레임워크 성능 최적화가 아니라, 오래 운영되는 프로젝트에서 반응성, 상태 정확성, 표현 비용 경계, 비동기 결과 안전성을 함께 보존하는 실행 지침으로 둔다.

## 기존 리포에서 가져온 기준

- `structure-first` 리포처럼 공개 문서는 한/영 페어를 유지하고, 스킬은 실제 작업 중 사용할 수 있는 실행 규칙을 중심에 둔다.
- `project-context` 리포처럼 README, direction/reference, shipped skill의 역할을 분리한다.
- 새 스킬의 `SKILL.md` 내부에서는 다른 스킬을 직접 참조하지 않는다. 같은 철학의 묶음 설명은 README와 origin reference 문서에서만 다룬다.
- 내부 작업 맥락은 `project-context` 구조로 남겨 다음 스레드가 `BRIEF.md`와 logs에서 이어받게 한다.
- ChatGPT 대화에서 나온 원점 맥락은 `docs/reference/origin.md`에 별도 정리한다. 이 문서는 조사와 리포 세팅 판단을 맡고, origin 문서는 문제 정의와 보존할 의도를 맡는다.

## 외부 문서 조사에서 반영한 점

- [OpenAI Academy Skills](https://academy.openai.com/public/resources/skills): skill은 반복 가능한 workflow를 `SKILL.md`와 관련 리소스로 패키징하고, name/description이 관련성 판단에 중요하다. 이 리포는 shipped skill을 간결한 실행 playbook으로 두고, 배경 설명은 repo docs에 분리한다.
- [Agent Skills specification](https://agentskills.io/specification): 최소 구조는 skill directory와 `SKILL.md`이며, `name`과 `description` frontmatter가 필수다. Progressive disclosure 원칙에 맞춰 `SKILL.md`를 500줄 아래로 유지하고, 현재는 별도 bundled resource를 만들지 않는다.
- [React `startTransition`](https://react.dev/reference/react/startTransition), [React `useDeferredValue`](https://react.dev/reference/react/useDeferredValue): urgent state와 non-blocking/deferred UI update를 구분하는 사례를 제공한다. 스킬에는 React API를 중심에 두지 않고 "source state vs presentation state" 판단으로 일반화했다.
- [MDN Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers), [MDN AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController/abort): background thread와 abortable async work는 유용하지만 수단일 뿐이다. 스킬에는 worker가 아니라 execution context, cancellation, freshness gate를 중심 개념으로 둔다.
- [Flutter isolates](https://docs.flutter.dev/perf/isolates), [Flutter background parsing](https://docs.flutter.dev/cookbook/networking/background-parsing): main isolate의 frame gap을 넘는 계산은 jank를 만들 수 있고, isolate 간 메시지 전달 비용과 메모리 경계가 있다. 그래서 "background execution is a boundary, not a goal" 규칙을 명시했다.
- [Android coroutines](https://developer.android.com/kotlin/coroutines), [Android slow rendering](https://developer.android.com/topic/performance/vitals/render): long-running task는 UI thread를 막을 수 있고, main-safe ownership이 필요하다. 스킬의 execution-boundary contract와 failure handling에 반영했다.
- [Apple Dispatch Queue](https://developer.apple.com/documentation/dispatch/dispatch-queue), [Swift MainActor](https://developer.apple.com/documentation/Swift/MainActor): main queue/main actor와 background queue는 UI 업데이트 소유권을 생각하게 한다. 플랫폼 매핑에는 API 이름만 예시로 둔다.
- [WPF threading model](https://learn.microsoft.com/en-us/dotnet/desktop/wpf/advanced/threading-model), [Swing Event Dispatch Thread](https://docs.oracle.com/javase/tutorial/uiswing/concurrency/dispatch.html): desktop UI에서도 UI thread/event dispatch thread의 짧은 작업 원칙이 중요하다. "interactive app 전반"이라는 범위를 뒷받침한다.

## 메운 판단

- 새 리포는 기존 skill 리포들과 나란히 `interactive-state-flow`로 둔다.
- `skills/interactive-state-flow/SKILL.md`는 영어 기본 shipped skill로 둔다. Agent Skills 메타데이터와 실제 자동 로딩 경로를 단순하게 유지하기 위해서다.
- `skills/interactive-state-flow/SKILL.ko.md`는 한국어 리뷰 페어로 둔다. frontmatter는 두지 않아 자동 skill 후보가 중복되지 않게 한다.
- `agents/openai.yaml`은 skill-creator의 현재 권장 형식에 따라 포함한다. 단, 동작 의존성은 두지 않고 UI-facing metadata만 둔다.
- 현재는 `references/`, `scripts/`, `assets/`를 skill bundle 안에 만들지 않는다. 이 스킬은 반복 실행 스크립트보다 판단 절차가 핵심이고, 세부 프레임워크별 패턴은 아직 검증된 공통 자산이 아니다.
- README에는 `Project Value Preservation Skills` 묶음을 둔다. 개별 `SKILL.md`는 독립성을 위해 cross-reference를 피한다.

## 리뷰 포인트

- `interactive-state-flow`라는 이름이 충분히 넓고 플랫폼 중립적인지 확인한다.
- README의 설치 명령은 GitHub 게시 후 유효하다. 게시 전에는 로컬 복사 설치 안내를 사용한다.
- `SKILL.md`가 너무 추상적으로 흐르지 않는지, 실패 냄새와 테스트 계약이 실제 코드 작업에 충분한지 검토한다.
- 향후 실제 사례가 쌓이면 framework-specific examples를 shipped skill이 아니라 별도 reference로 둘지 판단한다.
- origin 문서의 보존 의도와 shipped skill의 실행 규칙이 어긋나지 않는지 확인한다.
