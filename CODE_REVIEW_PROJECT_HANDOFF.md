# Code Review Module - 프로젝트 인수인계 문서

> **프로젝트 위치**: `c:\workspace\code-review` (dowa-lighting에서 분리됨)

## 📋 프로젝트 개요

**Hybrid Code Review Module** - AI 기반 코드 리뷰 시스템

- **목적**: dowa-lighting 프로젝트의 서브 모듈로 시작했으나, 독립 프로젝트로 분리
- **스택**: Angular 20 + NestJS + Prisma + PostgreSQL
- **UI 프레임워크**: Tailwind CSS 4 + daisyUI 5

---

## 🏗️ 프로젝트 구조

```
code-review/
├── apps/
│   ├── admin/                 # Angular Admin 앱 (포트 4201)
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── layout/    # 레이아웃 컴포넌트 (완성)
│   │   │   │   ├── pages/     # 페이지 컴포넌트
│   │   │   │   │   ├── sign-in-page/      # 로그인 (완성)
│   │   │   │   │   ├── dashboard-page/    # 대시보드 (스켈레톤)
│   │   │   │   │   ├── reports-page/      # 리포트 (스켈레톤)
│   │   │   │   │   └── guides-page/       # 가이드 (스켈레톤)
│   │   │   │   ├── services/
│   │   │   │   │   └── auth.service.ts    # 인증 서비스 (완성)
│   │   │   │   └── guards/
│   │   │   │       └── auth.guard.ts      # 인증 가드 (완성)
│   │   │   ├── styles/        # 스타일 파일 (dowa-lighting에서 복사)
│   │   │   └── styles.css     # 메인 스타일시트
│   │   └── project.json       # NX 프로젝트 설정
│   ├── admin-e2e/             # E2E 테스트
│   └── server/                # NestJS 서버 (포트 3000)
│       └── src/
│           ├── auth/          # 인증 모듈 (완성)
│           ├── admin/         # 어드민 모듈 (완성)
│           └── prisma/        # Prisma 서비스 (완성)
├── libs/
│   └── api-client/            # 자동 생성 API 클라이언트
├── prisma/
│   ├── schema.prisma          # DB 스키마 (code_review 스키마)
│   └── models/
│       └── admin.prisma       # Admin 모델
└── docker-compose.yml         # PostgreSQL 컨테이너
```

---

## ✅ 완료된 작업

### 1. 백엔드 (NestJS)

- [x] Prisma 설정 (`code_review` 스키마)
- [x] Admin 모델 정의
- [x] AuthModule (JWT 인증)
  - `POST /auth/sign-in` - 로그인
  - `POST /auth/refresh` - 토큰 갱신
  - `GET /auth/me` - 현재 사용자 정보
- [x] AdminModule (CRUD)
  - `GET /admins` - 목록 조회
  - `GET /admins/:id` - 상세 조회
  - `POST /admins` - 생성
  - `PATCH /admins/:id` - 수정
  - `DELETE /admins/:id` - 삭제
- [x] Swagger API 문서 (`/api-docs`)
- [x] 시드 데이터 (기본 관리자)

### 2. 프론트엔드 (Angular)

- [x] 라우팅 설정 (lazy loading)
- [x] AuthService (토큰 관리, 자동 갱신)
- [x] AuthGuard (인증 보호)
- [x] Layout 컴포넌트
  - [x] HeaderComponent
  - [x] SideMenuComponent
  - [x] SideMenuMobileComponent
- [x] SignInPage (로그인 UI)
- [x] API Client 자동 생성 설정

### 3. 인프라

- [x] Docker Compose (PostgreSQL)
- [x] 환경변수 설정 (.env)
- [x] NX 프로젝트 설정

---

## ❌ 현재 문제점 (해결 필요)

### 🔴 Critical: Tailwind CSS + daisyUI가 빌드되지 않음

**증상:**

- `styles.css` 빌드 크기: **47.93 kB** (정상: ~250+ kB)
- 브라우저에서 완전히 스타일 없이 렌더링됨
- daisyUI 클래스 (btn, btn-primary 등)가 CSS에 포함되지 않음

**시도한 해결책 (실패):**

1. `@angular/build:application` → `@angular-devkit/build-angular:application` executor 변경
2. postcss.config.mjs 생성/삭제
3. daisyUI 버전 변경 (5.5.14 → 5.0.46)
4. .angular, node_modules/.cache 캐시 삭제
5. dowa-lighting에서 styles 폴더 전체 복사

**원인 추정:**

- `@plugin "daisyui"` 문법이 postcss에서 처리되지 않음
- dowa-lighting 워크스페이스 내부에 있어서 node_modules 충돌 가능성
- Tailwind CSS 4의 `@plugin` 문법 인식 문제

**비교 (dowa-lighting vs code-review):**
| 항목 | dowa-lighting | code-review |
|------|---------------|-------------|
| styles.css 크기 | 555.69 kB ✅ | 47.93 kB ❌ |
| btn-primary 클래스 | 있음 ✅ | 없음 ❌ |
| @angular-devkit/build-angular | 20.0.5 | 20.0.5 |
| tailwindcss | 4.x | 4.x |
| daisyui | 5.0.46 | 5.0.46 |

---

## 🔧 다음 단계 (우선순위 순)

### 1. 스타일 문제 해결 (CRITICAL)

```bash
# 프로젝트 폴더로 이동
cd c:/workspace/code-review

# node_modules 재설치
rm -rf node_modules package-lock.json
npm install

# 캐시 삭제 후 빌드
rm -rf .angular
npx nx build code-review-admin --configuration=development

# styles.css 크기 확인 (250KB 이상이어야 함)
```

**해결 안 되면 시도할 것:**

1. postcss.config.mjs에 daisyui 명시적 추가
2. Tailwind CSS 3로 다운그레이드 + daisyUI 4 사용
3. dowa-lighting의 전체 스타일 설정 복사

### 2. 기능 구현

- [ ] 대시보드 페이지 구현
- [ ] 코드 리뷰 리포트 CRUD
- [ ] 코딩 가이드 관리
- [ ] AI 코드 분석 연동

---

## 🚀 실행 방법

### 1. 데이터베이스 시작

```bash
docker-compose up -d
```

### 2. Prisma 마이그레이션

```bash
npm run db:generate
npm run db:push
npm run db:seed  # 기본 관리자 생성
```

### 3. 서버 실행

```bash
npx nx serve code-review-server
# 또는 (프로젝트 이름 변경 전이면)
npx nx serve server
```

- API: http://localhost:3000
- Swagger: http://localhost:3000/api-docs

### 4. Admin 앱 실행

```bash
npx nx serve code-review-admin --port=4201
# 또는 (프로젝트 이름 변경 전이면)
npx nx serve admin --port=4201
```

- Admin: http://localhost:4201

---

## 🔑 인증 정보

### 기본 관리자 계정

- **이메일**: admin@example.com
- **비밀번호**: admin123!

### JWT 설정

- Access Token 만료: 1시간
- Refresh Token 만료: 7일
- Secret: `.env` 파일의 `JWT_SECRET`

---

## 📁 주요 파일 위치

### 스타일 관련 (문제 해결 시 확인)

- `apps/admin/src/styles.css` - 메인 스타일시트
- `apps/admin/src/styles/daisy-ui.css` - daisyUI 테마 설정
- `apps/admin/src/styles/tailwind.css` - Tailwind 커스텀
- `apps/admin/project.json` - 빌드 설정

### 인증 관련

- `apps/server/src/auth/` - 서버 인증 모듈
- `apps/admin/src/app/services/auth.service.ts` - 클라이언트 인증
- `apps/admin/src/app/guards/auth.guard.ts` - 라우트 가드

### API 클라이언트

- `libs/api-client/` - 자동 생성된 API 클라이언트
- 재생성: `npx nx serve code-review-server` 실행 시 자동

---

## ⚠️ 주의사항

1. **프로젝트 이름 변경됨**
   - `admin` → `code-review-admin`
   - `server` → `code-review-server`
   - `api-client` → `code-review-api-client`
   - `admin-e2e` → `code-review-admin-e2e`

2. **dowa-lighting과 완전 분리됨**
   - 현재 위치: `c:\workspace\code-review`
   - 독립적인 NX 워크스페이스로 운영

3. **Angular 문법 규칙**
   - `@if`, `@for` 사용 (ngIf, ngFor 금지)
   - `signal()`, `computed()` 사용
   - `inject()` 사용 (constructor 주입 금지)
   - `rxResource` 패턴 사용

---

## 📝 참고: dowa-lighting copilot-instructions

Admin 앱 개발 시 아래 규칙 준수:

```markdown
### Admin Modal Guidelines

- `<app-modal-container title="제목">` 사용
- Footer 버튼은 `[footer]` 슬롯 사용 (콘텐츠 내부에 넣지 않음)

### Angular Syntax

- `@if`, `@else`, `@for` 사용 (ngIf, ngFor 금지)
- `@defer` for lazy loading
- `inject()` for DI
- `signal()`, `computed()` for state
- `rxResource` for data fetching

### NestJS Syntax

- `@ApiTags`, `@ApiOperation`, `@ApiOkResponse` 필수
- `PrismaService` 주입
- `prisma.$transaction` for multiple operations
```

---

_마지막 업데이트: 2026년 1월 22일_
