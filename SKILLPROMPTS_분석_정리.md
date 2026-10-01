# SkillPrompts 전수조사 분석 & 활용 전략 정리

> 작성일: 2026-10-01
> 분석 대상 저장소: **https://github.com/bmshin94/SkillPrompts**
> 원본(Upstream) 저장소: **https://github.com/Ademking/SkillPrompts**
> 분석 기준 버전: **v1.1.0**
> 라이선스: **MIT**

---

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [전수조사 결과 — 폴더/파일 구조](#2-전수조사-결과--폴더파일-구조)
3. [동작 원리 해부](#3-동작-원리-해부)
4. [핵심 개념 — 프롬프트 / 변수 / 블록](#4-핵심-개념--프롬프트--변수--블록)
5. [내장 프롬프트 169개 분석](#5-내장-프롬프트-169개-분석)
6. [보안 & 프라이버시 검증](#6-보안--프라이버시-검증)
7. [설치 및 사용법](#7-설치-및-사용법)
8. [정체성 — 플러그인? 스킬? MCP?](#8-정체성--플러그인-스킬-mcp)
9. [API 토큰 필요 여부](#9-api-토큰-필요-여부)
10. [AI 에이전트 구축에 도움이 되는가](#10-ai-에이전트-구축에-도움이-되는가)
11. [React / PHP 로 만들 수 있는가](#11-react--php-로-만들-수-있는가)
12. [유튜브 강의 콘텐츠 제작 가능성](#12-유튜브-강의-콘텐츠-제작-가능성)
13. [수익화 전략 8가지](#13-수익화-전략-8가지)
14. [실행 로드맵](#14-실행-로드맵)
15. [법적 체크리스트](#15-법적-체크리스트)
16. [개선이 필요한 지점](#16-개선이-필요한-지점)
17. [참고 링크 모음](#17-참고-링크-모음)

---

## 1. 프로젝트 개요

### 한 줄 정의
> **AI 챗봇 입력창에서 `/` 를 입력하면 저장해 둔 프롬프트 목록이 떠서 즉시 삽입할 수 있는 브라우저 확장 프로그램**

Notion / Slack 의 슬래시 커맨드 UX 를 ChatGPT · Gemini 입력창에 이식한 도구다.

### 기본 정보

| 항목 | 내용 |
|---|---|
| 이름 | SkillPrompts |
| 버전 | 1.1.0 (2026-05-13) |
| 원작자 | Adem Kouki ([GitHub](https://github.com/Ademking) / [LinkedIn](https://www.linkedin.com/in/ademkouki/)) |
| 정체 | 브라우저 확장 프로그램 (Chrome MV3 / Firefox MV3) |
| 프레임워크 | Plasmo 0.90.5 |
| UI | React 18.2.0 + TypeScript 5.3.3 + TailwindCSS 3.4.1 |
| 스토리지 | `@plasmohq/storage` (= `chrome.storage.local` 래퍼) |
| 소스 규모 | 약 3,580줄 (src + docs) |
| 라이선스 | MIT (상업적 이용 · 수정 · 재배포 허용) |
| 랜딩 페이지 | https://skillprompts.surge.sh/ |

### 배포 채널

- Chrome Web Store: https://chromewebstore.google.com/detail/skillprompts/lmonnhccbnchmhgpdmfllmallciokckl
- Firefox Add-ons: https://addons.mozilla.org/en-US/firefox/addon/skillprompts/
- Releases (ZIP): https://github.com/Ademking/SkillPrompts/releases

### 지원 사이트 현황

| 상태 | 사이트 |
|---|---|
| ✅ 지원 | chatgpt.com, chat.openai.com, gemini.google.com |
| ⬜ 로드맵(미지원) | copilot.microsoft.com, huggingface.co/chat, chat.mistral.ai, poe.com, perplexity.ai, chat.deepseek.com, kimi.com, v0.app, github.com/copilot, grok.com, cursor.com/agents, **claude.ai** |

> 한국 서비스(뤼튼, 네이버 Cue: 등)는 로드맵에도 없음 → **차별화 기회**

---

## 2. 전수조사 결과 — 폴더/파일 구조

총 **50개 파일** (`.git` 제외) 전수 확인 완료.

```
SkillPrompts/
├── src/                          # 확장 본체
│   ├── content.tsx       (946줄) # ★ 페이지 주입 스크립트 (핵심)
│   ├── options.tsx       (785줄) # ★ 관리 대시보드 SPA
│   ├── background.ts      (71줄) # 서비스 워커 (설치 시 시딩)
│   ├── prompts.json              # 내장 프롬프트 169개
│   ├── types.ts           (19줄) # Prompt / Block / LibraryPrompt
│   ├── utils.ts           (17줄) # resolveBlocks(), blockNames()
│   ├── options.html              # 옵션 페이지 셸
│   ├── style.css                 # Tailwind 엔트리
│   └── components/        (14개)
│       ├── CommandPalette.tsx    (293줄)  슬래시 팔레트
│       ├── FormModal.tsx         (231줄)  프롬프트 생성/수정
│       ├── VariableModal.tsx     (170줄)  변수 입력
│       ├── PromptCard.tsx        (135줄)  그리드 카드
│       ├── BlockModal.tsx        (122줄)  재사용 블록 관리
│       ├── LibraryModal.tsx       (92줄)  169개 라이브러리 브라우저
│       ├── PromptRow.tsx          (89줄)  리스트 행
│       ├── ViewPromptModal.tsx    (76줄)  미리보기
│       ├── ViewToggle.tsx         (51줄)  그리드/리스트 전환
│       ├── DeleteConfirmModal.tsx (49줄)
│       ├── Logo.tsx / Icons.tsx / Toast.tsx / Field.tsx
│
├── docs/                         # 환영·랜딩 페이지 (별도 앱)
│   ├── src/App.tsx       (311줄)
│   ├── src/Logo.tsx / main.tsx / index.css
│   ├── public/logo.svg
│   └── package.json              # React 19 + Vite 8 + Tailwind 4
│
├── .github/workflows/submit.yml  # 웹스토어 자동 제출 CI (workflow_dispatch)
├── assets/icon.png
├── package.json                  # Plasmo 설정 + manifest 오버라이드
├── tsconfig.json / tailwind.config.js / postcss.config.js
├── .prettierrc.mjs / .gitignore / global.d.ts
├── pnpm-lock.yaml
├── CLAUDE.md                     # 프로젝트 전용 AI 페르소나 가이드
├── README.md
└── LICENCE                       # MIT, (c) 2026 Adem Kouki
```

### 커밋 히스토리 (최근)

```
3baeb7e  Merge pull request #1 from bmshin94/feat/claude-guide
f341f0d  docs: created CLAUDE.md persona guide
505b5a6  Enhance README with download links and badges
0acc1fa  feat: update README with version 1.1.0 details, add changelog
2eca6bb  feat: update version to 1.1.0
89de90b  feat: add reusable blocks functionality
25ea909  feat: add instructional text to FormModal, LibraryModal, OptionsIndex
727049d  feat: add favorite functionality to prompts
2ffe56b  feat: enhance FormModal and Toast with initialData and error handling
0fbc33f  feat: add import/export functionality
```

---

## 3. 동작 원리 해부

### 3-Tier 구조

```
┌──────────────────────────────────────────────┐
│ options.tsx — 관리 대시보드                   │
│   확장 아이콘 클릭 → 옵션 페이지              │
│   프롬프트 CRUD / 블록 / 테마 / Import·Export │
└──────────────────────────────────────────────┘
┌──────────────────────────────────────────────┐
│ background.ts — 서비스 워커 (상주)            │
│   onInstalled → 기본 프롬프트 16개 시딩       │
│   onInstalled → 환영 페이지 탭 오픈           │
│   action.onClicked → 옵션 페이지 열기         │
│   (Firefox 분기: chrome.browserAction)        │
└──────────────────────────────────────────────┘
┌──────────────────────────────────────────────┐
│ content.tsx — 페이지 주입 스크립트 (핵심)      │
│   matches: chatgpt.com / chat.openai.com /    │
│            gemini.google.com                  │
│   "/" 키 감지 → 팔레트 표시 → 텍스트 삽입      │
└──────────────────────────────────────────────┘
                     ↓
       chrome.storage.local (로컬 저장소)
```

### 스토리지 키

| 키 | 용도 |
|---|---|
| `skillprompts_prompts` | 프롬프트 배열 |
| `skillprompts_blocks` | 재사용 블록 배열 |
| `skillprompts_usage` | 프롬프트별 사용 횟수(추천 정렬용) |
| `skillprompts_enabled` | 확장 전체 ON/OFF |
| `skillprompts_theme` | light / dark |
| `skillprompts_view` | grid / list |

> `options.tsx` 에는 과거 `window.localStorage` 에 저장된 데이터를 `chrome.storage` 로 옮기는 **레거시 마이그레이션 로직**이 포함돼 있다.

### 삽입 플로우 (content.tsx)

```
1. keydown 리스너가 "/" 감지 (event.key === "/", isVisible === false)
   ※ composition(한글 입력) 상태 고려 — event.isComposing 체크
2. 캐럿 위치 계산 → CommandPalette 를 해당 좌표에 렌더
3. 검색어 입력 → 필터링 (즐겨찾기 우선 + 사용 횟수 가중)
4. ↑↓ 이동, Enter 선택, Esc 닫기
5. 선택된 템플릿에서 블록 먼저 치환 (resolveBlocks)
6. 남은 {{변수}} 가 있으면 VariableModal 오픈 → 값 수집
7. 최종 문자열 삽입
```

### 삽입이 기술적으로 어려운 이유 (하이라이트)

ChatGPT 입력창은 단순 `<textarea>` 가 아니라 **ProseMirror 기반 contenteditable** 이다.
`element.value = "..."` 로 값을 넣으면 화면에는 보이지만 React 내부 상태가 갱신되지 않아 **빈 메시지가 전송**된다.

이 프로젝트가 사용한 해법:

```ts
// 1) 브라우저가 "사용자가 직접 타이핑했다"고 인식하도록 유도
document.execCommand("insertText", false, processed)

// 2) React/Vue 등 프레임워크 상태 동기화를 위한 수동 이벤트 디스패치
element.dispatchEvent(new Event("input",  { bubbles: true }))
element.dispatchEvent(new Event("change", { bubbles: true }))

// 3) Shadow DOM 재귀 탐색으로 contenteditable 입력창 찾기
root.querySelectorAll('[contenteditable]:not([contenteditable="false"])')

// 4) 삽입 토큰 하이라이팅 — CSS Custom Highlight API
CSS.highlights.set("command-insert", new Highlight(range))
// 미지원 브라우저는 <span class="command-insert-fallback"> 폴백
```

### CSS 격리

```ts
// Plasmo CSUI — Shadow DOM 안에서 렌더하며 :root 를 :host 로 치환
let updatedCssText = cssText.replaceAll(":root", ":host(plasmo-csui)")
// rem → px 환산 (호스트 페이지의 font-size 영향 차단)
updatedCssText = updatedCssText.replace(/([\d.]+)rem/g, (m, v) => `${parseFloat(v) * 16}px`)
```

> Tailwind 클래스에 `plasmo-` 프리픽스를 사용해 호스트 페이지 스타일과 충돌을 방지한다.

---

## 4. 핵심 개념 — 프롬프트 / 변수 / 블록

### 타입 정의 (`src/types.ts`)

```ts
export interface Prompt {
    id: string
    label: string          // 슬래시 명령어 이름 (예: debug)
    description: string    // 팔레트에 표시될 설명
    template: string       // 실제 삽입될 본문
    favorite?: boolean
}

export interface Block {
    id: string
    name: string           // {{name}} 으로 참조
    value: string          // 고정 치환 값
}

export interface LibraryPrompt {
    label: string
    description: string
    prompt: string
}
```

### 변수(Variable) vs 블록(Block)

| 구분 | 변수 (Variable) | 블록 (Block) |
|---|---|---|
| 문법 | `{{name}}` | `{{name}}` (동일) |
| 값 | 사용할 때마다 다름 | 고정 |
| 입력 시점 | 삽입 직전 모달로 질의 | 미리 등록, 자동 치환 |
| 관리 위치 | 템플릿 안에 그냥 작성 | Blocks 메뉴에서 등록 |
| 비유 | 신청서 빈칸 | 도장 |
| 예시 | 번역 언어, 주제, 대상 독자 | 내 기술 스택, 회사 소개, 코딩 규칙 |

### 구분 로직

둘 다 `{{ }}` 문법을 쓰는데, 구분은 **이름이 Blocks 에 등록되어 있는지** 로 판정한다.

```ts
// src/utils.ts
export function resolveBlocks(template: string, blocks: Block[]): string {
    let result = template
    for (const b of blocks) {
        const regex = new RegExp(`\\{\\{\\s*${escapeRegex(b.name)}\\s*\\}\\}`, "g")
        result = result.replace(regex, b.value)
    }
    return result
}

export function blockNames(blocks: Block[]): Set<string> {
    return new Set(blocks.map(b => b.name.toLowerCase()))
}
```

```ts
// 블록으로 치환된 뒤 남은 토큰 = 변수
function extractVariables(template: string, blocks?: Block[]): string[] {
    const resolved = blocks ? resolveBlocks(template, blocks) : template
    const matches = resolved.match(/\{\{\s*([\w-]+)\s*\}\}/g)
    if (!matches) return []
    return [...new Set(matches.map(m => m.slice(2, -2).trim()))]
}
```

### 활용 예시

```
[블록 등록]
  my_stack = "Next.js 14, TypeScript, Prisma, PostgreSQL, Vercel"
  my_rule  = "코드에 한국어 주석과 에러 핸들링을 반드시 포함할 것"

[프롬프트 템플릿]
  너는 {{my_stack}} 전문 시니어 개발자야.
  아래 Pull Request 를 {{focus}} 관점으로 리뷰해줘.
  {{my_rule}}

[삽입 시]
  - my_stack, my_rule → 자동 치환 (질문 없음)
  - focus            → 모달에서 입력 요청
```

---

## 5. 내장 프롬프트 169개 분석

`src/prompts.json` 에 **169개** 프롬프트가 내장돼 있고, 그중 **16개**가 설치 시 자동 시딩된다.

### 자동 시딩되는 기본 16개 (`background.ts` DEFAULT_LABELS)

```
viral-post, facebook-post, reddit-post, linkedin-post,
debug, blog, formalizer, compare, expander, shortener,
simplifier, ideas, translate-to, corrector, tldr, explain
```

### 카테고리 분포

| 카테고리 | 대표 라벨 |
|---|---|
| 💻 개발 | `debug`, `code-reviewer`, `regex-generator`, `sql-query-builder`, `git-commit-writer`, `docker-expert`, `readme-generator`, `test-case-generator`, `documentation-generator`, `bug-report-writer`, `database-designer`, `json-generator`, `yaml-generator`, `api-response-generator`, `linux-server-admin`, `cybersecurity-auditor`, `it-architect`, `ux-ui-developer`, `ethereum-developer`, `data_model` |
| 📱 SNS/마케팅 | `linkedin-post`, `linkedin-thread`, `viral-post`, `x-twitter-thread`, `reddit-post`, `facebook-post`, `content-repurposer`, `tweet-generator`, `advertiser`, `social-media-manager` |
| 💼 비즈니스 | `business-plan`, `startup-mentor`, `business-coach`, `proposal-writer`, `salary-negotiation-coach`, `sales-coach`, `marketing-strategist`, `brand-consultant`, `business-idea-validator`, `product-requirements-document` |
| ✍️ 글쓰기/편집 | `formalizer`, `shortener`, `expander`, `corrector`, `tldr`, `clarifier`, `polisher`, `rewriter-professional`, `bullet-point-converter`, `bullet-to-paragraph`, `markdown-converter` |
| 🎓 교육/연구 | `teacher-assistant`, `thesis-adviser`, `research-paper-writer`, `student-coach`, `online-course-creator`, `math-teacher`, `philosophy-teacher`, `ai-writing-tutor` |
| 🧑‍💼 커리어 | `cv-resume-writer`, `cover-letter-writer`, `interview-coach`, `job-interviewer`, `recruiter`, `career-counselor`, `career-roadmap`, `linkedin-optimizer` |
| 🏠 라이프스타일 | `meal-planner`, `travel-itinerary-planner`, `budget-planner`, `fitness-routine-generator`, `habit-builder`, `recipe-generator`, `wedding-planner`, `pet-care-assistant`, `interior-design-helper`, `fashion-stylist` |
| 🎭 롤플레이/시뮬 | `linux-terminal`, `javascript-console`, `sql-terminal`, `excel-sheet`, `text-based-adventure-game`, `stand-up-comedian`, `rapper`, `storyteller`, `screenwriter`, `poet` |
| 🧠 사고/논증 | `fallacy-finder`, `socratic-method`, `debater`, `debate-coach`, `decision-helper`, `business-problem-solver`, `statistician`, `historian` |
| 🤖 메타 | `ai-prompt-engineer`, `prompt-generator` |

### 프롬프트 작성 패턴 (Role–Task–Format)

내장 프롬프트는 일관된 구조를 따른다. 이 패턴 자체가 좋은 학습 자료다.

```
[역할 부여] Act as a senior software engineer and debugging expert.
[입력 정의] I will provide code snippets, error messages, or unexpected behavior.
[작업 지시] Your task is to identify the root cause of the issue, explain it
            clearly, and provide a corrected, improved version of the code.
[추가 요구] Also suggest best practices or optimizations when relevant.
[입력 마커] My issue is:
```

---

## 6. 보안 & 프라이버시 검증

소스 전체를 패턴 검색하여 검증한 결과.

```bash
$ grep -rniE "api[_-]?key|token|bearer|secret|openai.com/v1" src/ docs/src/
→ 해당 없음
```

### 검증 결과표

| 항목 | 결과 | 근거 |
|---|---|---|
| API 키 / 토큰 사용 | ❌ 없음 | 전체 소스 검색 결과 0건 |
| 외부 서버 전송 | ❌ 없음 | `fetch` 호출은 `options.tsx:277` 1곳뿐 |
| 그 1곳의 용도 | ✅ 사용자가 **직접 입력한** GitHub URL 에서 `.md` 를 읽어오는 Import 기능 | `handleImportFromUrl()` |
| 데이터 저장 위치 | ✅ `chrome.storage.local` (사용자 기기) | `@plasmohq/storage` |
| 데이터 수집 선언 | ✅ `data_collection_permissions: { required: ["none"] }` | `package.json` (Firefox manifest) |
| 권한 범위 | ✅ ChatGPT / Gemini 3개 도메인만 | `host_permissions` |
| CSP | ✅ `script-src 'self'; object-src 'self'` | 외부 스크립트 실행 차단 |
| 애널리틱스 / 트래커 | ❌ 없음 | 의존성에 해당 패키지 없음 |

### 결론

**100% 로컬 동작. 서버 없음. 프롬프트 데이터가 외부로 나가지 않는다.**

### 다만 유의할 점

- `storage.local` 사용 → **기기 간 동기화 불가**. 백업은 JSON Export 수동 수행 필요.
- `storage.sync` 로 전환하면 동기화는 되지만 용량 제한(약 100KB)이 있어 트레이드오프 존재.

---

## 7. 설치 및 사용법

### 7-1. 설치 — 3가지 경로

#### 방법 A. 스토어 설치 (권장)

- Chrome 계열: https://chromewebstore.google.com/detail/skillprompts/lmonnhccbnchmhgpdmfllmallciokckl
- Firefox: https://addons.mozilla.org/en-US/firefox/addon/skillprompts/

#### 방법 B. ZIP 수동 설치

**Chromium 계열 (Chrome / Edge / Brave / Whale)**
1. Releases 에서 `chrome-mv3-prod.zip` 다운로드
2. 압축 해제
3. 주소창에 `chrome://extensions/`
4. 우측 상단 **개발자 모드** ON
5. **압축해제된 확장 프로그램을 로드합니다** → 압축 푼 폴더 선택

**Firefox**
1. `firefox-mv3-prod.zip` 다운로드 후 압축 해제
2. 주소창에 `about:debugging#/runtime/this-firefox`
3. **임시 부가 기능 로드** → `manifest.json` 선택
   - ⚠️ 임시 로드는 브라우저 재시작 시 사라짐

#### 방법 C. 소스 빌드 (개조용)

```bash
git clone https://github.com/bmshin94/SkillPrompts.git
cd SkillPrompts

npm install -g pnpm     # pnpm 미설치 시
pnpm install

# 개발 모드 (HMR)
pnpm dev                # → build/chrome-mv3-dev
pnpm dev:firefox
pnpm dev:verbose

# 프로덕션 빌드
pnpm build              # 기본 타깃
pnpm build:chrome       # → build/chrome-mv3-prod.zip
pnpm build:firefox      # → build/firefox-mv3-prod.zip (소스맵 포함)
pnpm build:all
pnpm package
```

**랜딩 페이지(docs/) 별도 실행**
```bash
cd docs
npm install
npm run dev      # http://localhost:5173
npm run build    # dist/
npm run lint
```

### 7-2. 사용법

#### STEP 1 — 첫 실행
설치 직후 `background.ts` 의 `onInstalled` 훅이 동작한다.
- 기본 프롬프트 16개 자동 시딩
- 환영 페이지(`https://skillprompts.surge.sh/`) 새 탭 오픈

#### STEP 2 — 프롬프트 삽입 (핵심 플로우)
1. ChatGPT 또는 Gemini 접속
2. 입력창 클릭(포커스)
3. **`/` 입력** → 커맨드 팔레트 등장
4. 검색어 타이핑 → `↑` `↓` 이동
5. `Enter` → 삽입
6. 변수가 있으면 모달에서 값 입력 후 Insert
7. `Esc` 로 닫기

#### STEP 3 — 관리 대시보드
툴바의 확장 아이콘 클릭 → 옵션 페이지

| 기능 | 설명 |
|---|---|
| **+ New Prompt** | label / description / template 직접 작성 |
| **Library** | 내장 169개에서 선택 추가 |
| **Blocks** | 재사용 블록 등록 · 수정 · 삭제 |
| **Import / Export** | JSON 백업·복원, **GitHub URL 의 `.md` 가져오기** |
| **⭐ 즐겨찾기** | 팔레트 상단 고정 |
| **👁 미리보기** | 변수 치환 후 최종 결과 확인 |
| **🌙 / ☀️** | 다크 · 라이트 테마 |
| **⊞ / ☰** | 그리드 · 리스트 뷰 |
| **ON / OFF** | 확장 전체 비활성화 |

#### STEP 4 — GitHub 에서 프롬프트 가져오기

Import/Export 패널의 URL 입력란에 GitHub `.md` 주소를 넣으면
`convertGithubUrl()` 이 raw URL 로 변환 → `parseMarkdownToPrompt()` 가 파싱 →
라벨 중복 시 `_2`, `_3` 자동 넘버링 후 폼에 채워준다.

---

## 8. 정체성 — 플러그인? 스킬? MCP?

### 결론: **브라우저 확장 프로그램 (Browser Extension)**

이름에 "Skill" 이 들어가지만 **Claude Skill 과는 무관**하다. 게임 스킬처럼 빠르게 쓴다는 네이밍이다.

| 구분 | 실행 위치 | 역할 | SkillPrompts |
|---|---|---|---|
| 🧩 브라우저 확장 | 브라우저 내부 | 웹페이지 조작 / UI 주입 | ✅ **해당** |
| 🔌 일반 플러그인 | 호스트 앱 내부 | 앱 기능 확장 (예: VSCode Extension) | ❌ |
| 🎯 Claude Skill | Claude 실행 환경 | AI 에게 전문 절차·지식 주입 | ❌ |
| 🔗 MCP 서버 | 별도 프로세스/서버 | AI 에게 도구·데이터 연결 제공 | ❌ |
| 🤖 ChatGPT GPTs | OpenAI 서버 | 커스텀 챗봇 | ❌ |

### 코드 근거

```json
// package.json — Plasmo + MV3 매니페스트
{ "dependencies": { "plasmo": "0.90.5" },
  "manifest": { "host_permissions": ["https://chatgpt.com/*", ...] } }
```
```ts
// background.ts — Chrome Extension API
chrome.action.onClicked.addListener(...)
chrome.runtime.onInstalled.addListener(...)
chrome.tabs.create({ url })
```
```tsx
// content.tsx — Plasmo Content Script 설정
export const config: PlasmoCSConfig = { matches: [...] }
```

### 핵심 차이

```
🧩 브라우저 확장 : AI 웹사이트 위에 덧씌우는 리모컨.
                  AI 능력은 그대로. 입력 속도만 빨라짐.
🎯 Claude Skill  : AI 에게 주는 전문 매뉴얼. AI 행동 자체가 변함.
🔗 MCP           : AI 에 꽂는 USB 포트. DB·파일·API 직접 접근 가능.
```

SkillPrompts 는 AI 를 전혀 건드리지 않는다. **AI 입장에서는 사람이 타이핑한 것과 동일**하다.

---

## 9. API 토큰 필요 여부

### 결론: **불필요. 가입 · 결제 · API 키 전부 0.**

### 이유

```
❌ 토큰이 필요한 구조
   내 앱 → [API 키] → OpenAI 서버 → 응답   (내가 과금됨)

✅ SkillPrompts 구조
   확장 → ChatGPT 입력창에 텍스트 삽입
        → 사용자가 Enter
        → ChatGPT 가 사용자 계정으로 처리
   (확장은 LLM 호출 자체를 하지 않음)
```

키보드 매크로 프로그램이 API 키를 요구하지 않는 것과 동일한 원리다.

| 항목 | 상태 |
|---|---|
| 회원가입 | ❌ 불필요 |
| API 키 | ❌ 불필요 |
| 결제 | ❌ 완전 무료 |
| 서버 | ❌ 없음 (100% 클라이언트) |
| 외부 전송 | 🚫 없음 |

---

## 10. AI 에이전트 구축에 도움이 되는가

### 결론: **직접적으로는 ❌ / 간접적으로는 ⭕⭕⭕**

### 직접적으로 부족한 이유

| 에이전트 필수 요소 | SkillPrompts |
|---|---|
| LLM API 호출 | ❌ |
| Tool / Function Calling | ❌ |
| 자율 루프 (Plan → Act → Observe) | ❌ |
| 메모리 · 대화 상태 관리 | ❌ |
| 태스크 플래닝 | ❌ |
| 멀티 에이전트 오케스트레이션 | ❌ |
| RAG / 벡터 DB | ❌ |

→ 이 프로젝트는 **수동 트리거 텍스트 삽입기**. 에이전트는 **자율 실행 시스템**. 레이어가 다르다.

### 간접적으로 매우 유용한 이유

#### ① 프롬프트 엔지니어링 실전 샘플 169개
Role–Task–Format 패턴이 일관되게 적용돼 있어 **에이전트 System Prompt 설계 레퍼런스**로 그대로 활용 가능.

#### ② 템플릿 엔진 설계 패턴
`resolveBlocks()` 는 LangChain `PromptTemplate`, Jinja2, Handlebars 가 하는 일의 축소판이다.
특히 **"변수 = 런타임 주입" vs "블록 = 정적 컨텍스트"** 분리 개념은 에이전트 설계에 그대로 대응된다.

#### ③ AI 웹 UI 조작 기술 (가장 가치 있는 부분)
`execCommand` 로 ProseMirror 우회, Shadow DOM 재귀 탐색, 수동 이벤트 디스패치 —
**브라우저 자동화 에이전트**(Browser-Use, Agent-E, Playwright 기반 에이전트)를 만들 때
반드시 마주치는 문제들의 실전 해법이 담겨 있다.

#### ④ Human-in-the-Loop UX 패턴
VariableModal = "에이전트가 실행 전에 사람에게 확인받는" 패턴의 좋은 예시.

### 에이전트로 진화시키는 로드맵

```
📍 현재 (v1.1.0)  수동 삽입
📍 Level 1        삽입 후 자동 Enter → 원클릭 실행
📍 Level 2        체인: A 실행 → DOM 에서 응답 파싱 → 응답을 변수로 B 실행
                  예) /research → /summarize → /blog
📍 Level 3        조건 분기: 응답에 "에러" 포함 시 /debug 자동 실행
📍 Level 4        멀티 사이트 오케스트레이션:
                  ChatGPT 질문 → Gemini 동일 질문 → Claude 가 비교·평가
📍 Level 5        평가 루프(Reflexion): LLM-as-Judge 로 채점 → 미달 시 재시도
```

> Level 2(체인)만 구현해도 **"API 키 없이 돌아가는 브라우저 기반 노코드 AI 워크플로우"** 라는
> 상당히 강력한 차별점이 생긴다.

---

## 11. React / PHP 로 만들 수 있는가

### React — 이미 React 로 만들어져 있음

```json
// 확장 본체
"react": "18.2.0", "react-dom": "18.2.0", "tailwindcss": "3.4.1"
// docs/ 랜딩 페이지 (더 최신)
"react": "^19.2.5", "vite": "^8.0.10", "tailwindcss": "^4.3.0"
```

| 파일 | React 사용 |
|---|---|
| `content.tsx` | ✅ Shadow DOM 안에서 렌더 |
| `options.tsx` | ✅ 풀 React SPA |
| `components/*` 14개 | ✅ 함수형 컴포넌트 + Hooks |
| `docs/src/App.tsx` | ✅ React 19 |

#### 사이트 추가 예시

```tsx
// src/content.tsx
export const config: PlasmoCSConfig = {
  matches: [
    "https://chatgpt.com/*",
    "https://chat.openai.com/*",
    "https://gemini.google.com/*",
    "https://claude.ai/*",            // 추가
    "https://www.perplexity.ai/*",    // 추가
    "https://wrtn.ai/*"               // 추가
  ]
}
```
```json
// package.json — host_permissions 에도 동일하게 추가 필요
"host_permissions": [
  "https://chatgpt.com/*", "https://chat.openai.com/*",
  "https://gemini.google.com/*", "https://claude.ai/*",
  "https://www.perplexity.ai/*", "https://wrtn.ai/*"
]
```
> ⚠️ 사이트마다 입력창 DOM 구조가 달라 삽입 로직 보강이 추가로 필요하다.

### PHP — 확장 본체는 불가, 백엔드로는 완벽히 가능

브라우저 확장은 HTML + CSS + JavaScript 로만 동작한다. PHP 로 content script 를 작성할 수는 없다.
대신 **백엔드 레이어**를 PHP(Laravel)로 구축하면 된다.

```
┌───────────────────────────────────┐
│ 🧩 확장 (React + TS + Plasmo)      │
│   팔레트 UI / 입력창 삽입          │
└──────────────┬────────────────────┘
               │ REST API (JSON)
┌──────────────▼────────────────────┐
│ 🐘 PHP 백엔드 (Laravel 11)         │
│   인증(Sanctum) / 클라우드 동기화   │
│   팀 공유 / 권한 / 마켓 / 통계      │
└──────────────┬────────────────────┘
               ▼  PostgreSQL · MySQL
```

```php
// routes/api.php
Route::middleware('auth:sanctum')->group(function () {
    Route::get   ('/prompts',           [PromptController::class, 'index']);
    Route::post  ('/prompts',           [PromptController::class, 'store']);
    Route::put   ('/prompts/{id}',      [PromptController::class, 'update']);
    Route::delete('/prompts/{id}',      [PromptController::class, 'destroy']);
    Route::post  ('/sync',              [SyncController::class,  'push']);
    Route::get   ('/team/{id}/prompts', [TeamController::class,  'prompts']);
});
Route::get('/marketplace/packs', [MarketController::class, 'index']);
```

```php
// app/Models/Prompt.php
class Prompt extends Model {
    protected $fillable = ['user_id','team_id','label','description','template','is_favorite'];
    public function user() { return $this->belongsTo(User::class); }
    public function team() { return $this->belongsTo(Team::class); }
}
```

```ts
// 확장 → 백엔드 호출
const res = await fetch("https://api.example.kr/api/sync", {
  method: "POST",
  headers: { Authorization: `Bearer ${userToken}`, "Content-Type": "application/json" },
  body: JSON.stringify({ prompts, blocks })
})
```
> ⚠️ 자체 API 도메인도 `host_permissions` 에 추가해야 한다.

### 추천 스택

| 레이어 | 추천 | 비고 |
|---|---|---|
| 확장 프론트 | React 18 + TS + Plasmo (현행 유지) | 완성도 높음 |
| 랜딩 | React 19 + Vite (현행 유지) | 최신 스택 |
| 백엔드 | **Laravel 11** 또는 **Supabase** | PHP 선호 시 Laravel, 속도 우선 시 Supabase |
| DB | PostgreSQL | JSON 컬럼 지원 |
| 결제 | 토스페이먼츠 / Stripe | 국내는 토스 |
| 배포 | Vercel(랜딩) + Railway / Cafe24(API) | |

---

## 12. 유튜브 강의 콘텐츠 제작 가능성

### 결론: **가능하며, 소재로서 상당히 우수함**

| 이유 | 설명 |
|---|---|
| 라이선스 | MIT → 코드 공개·설명·개조 전부 합법 (출처 표기 조건) |
| 규모 | 3,580줄 — 강의로 다루기 적당 |
| 시각적 임팩트 | `/` 입력 → 팝업. 데모가 화려해 썸네일·후킹에 유리 |
| 트렌드 키워드 | AI, ChatGPT, 생산성, 크롬 확장 — 전부 검색량 높음 |
| 한국어 공백 | Plasmo 한국어 강의 거의 없음 |
| 난이도 스펙트럼 | 초급(사용법) ~ 고급(ProseMirror 우회) 전부 커버 |

### 트랙 A — 일반 사용자 대상 (조회수)

| # | 제목 | 길이 |
|---|---|---|
| 1 | ChatGPT 10배 빠르게 쓰는 법 (무료 확장) | 8분 |
| 2 | 프롬프트 직접 만들기 — 변수 완전정복 | 12분 |
| 3 | 개발자용 프롬프트 TOP 10 | 15분 |
| 4 | 내 프롬프트 라이브러리 백업·공유하기 | 7분 |

### 트랙 B — 개발자 대상 (전문성 · 강의 전환)

| # | 제목 | 핵심 |
|---|---|---|
| 1 | Plasmo 로 크롬 확장 만들기 — 환경 세팅 | `pnpm create plasmo`, 폴더 구조 |
| 2 | Manifest V3 완전 이해 | background / content / options 3층 구조 |
| 3 | 남의 웹사이트에 내 UI 띄우기 | Shadow DOM, CSS 격리, `:host(plasmo-csui)` |
| 4 | **ChatGPT 입력창에 텍스트 꽂기 — ProseMirror 뚫기** | `execCommand`, 이벤트 디스패치 ★ |
| 5 | chrome.storage + React 상태 동기화 | `@plasmohq/storage`, watch |
| 6 | Command Palette UI 직접 만들기 | 키보드 네비게이션, 검색 |
| 7 | CSS Custom Highlight API | 신 API + 폴백 전략 |
| 8 | 웹스토어 심사 통과 + CI 자동 배포 | `submit.yml`, PlasmoHQ/bpp |

> **4편이 킬러 콘텐츠.** "React 가 관리하는 입력창에 값 넣기" 는 개발자들이 실제로 많이 막히는 지점이며,
> 한국어 콘텐츠가 사실상 없다.

### 트랙 C — 사이드프로젝트 · 수익화 대상

| # | 제목 |
|---|---|
| 1 | 오픈소스 포크해서 내 서비스 만들기 (MIT 라이선스 활용법) |
| 2 | 크롬 확장으로 월 100만원 버는 구조 설계 |
| 3 | 확장 + Supabase 로 클라우드 동기화 붙이기 |
| 4 | 크롬 웹스토어 ASO — 검색 노출 올리기 |

### 제작 시 주의사항

| 항목 | 지켜야 할 것 |
|---|---|
| 출처 표기 | 영상·설명란에 "원작: Adem Kouki, github.com/Ademking/SkillPrompts, MIT License" |
| MIT 조건 | 재배포 시 LICENSE 파일 반드시 포함 |
| 오해 방지 | "내가 만들었다" ❌ → "오픈소스를 분석·개조한다" ⭕ |
| 화면 녹화 | 개인정보 · API 키 블러 처리 |
| 상표 | 포크 시 새 이름 사용 |
| 버전 고지 | "v1.1.0 기준" 명시 |

### 제작 팁

- 녹화 OBS Studio / 편집 DaVinci Resolve / 자막 Vrew
- 코드 화면: VSCode, 폰트 18pt 이상
- 8분 영상 구성: 훅(0:15) → 문제제기(0:45) → 본론(6:30) → 정리 → 예고
- SEO 키워드: `크롬 확장 만들기`, `ChatGPT 확장프로그램`, `프롬프트 관리`, `Plasmo 튜토리얼`, `Manifest V3`, `AI 생산성`, `프롬프트 엔지니어링`, `사이드프로젝트`

---

## 13. 수익화 전략 8가지

### 시장 배경

| 요인 | 상황 |
|---|---|
| AI 사용자 | ChatGPT 주간 사용자 수억 명 규모, 국내도 수백만 |
| 한국어 공백 | UI · 프롬프트 100% 영어. 한국어 프롬프트 관리 도구 희소 |
| 미지원 사이트 | Claude, Perplexity, 뤼튼, 딥시크, Grok, Copilot 전부 비어 있음 |
| 라이선스 | MIT → 상업적 이용 합법 |
| 진입장벽 | 서버 없이 출시 가능 → 초기 비용 거의 0 |
| B2B 수요 | 사내 프롬프트 표준화 니즈 증가 |

### 경쟁 환경

| 서비스 | 특징 | 약점 |
|---|---|---|
| AIPRM | ChatGPT 프롬프트 확장, 유료 | UI 무거움, 영어, 광고 |
| PromptBase | 프롬프트 마켓 | 확장 아님 |
| Superpower ChatGPT | 다기능 확장 | 복잡함 |
| 한국어 서비스 | — | **거의 없음 → 기회** |

---

### 모델 1. 한국어 프리미엄 버전 ⭐⭐⭐⭐⭐

**컨셉**: 포크 → 완전 한글화 + 한국 AI 서비스 지원

**MVP (2~3주)**
- UI 전체 한글화 (`options.tsx`, `content.tsx`)
- 한국어 프롬프트 150개 신규 제작
- 뤼튼 · 클로드 지원 추가
- 한국형 온보딩

**추가 (1~2개월)**
- 네이버 Cue:, 카카오 지원
- 한글 초성 검색 (ㄱㅅ → 기술스택)
- 업무 템플릿 (보고서 · 기안서 · 주간보고)

**가격 (Freemium)**

| 플랜 | 가격 | 제공 |
|---|---|---|
| Free | 0원 | 프롬프트 20개, 블록 5개, 템플릿 50개 |
| Pro | 월 4,900원 / 연 49,000원 | 무제한, 클라우드 동기화, 템플릿 500개+, 폴더, 우선 지원 |
| Team | 1인 월 9,900원 (최소 3인) | Pro + 팀 공유, 권한 관리, 사용 통계 |

**시뮬레이션**
```
다운로드 10,000 → 활성 30% (3,000) → 유료 전환 5% (150명)
월 735,000원 / 연 약 880만원
다운로드 50,000 → 유료 750명 → 월 367만원 / 연 4,400만원
```

---

### 모델 2. 직군별 프롬프트 팩 판매 ⭐⭐⭐⭐⭐ (최우선 추천)

**컨셉**: 확장은 무료 배포, **프롬프트 팩(JSON)을 유료 판매**.
Import/Export 기능이 이미 존재하므로 **추가 개발이 사실상 불필요**.

| 팩 | 구성 | 가격 |
|---|---|---|
| 개발자 팩 | 코드리뷰·리팩토링·테스트·PR·디버깅·아키텍처 60개 | 19,900원 |
| 마케터 팩 | 광고카피·SNS·블로그SEO·이메일·퍼널 70개 | 24,900원 |
| 디자이너 팩 | 컨셉기획·피드백·포트폴리오·UX라이팅 50개 | 19,900원 |
| 기획자/PM 팩 | PRD·요구사항·유저스토리·회의록·KPI 60개 | 24,900원 |
| 직장인 팩 | 보고서·기안서·메일·주간보고·발표 80개 | 19,900원 |
| 학생/연구 팩 | 논문요약·리서치·과제·자소서 50개 | 14,900원 |
| 창업가 팩 | 사업계획서·IR덱·린캔버스·투자자메일 50개 | 29,900원 |
| **올인원 번들** | 전체 7팩 (420개) | **69,900원** |

**시뮬레이션**
```
보수적: 월 50개 × 평균 2만원 = 월 100만원 / 연 1,200만원
낙관적: 월 200개              = 월 400만원 / 연 4,800만원
원가 0원 → 마진율 약 95% (플랫폼 수수료 5~10% 차감)
```

**판매 채널**: 크몽 · 탈잉 · 클래스101 · 네이버 스마트스토어 · 자체 랜딩(토스페이먼츠) / Gumroad · Lemon Squeezy · PromptBase

**최우선 추천 이유**: 개발 불필요 · 원가 0 · 1주 내 출시 · 실패해도 손실 없음 · **수요 검증 수단으로 최적**

---

### 모델 3. 프롬프트 마켓플레이스 ⭐⭐⭐⭐

**컨셉**: 누구나 등록·판매하는 플랫폼

```
확장(구매 프롬프트 사용) ↔ 웹 마켓플레이스(Next.js/Laravel) ↔ PostgreSQL + S3
                              등록·검색·리뷰·결제·정산·미리보기
```

**수익**: 판매 수수료 20~30% + 상단 노출 광고(주 5만원) + 인증 배지(월 9,900원) + 구매자 멤버십(월 9,900원)

**시뮬레이션**: GMV 월 1,000만원 × 25% = 월 250만원 / GMV 5,000만원 → 월 1,250만원

**난관**: 닭과 달걀 문제(초기엔 자체 제작 팩으로 채우기), 품질 관리(심사제+리뷰), 복제(일부 미리보기), 법적 이슈(사업자등록·통신판매업)

---

### 모델 4. B2B 팀/기업용 SaaS ⭐⭐⭐⭐⭐ (수익성 최고)

**해결할 기업 Pain Point**
```
"직원마다 ChatGPT 쓰는 방식이 다르다"
"잘 만든 프롬프트가 공유되지 않는다"
"기밀 정보가 프롬프트에 들어갈까 걱정된다"
"누가 얼마나 쓰는지 모른다"
```

**기능**
- 조직: SSO(Google Workspace / Okta / SAML), 부서별 라이브러리, 역할 권한
- 거버넌스: 승인 워크플로우(초안→검토→승인→배포), 버전 관리·롤백, 필수 프롬프트 강제 배포
- 보안: 민감정보 패턴 감지(주민번호·카드번호 경고), 금지어 필터, 감사 로그, 온프레미스 옵션
- 분석: 프롬프트별 사용 빈도, 팀별 활용도, ROI 리포트

**가격**

| 플랜 | 가격 |
|---|---|
| Starter | 1인 월 9,900원 (최소 5석) |
| Business | 1인 월 19,900원 (최소 20석) |
| Enterprise | 별도 견적 (연 1,000만원~) |

**시뮬레이션**
```
50인 기업 10곳 = 50 × 19,900 × 10 = 월 995만원 / 연 약 1.2억원
+ Enterprise 3곳 (연 2,000만원) → 연 1.8억원
```

**난관**: 영업 사이클 3~6개월, 보안 심사(ISMS·개인정보보호), 커스터마이징 요구, 초기 레퍼런스 확보
**진입 전략**: 지인 회사 1곳 무료 도입 → 성공 사례 확보 → 영업

---

### 모델 5. 클라우드 동기화 Pro ⭐⭐⭐⭐ (가성비)

현재 가장 큰 불편인 **기기 간 동기화 부재**를 해결.

```ts
import { createClient } from '@supabase/supabase-js'
const supabase = createClient(URL, ANON_KEY)

await supabase.auth.signInWithOAuth({ provider: 'google' })
await supabase.from('prompts').upsert(prompts.map(p => ({ ...p, user_id: user.id })))

supabase.channel('prompts')
  .on('postgres_changes', { event: '*', schema: 'public', table: 'prompts' },
      payload => syncToLocal(payload))
  .subscribe()
```

| 플랜 | 가격 | 제공 |
|---|---|---|
| Free | 0원 | 로컬 저장 (현재 기능) |
| Pro | 월 3,900원 / 연 39,000원 | 무제한 기기 동기화, 30일 백업 히스토리, 버전 복원, 웹 관리 |

**시뮬레이션**: 사용자 20,000 × 전환 4% = 800명 × 3,900원 = 월 312만원 / 연 3,700만원
운영비 Supabase Pro $25/월 + 도메인 → 마진 95%+

---

### 모델 6. 콘텐츠 & 교육 수익화 ⭐⭐⭐⭐

```
유튜브(무료) → 뉴스레터(리드) → 강의·전자책(수익)
```

| 상품 | 가격 | 채널 | 예상 |
|---|---|---|---|
| 전자책 "AI 프롬프트 엔지니어링 실전" | 19,900원 | 크몽·브런치·리디 | 월 50권 = 100만원 |
| 강의 "Plasmo 로 크롬 확장 만들기(8h)" | 99,000원 | 인프런·패스트캠퍼스 | 월 20명 = 198만원 |
| 강의 "AI 생산성 200%" | 49,000원 | 클래스101·탈잉 | 월 40명 = 196만원 |
| 유료 뉴스레터 (주 1회) | 월 5,000원 | 메일리·스티비 | 300명 = 150만원 |
| 기업 출강 | 회당 150만원 | 직접 영업 | 월 2회 = 300만원 |
| 유튜브 광고 | — | YouTube | 월 50~200만원 |

**종합**: 보수적 월 300~500만원 / 적극적 월 800~1,000만원
**주의**: 성과까지 6개월~1년, 주 1회 이상 꾸준한 업로드 필요

---

### 모델 7. 화이트라벨 / 커스텀 개발 ⭐⭐⭐

| 서비스 | 가격 | 기간 |
|---|---|---|
| 기본 브랜딩 변경 | 300만원 | 2주 |
| + 사내 AI 도구 연동 | 800만원 | 4주 |
| + 백엔드 구축(동기화·권한) | 1,500만원 | 8주 |
| 풀 커스텀 + 온프레미스 | 3,000만원~ | 12주 |
| 유지보수 | 월 50~150만원 | 연 계약 |

**시뮬레이션**: 연 3건(평균 1,000만원) = 3,000만원 + 유지보수 3건 × 월 100만원 = 연 6,600만원
**단점**: 영업 난이도, 요구사항 변경 리스크, 확장성 없음(시간 = 매출)

---

### 모델 8. 제휴 마케팅 & 스폰서 ⭐⭐⭐

- AI 서비스 제휴 링크 (ChatGPT Plus, Claude Pro, 국내 AI 서비스)
- 옵션 페이지 하단 배너 / "이번 주 추천 AI 툴" 섹션 → 월 50~200만원
- AppSumo 라이프타임 딜 → 초기 현금 확보용 (단, 플랫폼 수수료 높음)

---

### 종합 비교

| # | 모델 | 난이도 | 초기비용 | 수익성 | 소요기간 | 추천 |
|---|---|---|---|---|---|---|
| 2 | 프롬프트 팩 판매 | ⭐ | 0원 | 💰💰💰 | 1주 | 🥇 최우선 |
| 1 | 한국어 프리미엄 | ⭐⭐ | 50만원 | 💰💰💰💰 | 2개월 | 🥈 |
| 5 | 클라우드 Pro | ⭐⭐⭐ | 100만원 | 💰💰💰💰 | 1개월 | 🥉 |
| 6 | 콘텐츠·교육 | ⭐⭐ | 100만원 | 💰💰💰💰 | 6개월 | ⭐⭐⭐⭐ |
| 4 | B2B SaaS | ⭐⭐⭐⭐⭐ | 500만원 | 💰💰💰💰💰 | 6개월 | ⭐⭐⭐⭐⭐ |
| 3 | 마켓플레이스 | ⭐⭐⭐⭐ | 1,000만원 | 💰💰💰💰💰 | 6개월 | ⭐⭐⭐ |
| 7 | 화이트라벨 | ⭐⭐⭐ | 0원 | 💰💰💰 | 수주 시 | ⭐⭐⭐ |
| 8 | 제휴·스폰서 | ⭐ | 0원 | 💰💰 | 즉시 | ⭐⭐⭐ |

---

## 14. 실행 로드맵

### Phase 1 (0~1개월) — 검증 · 리스크 0
```
□ 프롬프트 팩 3종 제작 (개발자 / 직장인 / 마케터)
□ Gumroad + 크몽 등록
□ 커뮤니티 홍보 (OKKY, 디스콰이엇, 긱뉴스, Reddit)
🎯 목표: 월 30만원 + 어떤 팩이 팔리는지 데이터 확보
💸 투자: 0원
```

### Phase 2 (1~3개월) — 제품화
```
□ 포크 → 한글화 → 리브랜딩 (새 이름 · 로고)
□ 클로드 / 뤼튼 / 퍼플렉시티 지원 추가
□ 크롬 웹스토어 출시 (무료)
□ 유튜브 채널 개설 (주 1회)
🎯 목표: 다운로드 3,000, 팩 매출 월 100만원
💸 투자: 약 50만원 (웹스토어 등록비 $5 + 디자인)
```

### Phase 3 (3~6개월) — 수익화
```
□ Supabase 연동 → Pro 플랜 출시 (월 3,900원)
□ 토스페이먼츠 연동
□ 전자책 출간
🎯 목표: MRR 300만원
💸 투자: 약 100만원
```

### Phase 4 (6~12개월) — 확장
```
□ B2B Team 플랜 출시
□ 지인 회사 1곳 무료 도입 → 레퍼런스 확보
□ 인프런 강의 출시
□ 마켓플레이스 베타
🎯 목표: MRR 1,000만원
```

### 권장 조합
> 무료 한국어 확장으로 사용자 확보 → 프롬프트 팩으로 1차 수익 →
> 클라우드 Pro 로 구독 수익(MRR) → B2B 로 스케일업

---

## 15. 법적 체크리스트

| 항목 | 내용 |
|---|---|
| ✅ MIT 라이선스 | 상업적 이용 가능. **LICENSE 파일과 저작권 고지 반드시 포함** |
| ✅ 원작자 크레딧 | "Based on SkillPrompts by Adem Kouki (MIT)" 표기 권장 |
| ⚠️ 이름 · 로고 변경 | 포크 시 반드시 새 이름 사용 (상표 분쟁 방지) |
| ⚠️ 사업자등록 | 유료 판매 시 필수. 간이과세자부터 시작 가능 |
| ⚠️ 통신판매업 신고 | 온라인 판매 시 필수 (관할 구청) |
| ⚠️ 개인정보처리방침 | 클라우드 동기화 도입 시 필수 작성 |
| ⚠️ 스토어 정책 | Chrome Web Store 결제·데이터 정책 준수 |
| ⚠️ AI 서비스 ToS | ChatGPT / Gemini 약관 준수 (자동 대량 요청 금지) |

---

## 16. 개선이 필요한 지점

전수조사 중 확인된 실제 이슈.

| 이슈 | 상세 | 우선순위 |
|---|---|---|
| 🇰🇷 한국어 미지원 | UI 전부 영어, 내장 프롬프트 169개 전부 영어 | 높음 |
| 🔗 동기화 부재 | `storage.local` 사용 → 기기 간 동기화 불가, 수동 JSON 백업만 | 높음 |
| 🧪 테스트 전무 | 테스트 코드 0줄, 테스트 러너 미설정 | 높음 |
| 🌐 지원 사이트 적음 | Claude · Perplexity · 뤼튼 등 미지원 | 높음 |
| 📜 README 뱃지 오류 | 뱃지가 원본(Ademking) 저장소를 가리킴 — 포크 기준 수정 필요 | 중간 |
| 🧱 거대 파일 | `content.tsx` 946줄, `options.tsx` 785줄 → 모듈 분리 필요 | 중간 |
| 📅 CI Node 버전 | `submit.yml` 이 Node 16.x 사용 (EOL) | 중간 |
| 🏷️ 라이선스 파일명 | `LICENCE` (영국식) — README 링크는 `LICENSE` 를 가리켜 깨짐 | 낮음 |
| ♿ 접근성 | 모달 포커스 트랩 · ARIA 속성 보강 여지 | 낮음 |

---

## 17. 참고 링크 모음

### 저장소
- **본 저장소 (bmshin94)**: https://github.com/bmshin94/SkillPrompts
- 원본 저장소 (Ademking): https://github.com/Ademking/SkillPrompts
- Releases: https://github.com/Ademking/SkillPrompts/releases

### 배포
- Chrome Web Store: https://chromewebstore.google.com/detail/skillprompts/lmonnhccbnchmhgpdmfllmallciokckl
- Firefox Add-ons: https://addons.mozilla.org/en-US/firefox/addon/skillprompts/
- 환영 페이지: https://skillprompts.surge.sh/

### 기술 문서
- Plasmo Framework: https://docs.plasmo.com/
- Chrome Extensions MV3: https://developer.chrome.com/docs/extensions/mv3/intro/
- Firefox Extension Workshop: https://extensionworkshop.com/
- CSS Custom Highlight API: https://developer.mozilla.org/en-US/docs/Web/API/CSS_Custom_Highlight_API
- ProseMirror: https://prosemirror.net/
- @plasmohq/storage: https://docs.plasmo.com/framework/storage
- Supabase: https://supabase.com/docs
- Laravel: https://laravel.com/docs

### 원작자
- GitHub: https://github.com/Ademking
- LinkedIn: https://www.linkedin.com/in/ademkouki/

---

## 부록 — 대화 진행 요약

| # | 질문 | 핵심 답변 |
|---|---|---|
| 1 | 전수조사 후 분석 | 50개 파일 전수 확인. ChatGPT/Gemini 입력창에 `/` 슬래시 커맨드를 붙이는 브라우저 확장. Plasmo + React 18 + TS 기반 |
| 2 | 더 쉽게 설명 | "ChatGPT용 초강력 단축어 등록기". 타이핑 40초 → 2초. 변수 = 신청서 빈칸, 블록 = 도장 |
| 3-1 | 설치 및 사용법 | 스토어 설치 / ZIP 수동 / 소스 빌드 3경로. 사용은 입력창에서 `/` |
| 3-2 | 플러그인? 스킬? MCP? | 전부 아님. **브라우저 확장 프로그램**. Claude Skill 과 무관 |
| 3-3 | API 토큰 필요? | 불필요. 가입·결제·키 전부 0. 100% 로컬 |
| 3-4 | AI 에이전트에 도움? | 직접적 ❌ / 간접적 ⭕ (프롬프트 패턴, 템플릿 엔진, 브라우저 자동화 기술) |
| 3-5 | 수익화 아이디어? | 있음 (4번에서 상세) |
| 3-6 | React / PHP 가능? | React 는 이미 사용 중. PHP 는 백엔드(Laravel)로 가능 |
| 3-7 | 유튜브 강의 가능? | 가능. 특히 "ProseMirror 입력창 뚫기" 편이 킬러 콘텐츠 |
| 4 | 수익화 상세 | 8개 모델 제시. 1순위 = 프롬프트 팩 판매(리스크 0, 1주 출시) |
| 5 | 정리 파일 생성 | 본 문서 |

---

_이 문서는 저장소 전체 파일 50개를 직접 읽어 작성한 분석 결과입니다._
_분석 시점: 2026-10-01 / 대상 커밋: `3baeb7e`_
