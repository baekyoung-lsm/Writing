# AI 캐릭터 외형 — 기본 신체 기준 이미지

향후 헤어·눈썹·눈·입·의상·신발·액세서리 등 외형 에셋을 개별 이미지로 만들어 HTML에서 레이어처럼 합성하기 위한 "신체 기준(reference)" 이미지 세트. 완성 캐릭터가 아니라 향후 모든 에셋이 맞춰야 할 비율·자세·화풍의 기준점.

## 파일 위치

```
assets/character-base/female/seed0_female_base_none.png         (가슴 크기: 없음, 원본)
assets/character-base/female/seed0_female_base_small.png        (가슴 크기: 작음, 원본)
assets/character-base/female/seed0_female_base_medium.png       (가슴 크기: 보통, 원본)
assets/character-base/female/seed0_female_base_big.png          (가슴 크기: 큼, 원본)
assets/character-base/female/seed0_female_base_verybig.png      (가슴 크기: 매우 큼, 원본)
assets/character-base/female/seed0_female_base_none_opt.png     (위 5장의 팔레트 축소판 -- bustSize: none, 앱에 실제 임베드)
assets/character-base/female/seed0_female_base_small_opt.png    (bustSize: small, 앱에 실제 임베드)
assets/character-base/female/seed0_female_base_medium_opt.png   (bustSize: medium, 앱에 실제 임베드)
assets/character-base/female/seed0_female_base_large_opt.png    (bustSize: large, 앱에 실제 임베드)
assets/character-base/female/seed0_female_base_xlarge_opt.png   (bustSize: xlarge, 앱에 실제 임베드)
assets/character-base/male/seed42_male_base.png
assets/character-base/male/seed42_male_base_opt.png  (팔레트 축소판, 앱에 실제 임베드된 버전)
```

`_opt.png` 파일명의 `none/small/medium/large/xlarge`는 `app/jipseul-note.html`의 `bustSize`/`VD_BUST_SIZE_MULT` enum 키와 그대로 맞춘 이름이고, 원본 파일명의 `big/verybig`는 이전 세션 산출물 이름을 그대로 유지한 것 — 매핑은 big=large, verybig=xlarge.

원본은 `C:\Users\youngjune\AppData\Local\Comfy-Desktop\ComfyUI-Shared\input\`(여자, 이전 세션에서 이미 검증됨)와 `...\ComfyUI-Shared\output\seed42_male_base_00003_.png`(남자, 이번 세션에서 생성)에도 남아있음.

## 공통 기준

- 캔버스 832×1216, 순수 흰색 배경(#FFFFFF), 그림자·비네트 없음.
- 화풍: **플랫 2D 벡터, 굵은 검정 외곽선, 단색 평면 채색, 그라데이션·부드러운 그림자·안티앨리어싱 없음** — `docs/jipseul-note-sync.md` §2-1에서 이미 확정된 요구사항이며, 처음 사용자 프롬프트에 있던 "레트로 16비트 픽셀아트"보다 이 기준이 우선함(스크린샷 실물 기준 vs 텍스트 프롬프트 충돌 시 프로젝트 기존 결정 우선).
- 정면, 좌우대칭, 머리부터 발끝까지 여백 포함 전체 노출.
- 대머리(헤어 없음), 이목구비 없음(이목구비는 이 앱에서 별도로 SVG 코드로 그리기로 이미 결정됨 — `docs/jipseul-note-sync.md` §2-1 "얼굴이 그려져 있음" 문제 참고. 즉 신체 기준 이미지에는 얼굴 디테일을 절대 추가하지 않는다).
- 의상·액세서리·소품 없음.
- 여자: A-pose(양팔 살짝 아래로 벌림), 가슴 크기만 5단계로 다르고 키·머리 크기·어깨너비·팔다리 길이·손발 크기·중심선·자세·캔버스 내 위치는 전 단계 동일.
- 남자: 정확한 수평 T-pose(양팔이 어깨높이에서 일직선), 손끝까지 캔버스 안에 여백을 두고 다 들어오도록 축소 비율 적용됨.

## 생성 방법 (재현용)

로컬 ComfyUI API(`http://127.0.0.1:8188`)에 Z-Image Turbo(`z_image_turbo_bf16.safetensors` + `qwen_3_4b.safetensors` CLIP + `ae.safetensors` VAE) 워크플로우를 직접 POST. 핵심은 `TextEncodeZImageOmni` 노드의 `image1`에 이미 검증된 여자 기준 이미지를 넣어 화풍·캔버스를 그대로 물려받게 하고, 프롬프트로 자세/성별만 바꾸는 방식(레퍼런스 이미지를 진짜 조건으로 사용 — 텍스트 설명만으로는 화풍이 재현되지 않음).

- KSampler: seed는 각 기준값(남자 42) 그대로, steps 8, cfg 1.0, sampler `euler`, scheduler `simple`.
- `ModelSamplingAuraFlow` shift 3.0 (Z-Image Turbo 권장값).
- `EmptySD3LatentImage` 832×1216.

## 앱에 이미 반영한 부분

- `APPEAR_RASTER_PREVIEW.m`(외형 편집 "얼굴 확인"·"키·체형 비교" 패널의 남자 래스터 미리보기)을 이번에 만든 `seed42_male_base_opt.png`로 교체함. 기존에 박혀 있던 남자 이미지는 짧은 머리 실루엣과 젖꼭지가 그려져 있고 손 모서리가 들쭉날쭉해 이 기준 문서 §공통 기준(대머리·이목구비 없음·화풍 일관성)과 어긋나 있었음. 색상 팔레트를 48색으로 낮춰 44KB로 최적화해서 임베드(기존 남자 임베드 약 99KB보다 오히려 작음).
- `APPEAR_RASTER_PREVIEW.f`를 문자열 하나에서 `{ none, small, medium, large, xlarge }` 객체로 바꾸고, `appearRasterSrc(apLike)` 헬퍼로 성별+`bustSize`에 맞는 이미지를 고르도록 함. 여자 5단계 기준 이미지 전부(48색 팔레트, 각 약 25KB)를 임베드해서 가슴 크기 픽커를 누르면 "얼굴 확인"·"키·체형 비교" 패널의 래스터 미리보기가 실제로 바뀜(`refreshPreview()`도 함께 수정 — 기존엔 피부색 필터만 갱신하고 `bustSize` 변경은 무시했음).
- 헤드리스 Chrome 스크린샷 + Node로 임베드된 base64를 직접 디코드해 5장 전부와 남자 이미지가 깨지지 않고 그대로 들어있는지 확인함.

## 체형(bodyType) 16종 메시 워프 — 해결됨, 앱에 반영함

처음엔 VectorDoll의 랜드마크 Y좌표(`VY`)를 기준 이미지에 비율로 매핑해서 풀려고 했는데, **A/T-pose로 뻗은 팔이 waist·hip Y밴드를 가로질러** 실루엣 반너비 측정에 팔 폭이 섞여 들어가는 문제가 있었음(예: medium 이미지 waist 지점이 314px로 나와 실제 허리보다 훨씬 넓게 측정됨).

해결책: SAM 등 새 마스킹 노드를 설치하는 대신, 이미 설치돼 있던 `comfyui_controlnet_aux`의 `OpenposePreprocessor`로 두 기준 이미지(여자 medium, 남자)에서 실제 관절 좌표(어깨/팔꿈치/손목/골반/무릎/발목)를 뽑았음. 관절 좌표를 알고 나면 어깨·가슴·허리·골반 Y지점에서 "몸통 중심 ± (어깨 반span + 40px)" 범위로만 좌우 스캔해서 반너비를 재는 것만으로 팔 간섭 없이 깨끗한 값을 얻을 수 있었음(무릎·발목은 그 자체로 팔과 안 겹쳐서 문제 없음). 실측값은 `app/jipseul-note.html`의 `APPEAR_RASTER_LANDMARKS`에 하드코딩돼 있음.

이 실측 반너비에 VectorDoll의 `VectorDoll.bodyMetrics(bodyType, gender, bustSize)` 비율(체형별 목표값 / normal 기준값)을 곱해서, 데모 아티팩트(`6e233096`)에서 검증된 것과 같은 `buildMesh`/`warpTriangle` 삼각형 워프를 실제 raster 미리보기에 적용함(`appearRasterWarpMesh`/`appearRasterWarpTriangle`/`appearRasterDrawInto`). "얼굴 확인"·"키·체형 비교" 패널의 `<img>`를 `<canvas>`로 바꾸고, `bodyType`을 포함한 모든 픽커가 `refreshPreview()`에서 캔버스를 다시 그리도록 연결함. 헤드리스 Chrome으로 여성(글래머+가슴 매우 큼)·남성(슬림) 등 여러 조합을 실제로 렌더링해서 팔이 안 뒤틀리고 몸통만 자연스럽게 변형되는 것을 스크린샷으로 확인함.

버그 하나 발견·수정: 이미지 로더(`appearRasterLoadImage`)가 같은 이미지를 기다리는 콜백을 하나만 저장해서 덮어쓰는 구조였음 — 남자처럼 기준(baseline)과 현재(current) 캔버스가 같은 소스 이미지(성별당 이미지 1장)를 동시에 요청하면 먼저 등록된 콜백이 씹혀서 캔버스 하나가 안 그려지는 문제가 있었음(스크린샷으로 발견). 콜백을 배열로 큐잉해서 전부 실행하도록 고쳐서 해결.

## 헤어 레이어 생성 — 해결됨 (편집 방식 → 진짜 인페인팅 방식으로 전환)

처음엔 기준 이미지를 그대로 두고 "대머리에 단발머리만 추가"하도록 `TextEncodeZImageOmni`에 편집 지시를 내리는 방식(이미지 1장 전체를 다시 그리게 하는 "omni 편집")으로 3번 시도했는데, 매번 실패했음:
1. 프레이밍 지시 없이 시도 → 두상 클로즈업으로 줌인되어 전신 캔버스 비율이 깨짐.
2. "전신 프레이밍 유지" 지시 추가 → 프레이밍은 고쳐졌지만 몸통 충실도가 무너짐(다리가 하나로 뭉개짐, 손가락 디테일 소실).

원인: `TextEncodeZImageOmni` + 전체 이미지 재생성은 "이미지 전체를 편집"하는 것이지 "지정한 영역만 남기고 나머지는 원본 그대로 보존"하는 게 아님 — `jipseul-note-sync.md` §2-1~2-2의 두 차례 실패와 같은 종류의 문제.

**해결책 (3번째 시도, 성공)**: "편집"이 아니라 진짜 **인페인팅**으로 전환함 — 이미 설치돼 있던 `DifferentialDiffusion` + `SetLatentNoiseMask` 조합을 씀. 머리 영역만 흰색인 마스크(PIL로 직접 그림, 얼굴 부분은 구멍을 뚫어 제외)를 만들어서, 마스크 밖 영역은 latent 노이즈가 전혀 섞이지 않게 하면 몸이 픽셀 단위로 100% 보존됨. 이렇게 하니 다리·손가락·팔 비율이 원본과 완전히 동일하게 나왔고, 마스크에서 얼굴 부분을 도려내서 눈·코·입이 전혀 그려지지 않게 만들 수 있었음(이 프로젝트의 "이목구비는 SVG로 그린다" 방침과 정확히 맞아떨어짐).

- 워크플로우: `LoadImage`(기준 몸) → `VAEEncode` → `SetLatentNoiseMask`(마스크) → `DifferentialDiffusion`이 패치한 모델로 `KSampler` → `VAEDecode`.
- 첫 시도는 색이 흐릿한 흰머리로 나왔음(cfg=1.0이 매우 낮아서 프롬프트의 "검정"이 약하게 반영됨) → 프롬프트에 "solid jet black, #000000, no highlights"처럼 색을 강하게 반복 명시해서 해결.
- 레이어 추출: mediapipe `hair_segmenter.tflite`로 시도했으나 이 모델은 실사 사진으로 학습돼 있어서 플랫 벡터 그림체에서는 완전히 오작동함(머리카락 대신 팔 윤곽선 한 조각을 "머리"로 잘못 인식). 대신 **기준 원본과 인페인팅 결과물을 픽셀 단위로 diff한 뒤, 인페인팅에 실제로 쓴 마스크 영역으로만 diff를 제한**하는 방식으로 깨끗하게 추출함(diff는 마스크 바깥에서도 VAE 인코드/디코드 왕복 때문에 미세한 재구성 노이즈가 생기므로, 마스크 영역으로 반드시 제한해야 함).
- 결과물: `assets/character-layers/hair/female_bob_black_medium.png`(투명 배경 헤어 레이어), 같은 폴더에 사용한 인페인팅 마스크도 참고용으로 같이 넣어둠.

## 의상 레이어 — 방향 검증 완료 (자체 생성 + rembg)

VTON 계열(외부 유료 API·실사풍 학습이라 화풍 불일치 위험) 대신, 지금까지와 같은 방식(Z-Image Turbo + 기준 이미지를 style reference로 사용)으로 의상 1개(블라우스)를 "몸 없이 옷만" 흰 배경에 단독으로 생성한 뒤, 이미 설치된 `rembg`로 배경을 투명화하는 방식을 테스트함 — 결과: 화풍이 일관되게 유지되는 깨끗한 투명 PNG를 얻음(`assets/character-layers/tops/female_blouse_blue_raw.png`). VTON/LayerDiffusion 없이도 방향이 유효하다는 게 확인됨.

**아직 안 된 것**: 옷 이미지가 캔버스 전체를 거의 채우도록 생성돼서, 기준 몸 이미지의 실제 어깨·가슴·허리 위치(§ 위 "체형 메시 워프"에서 이미 실측한 `APPEAR_RASTER_LANDMARKS` 좌표)에 맞춰 축소·정렬하는 후처리가 아직 없음 — 지금 그대로 몸 위에 겹치면 옷이 몸보다 훨씬 크게 나옴. 다음엔 생성된 옷 이미지를 그 실측 랜드마크 좌표 기준으로 리사이즈·크롭하는 스크립트가 필요함.

## 다음 단계 (미완료, `jipseul-note-sync.md` §③ 참고)

- "외형 확인"(AI 원화 베타) 패널을 실제 레이어 합성으로 바꾸는 작업 자체는 아직 안 함 — 지금까지는 개별 레이어(헤어 1종, 옷 1종)가 실제로 만들어질 수 있다는 걸 검증한 단계이고, 이걸 여러 종류로 늘리고 몸 위에 정확히 정렬해서 합성하는 앱 통합은 다음 단계임. 그 전까지는 기존 안내 문구("AI 원화(베타)는 아직 의상을 표현하지 못해요")를 그대로 유지함.
- 옷 레이어를 몸 이미지의 실측 랜드마크에 맞춰 자동 정렬·리사이즈하는 후처리 스크립트가 필요함(위 "의상 레이어" 절 참고).
- 지금 검증된 헤어·의상 파이프라인(인페인팅 or 단독 생성 + diff/rembg 추출)을 반복해서 종류를 늘리는 작업이 남음(헤어스타일 여러 종류, 하의·신발·액세서리 등).
