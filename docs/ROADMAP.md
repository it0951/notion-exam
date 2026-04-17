# 노션 Claude를 다루는 기술 — 개발 로드맵

> **기준 문서**: `docs/PRD.md`
> **목표**: Notion CMS 기반 Claude 학습 블로그 MVP 완성

---

## Phase 1: 프로젝트 초기 설정 (1일)

### 작업 목록

- [ ] `@notionhq/client` 패키지 설치 (`npm install @notionhq/client`)
- [ ] `notion-to-md` 패키지 설치 (선택, 블록 렌더링 유틸)
- [ ] `.env.local` 파일 생성 및 환경변수 추가
  - `NOTION_API_KEY=secret_xxxx`
  - `NOTION_DATABASE_ID=xxxx`
- [ ] Notion Integration 생성 및 데이터베이스 연결
  - Notion Developers에서 Integration 생성
  - 대상 DB에 Integration 공유(Share → Connections)
  - DB URL에서 ID 추출
- [ ] Notion 데이터베이스 스키마 구성
  - `Title` (제목), `Category` (Select), `Tags` (Multi-select), `Published` (Date), `Status` (Select)
- [ ] `lib/notion.ts` — Notion 클라이언트 싱글톤 초기화

### 이유

환경변수·API 키·DB 스키마가 없으면 이후 모든 개발이 막힘. 가장 먼저 확보해야 하는 인프라 기반.

### 완료 기준

- `lib/notion.ts`에서 `notion.databases.retrieve()` 호출 시 에러 없이 응답 반환
- `.env.local`의 두 환경변수가 Next.js 서버에서 정상 로드됨

---

## Phase 2: 공통 모듈 및 타입 개발 (1-2일)

### 작업 목록

- [ ] `lib/notion.ts` — 데이터 페칭 함수 구현
  - `getPosts()`: Status=발행됨 필터, 최신순 정렬
  - `getPageBlocks(pageId)`: 개별 페이지 블록 조회
- [ ] `types/notion.ts` — 공통 타입 정의
  - `NotionPost` (id, title, category, tags, published, status)
  - `NotionBlock` (id, type, content, children)
- [ ] Notion API 응답을 `NotionPost`로 변환하는 파서 함수 (`parsePost`)
- [ ] `app/layout.tsx` — 루트 레이아웃에 `Navbar` / `Footer` 슬롯 배치
- [ ] `components/layout/Navbar.tsx` — 로고(홈 링크) + 반응형 헤더 골격
- [ ] `components/layout/Footer.tsx` — 간단한 푸터

### 이유

`getPosts()` · `getPageBlocks()` · 공통 타입은 홈/상세 페이지 양쪽에서 재사용됨. 먼저 만들지 않으면 각 페이지 개발 시 중복 코드가 발생하고 타입 불일치 버그가 생길 수 있음.

### 완료 기준

- `getPosts()` 호출 결과가 `NotionPost[]`로 타입 오류 없이 반환됨
- `getPageBlocks()` 호출 결과가 `NotionBlock[]`로 반환됨
- Navbar·Footer가 모든 페이지에 공통으로 노출됨

---

## Phase 3: 핵심 기능 개발 (2-3일)

### 작업 목록

**글 목록 페이지 (F001, F010, F011)**

- [ ] `app/page.tsx` — 서버 컴포넌트, `getPosts()` 호출 + ISR(`revalidate = 3600`) 설정
- [ ] `components/PostCard.tsx` — 카드 UI (제목, 카테고리 배지, 발행일, 태그)
- [ ] `components/PostList.tsx` — 클라이언트 컴포넌트, 카드 그리드 레이아웃 + 반응형

**글 상세 페이지 (F002, F010, F011)**

- [ ] `app/posts/[id]/page.tsx` — 서버 컴포넌트, `getPageBlocks()` + `notion.pages.retrieve()` 병렬 호출
- [ ] `components/BlockRenderer.tsx` — Notion 블록 타입별 렌더링
  - 지원 블록: `paragraph`, `heading_1~3`, `bulleted_list_item`, `numbered_list_item`, `code`, `quote`, `image`, `divider`
- [ ] 메타 정보 표시 영역 (제목, 카테고리, 태그, 발행일)
- [ ] 홈으로 돌아가기 링크 (브레드크럼 또는 상단 버튼)

### 이유

블로그의 가장 기본 기능인 "목록 보기"와 "글 읽기"를 먼저 완성해야 서비스가 동작 가능한 상태가 됨. 추가 기능은 이 두 페이지 위에 얹는 방식으로 개발.

### 완료 기준

- 홈 페이지에서 Notion DB의 발행된 글 목록이 카드 형태로 렌더링됨
- 카드 클릭 시 `/posts/[id]`로 이동하여 본문 블록이 올바르게 렌더링됨
- 모바일(375px) / 태블릿(768px) / 데스크탑(1280px) 레이아웃 정상 동작

---

## Phase 4: 추가 기능 개발 (1-2일)

### 작업 목록

**카테고리 필터링 (F003)**

- [ ] `PostList.tsx`에 카테고리 탭 UI 추가 (전체 + 동적 카테고리 목록)
- [ ] 탭 선택 시 해당 카테고리 글만 표시되는 클라이언트 사이드 필터 로직

**검색 기능 (F004)**

- [ ] `PostList.tsx`에 검색 입력창(shadcn `Input`) 추가
- [ ] 제목 기준 실시간 텍스트 필터 로직 (클라이언트 사이드, `useState` + 디바운스)
- [ ] 카테고리 필터 + 검색어 필터 AND 조합 처리

**SEO 기본 설정**

- [ ] `app/layout.tsx` — 사이트 기본 `metadata` (title, description) 설정
- [ ] `app/posts/[id]/page.tsx` — 글별 `generateMetadata()` 구현 (제목, 카테고리 반영)

### 이유

핵심 기능(목록·상세)이 동작한 후에 탐색 편의 기능을 추가해야 기반 없이 UI만 만드는 낭비를 방지. SEO는 배포 전에 처리해야 초기 색인이 올바르게 잡힘.

### 완료 기준

- 카테고리 탭 클릭 시 해당 글만 필터링되고 "전체" 탭으로 초기화 가능
- 검색어 입력 시 제목에 키워드가 포함된 글만 실시간 표시
- 카테고리 + 검색어 동시 적용 시 두 조건 모두 충족하는 글만 노출
- `/posts/[id]` 페이지의 `<title>` 태그에 글 제목이 반영됨

---

## Phase 5: 최적화 및 배포 (1일)

### 작업 목록

**성능 최적화**

- [ ] `next/image`로 Notion 이미지 블록 최적화 (도메인 허용 설정 포함)
- [ ] ISR `revalidate` 값 검토 및 조정 (현재 3600초)
- [ ] `npm run build` 빌드 오류 0건 확인, `npm run lint` 경고 해소

**반응형 점검**

- [ ] 모바일 카드 레이아웃, 검색창, 카테고리 탭 UI 최종 확인
- [ ] 코드 블록 수평 스크롤 처리

**Vercel 배포**

- [ ] GitHub 저장소 연결 및 Vercel 프로젝트 생성
- [ ] Vercel 환경변수 설정 (`NOTION_API_KEY`, `NOTION_DATABASE_ID`)
- [ ] 프로덕션 배포 후 실제 URL에서 전체 사용자 여정 검증
  - 홈 → 카테고리 필터 → 검색 → 글 상세 → 홈 복귀

### 이유

기능 완성 후 실 환경 검증 없이 배포하면 환경변수 누락·이미지 도메인 오류 등 예측 가능한 문제가 발생. 마지막에 한 번에 정리하는 것이 효율적.

### 완료 기준

- Vercel 프로덕션 URL에서 모든 MVP 기능(F001~F004, F010, F011) 정상 동작
- Lighthouse 성능 점수 70점 이상 (모바일 기준)
- 빌드 오류 및 TypeScript 컴파일 오류 0건

---

## 전체 일정 요약

| Phase | 내용 | 예상 소요 |
|-------|------|----------|
| Phase 1 | 프로젝트 초기 설정 | 1일 |
| Phase 2 | 공통 모듈 및 타입 개발 | 1-2일 |
| Phase 3 | 핵심 기능 개발 (목록·상세) | 2-3일 |
| Phase 4 | 추가 기능 개발 (필터·검색·SEO) | 1-2일 |
| Phase 5 | 최적화 및 Vercel 배포 | 1일 |
| **합계** | | **6-9일** |

---

## MVP 기능 커버리지

| 기능 ID | 기능명 | 구현 Phase |
|---------|--------|-----------|
| F001 | 글 목록 조회 | Phase 3 |
| F002 | 글 상세 조회 | Phase 3 |
| F003 | 카테고리 필터링 | Phase 4 |
| F004 | 검색 기능 | Phase 4 |
| F010 | Notion API 연동 | Phase 1, 2 |
| F011 | 반응형 레이아웃 | Phase 3, 5 |
