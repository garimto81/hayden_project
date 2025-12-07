# PRD: 키워드 레이더 (Keyword Radar) v2.0

**Version:** 2.0.0
**Created:** 2025-12-08
**Status:** Ready for Development
**Owner:** Hayden Team

---

## 1. Executive Summary

### 1.1 프로젝트 배경

마케터들은 매일 트렌드를 모니터링하지만, 기존 도구들은 다음 문제를 가짐:
- Google Trends: 업데이트 지연 (실시간 X)
- 블로그 차트: 이미 대중화된 키워드 (선점 불가)
- 커뮤니티 직접 순회: 시간 소모 (비효율)

**기회**: 커뮤니티 실시간 데이터를 **주식 거래소 UX**로 시각화하여, "지금 바로 콘텐츠로 만들어야 할 키워드"를 직관적으로 발견하게 함.

### 1.2 제품 비전

> "지금 안 보면 떡락! 마케터를 위한 실시간 트렌드 호가창"

정적인 트렌드 리포트를 **도파민 자극형 실시간 대시보드**로 전환하여, 마케터가 트렌드 선점의 긴박함을 체감하고 행동하게 만든다.

### 1.3 핵심 가치 제안 (Value Proposition)

| 기존 솔루션 | 키워드 레이더 |
|------------|--------------|
| 정적인 텍스트 리스트 | 살아 움직이는 숫자 애니메이션 |
| 1일 단위 업데이트 | 5분 단위 실시간 갱신 |
| "정보 제공" | "지금 이거 안 보면 큰일!" (긴박감) |
| 단독 열람 | 종토방 커뮤니티 (소속감) |

---

## 2. 타겟 유저 (User Persona)

### 2.1 Primary Persona: "콘텐츠 마케터 민지"

| 항목 | 상세 |
|------|------|
| **연령/직업** | 28세, 뷰티 브랜드 콘텐츠 마케터 (3년차) |
| **일일 루틴** | 오전: 트렌드 리서치 (1시간+) → 콘텐츠 기획 → 제작 |
| **Pain Point** | 펨코/인스티즈/더쿠 직접 순회하며 "뭐가 뜨나" 체크 → 시간 낭비 |
| **Desired Outcome** | 출근하자마자 "오늘 뭐 뜨나" 한눈에 파악 → 빠른 콘텐츠 선점 |
| **행동 특성** | 숫자/순위에 민감, 빠른 판단 선호, SNS 헤비유저 |

### 2.2 Secondary Persona: "바이럴 대행사 PD 준혁"

| 항목 | 상세 |
|------|------|
| **연령/직업** | 32세, 바이럴 마케팅 대행사 PD |
| **Pain Point** | 클라이언트에게 "지금 이게 뜬다" 설득할 데이터 부족 |
| **Desired Outcome** | 차트/수치로 트렌드 증명 → 빠른 기획안 통과 |
| **행동 특성** | 데이터 기반 의사결정, 리포트 작성 빈번 |

### 2.3 Anti-Persona (타겟 아님)

- 장기 트렌드 분석이 필요한 전략 기획자 (우리는 "실시간" 특화)
- 해외 트렌드 추적 필요 (국내 커뮤니티 한정)

---

## 3. 유저 저니 (User Journey)

```
┌─────────────────────────────────────────────────────────────────────┐
│  [진입]          [발견]           [탐색]           [행동]           │
│                                                                     │
│  페이지 접속  →  숫자 롤링     →  급등 키워드    →  원문 확인      │
│  ↓              애니메이션        클릭              또는 저장       │
│  "뭔가 터지고    으로 긴박감      ↓                ↓               │
│   있다" 직감     형성            상세 차트 확인    콘텐츠 기획     │
│                                  ↓                                 │
│                                  종토방에서                        │
│                                  반응 확인                         │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.1 상세 시나리오

**Scenario 1: 오전 트렌드 체크**
1. 민지, 9시 출근 후 키워드 레이더 접속
2. 메인 화면: 숫자가 0% → +1,200%까지 롤링되며 "긴박함" 자극
3. 1위 키워드 "[연예인A 열애설]" 확인 → 🔥 아이콘 + 빨간 불기둥
4. 클릭 → 캔들 차트로 지난 2시간 급등 추이 확인
5. "종토방" 탭 → 다른 마케터들 반응 확인 ("이건 진짜 터진다")
6. 원문 링크 클릭 → 펨코 원글 확인 → 콘텐츠 기획 시작

**Scenario 2: VI 발동 알림**
1. 준혁, 업무 중 알림 수신 "🚨 [키워드B] VI 발동! +45% (5분)"
2. 즉시 대시보드 접속 → 빨갛게 깜빡이는 해당 Row 확인
3. 클라이언트에게 카톡: "지금 이거 급상승 중입니다. 바로 콘텐츠 제작할까요?"

---

## 4. 기능 요구사항 (Functional Requirements)

### 4.1 우선순위 매트릭스 (MoSCoW)

#### Must Have (MVP 필수)

| ID | 기능 | 설명 | 수용 기준 |
|----|------|------|----------|
| F01 | **실시간 랭킹 리스트** | 급상승 키워드 Top 20 표시 | 5분마다 자동 갱신, 새로고침 없이 숫자 변경 |
| F02 | **숫자 롤링 애니메이션** | 화력(%) 숫자가 0에서 목표치까지 롤링 | 페이지 로드 시 1초 내 애니메이션 완료 |
| F03 | **호가창 스타일 UI** | 상승률에 따른 배경색 그라데이션 | +10% 옅은 빨강 → +100% 진한 네온 레드 |
| F04 | **키워드 상세 페이지** | 시계열 차트 + 원문 링크 | 캔들 차트 형태, 5분 단위 X축 |
| F05 | **데이터 수집 파이프라인** | 5분 단위 크롤링 + DB 적재 | 펨코, 더쿠 최소 2개 커뮤니티 |

#### Should Have (1차 릴리즈)

| ID | 기능 | 설명 | 수용 기준 |
|----|------|------|----------|
| F06 | **VI 발동 연출** | 5분 내 30%+ 급등 시 강조 | Pulse 애니메이션 + [🚨VI] 배지 |
| F07 | **화력 아이콘 시스템** | 상승률 구간별 아이콘 | 🔥(+50%), 🚀(+100%), 💎(+200%) |
| F08 | **종토방 (익명 채팅)** | 키워드별 실시간 댓글 | 로그인 없이 닉네임 자동 생성 |
| F09 | **떡상/떡락 투표** | 키워드별 방향성 투표 | 클릭 즉시 게이지 애니메이션 |

#### Could Have (2차 릴리즈)

| ID | 기능 | 설명 |
|----|------|------|
| F10 | **알림 설정** | 관심 키워드/VI 발동 시 푸시 알림 |
| F11 | **랭킹 히스토리** | 일간/주간 Top 키워드 아카이브 |
| F12 | **내보내기** | 차트 이미지/리포트 PDF 다운로드 |

#### Won't Have (현재 범위 외)

- 해외 커뮤니티 수집
- 모바일 앱 (웹 반응형으로 대응)
- 유료 구독 모델 (광고 BM 우선)

---

### 4.2 상세 기능 명세

#### F01: 실시간 랭킹 리스트

```
┌─────────────────────────────────────────────────────────┐
│  순위  │  키워드           │  화력     │  출처        │
├─────────────────────────────────────────────────────────┤
│  🥇 1  │  [연예인A 열애설] │  +1,247%  │  펨코        │
│  🥈 2  │  [게임B 업데이트] │  +892%    │  더쿠        │
│  🥉 3  │  [정치이슈C]      │  +654%    │  블라인드    │
│  ...                                                    │
└─────────────────────────────────────────────────────────┘
```

**UI 요구사항:**
- 상승률 높을수록 Row 배경색 진하게 (네온 레드 그라데이션)
- 순위 변동 시 Row 위치 애니메이션 (slide up/down)
- 새 데이터 수신 시 숫자 깜빡임 효과 (Flash)

#### F04: 키워드 상세 페이지

```
┌─────────────────────────────────────────────────────────┐
│  [연예인A 열애설]                         +1,247% 🔥    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│      │    ┌──┐                                         │
│      │    │  │  ┌──┐                                   │
│      │ ┌──┤  │  │  │                                   │
│  언급량  │  │  │  │  │                                   │
│      │ │  │  │  │  │                                   │
│      └─┴──┴──┴──┴──┴────────────────────              │
│        09:00  09:05  09:10  09:15  (시간)              │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  📊 통계                                                │
│  - 최초 감지: 오늘 08:45                               │
│  - 누적 언급: 3,847건                                  │
│  - 주요 출처: 펨코(72%), 더쿠(18%), 인스티즈(10%)      │
├─────────────────────────────────────────────────────────┤
│  🔗 원문 바로가기                                       │
│  - [펨코] 속보) A 열애 인정함 ㄷㄷ (조회 12,453)       │
│  - [더쿠] A 열애 반응 모음 (댓글 892)                  │
└─────────────────────────────────────────────────────────┘
```

#### F08: 종토방 (익명 채팅)

```
┌─────────────────────────────────────────────────────────┐
│  💬 종토방 - [연예인A 열애설]                           │
├─────────────────────────────────────────────────────────┤
│  물린마케터1: 이거 지금 콘텐츠 만들면 터짐             │
│  무한매수중2: ㅋㅋㅋ 벌써 기사 나왔는데                │
│  단타전문가3: 아직 늦지 않음 밈 콘텐츠 ㄱㄱ            │
│  ...                                                    │
├─────────────────────────────────────────────────────────┤
│  [떡상 67%] ████████░░ [떡락 33%]                       │
├─────────────────────────────────────────────────────────┤
│  닉네임: 희망회로4  │  입력창...              │ [전송] │
└─────────────────────────────────────────────────────────┘
```

---

## 5. 비기능 요구사항 (Non-Functional Requirements)

### 5.1 성능 (Performance)

| 항목 | 요구사항 | 측정 방법 |
|------|---------|----------|
| **페이지 로드** | FCP < 1.5초, LCP < 2.5초 | Lighthouse |
| **실시간 갱신** | 데이터 변경 → UI 반영 < 500ms | Supabase Realtime latency |
| **애니메이션** | 60fps 유지 (숫자 롤링 중) | Chrome DevTools |
| **크롤링 주기** | 5분 간격 안정적 수집 | Cron 로그 모니터링 |
| **동시 접속** | 1,000명 동시 접속 처리 | 부하 테스트 |

### 5.2 확장성 (Scalability)

- 커뮤니티 소스 추가 시 크롤러 모듈만 확장
- DB 샤딩 고려한 스키마 설계 (키워드별 파티셔닝)
- CDN 활용한 정적 자산 배포

### 5.3 보안 (Security)

| 항목 | 요구사항 |
|------|---------|
| **API 인증** | Rate limiting (분당 60회) |
| **데이터 보호** | 사용자 IP 해싱 저장 (종토방) |
| **XSS 방지** | 사용자 입력 sanitize |
| **CORS** | 허용 도메인 화이트리스트 |

### 5.4 가용성 (Availability)

- 목표 SLA: 99.5% uptime
- 크롤러 장애 시 마지막 유효 데이터 표시 (graceful degradation)
- DB 백업: 일일 자동 백업

---

## 6. 기술 아키텍처 (Technical Architecture)

### 6.1 시스템 구성도

```
┌─────────────────────────────────────────────────────────────────────┐
│                         키워드 레이더 Architecture                   │
└─────────────────────────────────────────────────────────────────────┘

  [Data Sources]              [Backend]                [Frontend]
  ┌──────────┐              ┌───────────┐            ┌───────────┐
  │  펨코    │──┐           │  FastAPI  │◀──REST───▶│  Next.js  │
  │  더쿠    │  │           │  Server   │            │  14 App   │
  │ 인스티즈 │  │  Crawl    └─────┬─────┘            │  Router   │
  │ 블라인드 │──┼────────▶       │                   └─────┬─────┘
  │ 네이트판 │  │           ┌─────▼─────┐                  │
  └──────────┘  │           │  Python   │            Supabase
                │           │  Worker   │            Realtime
                │           │ (크롤러)  │                  │
                │           └─────┬─────┘                  │
                │                 │                        │
                │           ┌─────▼─────┐                  │
                └──────────▶│ Supabase  │◀─────────────────┘
                            │    DB     │
                            │ (Postgres)│
                            └───────────┘
```

### 6.2 기술 스택

| Layer | Technology | 선택 이유 |
|-------|-----------|----------|
| **Frontend** | Next.js 14 (App Router) | SSR + RSC, 빠른 초기 로드 |
| | Tailwind CSS | 빠른 스타일링, 다크모드 |
| | Framer Motion | 고성능 애니메이션 |
| | react-countup | 숫자 롤링 효과 |
| | TradingView Lightweight Charts | 주식 차트 UI |
| **Backend** | FastAPI | 빠른 API 개발, async 지원 |
| | Python 3.11+ | 크롤링/NLP 생태계 |
| | Playwright | 안정적인 브라우저 자동화 |
| **NLP** | Kiwi | 한국어 형태소 분석 (속도 우선) |
| | sentence-transformers | 유사어 클러스터링 |
| **Database** | Supabase (PostgreSQL) | Realtime 기능 내장, 빠른 구축 |
| **Infra** | Vercel | Next.js 최적화 배포 |
| | Railway / Render | Python Worker 배포 |

### 6.3 데이터 파이프라인

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│ COLLECT │ ──▶ │  PARSE  │ ──▶ │ ANALYZE │ ──▶ │  STORE  │ ──▶ │  SERVE  │
└─────────┘     └─────────┘     └─────────┘     └─────────┘     └─────────┘
     │               │               │               │               │
 Playwright      BeautifulSoup    Kiwi 형태소    Supabase      Realtime
 크롤링          HTML 파싱        분석 + 유사어   INSERT        Broadcast
 (5분 주기)      제목/조회수 추출  통합            + 화력 계산
```

**화력(Momentum) 계산 공식:**
```
화력(%) = ((현재_언급량 - 이전_언급량) / 이전_언급량) × 100

가중치 적용:
- 조회수: 1점
- 댓글수: 5점 (댓글이 많을수록 진짜 핫이슈)
```

---

## 7. 데이터 모델 (Database Schema)

### 7.1 ERD

```
┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
│    keywords      │       │   keyword_stats  │       │     sources      │
├──────────────────┤       ├──────────────────┤       ├──────────────────┤
│ id (PK)          │──┐    │ id (PK)          │       │ id (PK)          │
│ keyword          │  │    │ keyword_id (FK)  │──┐    │ name             │
│ cluster_id       │  └───▶│ source_id (FK)   │  │    │ base_url         │
│ created_at       │       │ views            │  └───▶│ crawl_interval   │
│ updated_at       │       │ comments         │       │ is_active        │
└──────────────────┘       │ momentum         │       └──────────────────┘
                           │ recorded_at      │
                           └──────────────────┘
                                   │
                                   ▼
┌──────────────────┐       ┌──────────────────┐
│   source_posts   │       │    chat_rooms    │
├──────────────────┤       ├──────────────────┤
│ id (PK)          │       │ id (PK)          │
│ keyword_id (FK)  │       │ keyword_id (FK)  │
│ source_id (FK)   │       │ message          │
│ title            │       │ nickname         │
│ url              │       │ created_at       │
│ views            │       └──────────────────┘
│ comments         │
│ crawled_at       │       ┌──────────────────┐
└──────────────────┘       │      votes       │
                           ├──────────────────┤
                           │ id (PK)          │
                           │ keyword_id (FK)  │
                           │ vote_type        │  (bull/bear)
                           │ ip_hash          │
                           │ created_at       │
                           └──────────────────┘
```

### 7.2 주요 테이블 DDL

```sql
-- 키워드 마스터
CREATE TABLE keywords (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    keyword VARCHAR(100) NOT NULL,
    cluster_id UUID,  -- 유사어 그룹
    first_seen_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 키워드 시계열 통계
CREATE TABLE keyword_stats (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    keyword_id UUID REFERENCES keywords(id),
    source_id UUID REFERENCES sources(id),
    views INT DEFAULT 0,
    comments INT DEFAULT 0,
    momentum DECIMAL(10,2),  -- 화력(%)
    recorded_at TIMESTAMPTZ DEFAULT NOW(),

    -- 5분 단위 파티셔닝 고려
    CONSTRAINT unique_stat UNIQUE (keyword_id, source_id, recorded_at)
);

-- 실시간 순위 뷰 (캐시용)
CREATE VIEW keyword_rankings AS
SELECT
    k.id,
    k.keyword,
    SUM(ks.views) as total_views,
    SUM(ks.comments) as total_comments,
    AVG(ks.momentum) as avg_momentum,
    MAX(ks.recorded_at) as last_updated
FROM keywords k
JOIN keyword_stats ks ON k.id = ks.keyword_id
WHERE ks.recorded_at > NOW() - INTERVAL '1 hour'
GROUP BY k.id, k.keyword
ORDER BY avg_momentum DESC
LIMIT 20;
```

---

## 8. API 명세 (API Specification)

### 8.1 REST Endpoints

| Method | Endpoint | 설명 | Response |
|--------|----------|------|----------|
| GET | `/api/v1/rankings` | 실시간 Top 20 | `RankingList` |
| GET | `/api/v1/keywords/{id}` | 키워드 상세 | `KeywordDetail` |
| GET | `/api/v1/keywords/{id}/chart` | 시계열 차트 데이터 | `ChartData[]` |
| GET | `/api/v1/keywords/{id}/posts` | 원문 링크 목록 | `SourcePost[]` |
| GET | `/api/v1/keywords/{id}/chat` | 종토방 메시지 | `ChatMessage[]` |
| POST | `/api/v1/keywords/{id}/chat` | 채팅 전송 | `ChatMessage` |
| POST | `/api/v1/keywords/{id}/vote` | 떡상/떡락 투표 | `VoteResult` |

### 8.2 Response Schema

```typescript
// GET /api/v1/rankings
interface RankingList {
  data: RankingItem[];
  updated_at: string;  // ISO 8601
}

interface RankingItem {
  rank: number;
  id: string;
  keyword: string;
  momentum: number;      // 화력 %
  momentum_delta: number; // 이전 대비 변화
  total_views: number;
  total_comments: number;
  primary_source: string; // 주요 출처
  icon: 'fire' | 'rocket' | 'diamond' | null;
  is_vi: boolean;        // VI 발동 여부
}

// GET /api/v1/keywords/{id}/chart
interface ChartData {
  timestamp: string;     // ISO 8601
  open: number;
  high: number;
  low: number;
  close: number;         // 해당 시점 언급량
  momentum: number;
}
```

### 8.3 WebSocket (Realtime)

```typescript
// Supabase Realtime Channel
const channel = supabase
  .channel('rankings')
  .on('postgres_changes',
    { event: 'INSERT', schema: 'public', table: 'keyword_stats' },
    (payload) => {
      // UI 자동 갱신 트리거
      updateRankingUI(payload.new);
    }
  )
  .subscribe();
```

---

## 9. UI/UX 디자인 가이드

### 9.1 컬러 팔레트

| 용도 | Color | Hex |
|------|-------|-----|
| **Background** | Dark Gray | `#121212` |
| **Surface** | Elevated | `#1E1E1E` |
| **Primary (상승)** | Neon Red | `#FF3B30` |
| **Primary Variant** | Binance Red | `#F6465D` |
| **Secondary (하락)** | Korean Blue | `#007AFF` |
| **Accent (1위)** | Gold | `#FFD700` |
| **Text Primary** | White | `#FFFFFF` |
| **Text Secondary** | Gray | `#8E8E93` |

### 9.2 타이포그래피

| 용도 | Font | Weight |
|------|------|--------|
| **본문** | Pretendard | 400, 600 |
| **숫자 데이터** | IBM Plex Mono | 500, 700 |
| **강조 (화력%)** | IBM Plex Mono | 700 |

### 9.3 애니메이션 명세

| 요소 | 트리거 | Duration | Easing |
|------|--------|----------|--------|
| **숫자 롤링** | 페이지 로드, 데이터 갱신 | 800ms | `easeOut` |
| **순위 변동** | 순위 UP/DOWN | 300ms | `spring` |
| **VI 발동** | momentum > 30% (5분) | 2s loop | `pulse` |
| **플래시 효과** | 데이터 변경 | 200ms | `linear` |
| **호버 효과** | Row hover | 150ms | `ease` |

---

## 10. 성공 지표 (KPIs)

### 10.1 North Star Metric

> **DAU (Daily Active Users)** - 매일 대시보드를 확인하는 마케터 수

### 10.2 Key Metrics

| 카테고리 | 지표 | 목표 (런칭 후 3개월) |
|---------|------|---------------------|
| **Acquisition** | 신규 방문자 | 10,000/월 |
| **Activation** | 첫 방문 → 상세페이지 진입률 | > 40% |
| **Retention** | D7 재방문율 | > 25% |
| **Engagement** | 평균 체류시간 | > 3분 |
| | 종토방 참여율 | > 5% |
| **Revenue** | 광고 CTR | > 2% |

### 10.3 측정 도구

- **Web Analytics**: Vercel Analytics, Google Analytics 4
- **Product Analytics**: PostHog (이벤트 트래킹)
- **Error Tracking**: Sentry

---

## 11. 리스크 및 완화 전략

### 11.1 기술 리스크

| 리스크 | 영향도 | 완화 전략 |
|--------|--------|----------|
| **크롤링 차단** | 높음 | User-Agent 로테이션, 프록시 풀, 요청 간격 랜덤화 |
| **실시간성 지연** | 중간 | Supabase Realtime 대신 WebSocket 직접 구현 대비 |
| **NLP 정확도** | 중간 | 초기 수동 검수 → 피드백 학습 |
| **서버 비용 폭증** | 중간 | 캐싱 레이어 (Redis), CDN 최적화 |

### 11.2 법적 리스크

| 리스크 | 영향도 | 완화 전략 |
|--------|--------|----------|
| **크롤링 법적 이슈** | 높음 | robots.txt 준수, 서버 부하 최소화, 원문 링크만 제공 (복제 X) |
| **저작권 문제** | 중간 | 제목/통계만 표시, 본문 미저장 |
| **개인정보** | 낮음 | 종토방 IP 해싱, 익명 처리 |

### 11.3 사업 리스크

| 리스크 | 영향도 | 완화 전략 |
|--------|--------|----------|
| **경쟁사 유사 서비스** | 중간 | "도파민 UX" 차별화 강화, 빠른 기능 이터레이션 |
| **타겟 시장 한계** | 중간 | 마케터 → 에디터 → 일반 트렌드 관심자로 확장 |

---

## 12. 마일스톤 (Milestones)

### Phase 1: Data Foundation (Week 1)
**목표:** 크롤링 파이프라인 구축 및 검증

- [ ] 펨코, 더쿠 크롤러 개발
- [ ] Supabase DB 스키마 구축
- [ ] 5분 주기 Cron 설정
- [ ] 데이터 적재 검증 (3일간 수집 테스트)

**Deliverable:** `keyword_stats` 테이블에 데이터 쌓이는 것 확인

### Phase 2: Core UI (Week 1-2)
**목표:** 메인 대시보드 UI 구현

- [ ] Next.js 프로젝트 셋업
- [ ] 랭킹 리스트 컴포넌트
- [ ] 숫자 롤링 애니메이션
- [ ] 호가창 스타일 Row 컴포넌트
- [ ] 더미 데이터 연동 테스트

**Deliverable:** 더미 데이터로 동작하는 메인 화면

### Phase 3: Integration (Week 2)
**목표:** 실시간 연동 및 상세 페이지

- [ ] Supabase Realtime 연동
- [ ] 키워드 상세 페이지 (차트)
- [ ] 원문 링크 목록
- [ ] VI 발동 연출

**Deliverable:** 실제 데이터 실시간 반영되는 대시보드

### Phase 4: Community & Polish (Week 3)
**목표:** 종토방 및 최종 마무리

- [ ] 종토방 익명 채팅
- [ ] 떡상/떡락 투표
- [ ] 성능 최적화
- [ ] 버그 수정

**Deliverable:** 베타 런칭 가능 상태

### Phase 5: Beta Launch (Week 3-4)
**목표:** 베타 테스트 및 피드백

- [ ] 베타 테스터 모집 (마케터 커뮤니티)
- [ ] 피드백 수집 및 반영
- [ ] 광고 슬롯 구현
- [ ] 정식 런칭 준비

**Deliverable:** 정식 서비스 런칭

---

## 13. 역할 분담 (R&R)

| 역할 | 담당 | 주요 업무 |
|------|------|----------|
| **Backend/Data** | 형 (개발자) | 크롤링, DB, API, NLP 파이프라인 |
| **Frontend/Design** | 본인 (기획자) | UI/UX, Next.js, 애니메이션, 마케팅 |

### 협업 포인트

| 단계 | 형 → 본인 | 본인 → 형 |
|------|----------|----------|
| Phase 1 | API 엔드포인트 문서 | UI 컴포넌트 요구사항 |
| Phase 2 | 더미 API 제공 | 디자인 시안 |
| Phase 3 | Realtime 연동 가이드 | 통합 테스트 피드백 |

---

## 14. 향후 로드맵 (Future Roadmap)

### v1.1 (런칭 후 1개월)
- 추가 커뮤니티 소스 (인스티즈, 블라인드, 네이트판)
- 알림 기능 (웹 푸시)
- 랭킹 히스토리 아카이브

### v1.2 (런칭 후 2개월)
- 카테고리 필터 (연예/정치/게임/IT)
- 키워드 북마크/워치리스트
- 리포트 내보내기 (PDF)

### v2.0 (런칭 후 6개월)
- 모바일 앱 (React Native)
- 유료 플랜 (광고 제거, 알림 무제한)
- 기업용 API (B2B)

---

## 15. 부록 (Appendix)

### A. 경쟁사 분석

| 서비스 | 장점 | 단점 | 차별화 |
|--------|------|------|--------|
| Google Trends | 글로벌 데이터 | 실시간 X, 국내 커뮤니티 미반영 | 국내 커뮤니티 특화 |
| 블로그 차트 | 공식 데이터 | 이미 대중화된 키워드 | 떠오르는 키워드 조기 감지 |
| 소셜메트릭스 | 상세 분석 | 비용 高, B2B 전용 | 무료, 개인 마케터 타겟 |

### B. 레퍼런스 UI

- 토스증권 '발견' 탭: 실시간 롤링 숫자
- 업비트/바이낸스: 호가창 UI, 캔들 차트
- Robinhood: 급등주 하이라이트

### C. 기술 라이브러리 목록

**Frontend:**
- `next@14.x`
- `tailwindcss@3.x`
- `framer-motion@11.x`
- `react-countup@6.x`
- `lightweight-charts@4.x` (TradingView)
- `@supabase/supabase-js@2.x`

**Backend:**
- `fastapi@0.109.x`
- `playwright@1.x`
- `beautifulsoup4@4.x`
- `kiwipiepy@0.x` (한국어 형태소)
- `sentence-transformers@2.x`
- `supabase-py@2.x`

---

## 변경 이력 (Changelog)

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 1.0.0 | 2025-05-20 | 초기 작성 | - |
| 2.0.0 | 2025-12-08 | 전면 개정: 비기능 요구사항, API 명세, 데이터 모델, KPI, 리스크 추가 | Claude |
