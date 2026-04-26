**2026-04-26**
- 사용자 브리프는 새 스킬을 기존 두 스킬과 같은 철학으로 보되, 각 SKILL.md 안에서는 결합하지 말라고 요구했다.
- 새 스킬은 독립 리포와 독립 SKILL.md로 두고, README와 repo-local reference에서만 Project Value Preservation Skills로 묶어 설명한다.
- 스킬 자동 활성화 기준과 설치 독립성을 흐리지 않고, 기존 structure-first/project-context의 책임을 비대하게 만들지 않기 위해서다.
- 다음 작업자는 SKILL.md 내부에 다른 스킬 사용 지시를 추가하지 말고 README/문서 레벨에서만 관계를 설명해야 한다.

**2026-04-26**
- 스킬 생성 규칙은 name/description frontmatter와 progressive disclosure를 중시하고, 기존 리포는 공개 문서 한/영 페어를 운영한다.
- 실제 shipped skill은 영어 SKILL.md 하나로 두고, SKILL.ko.md는 frontmatter 없는 한국어 리뷰 페어로 둔다.
- 자동 로딩 후보를 중복시키지 않으면서 사용자가 한국어로 리뷰하기 쉽게 만들기 위해서다.
- SKILL.md 변경 시 한국어 페어와 README 의미 동기화가 필요하다.

**2026-04-26**
- 사용자 브리프는 Web Worker 중심으로 흐르지 말고 인터랙티브 애플리케이션 전반의 구조 스킬로 만들라고 했다.
- SKILL.md의 중심 용어를 interaction path, source state, presentation state, freshness gate, execution context, owning boundary로 정했다.
- React, Web Worker, Flutter isolate, Android coroutine, Apple queue 같은 API는 플랫폼 매핑 예시일 뿐이며 공통 문제는 상태와 표현 비용, 비동기 결과 소유권의 분리이기 때문이다.
- 앞으로 프레임워크별 세부 패턴을 추가하더라도 shipped skill 본문보다 별도 reference로 분리할 가능성을 먼저 검토한다.

**2026-04-26**
- 사용자가 ChatGPT 대화 내용도 새 스킬의 맥락으로 docs에 남기는 것이 맞지 않냐고 지적했다.
- 원문 대화 전체가 아니라 문제 정의, 설계 결정, 보존할 의도를 요약한 docs/reference/interactive-state-flow-origin.md를 둔다.
- 원문 전체는 reference 문서로는 비대하고 진행 이력과 섞이기 쉬우며, 다음 스레드에는 현재 믿고 쓸 origin context가 더 유용하기 때문이다.
- 향후 스킬 문구를 바꿀 때 origin 문서의 보존 의도와 shipped skill의 실행 규칙이 어긋나지 않는지 함께 확인한다.

**2026-04-26**
- AGENTS.md가 project-context가 이미 책임지는 문서 배치와 로그 운영 규칙을 반복하고, skill-design reference가 세팅 작업 조사/판단 성격으로 남아 있었다.
- AGENTS.md는 항상 로드되는 repo-local 최소 규칙만 남기고, 세팅 조사/판단 문서는 docs/tasks/2026/04-26/initial-skill/SETUP-NOTES.md로 옮긴다.
- 항상 로드되는 파일의 토큰 비용을 줄이고, docs/reference에는 현재 반복 사용 가능한 기준 맥락만 두기 위해서다.
- 앞으로 세팅 과정 조사나 일회성 판단은 task-local 문서에 두고, reference에는 origin context처럼 재사용 가능한 기준만 남긴다.

**2026-04-26**
- origin 문서 파일명과 제목이 docs/reference 안에서도 스킬명을 반복해 prefix처럼 보였고, 원 대화에서 Codex 작업 브리프가 요구한 구현 제약이 별도 섹션으로 정리되어 있지 않았다.
- origin 문서를 docs/reference/origin.md로 줄이고 제목을 Origin으로 바꾸며, 초기 작업 브리프의 구현 제약 섹션을 추가한다.
- reference 경로 자체가 이미 이 리포의 스킬 맥락을 제공하므로 이름 반복은 불필요하고, 향후 리뷰에는 문제 정의뿐 아니라 작성 제약도 같이 필요하기 때문이다.
- 현재 문서 링크는 origin.md로 갱신하고, 이전 긴 경로는 append-only 로그의 역사로만 남긴다.

**2026-04-26**
- 두 dogfood subagent가 모두 source/presentation 분리와 freshness owner는 유용하다고 봤지만, owner-held identity와 direct UI mutation 금지를 더 명확히 할 필요를 드러냈다.
- SKILL.md에는 플랫폼별 세부 도구 선택 guidance를 추가하지 않고, Rule 4와 Rule 6에 일반화 가능한 두 문장만 추가한다.
- React-specific transition/worker guidance는 아직 표본이 부족하고 shipped skill을 웹 중심으로 만들 수 있지만, owner-held identity와 owner-only commit은 플랫폼 중립 핵심 규칙이기 때문이다.
- 추가 dogfood는 non-web UI 사례를 우선하고, 플랫폼별 패턴은 충분히 반복 확인된 뒤 reference 또는 별도 자료로 분리한다.

**2026-04-26**
- React 중심 dogfood만으로는 플랫폼 중립성과 과설계 방지를 충분히 확인하기 어렵다는 사용자의 지적이 있었다.
- Flutter, Android lifecycle, desktop file preview, game/realtime UI, minimal counter negative-control을 추가 dogfood로 실행하고, 일반화 가능한 결과만 shipped skill에 반영한다.
- 스킬이 웹 검색 문제에만 맞는지, non-web execution boundary와 small-clear-code guardrail에서도 작동하는지 확인해야 하기 때문이다.
- dogfood 결과는 task-local artifacts에 보존하고, platform-specific guidance는 아직 shipped skill에 넣지 않는다.

**2026-04-26**
- 가상 dogfood만으로는 실제 프로젝트의 작은 plain JavaScript 상호작용과 reload/navigation 정책을 충분히 확인하기 어렵다는 판단이 있었다.
- itdonga-cms의 admin-toast-editor.js와 common.js를 읽기 전용 real-project dogfood 입력으로 사용하고, 원본 프로젝트는 수정하지 않는다.
- 실제 코드에서 스킬이 문제 없는 경로와 실제 freshness/presentation 위험을 구분하는지 확인하기 위해서다.
- real-project 산출물은 task-local dogfood에만 저장하고, shipped skill에는 reload/navigation freshness boundary 같은 일반 규칙만 반영한다.

**2026-04-26**
- 실제 GitHub 배포를 앞두고 README와 shipped skill의 공개 표면을 기존 공개 스킬 리포와 맞춰야 했다.
- README는 즉시 설치 가능한 공개 안내로 전환하고 support 섹션을 추가하되, skill contract 자체는 현재 107줄 축약본을 유지한다.
- project-context 최신 관례는 README를 소개와 설치 진입점으로 작게 유지하고, shipped skill은 실행 계약만 갖게 하는 쪽이다. 현재 SKILL.md는 validation을 통과했고 dogfood 반영 후 핵심 판단 절차가 남아 있어 추가 확장보다 배포 안정화가 맞다.
- 배포 전 수정은 README와 reference 드리프트 정리에 제한하고, 스킬 본문은 release candidate로 취급한다.
