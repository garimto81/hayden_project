在此 프로젝트의 핵심은 \*\*"마케터의 도파민을 자극하는 UX(주식 호가창)"\*\*와 **"실제 데이터 수급의 기술적/법적 실현 가능성"** 사이의 균형을 맞추는 것입니다.

사용자가 제공한 기획안(v1, v2, concept)을 바탕으로, **최신 2025년 웹 기술 트렌드**와 \*\*크롤링 리스크 관리(Legal & Feasibility)\*\*를 보강한 **PRD v3.0**을 작성해 드립니다.

특히 개발자('형')가 가장 먼저 검증해야 할 **PoC(개념 증명) 단계**를 구체화했습니다.

-----

# [PRD] 키워드 레이더 (Keyword Radar) v3.0

**작성일:** 2025-12-08
**버전:** 3.0.0 (Tech & Legal Review Completed)
**목표:** 개발 착수 전 데이터 파이프라인 검증 및 UI 스택 확정

## 1\. 기술 스택 전략 (Tech Stack Strategy 2025)

2025년 트렌드에 맞춰 '빠른 개발'과 '화려한 인터랙션'을 동시에 잡는 스택을 제안합니다.

### 1.1 Frontend (Visual Dopamine)

주식 차트와 호가창은 데이터 갱신이 빈번하므로 렌더링 성능이 중요합니다.

  * **Core:** **Next.js 15 (App Router)** - 서버 컴포넌트(RSC)로 초기 로딩 속도 확보.
  * **State Management:** **Zustand** - Redux보다 가볍고, 실시간 데이터 흐름(Socket) 관리에 유리.
  * **UI/Styling:** **Tailwind CSS** + **Shadcn/ui** - 개발 속도 단축 및 커스터마이징 용이.
  * **Animation (핵심):**
      * **Framer Motion:** VI 발동 시 화면 전체가 붉게 깜빡이는(`Pulse`) 효과, 진입 시 숫자 카운트업.
      * **React Fast Marquee:** 상단 뉴스 티커(흐르는 자막) 구현에 최적화된 라이브러리.
  * **Charts:** **TradingView Lightweight Charts** - `Recharts`보다 훨씬 가볍고, '주식 차트' 감성을 그대로 구현 가능 (캔들 차트, 거래량 바 등).

### 1.2 Backend & Data (The Engine)

  * **Language:** **Python 3.11+** (FastAPI) - 크롤링 및 형태소 분석(Kiwi) 라이브러리가 Python에 집중되어 있음.
  * **DB & Realtime:** **Supabase** (PostgreSQL)
      * 별도의 소켓 서버 구축 없이, `Supabase Realtime` 기능을 켜면 DB에 데이터가 `INSERT` 되는 순간 프론트엔드로 즉시 전송됨. (개발 공수 50% 절감)
  * **Crawling:** **Playwright** - Selenium보다 빠르고, 최신 브라우저 감지 회피 기술(Stealth plugin) 적용이 용이함.

-----

## 2\. 핵심 리스크 점검: 데이터 수집 및 법적 이슈 (Critical)

개발자 형이 가장 우려할 부분입니다. **"전체 웹 트렌드"를 가져오는 것은 불가능**하며, 특정 커뮤니티 크롤링은 **법적/기술적 차단 이슈**가 있습니다. 이를 해결하기 위한 현실적인 가이드라인입니다.

### 2.1 법적 리스크 (Legal Review)

  * **크롤링 합법성 (국내 판례 기준):** 공개된 정보라도 \*\*'경쟁 서비스 제공'\*\*을 목적으로 대량 수집하거나, \*\*'서버에 과도한 부하'\*\*를 주면 위법(부정경쟁방지법 위반) 소지가 있습니다.
  * **대응 전략:**
    1.  **본문 수집 금지:** 게시글 내용은 저작권 위험이 큼. **'제목', '조회수', '댓글수', '작성시간'** 데이터만 수집하여 통계만 낸다. (단순 사실 데이터는 저작권 없음)
    2.  **딥링크 제공:** 상세 내용은 우리 앱에서 보여주지 않고, \*\*'원문 보러가기'\*\*로 해당 커뮤니티로 트래픽을 돌려준다. (커뮤니티 입장에서도 트래픽 유입이므로 관대할 수 있음)
    3.  **robots.txt 준수:** `User-Agent`에 봇임을 명시하지 않더라도, 과도한 요청(DDoS급)은 피해야 함.

### 2.2 기술적 차단 우회 (Anti-Scraping)

  * **Cloudflare 차단:** 펨코, 더쿠 등 대형 커뮤니티는 Cloudflare가 봇을 막습니다.
  * **해결책:** `Playwright` + `Stealth Plugin`을 사용하되, **가정용 IP(Residential Proxy)** 사용이 필요할 수 있습니다. 초기에는 개발자 PC(로컬 IP)에서 돌리는 것이 서버 IP보다 차단 확률이 낮습니다.

-----

## 3\. 기능 명세 및 로직 상세 (Functional Specs)

### 3.1 도파민 펌핑 알고리즘 (Momentum Logic)

단순 조회수 순위는 재미가 없습니다. **"지금 막 뜨기 시작한"** 것을 찾아야 합니다.

  * **화력 지수(Momentum Score) 공식:**
    $$Score = (V_{now} - V_{5min}) \times 1.0 + (C_{now} - C_{5min}) \times 5.0$$
      * $V$: 조회수, $C$: 댓글수
      * **댓글 가중치(5.0):** 조회수는 조작이 쉽지만, 댓글이 1분 만에 10개 달리는 글이 진짜 트렌드입니다.
  * **VI (변동성 완화 장치) 발동 조건:**
      * 직전 5분 대비 점수가 **300% 이상 급등** 시.
      * UI 효과: 해당 Row 배경이 붉은색으로 `Pulse` 애니메이션 + "🚨VI 발동" 배지 부착.

### 3.2 UI 상세 (호가창 컨셉)

  * **좌측 (매수 잔량 느낌):** 현재 랭킹 1\~10위 키워드.
  * **우측 (체결창 느낌):** 실시간으로 수집되는 개별 게시글들이 `타닥타닥` 올라가는 피드.
      * *예: [펨코] 09:41:22 | 지디 마약 무혐의 ㄷㄷ... (조회 512)*
  * **중앙 상단:** 종합 코스피 지수처럼 **"오늘의 도파민 지수"** (전체 커뮤니티 글 리젠 속도 합산).

-----

## 4\. 개발자('형')를 위한 PoC(개념 증명) 가이드

프로젝트 전체를 시작하기 전, \*\*"이게 진짜 가능한가?"\*\*를 검증하기 위해 형에게 아래 미션을 먼저 요청하세요.

### ✅ Step 1. 데이터 파이프라인 PoC (1주차)

**목표:** UI 없이 파이썬 스크립트로 2개 사이트(예: 펨코, 더쿠)의 인기글 목록을 5분 간격으로 가져와 DB에 쌓을 수 있는지 확인.

**요청사항:**

> "형, 일단 디자인 다 필요 없고 **Python**으로 아래 내용만 확인해줘."
>
> 1.  `Playwright`로 펨코 '포텐터짐' 게시판 1페이지 긁어지나? (Cloudflare 뚫리는지 확인)
> 2.  5분 뒤에 다시 긁었을 때, **조회수/댓글수 변화량**을 계산할 수 있나?
> 3.  이걸 **Supabase** 테이블에 `(키워드, 시간, 조회수_변화량)` 형태로 1시간만 쌓아봐줘.

### ✅ Step 2. 키워드 추출 성능 검증 (2주차)

**목표:** "지드래곤", "지디", "권지용"을 하나로 묶을 수 있는가?

**요청사항:**

> "형, 수집된 제목들에서 **Kiwi(형태소 분석기)** 돌려서 명사만 뽑고,
> `Sentence-Transformers` (SBERT) 같은 걸로 의미 비슷한 것끼리 묶어줄 수 있어?
> 너무 무거우면 그냥 텍스트 포함(`contains`) 로직으로 '지디' 들어가면 다 합치는 걸로 1차 퉁치자."

### ✅ Step 3. 실시간 소켓 연동 (3주차)

**목표:** DB에 데이터가 들어가면 프론트엔드 숫자가 자동으로 바뀌는지.

**요청사항:**

> "형, 내가 Next.js로 숫자 하나 띄워놓을게.
> 형이 파이썬에서 Supabase에 데이터 `INSERT` 하면, 내 화면에서 새로고침 안 해도 숫자가 `500` -\> `600`으로 바뀌게 해줘. (Supabase Realtime 기능 쓰면 된대)"

-----

## 5\. UI 예시 프롬프트 (For Generation)

개발자가 UI 구조를 잡을 때 참고할 수 있도록, v0나 AI 코딩 툴에 넣을 프롬프트입니다.

```markdown
Build a real-time crypto-exchange style dashboard for "Keyword Trends".
Use Next.js 15, Tailwind CSS, and Framer Motion.
Background should be #121212 (Dark Mode).

1. **Header:** A scrolling marquee ticker (like stock news) showing random trending keywords.
2. **Main Layout:** Split into 2 columns.
   - Left: "Ranking Board". Rows look like order books. Columns: Rank, Keyword, Momentum(%), Volume.
   - Right: "Live Feed". A list of recent posts appearing in real-time.
3. **Animations:**
   - When "Momentum" increases, the number should roll up (CountUp).
   - If Momentum > 500%, the row background should pulse in neon red.
4. **Chart:** Use TradingView Lightweight Charts to show a candlestick chart of a keyword's volume over time.
```

## 6\. 결론 및 제안

이 기획안은 \*\*"재미(UX)"\*\*와 \*\*"현실성(Tech)"\*\*을 모두 잡았습니다.
형에게는 \*\*"전체 웹 크롤링은 불가능하니, 딱 3개 커뮤니티(펨코/더쿠/인스티즈)만 파서 '실시간성'만 극대화하자"\*\*고 제안하십시오. 데이터의 양보다 **데이터가 갱신되는 속도와 연출**이 이 프로젝트의 핵심(도파민)입니다.