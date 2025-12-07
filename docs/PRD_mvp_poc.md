# PRD: 키워드 레이더 MVP (PoC 구현)

**Version:** 1.0.0
**Created:** 2025-12-08
**Status:** Ready for Implementation
**Goal:** 2주 내 PoC 완료 → 핵심 가설 검증

---

## 1. MVP 목표 및 범위

### 1.1 검증할 핵심 가설

| # | 가설 | 검증 방법 |
|---|------|----------|
| H1 | 펨코/더쿠 크롤링이 Cloudflare 차단 없이 가능하다 | 1시간 연속 수집 성공 |
| H2 | 5분 단위 데이터 갱신으로 "화력 지수" 계산이 유의미하다 | 실제 이슈와 랭킹 일치 여부 |
| H3 | 숫자 롤링 + 실시간 갱신이 "도파민" 효과를 준다 | 사용자 테스트 피드백 |

### 1.2 MVP 범위 (In Scope)

```
✅ 포함                          ❌ 제외 (v1.1 이후)
─────────────────────────────────────────────────────
크롤러: 펨코, 더쿠 2개           종토방 (익명 채팅)
UI: 랭킹 리스트 + 숫자 롤링      떡상/떡락 투표
실시간: Supabase Realtime        키워드 상세 차트
화력 계산: 조회수+댓글 가중치    VI 발동 애니메이션
원문 링크: 딥링크 제공           알림/푸시
```

### 1.3 성공 기준 (Success Criteria)

| 지표 | 목표 |
|------|------|
| 크롤링 성공률 | > 95% (1시간 기준) |
| 데이터 갱신 주기 | 5분 ± 30초 |
| UI 초기 로드 | < 2초 |
| 실시간 반영 지연 | < 1초 (DB INSERT → UI 갱신) |

---

## 2. 기술 스택 (확정)

### 2.1 아키텍처 개요

```
┌─────────────────────────────────────────────────────────────────┐
│                        MVP Architecture                          │
└─────────────────────────────────────────────────────────────────┘

[Communities]         [Backend]              [Frontend]
┌──────────┐        ┌───────────┐          ┌───────────┐
│  펨코    │───┐    │  Python   │          │  Next.js  │
│ (포텐)   │   │    │  Crawler  │          │    15     │
└──────────┘   │    │           │          │           │
               ├───▶│ (5분 주기)│          │  Zustand  │
┌──────────┐   │    └─────┬─────┘          │           │
│  더쿠    │───┘          │                │  Framer   │
│ (핫게)   │              │ INSERT         │  Motion   │
└──────────┘              ▼                └─────┬─────┘
                    ┌───────────┐                │
                    │ Supabase  │◀───Realtime────┘
                    │ PostgreSQL│
                    └───────────┘
```

### 2.2 스택 상세

| Layer | Technology | Version | 선택 이유 |
|-------|-----------|---------|----------|
| **Frontend** | Next.js | 15.x | App Router, RSC |
| | Zustand | 5.x | 가벼운 상태 관리 |
| | Tailwind CSS | 3.x | 빠른 스타일링 |
| | Framer Motion | 11.x | 숫자 롤링, 순위 변동 |
| | @supabase/supabase-js | 2.x | Realtime 연동 |
| **Backend** | Python | 3.11+ | 크롤링 생태계 |
| | Playwright | 1.x | Cloudflare 우회 |
| | Kiwi | 0.x | 한국어 형태소 (선택) |
| **Database** | Supabase | - | PostgreSQL + Realtime |
| **Infra** | Vercel | - | Next.js 배포 |
| | Local/Railway | - | Python Worker |

---

## 3. 데이터 모델 (최소 스키마)

### 3.1 ERD

```
┌──────────────────┐       ┌──────────────────┐
│     sources      │       │      posts       │
├──────────────────┤       ├──────────────────┤
│ id (PK)          │───┐   │ id (PK)          │
│ name             │   │   │ source_id (FK)   │──┐
│ base_url         │   └──▶│ title            │  │
│ board_path       │       │ url              │  │
│ is_active        │       │ views            │  │
└──────────────────┘       │ comments         │  │
                           │ author           │  │
                           │ created_at       │  │
                           │ crawled_at       │  │
                           └──────────────────┘  │
                                                 │
┌──────────────────┐       ┌──────────────────┐  │
│    keywords      │       │  keyword_stats   │  │
├──────────────────┤       ├──────────────────┤  │
│ id (PK)          │───┐   │ id (PK)          │  │
│ keyword          │   │   │ keyword_id (FK)  │──┘
│ first_seen       │   └──▶│ source_id (FK)   │
│ total_mentions   │       │ mention_count    │
└──────────────────┘       │ momentum         │
                           │ recorded_at      │
                           └──────────────────┘
```

### 3.2 DDL (Supabase SQL)

```sql
-- 1. 소스 (크롤링 대상 커뮤니티)
CREATE TABLE sources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) NOT NULL,           -- '펨코', '더쿠'
    base_url VARCHAR(255) NOT NULL,
    board_path VARCHAR(255) NOT NULL,    -- '/hotdeal', '/hot'
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 초기 데이터
INSERT INTO sources (name, base_url, board_path) VALUES
('펨코', 'https://www.fmkorea.com', '/hotdeal'),
('더쿠', 'https://theqoo.net', '/hot');

-- 2. 게시글 원본 (크롤링 결과)
CREATE TABLE posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_id UUID REFERENCES sources(id),
    external_id VARCHAR(50),             -- 원본 게시글 ID
    title VARCHAR(500) NOT NULL,
    url VARCHAR(500),
    views INT DEFAULT 0,
    comments INT DEFAULT 0,
    author VARCHAR(100),
    posted_at TIMESTAMPTZ,               -- 원글 작성 시간
    crawled_at TIMESTAMPTZ DEFAULT NOW(),

    UNIQUE(source_id, external_id)
);

-- 인덱스
CREATE INDEX idx_posts_crawled ON posts(crawled_at DESC);
CREATE INDEX idx_posts_source ON posts(source_id, crawled_at DESC);

-- 3. 키워드 (추출된 핵심 단어)
CREATE TABLE keywords (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    keyword VARCHAR(100) NOT NULL UNIQUE,
    first_seen TIMESTAMPTZ DEFAULT NOW(),
    total_mentions INT DEFAULT 0
);

-- 4. 키워드 시계열 통계
CREATE TABLE keyword_stats (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    keyword_id UUID REFERENCES keywords(id),
    mention_count INT DEFAULT 0,         -- 해당 시간대 언급 수
    momentum DECIMAL(10,2) DEFAULT 0,    -- 화력 지수
    recorded_at TIMESTAMPTZ DEFAULT NOW(),

    UNIQUE(keyword_id, recorded_at)
);

-- 인덱스
CREATE INDEX idx_stats_recorded ON keyword_stats(recorded_at DESC);
CREATE INDEX idx_stats_momentum ON keyword_stats(momentum DESC);

-- 5. 실시간 랭킹 뷰 (캐시용)
CREATE OR REPLACE VIEW keyword_rankings AS
SELECT
    k.id,
    k.keyword,
    k.total_mentions,
    COALESCE(latest.momentum, 0) as momentum,
    COALESCE(prev.momentum, 0) as prev_momentum,
    COALESCE(latest.momentum, 0) - COALESCE(prev.momentum, 0) as momentum_delta,
    latest.recorded_at as last_updated
FROM keywords k
LEFT JOIN LATERAL (
    SELECT momentum, recorded_at
    FROM keyword_stats
    WHERE keyword_id = k.id
    ORDER BY recorded_at DESC
    LIMIT 1
) latest ON true
LEFT JOIN LATERAL (
    SELECT momentum
    FROM keyword_stats
    WHERE keyword_id = k.id
    ORDER BY recorded_at DESC
    OFFSET 1 LIMIT 1
) prev ON true
WHERE latest.recorded_at > NOW() - INTERVAL '1 hour'
ORDER BY momentum DESC
LIMIT 20;

-- 6. Realtime 활성화 (Supabase 대시보드에서도 가능)
ALTER PUBLICATION supabase_realtime ADD TABLE keyword_stats;
```

---

## 4. 기능 명세

### 4.1 크롤러 (Python Worker)

#### 4.1.1 크롤링 대상

| 사이트 | 게시판 | URL | 수집 항목 |
|--------|--------|-----|----------|
| 펨코 | 포텐터짐 | `/hotdeal` | 제목, 조회수, 댓글수, 작성자, 시간 |
| 더쿠 | 핫게시판 | `/hot` | 제목, 조회수, 댓글수, 작성자, 시간 |

#### 4.1.2 크롤링 로직

```python
# 의사 코드
async def crawl_cycle():
    """5분마다 실행"""
    for source in active_sources:
        posts = await crawl_board(source)      # 1페이지 (20~30개)

        for post in posts:
            # 1. 게시글 저장/업데이트
            upsert_post(post)

            # 2. 키워드 추출 (제목에서 명사 추출)
            keywords = extract_keywords(post.title)

            for keyword in keywords:
                # 3. 키워드 카운트 증가
                increment_keyword_count(keyword)

        # 4. 화력 지수 계산 (5분 전 대비)
        calculate_momentum()

        await asyncio.sleep(30)  # 사이트별 30초 간격
```

#### 4.1.3 화력 지수 계산

```python
def calculate_momentum(keyword_id: str) -> float:
    """
    화력 지수 = (현재_언급수 - 5분전_언급수) * 가중치

    가중치:
    - 기본 언급: 1.0
    - 댓글 多 게시글: 1.5 (댓글 > 50)
    - 조회수 多 게시글: 1.2 (조회 > 1000)
    """
    now_count = get_mention_count(keyword_id, interval='5min')
    prev_count = get_mention_count(keyword_id, interval='5min', offset='5min')

    if prev_count == 0:
        return now_count * 100  # 신규 키워드

    return ((now_count - prev_count) / prev_count) * 100
```

#### 4.1.4 Cloudflare 우회 설정

```python
from playwright.async_api import async_playwright
from playwright_stealth import stealth_async

async def create_browser():
    playwright = await async_playwright().start()
    browser = await playwright.chromium.launch(
        headless=True,
        args=[
            '--disable-blink-features=AutomationControlled',
            '--no-sandbox',
        ]
    )
    context = await browser.new_context(
        viewport={'width': 1920, 'height': 1080},
        user_agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
    )
    page = await context.new_page()
    await stealth_async(page)  # Stealth 플러그인 적용
    return browser, page
```

### 4.2 Frontend (Next.js)

#### 4.2.1 페이지 구조

```
app/
├── page.tsx              # 메인 대시보드
├── layout.tsx            # 다크 테마 레이아웃
├── globals.css           # Tailwind + 커스텀 스타일
└── components/
    ├── RankingBoard.tsx  # 랭킹 리스트
    ├── RankingRow.tsx    # 개별 Row (애니메이션)
    ├── MomentumNumber.tsx # 숫자 롤링 컴포넌트
    └── LiveTicker.tsx    # 상단 티커 (선택)
```

#### 4.2.2 핵심 컴포넌트: RankingBoard

```tsx
// 의사 코드
'use client';

import { useEffect } from 'react';
import { createClient } from '@supabase/supabase-js';
import { useRankingStore } from '@/store/ranking';
import RankingRow from './RankingRow';

export default function RankingBoard() {
  const { rankings, setRankings, updateRanking } = useRankingStore();
  const supabase = createClient(URL, KEY);

  useEffect(() => {
    // 1. 초기 데이터 로드
    fetchInitialRankings();

    // 2. Realtime 구독
    const channel = supabase
      .channel('keyword_stats')
      .on('postgres_changes',
        { event: 'INSERT', schema: 'public', table: 'keyword_stats' },
        (payload) => {
          // 새 데이터 → UI 자동 갱신
          updateRanking(payload.new);
        }
      )
      .subscribe();

    return () => { supabase.removeChannel(channel); };
  }, []);

  return (
    <div className="bg-[#121212] min-h-screen p-4">
      <h1 className="text-2xl font-bold text-white mb-4">
        🔥 실시간 키워드 랭킹
      </h1>
      <div className="space-y-2">
        {rankings.map((item, index) => (
          <RankingRow
            key={item.id}
            rank={index + 1}
            data={item}
          />
        ))}
      </div>
    </div>
  );
}
```

#### 4.2.3 핵심 컴포넌트: MomentumNumber (숫자 롤링)

```tsx
'use client';

import { motion, useSpring, useTransform } from 'framer-motion';
import { useEffect } from 'react';

interface Props {
  value: number;
  duration?: number;
}

export default function MomentumNumber({ value, duration = 0.8 }: Props) {
  const spring = useSpring(0, {
    stiffness: 100,
    damping: 30,
    duration
  });

  const display = useTransform(spring, (v) =>
    `${v >= 0 ? '+' : ''}${v.toFixed(0)}%`
  );

  useEffect(() => {
    spring.set(value);
  }, [value, spring]);

  return (
    <motion.span
      className={`font-mono font-bold text-lg ${
        value >= 0 ? 'text-red-500' : 'text-blue-500'
      }`}
    >
      {display}
    </motion.span>
  );
}
```

#### 4.2.4 UI 스타일 가이드

```css
/* globals.css */
:root {
  --bg-primary: #121212;
  --bg-surface: #1E1E1E;
  --text-primary: #FFFFFF;
  --text-secondary: #8E8E93;
  --accent-bull: #FF3B30;      /* 상승 - 레드 */
  --accent-bull-light: #FF3B3033;
  --accent-bear: #007AFF;      /* 하락 - 블루 */
  --accent-gold: #FFD700;      /* 1위 강조 */
}

/* 호가창 Row 스타일 */
.ranking-row {
  @apply flex items-center justify-between p-3 rounded-lg;
  @apply transition-all duration-300;
  background: linear-gradient(
    90deg,
    var(--bg-surface) 0%,
    var(--accent-bull-light) var(--momentum-percent)
  );
}

/* 숫자 폰트 */
.font-mono-num {
  font-family: 'IBM Plex Mono', monospace;
  font-variant-numeric: tabular-nums;
}
```

### 4.3 API Endpoints (선택)

MVP에서는 Supabase 직접 연동으로 충분하지만, 확장을 위해 API 레이어 정의.

| Method | Endpoint | 설명 |
|--------|----------|------|
| GET | `/api/rankings` | 실시간 Top 20 |
| GET | `/api/rankings?source=펨코` | 소스별 필터 |

---

## 5. 구현 순서 (Step by Step)

### Phase 1: 데이터 파이프라인 (Day 1-3)

```
┌─────────────────────────────────────────────────────────┐
│  Day 1: 환경 설정                                        │
├─────────────────────────────────────────────────────────┤
│  □ Supabase 프로젝트 생성                               │
│  □ 테이블 생성 (DDL 실행)                               │
│  □ Python 환경 설정 (venv, playwright install)          │
│  □ .env 파일 설정 (SUPABASE_URL, SUPABASE_KEY)          │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Day 2: 크롤러 개발                                      │
├─────────────────────────────────────────────────────────┤
│  □ 펨코 크롤러 구현 (1페이지)                           │
│  □ Cloudflare 우회 테스트                               │
│  □ 더쿠 크롤러 구현                                     │
│  □ 데이터 파싱 (제목, 조회수, 댓글수)                   │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Day 3: 데이터 적재                                      │
├─────────────────────────────────────────────────────────┤
│  □ Supabase INSERT 로직                                 │
│  □ 5분 주기 스케줄러 (APScheduler)                      │
│  □ 1시간 연속 테스트                                    │
│  □ 데이터 검증 (Supabase 대시보드 확인)                 │
└─────────────────────────────────────────────────────────┘
```

**Phase 1 완료 기준:**
- [ ] 펨코/더쿠 데이터가 5분마다 `posts` 테이블에 적재됨
- [ ] 1시간 동안 차단 없이 수집 성공

---

### Phase 2: 키워드 추출 (Day 4-5)

```
┌─────────────────────────────────────────────────────────┐
│  Day 4: 키워드 추출                                      │
├─────────────────────────────────────────────────────────┤
│  □ Kiwi 형태소 분석기 설치                              │
│  □ 제목에서 명사 추출 로직                              │
│  □ 불용어 필터링 ('오늘', '진짜', 'ㅋㅋ' 등)            │
│  □ keywords 테이블 적재                                 │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Day 5: 화력 계산                                        │
├─────────────────────────────────────────────────────────┤
│  □ 화력 지수 계산 함수 구현                             │
│  □ keyword_stats 테이블 적재                            │
│  □ 랭킹 뷰 테스트                                       │
│  □ Realtime 활성화 확인                                 │
└─────────────────────────────────────────────────────────┘
```

**Phase 2 완료 기준:**
- [ ] `keyword_rankings` 뷰에서 Top 20 조회 가능
- [ ] 실제 이슈 키워드가 상위에 노출됨

---

### Phase 3: Frontend UI (Day 6-8)

```
┌─────────────────────────────────────────────────────────┐
│  Day 6: 프로젝트 셋업                                    │
├─────────────────────────────────────────────────────────┤
│  □ Next.js 15 프로젝트 생성                             │
│  □ Tailwind CSS 설정                                    │
│  □ Supabase 클라이언트 설정                             │
│  □ 다크 테마 레이아웃                                   │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Day 7: 랭킹 UI                                          │
├─────────────────────────────────────────────────────────┤
│  □ RankingBoard 컴포넌트                                │
│  □ RankingRow (호가창 스타일)                           │
│  □ 초기 데이터 페칭                                     │
│  □ 로딩/에러 상태 처리                                  │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Day 8: 애니메이션 & Realtime                            │
├─────────────────────────────────────────────────────────┤
│  □ MomentumNumber (숫자 롤링)                           │
│  □ 순위 변동 애니메이션                                 │
│  □ Supabase Realtime 연동                               │
│  □ 자동 갱신 테스트                                     │
└─────────────────────────────────────────────────────────┘
```

**Phase 3 완료 기준:**
- [ ] 다크 테마 랭킹 보드 UI 완성
- [ ] 숫자가 0에서 목표치까지 롤링됨
- [ ] DB INSERT 시 새로고침 없이 UI 갱신됨

---

### Phase 4: 통합 테스트 (Day 9-10)

```
┌─────────────────────────────────────────────────────────┐
│  Day 9: End-to-End 테스트                                │
├─────────────────────────────────────────────────────────┤
│  □ 크롤러 → DB → UI 전체 플로우 테스트                  │
│  □ 실시간 갱신 지연 시간 측정                           │
│  □ 에러 케이스 처리 (크롤링 실패 시)                    │
│  □ 성능 최적화 (불필요한 리렌더 방지)                   │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Day 10: 배포 & 데모                                     │
├─────────────────────────────────────────────────────────┤
│  □ Next.js → Vercel 배포                                │
│  □ Python Worker → Railway/로컬 실행                    │
│  □ 데모 시나리오 준비                                   │
│  □ 피드백 수집 계획                                     │
└─────────────────────────────────────────────────────────┘
```

**Phase 4 완료 기준:**
- [ ] 배포된 URL에서 실시간 랭킹 확인 가능
- [ ] 3명 이상 테스터 피드백 수집

---

## 6. 프로젝트 구조

### 6.1 Backend (Python)

```
backend/
├── pyproject.toml           # 의존성 관리 (Poetry/pip)
├── .env.example
├── src/
│   ├── __init__.py
│   ├── main.py              # 엔트리포인트 (스케줄러)
│   ├── config.py            # 환경변수 로드
│   ├── db/
│   │   ├── __init__.py
│   │   └── supabase.py      # Supabase 클라이언트
│   ├── crawlers/
│   │   ├── __init__.py
│   │   ├── base.py          # BaseCrawler 추상 클래스
│   │   ├── fmkorea.py       # 펨코 크롤러
│   │   └── theqoo.py        # 더쿠 크롤러
│   ├── processors/
│   │   ├── __init__.py
│   │   ├── keyword.py       # 키워드 추출
│   │   └── momentum.py      # 화력 계산
│   └── models/
│       ├── __init__.py
│       └── schemas.py       # Pydantic 스키마
└── tests/
    ├── test_crawlers.py
    └── test_processors.py
```

### 6.2 Frontend (Next.js)

```
frontend/
├── package.json
├── next.config.js
├── tailwind.config.js
├── .env.local.example
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
├── components/
│   ├── RankingBoard.tsx
│   ├── RankingRow.tsx
│   ├── MomentumNumber.tsx
│   └── ui/                  # Shadcn/ui (선택)
├── store/
│   └── ranking.ts           # Zustand store
├── lib/
│   ├── supabase.ts
│   └── utils.ts
└── types/
    └── index.ts
```

---

## 7. 환경 변수

### Backend (.env)

```env
# Supabase
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_SERVICE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Crawler
CRAWL_INTERVAL_MINUTES=5
HEADLESS_BROWSER=true

# Logging
LOG_LEVEL=INFO
```

### Frontend (.env.local)

```env
# Supabase (anon key - 클라이언트용)
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## 8. 리스크 및 완화

| 리스크 | 확률 | 영향 | 완화 전략 |
|--------|------|------|----------|
| Cloudflare 차단 | 중 | 높음 | Stealth 플러그인 + 로컬 IP 사용 |
| 사이트 구조 변경 | 중 | 중 | Selector 기반 → 정규식 병행 |
| 실시간 지연 | 낮 | 중 | Supabase → WebSocket 대안 준비 |
| 키워드 노이즈 | 중 | 낮 | 불용어 사전 지속 업데이트 |

---

## 9. MVP 이후 로드맵

| 버전 | 기능 | 예상 기간 |
|------|------|----------|
| **v1.1** | VI 발동 애니메이션, 키워드 상세 차트 | +1주 |
| **v1.2** | 종토방 (익명 채팅), 떡상/떡락 투표 | +1주 |
| **v1.3** | 추가 커뮤니티 (인스티즈, 블라인드) | +1주 |
| **v2.0** | 알림/푸시, 모바일 최적화 | +2주 |

---

## 10. 체크리스트 (개발자용)

### Phase 1 완료 체크

```
□ Supabase 프로젝트 생성 완료
□ 테이블 4개 생성 (sources, posts, keywords, keyword_stats)
□ 펨코 크롤러 1회 성공
□ 더쿠 크롤러 1회 성공
□ 5분 주기 스케줄러 동작 확인
□ 1시간 연속 크롤링 성공 (차단 없음)
```

### Phase 2 완료 체크

```
□ Kiwi 형태소 분석 동작 확인
□ 키워드 추출 (명사) 정상 작동
□ 불용어 필터링 적용
□ 화력 지수 계산 로직 구현
□ keyword_stats 테이블 데이터 적재
□ keyword_rankings 뷰 조회 성공
```

### Phase 3 완료 체크

```
□ Next.js 프로젝트 생성
□ Tailwind + 다크테마 적용
□ Supabase 클라이언트 연동
□ 랭킹 리스트 UI 완성
□ 숫자 롤링 애니메이션 구현
□ Realtime 구독 → UI 자동 갱신
```

### Phase 4 완료 체크

```
□ E2E 플로우 정상 동작
□ 실시간 갱신 지연 < 1초
□ Vercel 배포 완료
□ 데모 URL 공유 가능
□ 최소 3명 피드백 수집
```

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 |
|------|------|----------|
| 1.0.0 | 2025-12-08 | MVP PRD 초기 작성 |
