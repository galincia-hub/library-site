# 재현 핸드오프: "AI 에이전트 역공학으로 기존 앱 재구현" (리트머스 영상 기반)

- 작성일: 2026-10-26 (KST)
- 대상: 개발팀 (이 문서만 보고 바로 작업을 시작할 수 있도록 작성)
- 원본 영상: https://www.youtube.com/watch?v=g6yLR8sNTyk (리트머스, 2026-10-23, 22:06)
- 같은 폴더 문서: `summary.md`(영상 요약·팩트체크), `transcript_ts.txt`(타임스탬프 자막)
- 표기: **미확인** = 공개 자료로 확인하지 못한 내용

## 1. 목표 (한 문단)
기존 상용 소프트웨어의 기능을, 원본 소스코드나 보호 자산을 복제하지 않고 **공개 정보로 만든 기능 명세 + 독립 수용 테스트 + AI 코딩 에이전트 반복 빌드**로 재구현하는 절차를 우리 환경(SER5)에서 재현한다. 1차 산출물은 (a) 다른 프로젝트에도 쓸 수 있는 "명세 채굴 → 테스트 → 에이전트 빌드 → QA 점수화" 파이프라인과, (b) 그 파이프라인으로 만든 소규모 파일럿 앱 하나다. 파일럿 대상은 이사님이 정한다(12절 질문 1). 이사님 결정 전 기본값은 "블록 에디터 + 페이지 트리 + 간단한 DB 뷰를 갖춘 노션형 내부 위키"이며, 참고 구현으로 MIT 라이선스 오픈소스 Litpage를 쓴다.

## 2. 배경
- 영상 내용: 외주 개발사 리트머스가 오픈소스 **REA(Reverse Engineer Anything)**를 Claude Code·Codex에 붙여 엑셀·원노트·파워포인트·워드·캡컷·노션을 재구현했다.
  - MS 앱·캡컷: 앱을 띄워 보고 REA로 분석해 바로 만드는 방식이었고, 화면은 비슷했지만 독립 QA 점수는 10점 만점에 2~5점이었다(핵심 상호작용·성능 미구현).
  - 노션: 방식을 바꿨다. 소스코드는 가져오지 않고 **공식 사용설명서, 업데이트 로그, 웹앱을 열 때 브라우저에서 보이는 API 요청·JS 번들**로 기능을 추론해 **기능 명세(테스트 케이스) 약 3,000개**를 만들었다. 이어서 에이전트를 여러 라운드(약 6회) 돌려 카탈로그 정합성을 50%→90%로, QA 점수를 38→96/100으로 올렸다고 한다(점수 원자료는 미공개라 **미확인**).
  - 결과물은 Litpage로 공개됐다: https://github.com/litmers-dev/litpage (MIT).
- 레포 문서에 따른 실제 빌드 방식: 리드 Claude Code 세션(Claude Opus 5.5)이 작업을 영역(셸·내비, 에디터, 페이지·공유, DB, AI, 개발자 플랫폼)으로 나누고, 서브에이전트 44개를 약 21시간 돌렸다. Playwright E2E와 패키지별 단위 테스트를 가드레일로 썼고, 공개 전에 이름·문구·디자인 토큰·아이콘·템플릿을 전면 교체했다. 출처: https://github.com/litmers-dev/litpage/blob/main/docs/how-it-was-built.md
- 소요 시간은 출처마다 다르다: 영상 발언 "8시간", 영상 설명란 "노션 16.5시간(Codex+Claude Code)", 레포 문서 "약 21시간". 계획은 **21시간 이상**으로 잡는다.
- 영상이 말하는 한계: 서버 쪽 동작(권한, 외부 연동, 동시 편집 싱크, 대량 페이지 성능)은 바깥에서 관찰할 수 없어 재현되지 않는다. 영상 속 클론에서도 비공개 페이지가 인증 없이 iframe에 보이는 보안 결함이 나왔다.

## 3. 범위와 비범위
**범위**
1. 명세 채굴 절차: 공개 문서·변경 로그·자기 계정에서 관찰한 UI/네트워크 동작으로 기능 카탈로그(YAML/CSV)를 만든다.
2. 카탈로그를 수용 테스트(Playwright)로 바꾸고, 테스트는 개발 에이전트와 분리된 경로에 둔다.
3. 리드 에이전트 + 서브에이전트 분업 빌드, 라운드별 QA 점수 리포트.
4. 파일럿 앱 1개(기본값: 노션형 내부 위키, 기능 50~150개 규모).
5. (선택) Litpage를 SER5에 셀프호스팅해 참고 기준선이자 내부 도구 후보로 평가.

**비범위**
- 타사 바이너리 디컴파일이나 난독화 해제로 얻은 코드를 그대로 옮기는 일, 타사 아이콘·로고·문구·템플릿·폰트 복제.
- 약관상 역분석이 금지된 서비스를 상대로 한 REA 네이티브 분석(13절 참고).
- 실서비스 배포, 외부 공개, 결제·외부 계정 연동.
- GPU가 필요한 작업(로컬 LLM 추론 등). 모든 LLM은 클라우드 API나 구독형 코딩 에이전트를 쓴다.
- 3,000개 규모의 전체 노션 클론(비용·시간 때문에 1차 범위에서 제외).

## 4. 요구사항
**기능 요구사항**
- R1. 기능 카탈로그 스키마: `id, area, feature, user_story, steps, expected, source_url, priority(P0~P2), status`. 모든 항목은 출처 URL(공식 문서·변경 로그·직접 관찰 메모)을 갖는다.
- R2. 카탈로그의 P0·P1 항목은 각각 최소 1개의 Playwright 테스트로 연결한다(`tests/acceptance/<area>/<id>.spec.ts`).
- R3. 테스트 디렉터리는 개발 에이전트에게 읽기 전용이다. 실패 메시지만 전달하고 테스트 코드 수정은 금지한다(영상의 "독립 QA" 원칙).
- R4. 라운드마다 `qa/round-N.md`에 통과율(%)과 영역별 실패 목록을 자동 생성한다.
- R5. 파일럿 앱은 로컬 단일 프로세스 + SQLite로 동작한다(셀프호스팅, 외부 DB 없음).
- R6. 인증·권한은 테스트로 강제한다. 비공개 리소스를 비로그인으로 요청하면 401/403이어야 한다(영상 속 결함 재발 방지).

**비기능 요구사항**
- N1. SER5(CPU 전용, RAM 약 36GB, 디스크 여유 32GB) 안에서 빌드·테스트가 끝나야 한다. GPU 불필요.
- N2. 비밀값(API 키)은 `.env.local`에만 두고 커밋하지 않는다.
- N3. 결과물의 이름·문구·아이콘·디자인 토큰은 자체 것을 쓴다. 아이콘은 Lucide 같은 허용 라이선스만 사용한다.
- N4. 모든 외부 소스는 라이선스를 `THIRD_PARTY_NOTICES.md`에 기록한다.

## 5. 아키텍처와 기술 스택
### 5.1 파이프라인 (영상 방법론을 일반화한 것)
```
[공개 자료 수집]           [명세]                [테스트]               [빌드]                       [QA]
매뉴얼/헬프센터  ─┐
변경 로그        ─┼─▶ feature catalog ─▶ Playwright 수용 테스트 ─▶ 리드 에이전트(계획·분배) ─▶ 라운드 리포트
자기 계정 관찰    ─┤    (YAML, 출처 포함)     (개발 에이전트 읽기 전용)   └ 서브에이전트(영역별)       통과율·실패 목록
(DevTools 네트워크)─┘                                                  ▲                             │
REA(허용된 대상만)                                                     └───────── 실패 피드백 ◀───────┘
```
- REA의 역할: 우리가 분석 권한을 가진 대상(오픈소스 Electron 앱, 자사 앱, 라이선스상 허용된 대상)의 구조를 파악하는 보조 도구다. 영상에서 성과를 낸 노션 사례도 핵심은 REA가 아니라 공개 정보 기반 명세였다.

### 5.2 파일럿 앱 참고 아키텍처 (Litpage 문서 기준)
- 단일 Node.js 프로세스: JSON RPC(`/api/rpc`), WebSocket 실시간 동기화, REST API/웹훅(`/v1`), MCP 엔드포인트(`/api/mcp`), AI 서비스(OpenAI Responses 호환 또는 Anthropic, 자체 키)
- 저장소: `node:sqlite`(FTS5 trigram 검색) + 업로드 파일 폴더
- 데이터 모델: 모든 것을 record(`table:id`, version, JSON value)로 다룬다. block/collection/collection_view/space 등. 변경은 트랜잭션 연산(set/update/listAfter/listRemove/spliceText)으로 처리하고, 클라이언트는 낙관적 적용 후 약 80ms 배치로 서버에 보내 리베이스한다.
- 프론트: React 19 + Vite + TypeScript, 블록별 `contentEditable`, zustand, Lucide 아이콘
- 테스트: Vitest(단위), Playwright(E2E)
- 출처: https://github.com/litmers-dev/litpage/blob/main/docs/architecture.md

### 5.3 도구 스택
| 구분 | 도구 | 용도 |
|---|---|---|
| 코딩 에이전트 | Claude Code (Claude Opus 5.5 / Sonnet 5.5), 또는 Codex CLI | 리드·서브에이전트 빌드 |
| 역공학 보조 | REA (`rea-agents` npm, MCP) | 허용된 대상의 JS/Electron 정적 분석 |
| 네이티브 분석(선택) | Ghidra 12.1.x + JDK 21 | 허용된 바이너리만. 파일럿에는 불필요 |
| 테스트 | Playwright, Vitest | 수용·단위 테스트 |
| 런타임 | Node.js 26.x, pnpm 10 | 파일럿 앱·Litpage |
| DB | SQLite (`node:sqlite`) | 단일 파일 DB |

## 6. 단계별 작업 (각 단계에 수용 기준 포함)
**T0. 환경 준비**
```bash
node -v            # 26.12.1 이상 권장 (REA·Litpage 공통 조건 충족)
npm install -g pnpm@10.32.1
npx playwright install chromium   # 디스크 약 0.5GB
```
- 수용 기준: `node -v` ≥ 26.12.1, `pnpm -v` = 10.x, `npx playwright --version` 정상 출력, 남은 디스크 25GB 이상.

**T1. 기준선: Litpage 로컬 구동 (참고 구현 확인)**
```bash
git clone https://github.com/litmers-dev/litpage && cd litpage
pnpm install && pnpm dev     # API :3001, Web :5173
pnpm typecheck && pnpm test
```
- 수용 기준: http://127.0.0.1:5173 에서 로그인(개발 모드 코드 표시), 페이지 생성, 슬래시 메뉴, DB 보드 뷰가 동작. `pnpm test` 통과. 메모리 피크를 기록(`/usr/bin/time -v`).
- 참고: Litpage E2E는 기본적으로 macOS Chrome 경로를 쓴다(`e2e/lib.mjs`). Linux에서는 경로 수정이 필요하다.

**T2. REA 설치와 허용 대상 시험 분석**
```bash
npx rea-agents@latest setup          # Claude Code 선택, 변경 사항 검토 후 승인
npx -y rea-agents@latest analyze-javascript-application /abs/path/to/allowed-electron-app --json > rea-sample.json
```
- 분석 대상은 MIT/Apache 오픈소스 Electron 앱 또는 자사 앱으로 한정한다.
- Linux에서는 procps 호환 `ps`가 PATH에 있어야 한다(REA README).
- 수용 기준: `rea-sample.json`에 모듈·import·Electron 경계 정보가 출력되고, Claude Code에서 REA MCP 도구 목록이 보인다.

**T3. 기능 카탈로그 작성 (명세 채굴)**
- 입력: 대상 제품의 공식 헬프센터·사용설명서, 공개 변경 로그, 우리 계정으로 정상 사용하며 남긴 관찰 메모(화면 흐름, DevTools 네트워크 탭의 엔드포인트 형태. 응답 본문의 독점 콘텐츠는 저장하지 않음).
- 리드 에이전트 프롬프트 예시:
```
너는 요구사항 분석가다. 아래 공개 문서 URL 목록과 관찰 메모만 근거로, <대상 영역>의 기능 카탈로그를
YAML로 작성하라. 필드: id, area, feature, user_story, steps, expected, source_url, priority(P0~P2).
규칙: (1) 모든 항목에 source_url 필수, (2) 문서 문장을 복사하지 말고 자체 문장으로 요약,
(3) 서버 내부 동작을 추측한 항목은 priority P2 + note "추정"으로 표시, (4) 인증·권한 항목을 반드시 포함.
```
- 수용 기준: `catalog/*.yaml` 항목 50~150개(파일럿), 모든 항목에 `source_url` 존재(스크립트로 검사), P0 비율 30% 이하, 인증/권한 항목 5개 이상.

**T4. 수용 테스트 생성 (독립 QA)**
- 별도 에이전트 세션(QA 역할)이 카탈로그를 Playwright 테스트로 바꾼다. 개발 에이전트 세션에는 `tests/acceptance/` 쓰기 권한을 주지 않는다(별도 브랜치·CODEOWNERS·pre-commit 훅으로 강제).
- QA 프롬프트 예시:
```
catalog/*.yaml의 P0·P1 항목마다 Playwright 테스트 1개 이상을 tests/acceptance/<area>/<id>.spec.ts로 작성하라.
선택자는 role/text 기반으로 하고 구현 세부(클래스명)에 의존하지 마라. 각 테스트 상단 주석에 카탈로그 id를 적어라.
```
- 수용 기준: P0·P1 항목의 100%가 테스트와 매핑됨(id 대조 스크립트). 빈 앱을 상대로 실행하면 전부 실패한다(테스트가 실제로 검증하는지 확인).

**T5. 빌드 라운드 (리드 + 서브에이전트)**
- 리드 프롬프트 예시:
```
너는 리드 엔지니어다. 스택: Node 26 + Hono + node:sqlite + React 19/Vite. 단일 프로세스, 단일 SQLite 파일.
catalog/를 영역별로 나누어 서브에이전트에 배분하라. 각 서브에이전트는 자기 영역 폴더만 수정하고 notes/<area>.md에
결정 사항을 남긴다. tests/acceptance/는 읽기 전용이며 수정 금지다. 각 라운드 끝에 pnpm test와
npx playwright test를 실행하고, 실패 목록을 다음 라운드 작업으로 넘겨라. 타사 이름·문구·아이콘을 쓰지 마라.
```
- 라운드 상한: 6회(영상과 동일) 또는 통과율 90% 도달 시 종료.
- 수용 기준: 라운드마다 `qa/round-N.md` 생성. 최종 P0 통과율 100%, P0+P1 통과율 90% 이상. 단위 테스트 통과. 같은 라운드 안에서 테스트 파일 변경 0건(git diff로 확인).

**T6. 보안·동시성 보강 (영상에서 실패한 부분)**
- 비공개 페이지 비로그인 접근 차단, iframe/embed 응답의 인증 검사, 두 브라우저 동시 편집 시 텍스트 병합(Litpage `spliceText` 방식 참고).
- 수용 기준: 비로그인 요청 401/403 테스트 통과. 두 컨텍스트 동시 타이핑 E2E에서 양쪽 입력이 보존됨.

**T7. 정리·라이선스 점검**
- 이름·문구·아이콘·색상 토큰이 자체 것인지 점검하고, 대상 제품명이나 고유 문구를 grep으로 확인한다. `THIRD_PARTY_NOTICES.md`를 작성한다.
- 수용 기준: 대상 제품 상표·고유 문구 grep 결과 0건, 모든 의존성 라이선스 기록, 리포트 1장(`qa/final.md`: 기능 수, 통과율, 소요 시간, 토큰/비용).

## 7. 의존성과 버전
| 항목 | 버전/조건 | 링크 |
|---|---|---|
| Node.js | 26.12.1 이상 (REA: 22.x ≥22.23.4, 24.x ≥24.22.1, 26.12.1+ / Litpage: 26 이상) | https://github.com/morluto/rea · https://github.com/litmers-dev/litpage |
| pnpm | 10.32.1 (Litpage README 지정) | https://github.com/litmers-dev/litpage |
| REA (`rea-agents`) | 최신 (npm, 설정 시 `@latest`) | https://www.npmjs.com/package/rea-agents · https://rea.tools/ |
| Ghidra (선택) | 12.1.x + 64비트 JDK 21 | https://reversedwithai.com/learn/getting-started-with-rea · https://github.com/NationalSecurityAgency/ghidra |
| React / Vite | React 19 | Litpage `apps/web/package.json` |
| 서버 | Hono, `node:sqlite`(FTS5), WebSocket | https://github.com/litmers-dev/litpage/blob/main/docs/architecture.md |
| 테스트 | Playwright, Vitest | https://playwright.dev/ · https://vitest.dev/ |
| 코딩 에이전트 | Claude Code + Claude Opus 5.5(`claude-opus-5-5`) 또는 Sonnet 5.5 | https://www.anthropic.com/claude-opus-5-5 |
| (대안) | Codex CLI 0.153.0 이상(GPT-6 Astra 사용 시) | https://help.openai.com/en/articles/20001275-chatgpt-work-and-codex |

정확한 하위 패키지 버전은 Litpage `pnpm-lock.yaml`을 기준으로 한다(여기서는 전부 대조하지 않았으므로 **미확인**).

## 8. 예상 비용과 난이도, SER5 적합성
**비용 (추정)**
- 구독형: Claude Max 5x $100/월 또는 20x $200/월(공식 가격). 영상 팀은 Max 계정 2개로 노션 단계에서 한 계정의 주간 한도를 소진했다고 말했다. 파일럿(기능 50~150개)은 **Max 1계정, 1~2주** 규모로 추정한다(**추정치, 미검증**).
- API 종량제: Opus 5.5는 입력 $4 / 출력 $20 per MTok, 캐시 읽기 $0.20/MTok. 파일럿 규모는 대략 수백~수천 달러 범위로 추정하며 사용 패턴에 따라 크게 달라진다(**미확인**). 영상의 "2만 달러"는 MS 앱 전체 실험 기준 발언이고 역시 **미확인**이다.
- 인프라 비용: SER5 로컬이면 추가 비용이 없다.

**난이도**
- T0~T2: 하(반나절). T3~T4: 중(명세 품질이 결과를 좌우하므로 사람 검수 필요, 2~3일). T5: 중상(에이전트 운용·충돌 관리, 3~7일). T6: 상(동시성·보안). 전체 파일럿은 약 2~3주.

**SER5 적합성 (RAM 약 36GB, GPU 없음, 디스크 여유 32GB)**
| 구성요소 | 예상 자원 | 판정 |
|---|---|---|
| Node 서버 + Vite dev | RAM 1~3GB | 적합 |
| Playwright Chromium 병렬 4개 | RAM 2~4GB, 디스크 약 0.5GB | 적합 |
| node_modules(Litpage + 파일럿) | 디스크 약 1~3GB(**미확인**) | 적합 |
| REA (JS 정적 분석) | 경량 | 적합 |
| Ghidra 분석 (선택) | 기본 2GiB 힙, 2 CPU, 디스크 약 1~2GB | 적합(필요할 때만) |
| LLM | 클라우드(Claude/Codex) | 적합. **로컬 LLM·GPU 경로는 제외(거부)** |
- 디스크 32GB 안에서 여유를 두려면 Ghidra는 꼭 필요할 때만 설치하고, Playwright 리포트·트레이스는 라운드마다 정리한다.

## 9. 리스크
- 법적 리스크(13절): 대상 서비스 약관의 역분석 금지 조항, 상표·UI 문구·아이콘 저작권 문제.
- 품질 리스크: 영상에서 보듯 "화면만 비슷하고 핵심 상호작용이 안 되는" 결과가 나오기 쉽다 → P0 상호작용 테스트로 막는다.
- 보안 리스크: 바깥에서 관찰할 수 없는 서버 권한 로직이 빠진다 → T6 필수.
- 비용 리스크: 에이전트 라운드가 길어지면 한도·비용이 급증한다 → 라운드 상한 6회, 라운드별 비용 기록.
- 테스트 오염: 개발 에이전트가 테스트를 고쳐 통과시키는 일 → 경로 권한 분리, diff 검사.
- 수치 신뢰도: 영상의 점수·시간·비용은 원자료가 공개되지 않아 계획 근거로 쓰기 어렵다.

## 10. 법적·윤리적 주의
- **약관**: 대부분의 상용 SaaS·데스크톱 앱 약관에는 리버스 엔지니어링·디컴파일 금지 조항이 있다. 노션·마이크로소프트·캡컷의 현행 약관 원문은 이번 조사에서 직접 확인하지 못했다(페이지가 스크립트로 렌더링됨, **미확인**). 착수 전에 대상 약관의 해당 조항을 사람이 직접 확인한다.
  - 노션 약관: https://www.notion.com/notion/terms-and-privacy
  - 마이크로소프트 서비스 계약: https://www.microsoft.com/en-us/servicesagreement
- **한국 저작권법 제101조의4(프로그램코드역분석)**: 정당한 권한으로 쓰는 사람이 **호환에 필요한 정보**를 다른 방법으로 쉽게 얻을 수 없을 때, 호환에 필요한 부분에 한해서만 허용된다. 얻은 정보로 **대상과 실질적으로 유사한 프로그램을 개발·판매**하는 데 쓰면 안 된다. → "기능 클론"을 위한 코드 역분석은 이 조항의 보호를 받기 어렵다. 출처: https://www.law.go.kr/LSW/lsSideInfoP.do?docCls=jo&joBrNo=04&joNo=0101&lsiSeq=283335&urlMode=lsScJoRltInfoR · https://www.easylaw.go.kr/CSP/CnpClsMain.laf?ccfNo=3&cciNo=2&cnpClsNo=8&csmSeq=695&popMenu=ov
- **REA 라이선스 안내**: 합법적 연구, 상호운용성, 보존, "소유하거나 분석 권한이 있는 소프트웨어"용이다. 공식 배포처는 morluto/rea, rea.tools, `rea-agents` npm뿐이고 유사 이름 레포는 피한다.
- **자산**: 타사 로고·아이콘·폰트·템플릿·도움말 문구·가격표를 복제하지 않는다. Litpage도 공개 전에 이를 전면 교체했다.
- **권장 원칙**: 공개 문서와 정상 사용 관찰로 "무엇을 하는가"만 명세화하고, "어떻게 구현했는가"(코드)는 가져오지 않는다(클린룸). 대외 공개·상업화 전에는 법률 검토를 받는다.
- Litpage를 기반으로 쓸 경우 MIT 라이선스 고지를 유지한다.

## 11. 수용 체크리스트 (파일럿 완료 정의)
- [ ] 카탈로그 50~150개, 전 항목 출처 URL 존재
- [ ] P0·P1 테스트 매핑 100%, 개발 중 테스트 변경 0건
- [ ] 최종 P0 100%, P0+P1 90% 이상 통과
- [ ] 인증·동시성 테스트 통과
- [ ] 상표·고유 문구 grep 0건, 라이선스 고지 완료
- [ ] `qa/final.md`에 시간·비용 기록

## 12. 이사님께 여쭐 것
1. **무엇을 만들지**: 영상은 여러 대상(엑셀, 원노트, PPT, 워드, 캡컷, 노션)을 다룬다. 다음 중 어느 쪽인가요?
   - (a) 노션형 내부 위키/DB를 직접 만든다(기본안)
   - (b) Litpage를 그대로 셀프호스팅해 내부 도구로 쓴다(개발 최소)
   - (c) 방법론(명세 → 테스트 → 에이전트 빌드)만 가져와 크루즈 사업용 도구(예: 예약·선실 관리 화면)에 적용한다
   - (d) 엑셀·PPT 같은 오피스류 재현(영상상 품질이 낮아 비권장)
2. 용도: 사내 전용인가요, 외부 공개·판매 가능성이 있나요? (법률 검토 필요 여부가 달라짐)
3. 예산 상한: 구독(Claude Max 1~2계정) 또는 API 종량제 중 어느 쪽이며, 월 한도는 얼마인가요?
4. 기간: 파일럿 2~3주가 괜찮은가요?
5. 실행 위치: SER5에서 돌리는 것으로 확정하나요? (박스나 클라우드 VM 대안 여부)
6. AI 기능(문서 요약·번역 등)을 넣는다면 어느 공급자 키를 쓸까요?
7. 결과물 저장소: 새 repo를 만들지, 기존 repo(예: cruise-v3) 아래에 둘지

## 13. 출처
- 원본 영상: https://www.youtube.com/watch?v=g6yLR8sNTyk
- REA 레포: https://github.com/morluto/rea
- REA 웹사이트: https://rea.tools/
- REA npm: https://www.npmjs.com/package/rea-agents
- REA 시작 가이드(서드파티): https://reversedwithai.com/learn/getting-started-with-rea
- Litpage 레포: https://github.com/litmers-dev/litpage
- Litpage 아키텍처: https://github.com/litmers-dev/litpage/blob/main/docs/architecture.md
- Litpage 제작 과정: https://github.com/litmers-dev/litpage/blob/main/docs/how-it-was-built.md
- Litpage 데모: https://litpage-demo-production.up.railway.app
- Ghidra: https://github.com/NationalSecurityAgency/ghidra
- Claude Opus 5.5: https://www.anthropic.com/claude-opus-5-5
- Claude 플랫폼 릴리스 노트: https://platform.claude.com/docs/en/release-notes/overview.md
- Claude Max 요금: https://support.claude.com/en/articles/11049741-what-is-the-max-plan
- GPT-6 Astra / Codex: https://help.openai.com/en/articles/20001275-chatgpt-work-and-codex
- Playwright: https://playwright.dev/ · Vitest: https://vitest.dev/
- 노션 연혁: https://en.wikipedia.org/wiki/Notion_(productivity_software) · https://www.notion.com/blog/100-million-of-you
- 저작권법 제101조의4: https://www.law.go.kr/LSW/lsSideInfoP.do?docCls=jo&joBrNo=04&joNo=0101&lsiSeq=283335&urlMode=lsScJoRltInfoR
- 생활법령(프로그램 역분석): https://www.easylaw.go.kr/CSP/CnpClsMain.laf?ccfNo=3&cciNo=2&cnpClsNo=8&csmSeq=695&popMenu=ov
- 노션 약관: https://www.notion.com/notion/terms-and-privacy · MS 서비스 계약: https://www.microsoft.com/en-us/servicesagreement
