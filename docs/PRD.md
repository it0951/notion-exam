# 노션 Claude를 다루는 기술 MVP PRD

## 핵심 정보

**목적**: Notion을 CMS로 활용하여 Claude 관련 학습 자료, 조사 내용, 공식 가이드를 체계적으로 수집하고 웹에서 열람 가능하게 함
**사용자**: Claude AI를 학습하거나 활용하려는 1인 개발자 및 비개발자 독자

---

## 사용자 여정

```
1. 홈 페이지 (최근 글 목록)
   ↓ 카테고리 탭 클릭 또는 검색어 입력

2. 필터/검색 결과 반영 (동일 홈 페이지 내)
   ↓ 글 카드 클릭

3. 글 상세 페이지 (본문 표시)
   ↓ 뒤로가기 또는 헤더 로고 클릭

4. 홈 페이지 복귀
```

---

## 기능 명세

### 1. MVP 핵심 기능

| ID | 기능명 | 설명 | MVP 필수 이유 | 관련 페이지 |
|----|--------|------|--------------|------------|
| **F001** | 글 목록 조회 | Notion DB에서 Status=발행됨인 글을 최신순으로 조회 | 서비스의 핵심 콘텐츠 진입점 | 홈 페이지 |
| **F002** | 글 상세 조회 | Notion 페이지 블록을 파싱하여 본문 렌더링 | 콘텐츠 소비의 핵심 기능 | 글 상세 페이지 |
| **F003** | 카테고리 필터링 | Category 필드 기준으로 글 목록 필터 | 콘텐츠 탐색 효율화 | 홈 페이지 |
| **F004** | 검색 기능 | Title 기준 클라이언트 사이드 텍스트 검색 | 콘텐츠 직접 접근 수단 | 홈 페이지 |

### 2. MVP 필수 지원 기능

| ID | 기능명 | 설명 | MVP 필수 이유 | 관련 페이지 |
|----|--------|------|--------------|------------|
| **F010** | Notion API 연동 | `@notionhq/client`로 DB 쿼리 및 블록 조회 | 모든 데이터의 원천 | 홈 페이지, 글 상세 페이지 |
| **F011** | 반응형 레이아웃 | 모바일/태블릿/데스크탑 대응 레이아웃 | 다양한 환경에서 열람 가능 | 홈 페이지, 글 상세 페이지 |

### 3. MVP 이후 기능 (제외)

- 태그 기반 필터링
- RSS 피드
- 댓글/피드백 기능
- 다크모드 토글 (레이아웃 기본값으로 처리)
- OG 이미지 동적 생성
- 무한 스크롤 / 페이지네이션

---

## 메뉴 구조

```
노션 Claude를 다루는 기술 내비게이션
├── 로고/사이트명 (홈으로 이동)
│   └── 기능: F001 (글 목록 홈으로 리디렉션)
├── 카테고리 탭 목록
│   └── 기능: F003 (카테고리 필터링)
└── 검색 입력창
    └── 기능: F004 (제목 검색)

글 상세 페이지 내비게이션
└── 뒤로가기 / 홈 링크
    └── 기능: F001 (홈 복귀)
```

---

## 페이지별 상세 기능

### 홈 페이지

> **구현 기능:** `F001`, `F003`, `F004`, `F010`, `F011` | **접근:** 최초 진입 또는 로고 클릭

| 항목 | 내용 |
|------|------|
| **역할** | 발행된 모든 글의 목록을 보여주는 랜딩 페이지이자 탐색 허브 |
| **진입 경로** | 사이트 루트 URL 접속, 헤더 로고 클릭, 글 상세에서 뒤로가기 |
| **사용자 행동** | 최신 글 카드 탐색, 카테고리 탭으로 필터링, 검색창에 키워드 입력, 글 카드 클릭으로 상세 이동 |
| **주요 기능** | - Notion DB에서 Status=발행됨 글 전체 조회 (최신순)<br>- 카테고리 탭 UI (전체 / 각 카테고리값 동적 생성)<br>- 제목 기준 실시간 텍스트 필터링 (클라이언트 사이드)<br>- 글 카드: 제목, 카테고리 배지, 발행일, 태그 표시<br>- **글 카드 클릭** 시 상세 페이지로 이동 |
| **다음 이동** | 글 카드 클릭 → 글 상세 페이지, 카테고리/검색 변경 → 동일 페이지 내 필터 반영 |

---

### 글 상세 페이지

> **구현 기능:** `F002`, `F010`, `F011` | **접근:** 홈 페이지 글 카드 클릭 후

| 항목 | 내용 |
|------|------|
| **역할** | 특정 Notion 페이지의 블록 콘텐츠를 HTML로 렌더링하여 본문 열람 제공 |
| **진입 경로** | 홈 페이지에서 글 카드 클릭 (동적 라우팅: 글 ID 기반) |
| **사용자 행동** | 본문 읽기, 코드 블록/이미지/인용 등 Notion 블록 유형별 콘텐츠 소비, 홈으로 복귀 |
| **주요 기능** | - Notion 페이지 블록 API 조회 및 블록 타입별 렌더링<br>- 지원 블록: paragraph, heading_1~3, bulleted_list_item, numbered_list_item, code, quote, image, divider<br>- 메타 정보 표시: 제목, 카테고리, 태그, 발행일<br>- **홈으로 이동** 링크 (헤더 또는 상단 브레드크럼) |
| **다음 이동** | 홈 링크 클릭 → 홈 페이지 |

---

## 데이터 모델

### NotionPost (Notion DB 프로퍼티 매핑)

| 필드 | 설명 | 타입/관계 |
|------|------|----------|
| id | Notion 페이지 ID | string (UUID) |
| title | 글 제목 | string |
| category | 카테고리 | string (select) |
| tags | 태그 목록 | string[] (multi_select) |
| published | 발행일 | string (ISO date) |
| status | 상태 (초안 / 발행됨) | "draft" \| "published" |

### NotionBlock (본문 블록 렌더링용)

| 필드 | 설명 | 타입/관계 |
|------|------|----------|
| id | 블록 ID | string |
| type | 블록 유형 | string (paragraph, heading_1 등) |
| content | 렌더링할 텍스트/URL | string |
| children | 중첩 블록 목록 | NotionBlock[] |

---

## 기술 스택

### 프론트엔드 프레임워크

- **Next.js 16.2.3** (App Router, Turbopack) - 풀스택 React 프레임워크, ISR로 Notion 데이터 캐싱
- **TypeScript 5.6+** - 타입 안전성 보장, Notion API 응답 타입 매핑
- **React 19** - UI 라이브러리

### 스타일링 & UI

- **TailwindCSS v4** (CSS-first, `@import "tailwindcss"`, 설정파일 없는 방식) - 유틸리티 CSS
- **shadcn/ui** (`radix-ui` 기반) - Badge, Card, Input, Separator 등 활용
- **Lucide React** - 아이콘 (검색, 태그, 달력 등)

### CMS 연동

- **@notionhq/client** - Notion API 공식 SDK (DB 쿼리, 블록 조회)
- **notion-to-md** (선택) - Notion 블록을 Markdown으로 변환하는 유틸 라이브러리

### 환경 변수

```bash
NOTION_API_KEY=secret_xxxx          # Notion Integration 시크릿 키
NOTION_DATABASE_ID=xxxx             # 글 목록 데이터베이스 ID
```

### 배포 & 호스팅

- **Vercel** - Next.js 16.2.3 최적화 배포, 환경변수 관리

### 패키지 관리

- **npm** - 의존성 관리

---

## 구현 가이드

### 1단계: Notion API 패키지 설치

```bash
npm install @notionhq/client
# 블록 렌더링 유틸 (선택)
npm install notion-to-md
```

### 2단계: Notion Integration 설정

1. [Notion Developers](https://www.notion.so/my-integrations)에서 Integration 생성
2. 발급된 시크릿 키를 `.env.local`에 `NOTION_API_KEY`로 저장
3. 대상 Notion 데이터베이스에 Integration 연결 (Share → Connections)
4. 데이터베이스 URL에서 ID 추출 후 `NOTION_DATABASE_ID`에 저장

### 3단계: Notion 클라이언트 초기화

`lib/notion.ts` 파일 생성:

```typescript
import { Client } from "@notionhq/client";

// Notion 클라이언트 싱글톤
export const notion = new Client({
  auth: process.env.NOTION_API_KEY,
});

export const DATABASE_ID = process.env.NOTION_DATABASE_ID!;
```

### 4단계: 데이터 페칭 함수

`lib/notion.ts`에 추가:

```typescript
import type {
  PageObjectResponse,
  BlockObjectResponse,
} from "@notionhq/client/build/src/api-endpoints";

// 발행된 글 목록 조회 (Status = 발행됨, 최신순)
export async function getPosts(): Promise<PageObjectResponse[]> {
  const response = await notion.databases.query({
    database_id: DATABASE_ID,
    filter: {
      property: "Status",
      select: { equals: "발행됨" },
    },
    sorts: [{ property: "Published", direction: "descending" }],
  });
  return response.results as PageObjectResponse[];
}

// 개별 글 블록 조회
export async function getPageBlocks(
  pageId: string
): Promise<BlockObjectResponse[]> {
  const response = await notion.blocks.children.list({ block_id: pageId });
  return response.results as BlockObjectResponse[];
}
```

### 5단계: 글 목록 페이지 구현

`app/page.tsx` — 서버 컴포넌트로 Notion 데이터 페칭 후 클라이언트 필터 컴포넌트에 전달:

```typescript
// app/page.tsx (서버 컴포넌트)
import { getPosts } from "@/lib/notion";
import PostList from "@/components/PostList"; // 클라이언트 컴포넌트

export const revalidate = 3600; // 1시간 ISR 캐싱

export default async function HomePage() {
  const posts = await getPosts();
  return <PostList posts={posts} />;
}
```

### 6단계: 글 상세 페이지 구현

`app/posts/[id]/page.tsx` — 동적 라우팅으로 개별 글 렌더링:

```typescript
// app/posts/[id]/page.tsx (서버 컴포넌트)
import { getPageBlocks, notion } from "@/lib/notion";
import BlockRenderer from "@/components/BlockRenderer";

export const revalidate = 3600;

interface Props {
  params: Promise<{ id: string }>;
}

export default async function PostPage({ params }: Props) {
  const { id } = await params;
  const [page, blocks] = await Promise.all([
    notion.pages.retrieve({ page_id: id }),
    getPageBlocks(id),
  ]);
  return <BlockRenderer page={page} blocks={blocks} />;
}
```

### 7단계: Notion 데이터베이스 구조

Notion에서 아래 프로퍼티를 가진 데이터베이스 생성:

| 프로퍼티명 | 유형 | 값 예시 |
|-----------|------|--------|
| Title | 제목 (기본값) | "Claude 프롬프트 엔지니어링 가이드" |
| Category | 선택 (Select) | 학습, 조사, 공식가이드 |
| Tags | 다중 선택 (Multi-select) | claude, prompting, api |
| Published | 날짜 (Date) | 2026-04-15 |
| Status | 선택 (Select) | 초안, 발행됨 |

---

## 디렉토리 구조 (목표)

```
app/
  page.tsx                  # 홈 페이지 (서버 컴포넌트)
  posts/
    [id]/
      page.tsx              # 글 상세 페이지 (서버 컴포넌트)
  globals.css               # TailwindCSS v4 진입점
  layout.tsx                # 루트 레이아웃

components/
  PostList.tsx              # 글 목록 + 카테고리 필터 + 검색 (클라이언트)
  PostCard.tsx              # 개별 글 카드 UI
  BlockRenderer.tsx         # Notion 블록 타입별 렌더링
  layout/
    Navbar.tsx              # 헤더 내비게이션
    Footer.tsx              # 푸터

lib/
  notion.ts                 # Notion 클라이언트 및 데이터 페칭 함수
  utils.ts                  # cn() 유틸리티
```
