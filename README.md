# CROSSWIRE — 오늘의 AI·마켓 교차 브리핑

매일 아침, 구독 중인 두 뉴스레터(Medium Daily Digest · Investing.com Daily Digest)를
Claude Code **routine**이 읽어 하나의 교차 브리핑으로 재구성하고, GitHub Pages에 자동 게시한다.

이메일과의 차별점 = **종합 · 교차 · 축적**
- 두 사일로(AI코딩 / 국제금융)를 하나로 합침
- 매일 **오늘의 교차점**: AI 항목 ↔ 금융 항목의 실제 연결 1개
- **모멘텀**: 모델·기업·기법이 며칠에 걸쳐 반복되는지 누적 추적
- **아카이브**: 과거 에디션 열람 (이메일은 스크롤로 사라짐)

---

## 폴더 구조

```
crosswire/
├── index.html            # 고정 셸. JSON을 fetch해 렌더. routine은 이 파일을 건드리지 않음
│                         #  (fetch 실패 시 내장 폴백 데이터로 프리뷰 동작)
├── ROUTINE.md            # routine이 매일 따르는 실행 지침 + JSON 스키마(계약)
├── README.md
└── data/                 # routine이 쓰는 유일한 디렉터리
    ├── manifest.json     #  에디션 목록(최신순). 매 실행마다 갱신
    ├── entities.json     #  엔티티 누적 카운트(모멘텀 소스). 매 실행마다 갱신
    └── daily/
        ├── 2026-07-04.json   # 하루 = 파일 1개 (에이전트가 새로 씀)
        └── 2026-07-03.json
```

**설계 원칙:** 에이전트가 매일 커다란 HTML을 직접 편집하면 깨지기 쉽다.
그래서 **HTML은 고정**하고 routine은 **작은 JSON만** 쓴다. 화면은 클라이언트에서 그 JSON을 렌더한다.
가장 견고하고, 데이터가 곧 재사용 가능한 연구 자산(시계열)이 된다.

---

## 1. GitHub Pages 배포 (최초 1회)

```bash
git init && git add . && git commit -m "init crosswire"
git branch -M main
git remote add origin https://github.com/<user>/crosswire.git
git push -u origin main
```

리포 Settings → Pages → Source를 **main / (root)** 로 지정.
→ `https://<user>.github.io/crosswire/` 에서 열림.
`index.html`이 상대경로로 `./data/*.json`을 fetch하므로 별도 서버 불필요.

---

## 2. routine 설정 & 연동 (claude.ai/code/routines)

Pro/Max/Team/Enterprise + 웹용 Claude Code 활성화 필요. 하루 1회면 Pro 한도(하루 5회) 내.

| 항목 | 설정값 |
|---|---|
| **Repository** | `<user>/crosswire` (main 브랜치) |
| **Prompt** | ROUTINE.md의 "프롬프트" 블록을 붙여넣기 |
| **Trigger** | Scheduled → 매일 07:30 (KST 기준으로 입력, UTC 변환은 자동 처리) |
| **Connectors** | **Gmail** (필수). 나머지 커넥터는 제거해 권한 최소화 |
| **Environment** | Default (Trusted network) 로 충분 |

### 핵심 연동 포인트 3가지

**① Gmail은 스크래핑하지 말고 커넥터로 읽는다 (가장 안정적)**
이미 메일로 오는 콘텐츠라 사이트를 긁을 필요가 없다. Gmail 커넥터 트래픽은
Anthropic 서버를 경유하므로 Allowed domains에 호스트를 추가하지 않아도 동작한다.

**② main 브랜치 푸시 권한**
routine은 기본적으로 `claude/` 접두사 브랜치에만 푸시한다. Pages가 main을 서빙하므로,
리포별 설정에서 **"Allow unrestricted branch pushes"** 를 켜서 main 직접 커밋을 허용한다.
- 더 보수적으로 가려면: 이 옵션을 끄고 routine이 `claude/brief-YYYY-MM-DD` 브랜치로 PR을 열게 한 뒤,
  GitHub Action(auto-merge)이나 수동 머지로 반영. 자동화 취지엔 ①이 낫다.

**③ 원문 보강용 네트워크(선택)**
상위 항목 맥락 보강은 내장 `web_search`(Anthropic 경유)로 처리한다.
특정 언론사/RSS를 **직접 fetch**하려는 경우에만 Environment의 Allowed domains에 해당 도메인을 추가한다.
(medium.com 등은 페이월이라 직접 fetch 신뢰도가 낮음 → 메일 기반 권장)

---

## 3. 저작권 / 운영 메모

- 요약은 **원문 재서술 + 출처 링크**. 유료·회원 전용 전문을 그대로 옮기지 않는다.
- 공개 페이지라면 이 형태(짧은 의역 + 링크)가 안전하고 연구소 성격에도 맞는다.
- routine은 research preview 단계 — 초기 며칠은 실행 로그를 확인해 안정화한다.
- 메일 미수신 시 해당 소스는 빈 섹션 + thesis에 명시. 가짜 기사 생성 금지(프롬프트에 규정됨).

---

## 확장 아이디어

- `entities.json`에 일자별 시계열을 쌓아 **모멘텀 차트**(며칠째 상승/신규 급등) 추가
- 소스 추가(예: arXiv, KCI 신착)는 `sections[]`에 domain/label만 늘리면 됨
- `data/`가 그대로 데이터셋 → 교차 담론 분석·논문 소스로 재사용
