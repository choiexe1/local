# SignaLab

이 이미지는 미국 주식(나스닥) 트레이딩 관련 도구 모음 대시보드로 보여.

## 주요 카테고리별 정리

**시장 모니터링**

- 토스 급상승: 급등 종목 현황
- 뉴스: 나스닥 종합 뉴스
- 실시간 분석기: 시장 실시간 분석
- 차트 뷰 분석기: 차트 및 분석 도구

**스캐너 도구들**

- 레버리지 스캐너: ETF 가격 변동 분석
- 거래량 급등 스캐너: 거래량 폭발 종목 실시간 감지
- 모멘텀 스캐너: 기술적 지표 분석
- 나스닥 정지 감지기: 거래 정지/재개 감지

---

### SEC 고속 스캐너들

SEC 공시 정보를 실시간으로 스캔하는 도구야. 미국은 SEC에 공시가 올라오면 주가가 급등/급락하는 경우가 많아서, 이걸 남들보다 빨리 캐치하려는 목적이야.

- **김종수 SEC / 노현우 SEC1 / sh현우 SEC2**: 각각 다른 개발자가 만든 SEC 공시 스캐너 엔진으로 보여. 속도나 감지 방식이 조금씩 다를 수 있음.
- **SEC분석기**: SEC 공시 내용 분석 도구

---

### GEM 폭풍의 눈

**GEM = Growth Enterprise Market** 또는 소형주/저가주 관련 스캐너로 추정돼.

- "52주 바닥권 + 거래량 진공" 조건으로 종목을 찾는 도구
- 바닥권에서 거래량이 터지기 직전인 종목을 찾는 전략 (폭등 직전 포착 목적)

---

### Reg SHO Monitor Pro

**Regulation SHO** 관련 모니터링 도구. 공매도 제한 목록(Threshold Securities List)에 오른 종목을 추적하는 용도야. 숏스퀴즈 가능성 있는 종목 탐색할 때 쓰임.

![[Pasted image 20260127124854.png]]

SEC EDGAR 공시 모니터링 + 숏스퀴즈 시그널 탐지 서비스

## 프로젝트 개요

**목표**: SEC 8-K 공시 + FINRA 공매도 데이터 → LLM 분석 → 개인 투자자에게 푸시 알림

**타겟**: 밈 주식 / 숏스퀴즈 기회에 관심 있는 개인 투자자

## 기술 스택

- **런타임**: Bun
- **프레임워크**: Elysia
- **데이터베이스**: PostgreSQL + DrizzleORM
- **푸시**: Firebase Cloud Messaging
- **LLM**: OpenAI / Claude (교체 가능)

## 아키텍처

```
[SEC EDGAR] ─── 8-K 폴링 (5-15분) ───┐
                                      ├──→ [LLM 분석기] ──→ [시그널 엔진] ──→ [FCM 푸시]
[FINRA API] ─── 공매도 잔량 (월 2회) ─┘
```

**구조**: 모놀리스 (Elysia API + Cron Workers 단일 프로세스)

## 기능 로드맵

### Phase 1: MVP (핵심)

| 기능 | 설명 | 상태 |
|------|------|------|
| 숏스퀴즈 시그널 | SI/Float + 8-K 호재성 공시 알림 | 🔲 |
| 공시 요약 (한글) | LLM 기반 8-K 핵심 포인트 한글 요약 | 🔲 |
| 관심종목 | 사용자 즐겨찾기 종목 | 🔲 |
| 타임라인 뷰 | 종목별 공시 + SI 히스토리 | 🔲 |

### Phase 2: 내부자/기관 데이터

| 기능 | 데이터 출처 | 가치 |
|------|-------------|------|
| 내부자 매수 알림 | Form 4 | CEO가 자기 돈으로 매수 = 강력한 시그널 |
| 기관 포지션 | 13F (분기별) | 헤지펀드 동향 |
| 행동주의 투자자 | 13D | 5%+ 지분 공시, 주가 급등 가능성 |

### Phase 3: 고급 시그널

| 기능 | 데이터 출처 | 가치 |
|------|-------------|------|
| 락업 만료 캘린더 | S-1, 424B | IPO 후 락업 만료 → 매도 압력 |
| RegSHO Threshold | FINRA | FTD 누적 = 스퀴즈 후보 |
| 자사주 매입 공시 | 8-K Item 8.01 | 회사가 주가 저평가 판단 |

## 데이터 소스

| 소스 | 데이터 | 빈도 | 지연 | 비용 |
|------|--------|------|------|------|
| SEC EDGAR | 8-K 공시 | 실시간 폴링 | 없음 | 무료 |
| SEC EDGAR | Form 4 (내부자) | 실시간 폴링 | 없음 | 무료 |
| SEC EDGAR | 13F (기관) | 분기별 | 45일 | 무료 |
| SEC EDGAR | 13D (행동주의) | 실시간 | 없음 | 무료 |
| FINRA Consolidated SI | 공매도 잔량 | 월 2회 (15일, 말일) | 2주 | 무료 |
| FINRA Threshold List | RegSHO 종목 | 일일 | T+1 | 무료 |
| SEC FTD | 결제실패 데이터 | 월 2회 | 2주 | 무료 |

### SEC EDGAR API 엔드포인트

```
# 회사 제출물 (JSON) - 10 req/sec 제한, User-Agent 필수
https://data.sec.gov/submissions/CIK{10자리}.json

# 공시 문서
https://www.sec.gov/Archives/edgar/data/{CIK}/{accession-number}/

# 티커 → CIK 매핑
https://www.sec.gov/files/company_tickers.json
```

### FINRA API 엔드포인트

```
POST https://api.finra.org/data/group/otcMarket/name/weeklySummary
POST https://api.finra.org/data/group/otcMarket/name/weeklyDownloadDetails
```

## 프로젝트 구조

```
src/
├── index.ts              # 엔트리 포인트
├── app.ts                # Elysia 앱 설정
├── config/
│   └── env.ts            # 환경 변수
├── db/
│   ├── index.ts          # DB 연결 ✅
│   └── schema.ts         # Drizzle 스키마 ✅
├── workers/
│   ├── edgar-poller.ts   # SEC 8-K 폴링 (5분 간격)
│   └── finra-poller.ts   # FINRA SI 폴링 (일일 체크)
├── services/
│   ├── edgar.service.ts  # SEC API 호출
│   ├── finra.service.ts  # FINRA API 호출
│   ├── filing.service.ts # 공시 CRUD
│   └── signal.service.ts # 시그널 탐지
├── analyzers/
│   └── llm.analyzer.ts   # LLM 추상화 레이어
└── api/
    ├── filings.ts        # GET /filings
    ├── signals.ts        # GET /signals
    └── companies.ts      # GET /companies
```

## 데이터 수집 전략

### 대상 기업 (시드 리스트)

모든 8-K를 폴링하는 대신, 특정 기업만 모니터링:

| 카테고리 | 기준 | 예시 |
|----------|------|------|
| 밈/스퀴즈 이력 | 과거 스퀴즈 이벤트 | GME, AMC, KOSS |
| 고 SI/Float | SI/Float > 20% | FINRA 상위 50개 |
| 리테일 인기 | 고거래량, 소셜 버즈 | TSLA, NVDA, PLTR |
| 소형주 + 고 SI | 시총 < $1B + SI > 15% | 스퀴즈 후보 |

### 동적 추가

| 트리거 | 액션 |
|--------|------|
| SI/Float 20% 돌파 | 자동 모니터링 추가 |
| 사용자 관심종목 추가 | 모니터링 풀에 포함 |
| RegSHO Threshold 편입 | 자동 추가 |

### 데이터 플로우

```
[시드 기업] → companies 테이블 (isActive=true)
                      ↓
[Cron 5분] → 활성 기업 쿼리
                      ↓
      각 CIK별: SEC API에서 신규 8-K 확인
                      ↓
      신규 공시? → filings 테이블에 저장
                      ↓
      LLM 분석 큐 등록
                      ↓
      analyses 테이블에 저장
                      ↓
      시그널 엔진 체크 (SI 데이터와 결합)
                      ↓
      시그널 탐지? → 저장 + 푸시 알림
```

## 데이터베이스 스키마

상세 필드 설명은 [[SCHEMA]] 참조.

### 핵심 테이블 (Phase 1)
- **companies** - 모니터링 대상 종목 마스터
- **filings** - SEC 제출물 (8-K 중심)
- **short_interest** - FINRA 공매도 스냅샷
- **analyses** - LLM 분석 결과 (1:1 관계)
- **signals** - 탐지된 시그널

### 사용자 테이블 (Phase 2)
- **users** - 앱 사용자
- **subscriptions** - 관심종목 (user ↔ company)
- **push_logs** - 푸시 알림 히스토리

### 확장 테이블 (Phase 3)
- **insider_transactions** - Form 4 데이터
- **institutional_holdings** - 13F 데이터
- **lockup_expirations** - IPO 락업 캘린더

## 시그널 유형

| 시그널 | 트리거 | 중요도 |
|--------|--------|--------|
| `SHORT_SQUEEZE_CANDIDATE` | 고 SI/Float + 호재성 8-K | high |
| `INSIDER_BUY_HIGH_SI` | Form 4 내부자 매수 + 고 SI | high |
| `ACTIVIST_STAKE` | 13D 공시 (5%+ 지분) | high |
| `BUYBACK_HIGH_SI` | 자사주 매입 8-K + 고 SI | medium |
| `MATERIAL_EVENT` | 8-K Item 8.01 고영향 | medium |
| `LOCKUP_EXPIRY` | 7일 내 락업 만료 | medium |
| `THRESHOLD_LIST_ADD` | RegSHO threshold list 편입 | low |

## 개발 로드맵

### Phase 1: 핵심 인프라 ✅
- [x] Docker + PostgreSQL 설정
- [x] Drizzle ORM 구성
- [x] 데이터베이스 스키마 (companies, filings, short_interest, analyses, signals)

### Phase 2: 데이터 수집
- [ ] 시드 기업 리스트 (50-100개 기업)
- [ ] SEC EDGAR poller (CIK별 8-K)
- [ ] FINRA 공매도 수집기
- [ ] 공시 파서 (8-K 항목 추출)

### Phase 3: 분석 파이프라인
- [ ] LLM 추상화 레이어 (OpenAI/Claude 교체 가능)
- [ ] 8-K 감성 분석
- [ ] 시그널 탐지 엔진

### Phase 4: 사용자 기능
- [ ] 사용자 인증
- [ ] 구독 관리 (관심종목)
- [ ] FCM 푸시 연동
- [ ] 모바일 앱용 API 엔드포인트

### Phase 5: 내부자/기관
- [ ] Form 4 파서 (내부자 거래)
- [ ] 13F 파서 (기관 보유)
- [ ] 13D 파서 (행동주의 투자자)

### Phase 6: 고급
- [ ] S-1/424B 락업 추출
- [ ] RegSHO threshold list 연동
- [ ] SEC FTD 데이터 연동
- [ ] 히스토리컬 백테스팅

## 경쟁 환경

| 서비스 | 유형 | 가격 | 갭 |
|--------|------|------|-----|
| ORTEX | 공매도 | $50+/월 | 공시 분석 없음 |
| Fintel | SI + 기관 | $25+/월 | LLM 요약 없음 |
| SEC Filing Alerts | 공시 알림 | $10+/월 | SI 데이터 없음 |
| KFilings | 공시 알림 | 무료 | 이메일만, 분석 없음 |

**우리의 강점**: 8-K + SI 결합 시그널 + LLM 한글 요약 + 푸시 알림

## 명령어

```bash
bun run dev           # 개발 서버 시작
bun run docker:up     # PostgreSQL 시작
bun run docker:down   # PostgreSQL 중지
bun run docker:clean  # PostgreSQL + 데이터 삭제
bun run db:generate   # 마이그레이션 생성
bun run db:migrate    # 마이그레이션 실행
bun run db:push       # 스키마 직접 적용
bun run db:studio     # Drizzle Studio 열기
```

## 환경 변수

```
DATABASE_URL=postgres://signalab:signalab_dev_password@localhost:5434/signalab
PORT=3000
NODE_ENV=development
```

## 관련 문서

- [[README]] - 빠른 시작 가이드
- [[SCHEMA]] - 데이터베이스 스키마 상세
- [[BUSINESS]] - 비즈니스 모델
