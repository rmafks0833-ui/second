# SecondBrain — Schema

이것은 나 자신에 대한 개인 위키다. 외부 지식이 아니라 내가 누구인지, 뭘 경험했는지, 어떤 생각을 하는지를 기록하고 연결하는 시스템이다. Claude Code가 wiki를 유지하고, 나는 Obsidian으로 읽고 탐색한다.

---

## 역할 분담

- **나 (사용자)**: 소스를 제공한다. 일기, 생각, 대화, 경험, 읽은 것들. 방향을 잡고 질문을 던진다.
- **Claude**: wiki를 쓰고 유지한다. 요약, 연결, 분류, 업데이트, 교차 참조 — 모든 유지보수를 담당한다.
- **Obsidian**: 결과물을 읽는 IDE. 그래프 뷰로 연결을 탐색하고, 페이지를 열어 읽는다.

---

## 디렉토리 구조

```
SecondBrain/
├── CLAUDE.md                  ← 이 파일. 스키마.
├── raw/                       ← 내가 추가하는 소스 파일 (불변. Claude는 읽기만)
│   ├── informations/          ← 1인칭 직접 기록 (가장 높은 신뢰도)
│   ├── memories/              ← 과거 시점의 AI가 본 나 (참고용, 검증 필요)
│   └── external/              ← 외부 관점 2차 자료 (출처 명시 후 신중히 통합)
└── wiki/                      ← Claude가 쓰고 유지하는 파일들
    ├── index.md               ← 전체 페이지 카탈로그
    ├── log.md                 ← 시간순 작업 로그
    ├── overview.md            ← 나에 대한 최상위 요약 (진화하는 자화상)
    ├── sources/               ← 소스 요약 페이지
    ├── people/                ← 등장 인물 페이지
    ├── concepts/              ← 개념/가치/패턴 페이지
    ├── experiences/           ← 사건/기억 페이지
    └── threads/               ← 진행 중인 서사 페이지
```

---

## 소스 신뢰도

소스가 어느 하위 폴더에 있는지에 따라 가중치가 다르다:

| 폴더 | 의미 | 신뢰도 | 처리 방식 |
|------|------|--------|----------|
| `raw/informations/` | 낮것의 나. 지금 내가 직접 쓴 것. | **최고** | 그대로 통합 |
| `raw/memories/` | 과거 시점의 AI가 본 나. | **참고** | 지금 나와 비교해 검증 후 통합 |
| `raw/external/` | 외부 관점 2차 자료. | **낮음** | 출처 명시 후 신중히 통합 |

---

## 페이지 타입

### `sources/` — 소스 요약
처리한 소스 하나당 한 페이지.

```yaml
---
type: source
source_folder: informations / memories / external
date_ingested: YYYY-MM-DD
original_file: raw/informations/파일명.md
---
```

### `people/` — 등장 인물
소스에서 언급된 중요한 사람들.

```yaml
---
type: person
relationship: family/friend/mentor/colleague/self
updated: YYYY-MM-DD
---
```

### `concepts/` — 개념/가치/패턴
반복 등장하는 주제, 가치, 행동 패턴, 멘탈 모델.

```yaml
---
type: concept
category: value / pattern / belief / mental-model
confidence: high/medium/low
updated: YYYY-MM-DD
---
```

### `experiences/` — 사건/기억
시간에 붙어있는 사건, 기억, 전환점.

```yaml
---
type: experience
period: YYYY or YYYY-MM-DD
updated: YYYY-MM-DD
tags: [work, relationship, turning-point]
---
```

### `threads/` — 진행 중인 서사
지금도 진행 중인 이야기 — 프로젝트, 관계의 궤적, 반복되는 고민.

```yaml
---
type: thread
status: active/resolved/paused
updated: YYYY-MM-DD
---
```

---

## 연산 (Operations)

### 1. Ingest — 새 소스 흡수

사용자가 `raw/<하위폴더>/`에 파일을 넣고 "이거 읽어줘" 유형의 요청을 하면:

1. **소스 신뢰도 확인** — 어느 하위 폴더인지 파악해 가중치를 결정한다.
2. **소스를 끝까지 읽는다.**
3. **핵심을 사용자와 짧은 대화로 정리한다.** 사용자가 강조하고 싶은 지점을 확인한다.
4. **`wiki/sources/<소스명>.md`에 요약 페이지를 쓴다** (frontmatter 포함).
5. **영향받는 페이지를 갱신한다:**
   - 등장 인물 → `wiki/people/<이름>.md` (없으면 생성)
   - 개념/가치/패턴 → `wiki/concepts/<주제>.md` (없으면 생성)
   - 사건/기억 → `wiki/experiences/<시간>.md` (없으면 생성)
   - 진행 중 서사 → `wiki/threads/<스레드>.md` (없으면 생성)
6. **`wiki/overview.md`에 현재 상태 반영이 필요하면 미세 조정한다.**
7. **`wiki/index.md`에 신규 페이지를 추가하고**, 갱신된 페이지의 `updated` 날짜를 바꾼다.
8. **`wiki/log.md`에 ingest 항목을 append한다.**

하나의 소스가 보통 5~15개 페이지를 건드린다. 이게 정상이다.

---

### 2. Query — 나에 대한 질문

"내가 요즘 X에 대해 어떻게 느끼고 있지?", "나는 왜 반복해서 Y하지?" 같은 질문:

1. `wiki/index.md`를 먼저 스캔해 관련 페이지를 찾는다.
2. 페이지를 읽고 종합해 답한다. 반드시 `[[페이지]]` 위키링크로 근거를 인용한다.
3. 답 자체가 가치 있으면 (패턴 분석, 비교, 새로 발견한 연결 등) 사용자에게 "이걸 새 페이지로 저장할까요?" 물어본다. YES면 적절한 폴더에 저장.
4. `wiki/log.md`에 query 항목을 append.

**답변 형식:**
- 복잡한 분석 → 새 markdown 페이지
- 비교 → 표
- 시간순 → 타임라인

---

### 3. Lint — 건강 점검

사용자가 "lint해줘" 요청 시:

- **페이지 간 모순** — 단, "과거의 나 ↔ 현재의 나"는 모순이 아니라 변화로 기록한다.
- **최근 소스가 뒤집은 오래된 단정**
- **고아 페이지** (들어오는 링크 없음)
- **자주 언급되지만 독립 페이지가 없는 주제** → 생성 제안
- **누락된 상호 링크**
- **다음에 탐구할 만한 열린 질문 제안**

---

## `overview.md` 관리

가장 중요한 파일. "나는 누구인가"에 대한 진화하는 답. 새 소스를 처리할 때마다 필요한 경우 업데이트한다.

필수 섹션:
1. **핵심 정체성** — 가장 근본적인 자기 규정
2. **현재 단계** — 지금 어느 시점에 있는가
3. **핵심 가치** — 행동을 이끄는 것들
4. **현재 집중** — 지금 무엇을 하고 있는가
5. **반복 패턴** — 자주 나타나는 경향들
6. **열린 질문** — 아직 답이 없는 나에 대한 질문들

---

## `log.md` 형식

```markdown
## [YYYY-MM-DD] ingest | 소스 제목
- 처리한 내용 요약
- 업데이트한 페이지 목록
- 발견한 새 연결/인사이트

## [YYYY-MM-DD] query | 질문 내용
- 답변 요약
- 생성된 페이지 (있는 경우)

## [YYYY-MM-DD] lint
- 발견된 문제
- 수행한 수정
```

각 항목은 `## [` 로 시작한다 — `grep "^## \[" wiki/log.md | tail -5` 로 최근 5개 추출 가능.

---

## `index.md` 형식

```markdown
# Index

페이지 수: N | 마지막 업데이트: YYYY-MM-DD

## Overview
- [[overview]] — 나에 대한 최상위 요약

## Sources
- [[sources/소스명]] — 한줄 설명 (YYYY-MM-DD)

## People
- [[people/이름]] — 관계

## Concepts
- [[concepts/주제]] — 한줄 설명

## Experiences
- [[experiences/시간]] — 한줄 설명

## Threads
- [[threads/스레드명]] — 현황
```

---

## 교차 참조 규칙

- Obsidian 위키링크 문법: `[[파일명]]` 또는 `[[파일명|표시할 텍스트]]`
- 페이지 내 **첫 번째 언급에만** 링크 추가 (반복 링크 금지)
- 관련성 있는 경우만 링크 — 억지로 연결하지 않는다

---

## 원칙

1. **판단 금지**: 관찰하고 기록한다. 좋고 나쁨을 평가하지 않는다.
2. **진화 허용**: 과거의 생각과 현재의 생각이 다르면 둘 다 기록한다. 시간 딱지를 붙인다.
3. **모순 플래그**: 충돌하는 정보를 삭제하지 않는다. 명시적으로 표시한다.
4. **사용자 언어 따르기**: 한국어로 대화하면 한국어로 쓴다.
5. **간결함**: 페이지는 길게 쓰지 않는다. 핵심만, 명확하게.
