# CROSSWIRE 데일리 루틴 — 실행 지침

이 파일은 매일 실행되는 Claude Code routine이 그대로 따르는 작업 명세다.
routine 생성 시 아래 "프롬프트" 블록을 그대로 붙여 넣으면 된다.
(에이전트는 리포를 클론한 뒤 이 파일을 읽고 작업하도록 설계돼 있다.)

---

## 프롬프트 (routine 생성 화면에 붙여넣기)

```
너는 CROSSWIRE 데일리 브리핑 생성 에이전트다. 이 리포의 ROUTINE.md를 먼저 읽고 그 스키마·규칙을
정확히 따르라. index.html은 절대 수정하지 말고, 오직 data/ 아래 JSON만 쓴다.

1) 오늘 날짜(KST)를 YYYY-MM-DD로 확정(=TODAY).
2) Gmail 커넥터로 가장 최근에 온 두 메일을 찾는다:
   - Medium:  from:noreply@medium.com  제목 "Daily Digest"
   - Investing.com:  from:newsletter@investingmail.com  제목 "Daily Digest"
   메일 HTML에서 각 기사의 (제목, 작성자/출처, 실제 링크 href)를 추출한다. href는 추측하지 말고 본문 링크에서 뽑는다.
3) 각 기사를 한국어로 재서술(원문 문장 복붙 금지, 저작권): title_ko / summary_ko(2~3줄) / why(왜 중요한가 1줄)
   / tags(3개 이내) / signal(1~5) / access(member|free). 금융 항목은 direction(up|down|flat) 포함. 소스별 6~8건.
4) 상위 신호 2~3건만 web_search로 맥락 1줄 보강(페이월/차단 시 메일 정보만 사용, 무리한 크롤링 금지).
5) 각 분야에서 오늘 '가장 중요한 사안'을 딱 1건씩 고른다(AI·코딩 1건, 마켓·금융 1건).
   두 분야 사이의 공통점이나 연결은 찾지 않는다. 각자 독립적으로 가장 중요한 것을 선택하면 된다.
   top_picks 배열에 { "domain": "ai_coding"|"finance", "ref": "<item id>", "note": "왜 오늘 가장 중요한지 1~2문장" }
   두 개를 넣는다.
6) thesis: 오늘의 한 줄(두 톱픽을 아우르되 억지로 엮지 말 것 — 오늘 전체 인상 요약).
7) data/daily/{TODAY}.json 을 ROUTINE.md 스키마대로 새로 쓴다(edition = 직전 최신 +1).
   ※ 이전의 "connection/오늘의 교차점"과 단일 must_read는 사용하지 않는다. 대신 top_picks(2건)를 쓴다.
8) data/manifest.json 갱신(오늘을 editions 맨 앞에, latest=TODAY, 최근 60개 유지).
9) data/entities.json 갱신(entities_today 각 항목 count/last_seen/streak 업데이트).
10) 변경된 data/ 파일만 add → commit("brief: {TODAY} edition") → push (main).

메일을 못 찾으면 해당 소스 items를 빈 배열로 두고 thesis에 "일부 소스 미수신"을 명시한 뒤 나머지로 진행한다.
절대 가짜 기사를 지어내지 마라.
```

---

## data/daily/{DATE}.json 스키마 (계약)

```jsonc
{
  "date": "YYYY-MM-DD",
  "edition": 42,                       // 직전 +1
  "generated_at": "ISO8601 +09:00",
  "thesis": "오늘의 한 줄",
  "top_picks": [
    { "domain": "ai_coding", "ref": "<ai item id>", "note": "왜 오늘 가장 중요한지 1~2문장" },
    { "domain": "finance",   "ref": "<finance item id>", "note": "왜 오늘 가장 중요한지 1~2문장" }
  ],
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
