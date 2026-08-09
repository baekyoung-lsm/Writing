# 핸드오프 문서 — 집필 모드(양장본/일반/원고지/조아라/문피아/네이버시리즈) UI/UX 통합 정리

> **이 문서의 목적**: 이 작업은 사용자가 "다른 대화창에서 백그라운드 생성형 AI에게 실행시키겠다"고 명시한 작업입니다. 이 문서 하나만 읽고도 별도 질문 없이 작업을 시작할 수 있도록, 요구사항 원문의 의도를 그대로 보존하면서 실제 코드 위치·현재 구조 조사 결과·구체적 실행 순서까지 함께 정리했습니다. 실행하는 에이전트는 아래 "0. 작업 원칙"부터 반드시 읽으십시오.

- **대상 파일**: `app/jipseul-note.html` (단일 HTML 파일, 빌드 도구 없음, 순수 vanilla JS/CSS. 약 18,000줄+)
- **저장소/브랜치**: 이 저장소(`baekyoung-lsm/Writing`)의 `claude/project-redesign-completion-uk2bp2` 브랜치에서 최근 대규모 작업들이 이어지고 있음. 작업 시작 전 `git log --oneline -10`으로 최신 커밋을 확인하고, 이 브랜치 최신 상태 위에서 이어서 작업할 것(새 브랜치를 파지 말 것 — 별도 지시가 없는 한).
- **요청 시각 기준 최신 커밋**: `소설 원고 탐색 구조 개선` 커밋 이후. 이 문서가 다루는 범위는 그 이후의 **신규** 요청입니다.

---

## 0. 작업 원칙 (반드시 지킬 것)

1. **기존 기능을 손상시키지 않는다.** 이 코드베이스는 이미 여러 라운드에 걸쳐 원고지 레이아웃 엔진, 대용량 원고 가상 렌더링, 자동 들여쓰기, 부분 서식(이탤릭/글씨크기), 소설 원고 탐색/검색, 시나리오·희곡 편집기 등이 촘촘하게 구현·검증되어 있습니다. 이번 작업은 **UI 레이어의 재정리**이지 데이터 구조나 저장 로직을 갈아엎는 작업이 아닙니다.
2. **`<textarea>`만 실시간 편집 표면으로 쓴다(예외 1건 제외).** 이 앱은 한국어/일본어 IME 조합 안전성 때문에 편집 가능한 텍스트 표면에 `contenteditable`을 쓰지 않고 순수 `<textarea>`만 씁니다. 유일한 예외는 양장본 두쪽 보기 모드의 `#bookEditFlow`(`e.isComposing`을 체크하며 조합 중에는 절대 DOM을 프로그램적으로 건드리지 않는 방식으로 안전하게 구현되어 있음)와, 시나리오·희곡 편집기의 서식 있는 요소 본문(작은 `contenteditable` 조각들, 같은 안전 원칙 적용)뿐입니다. **이번 UI 통합 작업에서 원고지/일반집필창/조아라/문피아/네이버 계열 텍스트 입력 표면을 `contenteditable`로 바꾸지 마십시오.**
3. **작업 후 반드시 Playwright로 실제 브라우저에서 검증한다.** `git diff` 확인 → `node -e "new Function(...)"` 방식으로 두 `<script>` 블록 문법 검사 → Playwright(headless Chromium, 경로는 `/opt/pw-browsers/chromium`, 모듈은 `/opt/node22/lib/node_modules/playwright`)로 각 모드 전환·글쓰기·미리보기·규격 변경을 실제로 클릭/타이핑해서 확인. 스크린샷 비교도 권장(아래 "6. 검증 체크리스트" 참고).
4. **커밋 메시지는 한국어, "왜"를 설명하는 톤으로.** 이 저장소의 기존 커�밋 로그 스타일을 참고할 것(`git log`).
5. **완료 후 아티팩트 재배포.** 이 앱은 `claude.ai/code/artifact/cd68e9d0-bb1f-4dd4-9dfa-225d027b06b5` 로 배포되어 있음(제목 "집썰"). 작업 완료 후 같은 URL로 재배포할 것(Artifact 도구가 있다면 `url` 파라미터에 위 주소를 넣고 `force: true` 없이 먼저 시도 — 충돌 나면 최신본을 읽고 병합 없이 이번 세션의 git 커밋 내용으로 덮어써도 됨, git이 항상 진실의 원천).

---

## 1. 사용자 요구사항 원문 (요약 없이 전달)

> 현재 원고지 관련 UI를 전반적으로 개선하고, 각 집필 방식의 개성을 유지하면서도 하나의 통일된 웹페이지 안에 있는 기능이라는 느낌이 들도록 전체 모드의 UI/UX를 정리해줘.

핵심 원칙(사용자 원문 그대로):

- **① 각 집필 방식은 확실히 달라 보여야 한다.** 양장본을 쓰는 것과 원고지를 쓰는 것이 똑같아 보이면 안 된다. 조아라·문피아·네이버시리즈도 각각의 특성이 느껴져야 한다.
- **② 하지만 전체 서비스는 하나의 웹페이지처럼 보여야 한다.** 모드를 바꿨다고 헤더·메뉴·버튼·레이아웃·인터랙션까지 완전히 다른 사이트처럼 변하면 안 된다.
- 최종 목표: **"서로 다른 6개의 웹사이트를 모아놓았다"가 아니라 "하나의 웹페이지에서 6가지 집필 경험을 선택해서 쓸 수 있다"**는 느낌.

### 1-1. 6가지 집필 방식과 각각의 정체성 (유지해야 할 개성)

| 모드 | 코드상 키 | 유지해야 할 분위기 |
|---|---|---|
| 양장본 | `general` | 실제 책을 읽고 쓰는 듯한 고급스럽고 물성 있는 느낌. 표지·종이·제본을 연상시키는 디자인 |
| 일반 집필창 | `plain` | 가장 익숙하고 편안한 현대적 에디터. 불필요한 장식 없이 집중할 수 있는 환경 |
| 원고지 | `paper` | 실제 원고지를 연상시키는 격자와 작성 경험. 글자수·칸을 직관적으로 확인 |
| 조아라 | `joara` | 웹소설 플랫폼에서 연재하는 듯한 가볍고 친숙한 감각 |
| 문피아 | `munpia` | 장르소설/웹소설 연재 환경. 작품 관리·연재를 의식한 분위기 |
| 네이버시리즈 | `naver` | 정돈되고 현대적인 콘텐츠 플랫폼 느낌. 깔끔하고 콘텐츠 중심적 |

각 테마의 **색상, 배경, 장식, 집필 영역의 형태, 문서 표현 방식**은 개성을 유지하되, 그것 때문에 웹페이지 전체 구조까지 완전히 다른 사이트처럼 바뀌면 안 됨.

### 1-2. "테마"와 "웹페이지 UI"의 분리 — 공통으로 유지해야 하는 요소

다음은 **모든 모드에서 최대한 동일하게** 유지해야 하는 공통 요소(사용자 원문 목록):

- 전체 페이지의 기본 레이아웃
- 헤더/상단 네비게이션
- 모드 전환 UI
- 주요 메뉴의 위치
- 기본적인 버튼 형태
- 아이콘 스타일
- 폰트 체계
- 기본 여백과 간격 체계
- 패널의 기본 구조
- 화면 전환 방식
- 설정 및 공통 기능의 접근 방식

반대로 **집필 영역(글을 실제로 쓰는 공간) 안쪽**은 각 모드의 정체성을 보여주는 핵심 영역이므로 특이하고 개성 있어도 됨.

### 1-3. 모드 전환 시 연속성

양장본 → 일반 집필창 → 원고지 → 조아라 → 문피아 → 네이버시리즈로 연속해서 바꿔도, 다음은 예측 가능하게 유지되어야 함:

- 헤더의 위치
- 모드 전환 위치
- 공통 도구의 위치
- 페이지의 기본 폭
- 주요 버튼의 위치
- 인터랙션 방식

사용자가 모드를 바꿀 때마다 UI를 다시 학습해야 하는 상황을 만들지 않는다.

### 1-4. 현재 원고지 UI 문제 — 반드시 점검할 항목

원고지 모드가 다른 모드보다 시각적으로 지나치게 튀는 문제. 격자와 원고지 형태 자체는 유지하되, 다음 항목이 다른 모드와 자연스럽게 연결되는지 점검:

- 상단 영역의 높이
- 모드 전환 영역
- 버튼 크기 / 버튼 스타일
- 아이콘 스타일
- 페이지 여백
- 컨테이너 크기
- 패널 구조
- 배경 처리
- 테두리와 그림자
- 폰트와 글자 크기
- 각 요소 사이의 간격

원고지 특유의 디자인은 **집필 영역 안에서** 강조하고, 주변 UI는 다른 모드와 자연스럽게 연결.

### 1-5. 200자/400자/1000자 조정 기능 위치 변경 (기능 요구사항)

- **쓰기 모드**: 200/400/1000자 조정 기능을 **완전히 제거**. 그 공간은 자연스럽게 재배치(빈 공간·어색한 레이아웃 금지).
- **미리보기 모드**: 200/400/1000자 선택 기능을 **추가**. 사용자가 미리보기 상태에서 규격을 바꾸면 **원문 데이터는 그대로 유지한 채 미리보기만** 해당 규격에 맞게 다시 계산되어야 함(200→400→1000→200으로 계속 바꿔도 원문 불변). 규격이 바뀌면 원고지 칸 구성/줄 수/페이지 구성/페이지 수/문서 배치가 함께 갱신.

### 1-6. 모드별 역할 명확화

- **쓰기 모드** → 실제 글을 작성·수정하는 공간
- **미리보기 모드** → 작성된 글의 결과물을 확인하고 200/400/1000자 규격을 바꾸며 결과를 보는 공간
- **원고지 모드**(원고지라는 형식 자체) → 원고지 형태에 집중해 실제 원고지에서 어떻게 배치되는지 확인하는 공간
- **양장본/조아라/문피아/네이버시리즈** → 각각의 집필/연재 환경을 체험하고 결과물을 확인하는 특화된 방식

각 기능의 목적이 겹치지 않게 한다.

---

## 2. 현재 코드 구조 조사 결과 (이번 세션에서 확인한 사실)

아래는 이번 핸드오프를 준비하며 실제 코드를 읽고 확인한 내용입니다. 처음부터 다시 조사할 필요 없이 바로 활용하십시오.

### 2-1. 포맷 전환의 중심 — `MS_FORMATS`와 `renderManuscriptPage`

- `MS_FORMATS` 객체(약 1893번째 줄 부근, `grep -n "var MS_FORMATS"`로 재확인)가 6개 포맷의 라벨과 일부 스타일 값을 갖고 있음:
  ```js
  var MS_FORMATS = {
    general: { label:"양장본" },
    plain:   { label:"일반 집필창", lineHeight:"1.9", pad:"56px 64px", bg:"#FFFFFF", color:"#1A1A1A", spacing:"0" },
    paper:   { label:"원고지" },
    joara:   { label:"조아라", lineHeight:"1.55", pad:"14px 12px", bg:"#F4F4F1", color:"#222222", spacing:"-0.01em" },
    munpia:  { label:"문피아", lineHeight:"1.9", pad:"18px 16px", bg:"#FFFFFF", color:"#1E1E1E", spacing:"0" },
    naver:   { label:"네이버시리즈", lineHeight:"1.95", pad:"22px 18px", bg:"#FFFFFF", color:"#191919", spacing:"-0.01em" }
  };
  ```
  **중요한 발견**: `joara`/`munpia`/`naver`는 현재 `applyMsStyle(m)` 함수가 `#msText`에 `lineHeight`/`padding`/`background`/`color`/`letterSpacing`/`fontFamily`만 인라인 스타일로 적용하는 수준이고, **같은 `.ms-textarea` 또는 `.paper-text` 마크업을 그대로 재사용**합니다. 즉 지금은 이 세 플랫폼 테마가 사실상 "패딩/색만 다른 같은 에디터"이고, 사용자가 요구하는 "웹소설 연재 감각", "장르소설 연재 환경", "콘텐츠 플랫폼 느낌" 같은 뚜렷한 정체성은 **아직 거의 구현되어 있지 않습니다.** 이 3개 테마의 집필 영역 디자인을 실제로 새로 만들어야 합니다(양장본/원고지는 이미 전용 렌더 함수가 있어 개성이 뚜렷함).
  - `general`(양장본)과 `paper`(원고지)는 `MS_FORMATS`에 스타일 값이 없는 이유: 이 둘은 `applyMsStyle`을 안 쓰고 각각 **전용 렌더 함수**(`renderBookArea`, `renderManuscriptPaperArea`)로 완전히 다른 마크업을 그림.

- `renderManuscriptPage(m)` 함수(약 15168번째 줄, 이 문서 작성 시점 기준 — 커밋이 쌓이면 밀릴 수 있으니 `grep -n "^function renderManuscriptPage"`로 재확인)가 포맷에 따라 분기:
  - `isBook`(`format==="general"`) → `renderBookArea(m)` 호출, `#bookArea`에 그림 (책 형태: `.book-outer`, `.book-shell`, `.book-page-content`, 두쪽 보기 시 `#bookEditFlow`)
  - `isPaper`(`format==="paper"`) → `renderManuscriptPaperArea(m)` 호출, `#paperArea`에 그림. 내부적으로 `m.paperMode`가 `"write"`면 `renderManuscriptPaperWrite`, `"view"`면 `renderManuscriptPaperView` (쓰기/미리보기가 완전히 분리된 별도 화면 — 아래 2-3 참고)
  - 나머지(`isPlain` 포함 `plain`/`joara`/`munpia`/`naver`) → 공통 `else` 분기. `<textarea id="msText">`를 `isPlain`이면 `.paper-text` 클래스로, 아니면 `.ms-textarea` 클래스로 감싸고, `isPlain`이면 `.paper-outer > .paper-sheet-wrap`으로, 아니면 `.ms-frame > .ms-notch + .ms-screen`(휴대폰 프레임처럼 노치가 있는 디자인)으로 한 번 더 감쌈.
  - **문제점**: `joara`/`munpia`/`naver`가 전부 이 `else` 분기를 타면서 `.ms-frame`(노치 있는 폰 프레임 느낌)으로 감싸지는데, 이건 애초에 "양장본과 조금 다른 대안" 정도로 설계된 것이라 세 플랫폼의 실제 UI 관례(예: 조아라/문피아/네이버시리즈 각각의 에디터 화면 특징)와는 무관합니다. 이 부분을 포맷별로 분화시키는 것이 이번 작업의 핵심 중 하나입니다.

- 공통 헤더: `titleBarHtml`(제목 입력·저장 표시·글자수·삭제 버튼)과 새로 추가된 `epSwitcherHtml`(이전/현재/다음 화 전환줄)이 `main.innerHTML` 조립부에서 **모든 포맷 공통으로** 먼저 오고, 그 아래 `.page-pad` 안에 포맷별 `bodyHtml`이 들어감. 즉 **헤더/화 전환/등장인물 칩(`povHtml`)/통계바(`statsHtml`)/포맷 탭(`tabsHtml`)은 이미 구조적으로 공통**이고, 정말 갈라지는 지점은 `bodyHtml`(포맷 탭 아래, 실제 집필 영역)입니다. 이 경계선을 유지하면서 작업하면 "공통 UI + 개성 있는 집필 영역"이라는 목표에 자연스럽게 부합합니다.

- 포맷 전환 탭 자체: `tabsHtml`은 `.ms-tabs` 안에 `.ms-tab` 버튼들(각 포맷 라벨), `data-msformat` 클릭 시 `m.format` 변경 후 `renderManuscriptPage(m)` 재호출. 이 탭의 위치·스타일은 이미 모든 포맷에서 동일 — 유지할 것.

### 2-2. `isPaper`일 때만 헤더 높이가 달라지는 문제 (요구사항 1-4의 "상단 영역 높이"와 정확히 일치)

```css
.graph-title-bar { display:flex; align-items:center; gap:10px; padding:16px 22px; border-bottom:1px solid var(--border); flex-wrap:wrap; }
.paper-compact-bar { padding:10px 22px; gap:8px; }
```

원고지 포맷일 때만 `titleBarHtml`에 `paper-compact-bar` 클래스가 추가로 붙어 세로 패딩이 `16px`→`10px`로 줄어듭니다(화수 배지를 넣기 위한 이전 작업의 결과). 이게 사용자가 지적한 "상단 영역 높이가 다른 모드와 다르다"의 실제 원인 중 하나입니다. 원고지만 헤더가 얕아지는 대신, **화수 배지·표시방식 설정 버튼을 포함해도 다른 포맷과 같은 높이를 유지하는 방식**으로 다시 설계할 것을 권장합니다(예: 배지를 제목 입력 왼쪽에 작게 붙이되 세로 패딩 자체는 건드리지 않기).

### 2-3. 200자/400자/1000자 토글의 현재 위치 (요구사항 1-5의 이동 대상)

**현재 위치 — 쓰기 모드 툴바** (`renderManuscriptPaperWrite` 내부, `refreshToolbarExtras` 함수, 약 14691번째 줄):

```js
var sizeToggleHtml = st.rules.grid ? ('<span class="paper-size-toggle">' + [200,400,1000].map(function(sz){
    return '<button type="button" class="'+(m.paperSize===sz?"on":"")+'" data-papersize="'+sz+'">'+sz+'자</button>';
  }).join("") + '</span>') : '';
...
extras.innerHTML = langBadgeHtml + sizeToggleHtml + verticalToggleHtml + modeToggleHtml;
extras.querySelectorAll("[data-papersize]").forEach(function (btn) {
  btn.addEventListener("click", function () {
    m.paperSize = parseInt(btn.dataset.papersize, 10); saveData(); refreshStatsBar();
  });
});
```

이 블록(`sizeToggleHtml` 선언 + `extras.innerHTML`에 포함시키는 부분 + `querySelectorAll("[data-papersize]")` 리스너)을 **제거**하면 됩니다. `extras`는 `flex` 컨테이너(`#paperToolbarExtras`, `display:flex; gap:8px`)라 항목 하나를 빼도 나머지(언어 배지·세로쓰기 버튼·쓰기/미리보기 전환)가 자동으로 재배치되므로 별도 여백 처리가 거의 필요 없습니다. 다만 제거 후 남은 요소들의 정렬·간격이 어색하지 않은지 육안 확인은 필요합니다.

**이동할 위치 — 미리보기 화면 툴바** (`renderManuscriptPaperView` 함수, 약 14901번째 줄, `.paper-view-toolbar`):

```js
'<div class="paper-view-toolbar">' +
  '<div class="paper-mode-toggle">...</div>' +
  '<div class="paper-view-spread-toggle">...</div>' +
  '<button type="button" id="paperViewLineColorBtn">...</button>' +
'</div>'
```

여기에 동일한 `[200,400,1000].map(...)` 패턴으로 버튼을 추가하고, 클릭 시:
```js
m.paperSize = parseInt(btn.dataset.papersize, 10);
saveData();
renderManuscriptPaperView(m); // 레이아웃(getManuscriptLayoutCached)이 m.paperSize를 키로 다시 계산되므로 재호출만 하면 됨
```
이렇게만 하면 됨. **중요**: `getManuscriptLayoutCached(m, size)`는 `(id, text, paperSize)`를 키로 메모이즈되어 있어서(이전 라운드에서 이미 이렇게 설계됨) `m.paperSize`만 바뀌고 `m.text`는 그대로이므로 **원문 데이터는 자동으로 보존**됩니다. "200→400→1000→200으로 바꿔도 원문 불변"이라는 요구사항은 이 캐시 키 설계 덕분에 이미 충족되어 있으니, 이동 작업 자체는 순수 UI 이동이지 데이터 로직을 새로 짤 필요가 없습니다.

새 버튼의 활성 상태(`.on`) 표시는 현재 쓰기 모드 토글과 동일하게 `m.paperSize===sz` 비교로 처리(`.paper-size-toggle` CSS 클래스를 그대로 재사용 가능: `display:flex; gap:2px; background:var(--bg); border-radius:6px; padding:2px;` 이미 정의되어 있음, 970번째 줄 부근).

### 2-4. 원고지 실물 미리보기(격자) 관련 클래스

- `.paper-view-cell`(격자 한 칸), `.paper-view-grid`, `.paper-view-sheet`(낱장 한 장), `.paper-view-row` — `renderPaperViewSheetHtml`이 그림. `--paper-cell`(칸 크기, JS가 뷰포트에 맞춰 동적 계산), `--paper-line`(칸선 색상, 사용자가 색상 선택 가능)을 CSS 변수로 씀.
- 이 격자 자체는 "원고지 형태"라는 요구사항 1-1/1-4의 핵심이므로 **유지·강조 대상**입니다. 손대지 말아야 할 부분은 이 격자를 감싸는 **바깥쪽**(`.paper-view-wrap`, `.paper-view-toolbar`)의 버튼 스타일·여백이 다른 모드의 툴바(`.ms-controls`, `.paper-toolbar`)와 시각적으로 이질적이지 않은지입니다.
- 참고로 `.paper-toolbar`(쓰기 모드 툴바)와 `.ms-controls`(다른 포맷들의 툴바)는 **이미 거의 동일한 CSS**입니다(`display:flex; ...; background:var(--bg-card); border:1px solid var(--border); border-radius:10px; padding:6px 8px;`). 즉 "툴바 한 줄"의 뼈대는 이미 통일되어 있고, 실제로 지적받을 만한 불일치는 (a) 원고지 전용 헤더 높이(2-2), (b) 원고지 격자 뷰 전용 하위 툴바들(`.paper-mode-toggle`, `.paper-view-spread-toggle`)의 버튼 크기/폰트가 `.ms-controls button`과 미묘하게 다른 수치를 쓰는 것, (c) 원고지 격자 자체의 테두리 색(`--paper-line`, 기본값이 빨강 계열)과 그림자가 다른 모드의 카드형 컴포넌트(`--border`, `--shadow-1`)와 톤이 다른 것입니다. (b)는 공통 토큰으로 통일하고, (c)는 "격자 안에서는 원고지다워야 한다"는 원칙에 따라 의도적으로 남겨도 됩니다 — 단, 격자를 감싸는 카드(`.paper-view-sheet`)의 바깥 여백·그림자 정도는 다른 모드 카드들과 비슷한 톤(`--shadow-1`/`--shadow-2`, `--radius-md`)으로 맞추는 것을 권장.

### 2-5. 공통 디자인 토큰 (그대로 활용할 것)

`:root`에 이미 정의된 토큰(라이트/다크/RGB 커스텀 테마 전부 대응):
```css
--bg, --bg-panel, --bg-card, --bg-card-hover
--ink, --ink-soft, --ink-faint
--border, --border-strong
--accent, --accent-soft, --accent-strong
--tide, --tide-soft, --rose, --rose-soft, --moss, --moss-soft
--shadow, --radius, --radius-sm, --radius-md, --radius-lg, --radius-pill
--shadow-1, --shadow-2, --shadow-3
--sp-1 ~ --sp-10 (간격 스케일), --fs-xs ~ --fs-2xl (폰트 크기 스케일)
--serif ("Noto Serif KR" 등), --sans ("Pretendard Variable" 등)
```
새로 만드는 포맷별 집필 영역 스타일도 **이 토큰들을 기반으로** 색상 변주를 주는 것을 권장합니다(라이트/다크/RGB 테마 전환 시 자동으로 대응되도록). 완전히 하드코딩된 색(`#F4F4F1` 같은 것)을 새로 추가한다면 최소한 다크 모드에서 가독성이 깨지지 않는지 확인할 것 — 기존 `MS_FORMATS`의 `bg`/`color` 값들도 실은 라이트 모드 하드코딩이라 다크 모드에서는 `applyMsStyle`이 `m.previewDark` 플래그로 별도 분기 처리하고 있으니 그 패턴을 참고.

### 2-6. 공통 아이콘 시스템

`ic(name, size)` 함수가 SVG 아이콘을 문자열로 반환(예: `ic("docIcon",13)`, `ic("chevL",13)`). 새 포맷별 장식 요소를 추가할 때도 임의의 이모지나 별도 아이콘 세트를 쓰지 말고 기존 `ic()` 세트를 우선 활용하거나, 정말 필요하면 같은 `ic()` 함수의 아이콘 정의부(파일 상단 근처)에 같은 스타일(선 굵기 `stroke-width:1.5`, `stroke-linecap:round` 등)로 추가할 것 — "아이콘 스타일 통일"이라는 요구사항(1-2)에 직접 해당.

---

## 3. 실행 순서 제안

작업량이 크므로 아래 순서로 나눠서 진행하고, 각 단계마다 Playwright로 확인 후 다음 단계로 넘어갈 것을 권장합니다.

### 3단계 A — 공통 골격 감사 및 미세 조정 (위험도 낮음)
1. `.graph-title-bar`/`.paper-compact-bar` 높이 통일 (2-2 참고). 원고지 포맷이어도 다른 포맷과 세로 패딩이 같도록 조정.
2. `.ms-controls`/`.paper-toolbar`/`.paper-view-toolbar`/`.paper-mode-toggle`/`.paper-view-spread-toggle` 버튼들의 `padding`/`font-size`/`border-radius`를 하나의 공통 스케일로 통일(예: 버튼 높이 기준값 하나 정해서 전부 맞추기).
3. `.ms-frame`(노치 프레임)이 정말 필요한 포맷이 어디인지 재검토 — 이건 원래 "양장본의 대안 슬림 버전" 정도로 만들어진 장식이라, 조아라/문피아/네이버 세 플랫폼 각각의 정체성과는 무관합니다. 3단계 B에서 각 플랫폼 전용 프레임으로 교체될 예정이므로, 우선 순위가 낮다면 이 감사 단계에서는 손대지 않아도 됨.

### 3단계 B — 조아라/문피아/네이버시리즈 전용 집필 영역 디자인 (신규 구현, 이 작업의 핵심)
현재 이 3개 포맷은 `MS_FORMATS`의 몇 가지 인라인 스타일 값 외에는 **사실상 차별화된 마크업이 없습니다.** `renderManuscriptPage`의 `else` 분기를 세분화해서 포맷별로 다른 wrapper 마크업 + 전용 CSS 클래스를 만들어야 합니다.
- 조아라: 가볍고 친숙한 "연재 화면" 느낌 — 예: 상단에 작품/화 정보 바(연재 사이트의 챕터 헤더를 연상시키는 얇은 정보줄), 본문 폰트는 플랫폼에서 흔한 고딕 계열, 배경은 옅은 회백색.
- 문피아: 장르소설 연재 관리 느낌 — 예: 좌측 또는 상단에 "회차 관리" 느낌의 정보 배지(이미 있는 `epSwitcherHtml`과 자연스럽게 어울리게), 본문은 신뢰감 있는 세리프/고딕 조합.
- 네이버시리즈: 정돈된 콘텐츠 플랫폼 느낌 — 예: 카드형 문서 프레임, 여백을 넉넉히 주는 미니멀한 배치, 브랜드 그린 계열 포인트 컬러는 `--accent`가 아니라 해당 포맷 전용 accent 변수로 국소적으로만 사용(전역 accent를 바꾸면 다른 모드에도 영향이 가므로 금지).
- **주의**: 이 3개는 실제 서비스 브랜드명이므로, 저작권/상표 관련 텍스트나 실제 로고를 그대로 가져다 쓰지 말 것 — "그 플랫폼에서 연재하는 듯한 감각"을 UI 관례(레이아웃·톤)로 표현하되, 로고나 상표를 복제하지 않는 선에서 구현.
- 구현 방법은 `MS_FORMATS`에 각 포맷별 `wrapperClass` 같은 필드를 추가하고, `renderManuscriptPage`의 `else` 분기에서 `format`별로 다른 outer 마크업을 감싸는 방식을 권장(기존 `isPlain` 삼항 분기를 `switch`나 맵 테이블 방식으로 확장).

### 3단계 C — 200자/400자/1000자 토글 이동 (2-3 참고, 위험도 낮고 명확함)
- 쓰기 모드에서 제거, 미리보기 모드에 추가. 위 2-3에 정확한 코드 위치와 이동 방법이 있음.

### 3단계 D — 전체 모드 순회 회귀 확인
- 6개 포맷을 순서대로 전환하며 헤더 높이·버튼 위치·아이콘이 튀지 않는지, 각 포맷의 정체성이 느껴지는지 스크린샷으로 비교.
- 원고지 쓰기→미리보기 전환, 미리보기에서 200/400/1000 변경, 다시 쓰기로 돌아왔을 때 원문이 그대로인지 확인.
- 기존 회귀 대상(6장 체크리스트) 전부 확인.

---

## 4. 건드리지 말아야 할 것 (명시적 경계)

- `computeManuscriptLayout`, `getManuscriptLayoutCached`, `manuscriptLayoutPosAt` 등 레이아웃 계산 엔진 — 이미 여러 라운드에 걸쳐 검증된 정확한 글자수/금칙처리 엔진입니다. UI만 바뀌어야 하므로 이 함수들의 시그니처나 캐시 키 구조를 바꾸지 마십시오.
- `m.text`/`m.formatRanges`(부분 서식) — 순수 텍스트 + 오프셋 기반 서식 구조. 어떤 포맷으로 보든 같은 데이터를 공유합니다. 포맷별로 텍스트를 별도 필드에 복제하는 방식으로 가면 안 됩니다(원문은 항상 하나).
- 자동 들여쓰기(`wireAutoIndent`), 저장 디바운스(`schedulePaperSave`/`flushPaperSave`), 부분 서식 툴바(`formatToolbarHtml`/`wireFormatToolbarButtons`) — 그대로 재사용. 포맷별 wrapper를 새로 만들어도 이 함수들에 textarea 엘리먼트만 넘겨주면 그대로 동작합니다.
- 소설 원고 탐색/검색, 회차 전환 팝오버(`epSwitcherHtml`, `openEpSwitchPopover`) — 방금 전 라운드에서 추가된 기능. 헤더 바로 아래에 위치가 고정되어 있고 모든 포맷에서 공통으로 보여야 합니다(이 문서의 요구사항과도 부합 — "공통 도구의 위치"를 지키는 대표적인 예).
- 시나리오·희곡 편집기(`renderPlayScriptPage`)와 수필·자서전(`renderEssayEntryPage`) — 이번 요청 범위 밖입니다. 다만 수필·자서전도 내부적으로 `.ms-doc-page`/`.ms-doc-ruler` 등 원고지 쓰기 모드와 같은 클래스를 재사용하고 있으므로, 공통 클래스의 CSS 값을 바꾸면 수필 화면에도 영향이 갑니다 — 수필 화면도 함께 확인할 것.

---

## 5. 참고 — 관련 함수/셀렉터 빠른 목록

| 이름 | 위치(대략) | 역할 |
|---|---|---|
| `MS_FORMATS` | ~1893줄 | 포맷별 라벨/스타일 값 |
| `applyMsStyle(m)` | ~8388줄 | `#msText`에 포맷 스타일 인라인 적용 |
| `renderManuscriptPage(m)` | ~15168줄 | 원고 편집 화면 최상위 렌더 함수, 포맷 분기 지점 |
| `renderBookArea(m)` | ~14477줄 | 양장본 전용 렌더(쓰기/미리보기) |
| `renderManuscriptPaperArea(m)` | ~14643줄 | 원고지 쓰기/미리보기 분기 진입점 |
| `renderManuscriptPaperWrite(m)` | ~14651줄 | 원고지 쓰기 모드(현재 200/400/1000 토글이 있는 곳) |
| `renderManuscriptPaperView(m)` | ~14901줄 | 원고지 미리보기(실물 격자, 200/400/1000 토글을 옮길 곳) |
| `paintPaperSingle` / `paintPaperOverview` | ~14945줄대 | 미리보기 낱장/개요 그리기 |
| `epSwitcherHtml` 조립부 | `renderManuscriptPage` 내부 | 이전/현재/다음 화 전환줄(공통 헤더 영역) |
| `.graph-title-bar` / `.paper-compact-bar` | CSS ~1065, ~1071줄 | 공통 헤더 vs 원고지 전용 헤더 높이 차이 |
| `.ms-controls` / `.paper-toolbar` / `.paper-view-toolbar` | CSS ~880, ~963, ~1002줄 | 각 모드의 툴바 한 줄 |
| `.ms-frame` / `.ms-notch` / `.ms-screen` | CSS ~886줄대 | 현재 plain/joara/munpia/naver가 공유하는 "폰 프레임" 감싸개 |
| `.paper-outer` / `.paper-sheet-wrap` | CSS ~985줄대 | `isPlain`(일반 집필창) 전용 감싸개 |
| `.paper-view-cell` / `.paper-view-grid` / `.paper-view-sheet` | CSS ~973줄대 | 원고지 실물 격자 |

정확한 줄 번호는 이번 문서 작성 이후 다른 커밋이 쌓이면서 밀릴 수 있으므로, 실제 작업 시에는 함수/클래스 이름으로 `grep -n`해서 다시 확인할 것.

---

## 6. 검증 체크리스트 (완료 기준)

1. 양장본/일반집필창/원고지/조아라/문피아/네이버시리즈 6개 포맷 각각 전환했을 때 콘솔 에러 없이 정상 렌더되는가?
2. 6개 포맷을 연속으로 전환할 때 헤더 위치·높이, 포맷 탭 위치, 화 전환줄 위치, 저장 표시·글자수 위치가 흔들리지 않는가?
3. 조아라/문피아/네이버시리즈가 서로 구분되는 시각적 정체성을 갖는가(현재는 색상 몇 개 차이뿐이었음 — 실제로 달라 보이는지)?
4. 원고지 모드의 헤더 높이가 다른 포맷과 같은가?
5. 원고지 쓰기 모드에 200/400/1000자 토글이 더 이상 없는가? 제거 후 툴바 레이아웃이 어색하지 않은가?
6. 원고지 미리보기 모드에 200/400/1000자 토글이 있고, 클릭 시 즉시 격자/줄 수/페이지 수가 바뀌는가?
7. 미리보기에서 200→400→1000→200으로 반복 전환해도 원문(글자 수)이 그대로인가(쓰기 모드로 돌아가서 확인)?
8. 수필·자서전 화면(`.ms-doc-page` 등 공용 클래스를 쓰는 다른 화면)이 이번 CSS 변경으로 깨지지 않았는가?
9. 라이트/다크/RGB 커스텀 테마 각각에서 새로 만든 포맷별 스타일이 가독성 문제 없이 보이는가?
10. 기존 부분 서식(이탤릭/글씨크기) 툴바, 자동 들여쓰기, 찾아바꾸기, 화 전환(이전/현재/다음), 회차 검색이 6개 포맷 전부에서 여전히 정상 동작하는가?
11. 모바일 폭(예: 375px)에서도 공통 헤더/툴바가 깨지지 않는가?

---

## 7. 이번 세션에서 함께 참고하면 좋은 관련 커밋

`git log --oneline`으로 아래 관련 커밋들의 diff를 참고하면 이 앱의 기존 UI 작업 패턴(공통 골격 유지하며 부분 재설계하는 방식)을 파악하는 데 도움이 됩니다:
- `원고지 기능 전면 재설계: 집필=현대적 에디터, 미리보기=별도 화면(실물 원고지)` — 쓰기/미리보기 분리 설계의 원형
- `원고지·일반 집필창 본문 입력 UI를 서로 교체` — CSS 클래스/wrapper 재할당만으로 두 포맷의 시각적 정체성을 맞바꾼 선례(이번 작업과 접근 방식이 유사함 — 참고할 만함)
- `상단 네비게이션 하이브리드 전환`, `2단계 메뉴(상위 카테고리 → 하위 탭) UI 구현` — 공통 골격(헤더/탭 구조)을 유지하며 재편한 선례
- `소설 원고 탐색 구조 개선` — 가장 최근 작업, `epSwitcherHtml` 등 이번 작업이 건드릴 공통 헤더 영역의 최신 상태
