# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

React 19 + TypeScript + Vite 기반 노트 앱 실습 프로젝트. 강의용으로 tags 등 기능이 의도적으로 누락되어 있음 (Note 타입에 주석으로 표시).

## Commands

| 명령어 | 설명 |
|--------|------|
| `npm run dev` | 프론트엔드 + JSON Server 동시 실행 (concurrent) |
| `npm run build` | `tsc && vite build` |
| `npm run lint` | ESLint (자동 수정) |
| `npm run format` | Prettier 포맷 |
| `npm test` | Vitest 1회 실행 |
| `npm run test:watch` | Vitest watch 모드 |
| `npm run server` | JSON Server 단독 실행 (port 3001) |

## Architecture

**단일 페이지, 사이드바 + 에디터 레이아웃 구조:**

- `App.tsx` — 최상위 상태 관리 (selectedNoteId, isCreating). Layout에 sidebar/main slot 전달
- `Layout.tsx` — 반응형 2컬럼 레이아웃 (사이드바 + 메인)
- `NoteList` → `NoteItem` — 노트 목록 조회/선택/삭제
- `NoteEditor` — 노트 생성/수정 폼 (isCreating 상태로 모드 전환)

**상태 관리 패턴:**
- `NotesContext` (React Context) — 노트 CRUD + loading/error 상태를 전역 제공
- `useNotes()` 커스텀 훅으로 접근 (Provider 외부 사용 시 에러 throw)

**API 계층:**
- `src/api/notes.ts` — JSON Server (`http://localhost:3001`) 대신 순수 fetch 사용
- CRUD: getNotes, createNote, updateNote(PATCH), deleteNote
- `db.json`이 JSON Server의 데이터베이스 역할

**스타일링:**
- Tailwind CSS v4 (`@tailwindcss/vite` 플러그인)
- `index.css`의 `@theme` 블록에서 디자인 토큰 정의 (background, card, foreground, muted, border, destructive 커스텀 컬러)
- Pretendard Variable 기본 폰트, 한국어 UI

## Code Conventions

- Prettier: singleQuote, semi true, trailingComma all, printWidth 100, tabWidth 2
- 타입 정의는 `src/types/`에 별도 파일로 분리
- 한국어 UI 텍스트, 한국어 날짜 포맷 (`ko-KR`)

### 상태 관리 패턴

- `NotesContext` (React Context) → `useNotes()` 커스텀 훅으로 전역 CRUD + loading/error 제공
- Provider 외부 사용 시 에러 throw 가드 패턴 (`NotesContext.tsx:53`)
- 로컬 UI 상태(selectedNoteId, isCreating)는 App.tsx에서 useState로 관리, Context와 분리

### 컴포넌트 패턴

- 함수 선언문 + named export (`export function Component()`)
- Props는 컴포넌트 파일 상단에 `interface ComponentProps` 정의
- 컴포지션 패턴: Layout에 `sidebar`/`main` slot을 children 대신 props로 전달

### API 호출 패턴

- `src/api/notes.ts`에 순수 async function으로 CRUD 함수 정의
- 모든 함수: 명시적 Promise 반환 타입 + `if (!res.ok) throw new Error(...)` 에러 처리
- 에러 처리: `alert()` 사용 금지, `console.error()`로 통일
- API_URL 상수로 베이스 URL 관리, template literal로 엔드포인트 조합

### 네이밍 패턴

- 이벤트 핸들러 내부: `handle*` (handleSave, handleSelectNote)
- 핸들러 props: `on*` (onSelect, onDelete, onDone, onNewNote)
- 컴포넌트: PascalCase, 파일명도 PascalCase (NoteEditor.tsx)
- API 함수: camelCase CRUD 동사 통일 (getNotes, createNote, updateNote, deleteNote)
- CSS 커스텀 컬러: 시맨틱 네이밍 (background, card, foreground, muted, border, destructive)

### ⚠️ 일관성 없는 패턴

- **App.tsx export**: `export default App` 사용, 나머지 모든 컴포넌트는 named export. 새 컴포넌트 추가 시 named export로 통일 권장
- **NoteEditor.tsx:27**: `useEffect` 의존성 배열에 eslint-disable-line 주석으로 경고를 무시하는 패턴. 필요 시 `selectedNote` 객체를 직접 의존성에 넣는 방식 고려

## Test Setup

Vitest + jsdom + @testing-library/react. 설정은 `vite.config.ts`의 `test` 필드와 `src/test-setup.ts`에 정의.
