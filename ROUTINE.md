# CROSSWIRE 데일리 루틴 — 실행 지침

이 파일은 매일 실행되는 Claude Code routine이 그대로 따르는 작업 명세다.
routine 생성 시 아래 "프롬프트" 블록을 그대로 붙여 넣으면 된다.
(에이전트는 리포를 클론한 뒤 이 파일을 읽고 작업하도록 설계돼 있다.)

---

## 프롬프트 (routine 생성 화면에 붙여넣기)

```
너는 CROSSWIRE 데일리 브리핑을 생성하는 에이전트다. 이 리포의 ROUTINE.md를 먼저 읽고,
아래 절차를 순서대로 수행한 뒤 커밋·푸시하라. HTML(index.html)은 절대 수정하지 마라.
오직 data/ 아래 JSON만 쓴다.

[1] 오늘 날짜(KST)를 YYYY-MM-DD로 확정한다. 이 값이 TODAY.

[2] Gmail 커넥터로 오늘 도착한 두 메일을 찾는다.
    - Medium: from:noreply@medium.com  제목에 "Daily Digest"
    - Investing.com: from:newsletter@investingmail.com  제목에 "Daily Digest"
    각 메일의 HTML 본문에서 개별 기사의 (제목, 작성자/출처, 실제 href URL)을 추출한다.
    href는 반드시 메일 본문의 링크에서 뽑는다(추측 금지). member-only 표시는 access="member".

[3] 각 기사를 한국어로 재서술한다. 원문 문장을 그대로 옮기지 말 것(저작권).
    - title_ko: 제목 의역(핵심 보존)
    - summary_ko: 2~3줄, 자기 문장으로 재작성한 요약
    - why: "왜 중요한가" 한 줄 (교차·맥락·독자 관점)
    - tags: 3개 이내 (모델/기업/기법 등 고유명 우선)
    - signal: 1~5 정수 (오늘 기준 중요도)
    - 금융 항목은 direction: "up"|"down"|"flat" 를 추론해 넣는다.
    항목 수는 소스별 6~8개로 제한한다(신호 높은 순).

[4] 가능하면 상위 신호 2~3개만 web_search로 맥락을 보강한다(수치·배경 1줄).
    페이월/직접 fetch가 막히면 메일 정보만으로 진행한다. 무리하게 원문 긁지 말 것.

[5] "오늘의 교차점(connection)"을 만든다 — 이게 이 제품의 핵심이다.
    AI 항목 1개와 금융 항목 1개 사이의 '실제' 연결을 찾아 3~4문장으로 서술한다.
    억지 연결 금지. 없으면 가장 강한 매크로-기술 접점을 택하되 근거를 밝힌다.
    ai_ref / market_ref 에 해당 item id를 넣는다.

[6] must_read: 오늘 가장 볼 가치 있는 항목 id 1개 + note 한 줄.

[7] thesis: 오늘 전체를 관통하는 한 줄(편집자 헤드라인).

[8] data/daily/{TODAY}.json 을 아래 스키마로 새로 쓴다(있으면 덮어쓴다).
    edition 번호는 직전 최신 edition +1.

[9] data/manifest.json 갱신: editions 배열 맨 앞에 오늘 항목 추가(중복 날짜면 교체),
    latest=TODAY, updated=TODAY. 최근 60개만 유지.

[10] data/entities.json 갱신: 오늘 entities_today의 각 항목에 대해
     - 신규면 추가(count=1, streak=1, first_seen=TODAY)
     - 기존이면 count+1, last_seen=TODAY, 어제도 등장했으면 streak+1 아니면 streak=1
     updated=TODAY.

[11] 변경된 data/ 파일만 git add → commit (메시지: "brief: {TODAY} edition") → push.
     푸시 대상 브랜치는 리포 설정을 따른다(무제한 푸시 허용 시 main).

메일을 못 찾으면: 해당 소스 items를 빈 배열로 두고, thesis에 "일부 소스 미수신"을 명시한 뒤
나머지만으로 에디션을 만든다. 절대 가짜 기사를 지어내지 마라.
```

---

## data/daily/{DATE}.json 스키마 (계약)

```jsonc
{
  "date": "YYYY-MM-DD",
  "edition": 42,                       // 직전 +1
  "generated_at": "ISO8601 +09:00",
  "thesis": "오늘의 한 줄",
  "connection": {
    "label": "오늘의 교차점",
    "body": "3~4문장 연결 서술",
    "ai_ref": "<ai item id>",
    "market_ref": "<finance item id>"
  },
  "must_read": { "ref": "<item id>", "note": "한 줄" },
  "sections": [
    {
      "source": "Medium Daily Digest",
      "source_url": "https://medium.com/",
      "domain": "ai_coding",           // 렌더러가 파란 컬럼으로
      "label": "AI · 코딩",
      "items": [ /* Item */ ]
    },
    {
      "source": "Investing.com Daily Digest",
      "source_url": "https://www.investing.com/news/",
      "domain": "finance",             // 렌더러가 앰버 컬럼으로
      "label": "마켓 · 국제금융",
      "items": [ /* Item(+direction) */ ]
    }
  ],
  "entities_today": ["China","GLM-5.2", "..."]
}
```

### Item
```jsonc
{
  "id": "kebab-case-고유",
  "title_ko": "…",
  "summary_ko": "…",            // 원문 재서술
  "why": "…",
  "author": "…",
  "url": "https://…",           // 메일에서 추출한 실제 링크
  "tags": ["…","…"],
  "access": "member" | "free",
  "signal": 1-5,
  "direction": "up|down|flat"    // 금융 항목만
}
```

> 렌더러(index.html)는 이 스키마만 안다. 필드명을 바꾸면 화면이 깨지므로,
> 스키마를 바꿀 때만 index.html도 함께 수정한다. 평상시 routine은 index.html을 건드리지 않는다.
