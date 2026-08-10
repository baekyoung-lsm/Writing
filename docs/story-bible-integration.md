# 집필 노트 × 작품 바이블 통합 — 분석·설계·구현 보고서

첨부된 "작품 바이블(STORY BIBLE v1.0)" docx(49개 섹션, 전부 빈 템플릿)를 기존 「집필 노트」 앱(`app/jipseul-note.html`)에 통합한 작업의 최종 기록. 원칙: 기존 기능 삭제 금지·UI 전면 교체 금지·데이터 구조 파괴 금지·중복 기능 생성 금지·AI 의존 신규 기능 금지·오프라인 우선.

---

## 1. 기존 기능 분석 (핵심만)

이미 구현되어 있던 것 중 이번 작업과 직접 관련된 부분:

- **DOCS**(설정 문서): `category` 필드로 이미 `등장인물/세력/장소/세계관/사건/아이템/떡밥/부·사이드/기타` 9종을 다룬다. 카테고리마다 `props`(타입 있는 커스텀 필드: 텍스트/링크/숫자/날짜/능력치바/등급배지), `blocks`(자유 서술), 계층 구조(`parentHouseId`/`parentLocationId`), 관계(`RELATIONSHIPS`+`GRAPHS`)를 이미 지원.
- **필드 템플릿**: `work.fieldTemplates[category]`로 작품마다 카테고리별 기본 속성 세트를 커스터마이즈하는 기능이 이미 있음(`renderFieldTemplatesPage`).
- **원고 자동 연결**: 원고 본문에 등장인물·장소의 이름/별칭이 등장하면 자동으로 그 원고에 연결하고(`detectCharactersInManuscript`/`detectLocationsInManuscript`), 컨텍스트 패널·"등장 화" 백링크 카드로 보여주는 기능이 이미 있었음. 단 세력·사건·떡밥은 빠져 있었음(2번 참고).
- **연표**(`TIMELINES`), **지도**(`MAPS`, 지역·마커에 장소 문서 연결 가능), **원고 시스템**(`NOVELS`/`MANUSCRIPTS`, 화 단위, 원고지·양장본 등 6개 포맷), **내보내기**(ZIP/HWPX) 전부 기존대로 유지·무변경.
- **국가**: 별도 카테고리가 없고 `장소` 카테고리의 `locationType: "country"`로 이미 표현됨(계층 구조도 지원).
- **설정 충돌 검증**: 코드 전체를 검색한 결과 존재하지 않았음(사용자가 "기존 기능"으로 나열했으나 실제로는 미구현 — 지도 라벨 겹침 자동 보정 기능과 혼동된 것으로 추정). 이번에 신규 추가(4장).

## 2. 작품 바이블 49개 섹션 매핑

**A. 연결만으로 충분(코드 변경 없음)** — 아래는 전부 기존 기능을 그대로 쓰면 됨:
작품 개요·목표·독자층 분석 등 1~7(기획 파트, `DOCS category:"기타"` 또는 자유 문서로 이미 작성 가능) · 세계관 개요·역사·지리·언어·문화·경제·종교·기술·마법·규칙(8,9,13~20 → `세계관`/`장소` 카테고리 문서) · 국가(11 → `장소`+`locationType:country`) · 세력(12 → `세력` 카테고리, 이미 계층·정렬·유형 지원) · 연표(10 → `TIMELINES`) · 등장인물·관계도·성장아크·비밀(21~24 → `등장인물` 카테고리 + `RELATIONSHIPS` + `props`) · 메인/서브 플롯·장별/화별 요약(26~29 → `TIMELINES` 또는 원고별 요약은 원고 자체의 요약/메모) · 상징·모티프(34,35 → `기타` 카테고리) · 독자/시장 분석(36,37 → 자유 문서) · 집필 관리 기록 전부(41~46 → 자유 문서, 날짜순 정렬은 기존 문서 정렬 재사용) · 후속작·외전·세계관 확장 아이디어(47~49 → `기타` 카테고리 또는 새 `작품(WORK)`으로 별도 관리, 이미 가능).

**B. 일부 필드 추가로 해결(이번에 구현)**:
- 복선 관리(30) — 기존 `떡밥` 카테고리와 메커니즘이 동일해 새 시스템 대신 `복선` 카테고리를 추가하고 회수 상태(미회수/회수완료/폐기)·회수 예정 화·실제 회수 화 필드를 `떡밥`과 공유(3장).
- 떡밥 관리(31) — 위와 동일 블록 재사용 + 힌트 강도/진짜 답은 기존 "속성 추가" 프리셋에 추가(코드 최소 변경).
- 회수 현황(32) — 새 대시보드 화면 대신 4장의 "설정 검증" 페이지의 "미회수 복선·떡밥" 카드가 동일 정보를 실시간으로 대체 제공.
- 원고 연동: 연결 인물/지역/세력/사건/복선(데이터 연동 관리 항목) — 기존에 인물·장소만 자동 연결되던 것을 세력·사건·떡밥/복선까지 확장(3장).
- 설정 충돌 검증(39) — 구조적으로 결정 가능한 항목(계층 순환 참조, 끊어진 참조, 미회수 복선)만 규칙 기반으로 신규 구현(4장).

**C. 완전 신규가 필요하지만 이번엔 미구현(13,14장에 설계만 기록)**:
- 세계관 버전 시스템(무제한 버전 + 지도/국가/세력/인물상태/사건/연표 스냅샷) — 데이터량이 크고 위험도가 높아 이번 세션에서는 보류.
- AI 생성 자산 카탈로그(원화/SVG버전/의상세트/헤어세트/표정세트/프롬프트/모델/날짜) — 이 앱의 "외견" 시스템은 코드로 그리는 SVG 방식이라 래스터 이미지 자산 개념 자체가 없음. 이번 세션 범위 밖(다른 대화에서 이미 확인했듯 이미지 생성 파이프라인 자체가 미해결 상태).
- 캐릭터 붕괴 자동 검증(40) — 텍스트 의미를 읽어야 하는 판단이라 규칙으로 불가능. AI 없이 할 수 있는 유일한 형태인 "자가 점검 체크리스트"로 대체 구현(4장).

## 3. 데이터 연동 관리 / 자동 연결 로직 (구현 완료)

- `CATS`에 `복선` 카테고리 추가(`떡밥` 옆, 아이콘 `connect`).
- `detectDocRefsInManuscript(m, workId, idsField, excludedField, categories)` 공통 함수 하나로 기존 `detectCharactersInManuscript`/`detectLocationsInManuscript`를 재정의하고, `detectFactionsInManuscript`/`detectEventsInManuscript`/`detectForeshadowsInManuscript`(떡밥+복선 공유)를 추가. 원고 본문에 이름/별칭이 등장하면 자동으로 `m.factionIds`/`m.eventIds`/`m.foreshadowIds`에 연결.
- 원고 편집기 옆 컨텍스트 패널(`updateContextPanel`)에 세력·사건·떡밥/복선 3개 섹션 추가(기존 인물·장소 섹션과 동일한 컴포넌트 재사용, 새 UI 없음).
- `renderCharacterAppearancesCard`(기존 "등장 화" 백링크 카드)를 `APPEARANCE_BACKLINK` 맵으로 일반화해 등장인물뿐 아니라 장소·세력·사건·떡밥·복선 문서에서도 "이 문서가 등장하는 화" 목록이 자동으로 뜨도록 확장.
- ID 체계: 별도의 새 ID 필드(Character ID 등)를 추가하지 않고 기존 `uid()` 기반 문서/원고 `id`를 그대로 재사용 — 바이블이 요구한 "모든 데이터가 서로 연결"은 이미 있는 참조 배열(`characterIds` 등)로 충분히 충족되므로 새 ID 스키마를 만들지 않음(중복 방지 원칙).

## 4. 검증 규칙 설계 (Rule-Based, 신규)

작품 홈 → "설정 노트" 카테고리 → 새 탭 "설정 검증"(기존 탭 확장 방식, 새 메뉴 아님). `renderValidationPage(work)`가 아래를 전부 로컬 데이터만으로, 인터넷·AI 없이 계산:

| 검사 | 방식 |
|---|---|
| 미회수 복선·떡밥 | `category`가 떡밥/복선이고 `!resolved && !abandoned`인 문서 나열 |
| 세력/장소 계층 순환 참조 | `parentHouseId`/`parentLocationId` 체인을 따라가며 자기 자신 재방문 시 감지(`findHierarchyCycles`) |
| 끊어진 관계도 참조 | `RELATIONSHIPS`의 `fromId`/`toId` 중 하나가 삭제된 문서를 가리키는 경우 |
| 원고의 끊어진 연결 참조 | 원고의 `characterIds` 등 5개 참조 배열에 이미 삭제된 문서 id가 남아있는 경우(`findOrphanManuscriptRefs`) |
| 복선·떡밥의 끊어진 화 연결 | `resolvedMsId`/`expectedResolveMsId`가 삭제된 원고를 가리키는 경우 |
| 캐릭터 붕괴 자가 점검 | 자동화 불가 항목이라 바이블 40번 항목의 체크리스트 6개를 그대로 참고용으로 표시(자동 판정 아님) |

**부수 수정(정합성 버그)**: `deleteDoc`이 인물만 정리하고 장소/세력/사건/복선 참조는 안 지우던 것, `deleteManuscript`가 떡밥/복선의 `resolvedMsId`를 안 지우던 것 — 검증 기능이 스스로 만드는 오류를 막기 위해 두 삭제 함수를 확장해 참조 정리를 완전하게 만듦.

## 5. DB(데이터 구조) 변경안 — 전부 additive, 기존 필드 삭제 없음

```
DOCS[]                    (+) category: "복선" 값 추가 허용
  .resolved                (기존, 이제 떡밥+복선 공유)
  .abandoned          (신규, boolean, optional) — 폐기 상태
  .resolvedMsId             (기존)
  .expectedResolveMsId(신규, string|null, optional) — 회수 예정 화

MANUSCRIPTS[]
  .factionIds / .excludedFactionIds       (신규, string[], optional)
  .eventIds / .excludedEventIds           (신규, string[], optional)
  .foreshadowIds / .excludedForeshadowIds (신규, string[], optional)

PROP_PRESETS[]     (+) 수도/정치체제/지도자/힌트 강도/진짜 답 5개 추가(자동완성 목록일 뿐, 스키마 아님)
DEFAULT_FIELD_TEMPLATES["떡밥"]  (+) 신규 추가(힌트 강도, 진짜 답)
```

모든 신규 필드는 optional이며 `undefined`일 때 기존 코드가 하던 대로 falsy/빈 배열로 동작하도록 감지 함수 내부에서 방어 처리(`if (!m[field]) m[field] = []`) — 과거 데이터에 대한 별도 마이그레이션 스크립트가 필요 없음(11장).

## 6. 상태 관리 구조

새 상태는 전부 기존 전역 배열(`DOCS`/`MANUSCRIPTS`)의 필드로만 존재하고, 별도의 새 전역 배열이나 `state.*` 키를 만들지 않았다(검증 페이지 진입만 `state.mode = "validate"`로 기존 라우팅 패턴 재사용). `localStorage` 저장(`saveData()`)도 기존 배열이 그대로 직렬화되므로 백업/복원·ZIP 내보내기 로직 변경 불필요.

## 7. 세계관 버전 시스템 — 설계만(미구현, 향후 확장)

바이블이 요구하는 "세계관 v1/v2/v3, 각 버전이 지도·국가·지역·세력·인물상태·사건·연표를 스냅샷 보관"은 대상 데이터가 6종(DOCS 여러 카테고리 + MAPS + TIMELINES)에 걸쳐 있고 깊은 복제·용량·UI가 만만치 않아 이번 세션에서는 구현하지 않았다. 제안 설계:

- `work.worldVersions: [{ id, label, createdAt, note }]` — 버전 자체는 태그 목록으로 가볍게 시작.
- 각 스냅샷 대상 엔티티에 `worldVersionId`(optional)를 달아 "이 국가는 v2부터 존재" 식으로 부분 태깅하는 **가벼운 버전**을 1단계로, 필요해지면 버튼 한 번으로 해당 시점 DOCS/MAPS/TIMELINES를 깊은 복제해 별도 스냅샷 레코드로 저장하는 **완전 스냅샷**을 2단계로 나눠 구현할 것을 권장(리스크를 나눠서 검증 가능하게).

## 8. UI 변경안

새 메뉴를 만들지 않는다는 원칙에 따라 우선순위 그대로 적용했다.

1. 기존 기능 재사용 — 복선/떡밥 회수 상태, 원고 자동 연결 전부 기존 문서 편집 화면·컨텍스트 패널 안에서 해결.
2. 기존 화면 확장 — 문서 편집 화면(`renderDocPage`)의 "떡밥" 전용 블록을 "떡밥·복선" 공용 블록으로 확장.
3. 기존 탭 확장 — "설정 노트" 카테고리에 "설정 검증" 탭 1개만 추가(바이블이 요구한 39/40/32번 항목을 이 탭 하나로 통합).
4. 신규 메뉴 — 사용하지 않음.

새로 만든 CSS는 0줄 — 전부 기존 `.unit-cat`/`.backlink-list`/`.life-badge`/`.life-pick-row` 컴포넌트를 재사용했다.

## 9. 마이그레이션 계획

코드가 이미 "필드가 없으면 만든다" 방식(`if (!m.factionIds) m.factionIds = []`)으로 짜여 있어, 별도의 데이터 마이그레이션 스크립트나 버전 플래그 없이 **기존에 저장된 원고/문서를 열거나 검증 탭을 켜는 순간 자동으로 새 필드가 채워진다.** 기존 사용자 데이터에 대해 실행할 일회성 작업은 없음.

## 10. 누락된 기능 / 향후 확장 포인트

- 세계관 버전 시스템(7장 설계만 존재).
- AI 생성 자산 카탈로그(등장인물 문서에 이미지 자산 메타데이터를 붙이는 소규모 기능으로 추후 추가 가능 — `doc.assetRefs: [{label, kind, promptText, model, generatedAt}]` 형태를 제안, 전부 optional).
- 수필·시나리오 원고에도 세력/사건/떡밥 자동 연결 확장(현재는 소설 원고만 대상 — `ESSAY_ENTRIES`/`PLAY_SCRIPTS`는 `relatedCharacterIds`만 있고 이번 확장 대상에서는 제외).
- "화별 요약" 표(29번)처럼 화 단위 메타데이터(복선 삽입/회수, 다음화 예고)를 원고 목록 페이지에 열로 보여주는 대시보드.

## 11. AI 관련 항목 정리

- **AI 제거 가능(=애초에 코드에 없었음)**: 자동 설정 생성, 자동 인물/세계관 생성, 자동 분류·태깅·검증·관계도·지도·연표·복선 생성 — 바이블이 금지한 항목들은 원래 이 앱에 구현된 적이 없다.
- **AI 유지 필요 항목**: 없음. 이번 통합 작업 범위 안에서 AI에 의존해야만 가능한 기능은 없다.
- **오프라인 대체 방안**: 캐릭터 붕괴 검증(의미 판단 필요)은 자동화 대신 자가 점검 체크리스트로, 설정 충돌 검증은 구조적으로 결정 가능한 부분만 규칙 기반으로 구현했다(4장). 전체 앱은 여전히 `localStorage` 기반 단일 HTML 파일로, 네트워크 요청이 전혀 없다.

## 12. 실제 적용 코드

전부 `app/jipseul-note.html` 한 파일에 반영(브랜치 `claude/md-files-review-work-39x74u`). 변경 지점 요약:
`CATS`/`CAT_ICON`(복선 카테고리) · `PROP_PRESETS`/`DEFAULT_FIELD_TEMPLATES`(신규 프리셋) · `renderDocPage`의 떡밥/복선 공용 블록 · `detectDocRefsInManuscript` 계열 함수 · `updateContextPanel` · `APPEARANCE_BACKLINK`/`renderCharacterAppearancesCard` · `deleteDoc`/`deleteManuscript`(참조 정리) · `WS_TABS`/`WS_CATEGORIES`/`WS_SINGLE_ITEM_TABS`/`renderWorkSections`(설정 검증 탭) · `findHierarchyCycles`/`findOrphanManuscriptRefs`/`renderValidationPage`(신규 검증 페이지).
