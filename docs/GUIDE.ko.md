# a11y-audit 설치 및 사용 설명서

Claude Code에서 화면이나 컴포넌트를 WCAG 2.2 기준으로 점검하는 스킬입니다.
체크리스트로 검토하고, 대비(contrast)는 실제 브라우저로 렌더링해서 잽니다.

## 1. 준비물

| 항목 | 버전 | 확인 명령 |
|---|---|---|
| Claude Code | 최신 | `claude --version` |
| Node.js | 18 이상 | `node --version` |
| Python | 3.8 이상 | `python --version` |
| Chrome 또는 Chromium | 아무 버전 | 설치되어 있으면 그대로 씀 |

Python은 색 두 개의 대비만 계산하는 `contrast.py`에만 필요합니다.

## 2. 설치

### 2-1. 스킬 폴더 복사

한 프로젝트에서만 쓰려면 그 프로젝트의 `.claude/skills/`에, 모든 프로젝트에서
쓰려면 `~/.claude/skills/`에 `a11y-audit` 폴더를 넣습니다.

macOS / Linux / Git Bash:

```bash
git clone https://github.com/civilian7/a11y-audit-skill.git
mkdir -p <프로젝트>/.claude/skills
cp -r a11y-audit-skill/a11y-audit <프로젝트>/.claude/skills/
```

Windows PowerShell:

```powershell
git clone https://github.com/civilian7/a11y-audit-skill.git
New-Item -ItemType Directory -Force <프로젝트>\.claude\skills | Out-Null
Copy-Item -Recurse a11y-audit-skill\a11y-audit <프로젝트>\.claude\skills\
```

### 2-2. Playwright 설치 (한 번만)

```bash
npm install --prefix <프로젝트>/.claude/skills/a11y-audit/kit
```

`kit/node_modules`에만 설치되므로 프로젝트의 `package.json`은 바뀌지 않습니다.
`kit/.gitignore`가 `node_modules/`를 제외하므로 커밋에도 들어가지 않습니다.

측정 스크립트는 먼저 시스템에 설치된 Chrome을 쓰고, 없으면 Playwright의
Chromium을 찾습니다. Chrome이 없다면 다음을 한 번 실행합니다.

```bash
cd <프로젝트>/.claude/skills/a11y-audit/kit
npx playwright install chromium
```

### 2-3. 설치 확인

일부러 대비가 낮은 페이지로 측정해 봅니다. 실패가 나와야 정상입니다.

```bash
cd <프로젝트>/.claude/skills/a11y-audit/kit
printf '<!doctype html><body style="background:#fff"><h1 style="color:#eee">test</h1></body>' > probe.html
DS_REQUIRE_BROWSER=1 node scripts/measure_render.mjs probe.html
rm probe.html
```

PowerShell에서는 `$env:DS_REQUIRE_BROWSER = '1'`을 먼저 실행한 뒤
`node scripts/measure_render.mjs probe.html`을 실행합니다.

기대 결과:

```
FAIL .../probe.html — 1 text pair(s) below WCAG AA:
  x <h1> "test" 1.16:1 (need 3)  [rgb(238,238,238) on rgb(255,255,255)]
```

`playwright not installed — REQUIRED, FAILING`이 나오면 2-2를 다시 확인합니다.

Claude Code가 이미 열려 있었다면 새 스킬을 읽도록 세션을 다시 시작합니다.

## 3. Claude Code에서 쓰기

자연어로 요청하면 스킬이 자동으로 실행됩니다.

```
설정 화면 접근성 점검해줘
src/components/Button.tsx 가 WCAG AA를 만족하는지 봐줘
다크모드 대비 확인해줘
```

직접 부르려면 `/a11y-audit`를 입력합니다.

스킬은 다음 순서로 점검합니다.

1. WCAG 체크리스트와 ARIA 패턴 문서를 읽습니다.
2. 컴포넌트마다 필수 항목(P0)을 봅니다: 키보드 조작, 포커스 표시(3:1 이상),
   스크린리더 이름·역할·상태, 대비(본문 4.5:1, UI 3:1), 터치 영역 24×24 이상,
   색에만 의존하지 않는 신호.
3. WCAG 2.2 추가 항목을 봅니다: 2.4.11 포커스 가림, 2.5.8 대상 크기,
   3.3.8 접근 가능한 인증.
4. 렌더링된 HTML이 있으면 대비를 실제로 측정합니다. 재지 않은 수치는 말하지
   않습니다.
5. 움직임 줄이기(`prefers-reduced-motion`) 처리를 확인합니다.

결과는 표로 나옵니다.

| WCAG 기준 | 심각도 | 무엇이 실패했나 | 고치는 방법 |
|---|---|---|---|
| 1.4.3 | P0 | 보조 버튼 글자 2.8:1 | `--text-muted`를 #6b6b6b 이상으로 |

## 4. 측정 스크립트 직접 실행

모든 명령은 `kit` 폴더 기준입니다. 브라우저가 없을 때 조용히 통과하지 않도록
항상 `DS_REQUIRE_BROWSER=1`을 붙입니다.

### 화면의 모든 글자 대비

```bash
DS_REQUIRE_BROWSER=1 node scripts/measure_render.mjs page.html
DS_REQUIRE_BROWSER=1 node scripts/measure_render.mjs --dark page.html
DS_REQUIRE_BROWSER=1 node scripts/measure_render.mjs pages/      # 폴더 안 *.html 전부
```

### 버튼·링크의 기본/호버/포커스 상태 대비

```bash
DS_REQUIRE_BROWSER=1 node scripts/verify_states.mjs page.html
DS_REQUIRE_BROWSER=1 node scripts/verify_states.mjs --dark page.html
```

이 스크립트는 파일 하나만 받습니다.

### 색 두 개의 대비

```bash
python scripts/contrast.py "#1d1d1f" "#ffffff"
python scripts/contrast.py 1d1d1f f5f5f7 --large
```

### 종료 코드

| 코드 | 뜻 |
|---|---|
| 0 | 기준 통과 |
| 1 | 기준 미달이 있음 (또는 `DS_REQUIRE_BROWSER=1`인데 브라우저가 없음) |
| 2 | 대상 파일이 없거나 인자가 없음: 측정 자체를 못 함 |

## 5. 꼭 알아야 할 제한

### 5-1. Vite·React 같은 앱 빌드는 그대로 넣으면 안 됩니다

스크립트는 페이지를 `file://`로 엽니다. 브라우저는 `file://`에서
`<script type="module">`을 실행하지 않습니다. Vite, Next 같은 도구가 만든
`index.html`을 넣으면 앱이 그려지지 않은 빈 껍데기만 측정되고, **결과는 OK로
나옵니다.** 통과가 아니라 측정을 안 한 것입니다.

대신 다음 중 하나를 측정합니다.

- **정적 하네스 페이지:** 점검할 컴포넌트의 마크업과 CSS를 담은 HTML 파일을
  만듭니다. CSS는 상대경로 `<link rel="stylesheet" href="./app.css">`로 연결하고
  모듈 스크립트는 쓰지 않습니다. 상태별(기본, 비활성, 오류)로 나란히 놓으면 한
  번에 볼 수 있습니다.
- **렌더링 결과 저장:** 앱을 실행한 브라우저에서 개발자 도구를 열고
  `<html>` 요소를 "Copy outerHTML"로 복사해 파일로 저장합니다. 스타일시트가
  절대경로(`/assets/...`)로 연결되어 있으면 저장한 파일 옆에 CSS를 두고 경로를
  상대경로로 고칩니다. `<script>` 태그는 지워도 됩니다.

측정 결과의 "all N text element(s)"에서 N이 화면에 보이는 글자 수와 비슷한지
확인하면 빈 페이지를 잰 것인지 알 수 있습니다.

### 5-2. `--dark`는 `data-theme="dark"`만 켭니다

`--dark`는 `<html>`에 `data-theme="dark"` 속성을 붙일 뿐,
`prefers-color-scheme: dark`를 흉내 내지는 않습니다. 다크모드를 미디어
쿼리로만 구현했다면 `--dark` 결과는 라이트 모드와 같습니다. 이 경우 하네스
페이지에서 다크 스타일을 `[data-theme="dark"]` 선택자에도 걸어 두고 측정합니다.

### 5-3. 자동 측정은 대비만 봅니다

키보드 조작, 스크린리더 읽기, 포커스 순서, 화면 400% 확대는 스크립트로 재지
않습니다. Claude가 코드와 체크리스트로 검토하고, 실제 기기에서 사람이 확인하는
단계가 따로 필요합니다.

## 6. 문제 해결

| 증상 | 원인과 해결 |
|---|---|
| `playwright not installed — REQUIRED, FAILING` | `kit`에서 `npm install`을 하지 않았음. 2-2 실행 |
| `REQUIRED, FAILING: no browser available` | Chrome도 Playwright Chromium도 없음. `kit`에서 `npx playwright install chromium` |
| `ERROR: path(s) not found` (종료 코드 2) | 파일 경로 오타. 경로는 `kit` 폴더 기준 상대경로이거나 절대경로 |
| 앱 화면인데 글자 수가 1~2개로 나옴 | 모듈 스크립트가 실행되지 않음. 5-1 참고 |
| `--dark` 결과가 라이트와 같음 | 다크모드가 미디어 쿼리로만 구현됨. 5-2 참고 |
| Claude가 스킬을 쓰지 않음 | 폴더 위치가 `.claude/skills/a11y-audit/SKILL.md`인지 확인하고 세션 재시작 |
| `python`을 찾을 수 없음 | Python 설치, 또는 macOS·Linux에서는 `python3`로 실행 |

## 7. 업데이트와 삭제

업데이트: 저장소를 다시 받아 `a11y-audit` 폴더를 덮어쓴 뒤 2-2를 다시 실행합니다.

삭제: `.claude/skills/a11y-audit` 폴더를 지우면 끝입니다. 다른 곳에는 아무것도
설치하지 않습니다.

## 8. 출처와 라이선스

[plugin87/ux-ui-agent-skills](https://github.com/plugin87/ux-ui-agent-skills)
v2.10.0의 `a11y-audit` 스킬에서 필요한 파일만 떼어 왔습니다. MIT 라이선스,
Copyright (c) 2026 Thientan Soparat. 자세한 내용은 `LICENSE`를 보세요.
