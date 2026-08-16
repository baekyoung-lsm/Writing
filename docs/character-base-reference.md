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
assets/character-layers/hair/female_bob_black_medium.png            (헤어 레이어, 아직 앱에 미연결)
assets/character-layers/tops/female_blouse_blue_transparent.png     (의상 원본, rembg 투명화만 된 상태)
assets/character-layers/tops/female_blouse_blue_aligned_opt.png     (몸 랜드마크에 맞춰 정렬·축소, 앱에 실제 임베드)
assets/character-layers/hair/female_<style>_opt.png       (헤어 11종, 정렬 불필요 -- 몸과 같은 832x1216 캔버스에 인페인팅해서 바로 겹쳐짐, 앱에 실제 임베드)
assets/character-layers/tops/female_<id>_opt.png          (상의 8종 -- shirt는 위 blouse 재사용, 나머지 7종 신규, 앱에 실제 임베드)
assets/character-layers/bottoms/female_<id>_opt.png       (하의 7종, 앱에 실제 임베드)
assets/character-layers/shoes/female_<id>_opt.png         (신발 6종, 앱에 실제 임베드)
```

헤어/상의/하의/신발의 `<style>`/`<id>`는 `app/jipseul-note.html`의 `PixelDollParts.hair`/`tops`/`bottoms`/`shoes` enum 키와 그대로 맞춘 이름. `bald`(헤어)·`none`(상의/하의)·`barefoot`(신발)은 "에셋 없음"이 곧 정답이라 이미지가 없다.

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

## 의상 레이어 — 해결됨, 앱에 반영함 (자체 생성 + rembg + 랜드마크 정렬 + 메시 워프 공유)

VTON 계열(외부 유료 API·실사풍 학습이라 화풍 불일치 위험) 대신, Z-Image Turbo + 기준 이미지를 style reference로 삼아 의상 1개(블라우스)를 "몸 없이 옷만" 흰 배경에 단독 생성 → `rembg`로 배경 투명화까지는 지난 시도에서 이미 검증함(`assets/character-layers/tops/female_blouse_blue_transparent.png` — 파일명이 `_raw`였던 걸 실제 내용에 맞게 `_transparent`로 정정함, rembg를 거친 투명 PNG가 맞음).

이번에 마저 풀어서 실제로 앱에 연결한 부분:
- **정렬/스케일 문제 해결**: 생성된 옷이 캔버스를 거의 다 채워서 몸보다 훨씬 크게 나오는 문제를, 옷 이미지에서 칼라(옷깃) 폭·Y좌표를 알파 채널로 직접 실측한 뒤, 몸의 어깨 랜드마크(`APPEAR_RASTER_LANDMARKS`, 반너비 123px)에 여유폭 15%를 더한 값에 맞춰 등비 축소하고, 칼라 Y를 몸의 어깨 Y 근처로 옮기는 방식으로 정렬함(`assets/character-layers/tops/female_blouse_blue_aligned_opt.png`). 몸 이미지 위에 합성해서 실제로 잘 맞는지 눈으로 확인함.
- **체형 변화에 옷도 같이 반응**: 옷을 위한 별도 계산을 만들지 않고, 몸에 이미 쓰는 `appearRasterWarpMesh`(bodyType·gender 기반 곡선 워프)를 옷 이미지에도 **그대로** 적용함(`appearRasterWarpDraw`로 삼각형 그리기 루프를 공용 함수로 뽑아냄). 이렇게 하면 체형이 바뀔 때 몸과 옷이 항상 같은 비율로 같이 늘어나서 어긋날 일이 없음 — 지난번에 검토했던 CSS 3분할 기법 대신 이 방식을 선택한 이유이기도 함(같은 코드를 재사용하고, 옷마다 사람이 분할 경계를 새로 잡아줄 필요가 없음).
- **앱 반영**: "외형 확인" 패널에서 `gender !== "m"`이면(`appearRasterHasOutfit`) 안내 문구 대신 `<canvas id="appearOutfitCanvas">`에 몸+옷을 합성해서 보여줌(`appearRasterDrawOutfitInto`). 남자는 아직 의상 자산이 없어서 기존 안내 문구를 그대로 유지함(회귀 없음, 헤드리스로 확인). 헤드리스 Chrome으로 체형(글래머+가슴 매우 큼)·기본 체형 등에서 실제로 옷이 몸을 따라 늘어나는 것을 스크린샷으로 확인함.

**남은 다듬을 점**: 기장이 다소 길고 헐렁하게(원피스에 가깝게) 나옴 — 옷 자체를 다시 생성하거나 밑단을 크롭하면 더 자연스러워짐. 지금은 상의 1종뿐이라 패널에 "지금은 상의 1종만 미리 입혀볼 수 있어요" 안내를 추가해 기대치를 정확히 알려줌.

## 헤어 11종 + 상의 7종 + 하의 7종 + 신발 6종 완성, enum 기반으로 앱에 전면 연동함

이전 세션까지는 상의 1종(블라우스)만 하드코딩으로 붙어 있었음. 이번 세션에서 `PixelDollParts.hair`/`tops`/`bottoms`/`shoes` enum 전체(대머리·없음류 제외)를 실제 이미지로 채우고, "외형 확인" 패널이 캐릭터가 고른 조합(헤어+상의+하의+신발)을 실제로 합성해서 보여주도록 코드 구조 자체를 바꿈.

**생성 파이프라인 (ComfyUI 재사용, `python -m pip install rembg`는 결국 안 씀)**
- 헤어: 기존에 검증된 인페인팅 방식(`DifferentialDiffusion`+`SetLatentNoiseMask`, 마스크는 얼굴 부분 구멍) 그대로 반복. steps=8~12, cfg=2.0~2.5(기존 세션의 cfg=1.0보다 살짝 올림 -- 마스크 밖은 어차피 100% 보존되니 마스크 안 색상 표현력만 올라감, 몸 보존에는 영향 없음을 확인).
- 상의/하의/신발: 몸 이미지를 참조로 안 걸고(`TextEncodeZImageOmni` reference 방식은 latent shape mismatch 에러로 실패, 디버깅 포기) 순수 `CLIPTextEncode` 텍스트 프롬프트만으로 흰 배경에 단독 생성. cfg=1.0(원래 시도)은 그라데이션/음영이 남는 문제가 있었음 -- cfg=3.0~4.0으로 올리고 프롬프트에 "flat solid color, no gradient, no shading, no highlight, cel-shaded icon style"을 강하게 반복하니 대부분 해결됨. 실측: 토르소 영역 RGB 표준편차가 cfg=1.0에서 채널당 약 120~150 → cfg=3.0에서 1~3으로 감소(픽셀 단위로 거의 완전한 단색).
- 갑옷류(armor, armorLegs)는 금속 표현 특성상 그라데이션이 특히 잘 안 없어짐 -- "children's book flat silhouette icon, 두 가지 색상만 사용, 두 색 모두 100% 균일" 식으로 프롬프트를 더 강하게 제한하고 cfg=4.0까지 올려서 겨우 통과 수준으로 만듦.

**품질 검수에서 실제로 걸러낸 것들 (전수 육안 검수, "매우 까다롭게")**
- top_armor, top_qipaoDress, top_romanceUniform, bottom_wuxiaRobeSkirt, bottom_armorLegs, bottom_leggings(사실적 사진 화풍으로 나와서 전면 재생성), 신발 boots/sandals(그라데이션 또는 좌우 짝짝이 색상) -- 전부 재생성 후 통과.
- shoe_heels는 좌우 신발이 겹쳐서 그려지는(한쪽은 펌프스, 한쪽은 부츠처럼 보이는) 비대칭 구도라 별도로 다시 생성함.

**정렬 파이프라인 -- `align_garment()` 버그 근본 수정**
기존 방식은 옷 이미지의 고정된 한 행(collar Y 등)에서 반너비를 재서 스케일을 계산했는데, ballDress처럼 그 행이 우연히 비어있거나 극단적으로 넓은 옷(off-shoulder 등)을 만나면 스케일 값이 깨졌음(`scale=94.3` 등). **수정**: 옷 자체 bbox 상단 12~18% 밴드 전체에서 각 행의 폭을 재고 **중앙값(median)**을 반너비로 씀 -- 한 행이 어쩌다 비정상이어도 중앙값은 영향을 거의 안 받음. 상의는 어깨 랜드마크(halfw×2×1.15)에, 하의는 허리 랜드마크에, 신발은 발목 랜드마크(×1.9, 두 짝이 나란히 서는 폭 고려)에 맞춰 등비 축소 후 배치. ballDress로 재현·수정 확인, jeans/tank/romanceSkirt/wuxiaRobe/shorts/qipaoDress로 회귀 없음 확인(전부 몸 위에 합성해서 육안 검수).

**헤어 레이어 추출 -- VAE 왕복 노이즈 버그 발견·수정**
diff 픽셀 임계값을 낮게(10) 잡았더니, 인페인팅이 실제로 그리지 않은 마스크 영역(어깨·팔 등)에도 VAE 인코드/디코드 왕복에서 생기는 미세한 재구성 오차가 "변경된 픽셀"로 잡혀서, 팔레트 축소 후 피부색/헤어색이 뒤섞인 반점 노이즈로 눈에 띄게 나타났음(twintail/hoodie 조합에서 발견). **수정**: 임계값을 45로 올리고, 연결omponent 분석(`scipy.ndimage.label`)으로 40px 미만의 작은 조각(=노이즈)은 버리고 큰 덩어리(=실제 머리카락)만 남김.

**longFantasy 헤어 -- 마스크 모양 자체가 원인이었던 버그**
기존 마스크가 가슴 높이(캔버스의 40%)에서 끝나 있어서 "허리 아래까지 오는 긴 머리"를 절대 그릴 수 없는 모양이었음(길이 부족의 진짜 원인). 마스크를 허벅지 중간(캔버스의 80%)까지 두 갈래로 늘려 새로 그려서 해결. 추가로 머리 돔과 갈래 사이 연결부가 오목한(concave) 모양이면 그 자리에 흰 반점(미채색 구멍) 아티팩트가 생기는 것도 발견 -- 연결부를 매끈한 타원 돔으로 단순화해서 같이 해결.

**afro 정수리 대머리** -- 이번 세션 재시도(cfg=2.0, 기존 마스크 재사용)에서 자연스럽게 해결됨. 정수리까지 완전히 덮인 둥근 아프로로 나옴.

**앱 코드 반영 (`app/jipseul-note.html`)**
- `APPEAR_RASTER_HAIR_PREVIEW`/`APPEAR_RASTER_BOTTOM_PREVIEW`/`APPEAR_RASTER_SHOES_PREVIEW` 신설, `APPEAR_RASTER_TOP_PREVIEW`를 `{f: {shirt: ..., hoodie: ..., ...}}` 형태로 확장(기존 블라우스 base64는 그대로 재사용, 새로 인코딩 안 함). 큰 base64 삽입은 항상 그렇듯 Python 스크립트(`embed_assets.py`, 스크래치패드)로 문자열 치환.
- `appearRasterDrawOutfitInto`를 하드코딩 2-레이어(몸+블라우스)에서 하의→신발→상의→헤어 순 범용 레이어 합성으로 재작성. `PixelDollParts.tops[ap.top].dress === true`(치파오/드레스류)면 SVG 벡터돌과 동일한 규칙으로 하의 레이어를 생략. 모든 레이어에 같은 `appearRasterWarpMesh`를 공유해서 체형 반응성 유지.
- `appearHairFilterCss(targetHex)` 신설 -- `appearSkinFilterCss`와 똑같은 hue-rotate/saturate/brightness 방식, 기준색 `APPEAR_RASTER_HAIR_BASE = "#3B2A1E"`(헤어 이미지를 생성한 색이자 앱 기본 머리색 `APPEAR_HAIR_COLORS[1]`과 정확히 일치 -- 기본 캐릭터는 필터가 완전 no-op). 헤어 레이어에만 적용, 의상 레이어는 아직 색상 필터 없음(기존 블라우스도 마찬가지였던 범위 유지 -- topColor/bottomColor 필터는 향후 과제).
- 헤드리스 대신 실제 Chrome(로컬 정적 서버 + `appearRasterDrawOutfitInto` 직접 호출 + canvas dataURL을 fetch로 로컬 서버에 저장)로 다양한 조합(로맨스 유니폼+업두, 아머+아프로, 볼드레스+롱판타지 헤어(보라색 필터 적용 확인), 치파오+글래머 체형, 탱크+슬림 체형+하복치마)을 실제 렌더링해서 확인 -- 워프·레이어 순서·드레스일 때 하의 생략·헤어 색상 필터 전부 정상 동작.

## 남자 에셋 완성 + 상의/하의 색상 필터(topColor/bottomColor) 추가

바로 다음 세션에서 이어서 완료함. `appearRasterHasOutfit`가 이제 항상 `true`를 반환 -- 남자도 실제 합성 미리보기가 나옴(기존 안내 문구 분기는 도달 불가능한 죽은 코드로 남아있으나 제거하지 않음, 위험 없는 상태라 그대로 둠).

**남자 헤어 7종** (dandy/topknot/mohawk/buzz는 남자 전용, longFantasy/ponytail/afro는 공용): 여자용 마스크를 그대로 재사용하되 남자 기준 이미지의 머리 위치가 12px 더 높다는 것을 어깨 랜드마크 차이(300 vs 312)로 확인하고 마스크를 위로 12px 시프트해서 재사용. 다만 두 가지가 시프트만으로는 안 통했음:
- **afro**: 마스크가 (여자 몸에서는 우연히 잘 나왔지만) 실제로는 머리~어깨까지 넓게 퍼진 모양이었는데, 남자 몸(목이 짧음)에서는 모델이 그 넓은 영역을 "어깨 위 목도리"로 해석해버림 -- 어깨까지 닿지 않는 머리만의 좁은 원형 마스크로 새로 그려서 해결(6번째 시도 만에 성공, 색이 안 채워지거나 빈 윤곽선만 나오는 실패를 두 번 더 거침).
- **topknot**: 마스크 위쪽 경계가 캔버스 y=0에 거의 닿아 있어서 상투가 위로 솟아오를 여백이 아예 없었음(여자 쪽도 사실 여백이 빠듯했는데 남자는 12px 위로 시프트하면서 그마저 없어짐) -- 머리 위에 여백을 더 두고 상투 봉우리용 타원을 따로 얹은 마스크를 새로 그려서 해결.

**남자용 셔츠(top=shirt) 신규 생성**: 기존 블라우스는 명백히 여성적인 디자인이라 재사용하지 않고, 캐주얼 버튼다운 셔츠를 별도로 생성.

**나머지 상의/하의/신발(gender="u")은 재생성 없이 재정렬만**: hoodie/romanceUniform/wuxiaRobe/armor/tank, jeans/shorts/wuxiaRobeSkirt/armorLegs/leggings, sneakers/dressShoes/boots/leatherBoots/sandals는 몸 없이 단독으로 생성된 이미지라 애초에 성별색이 없음 -- 같은 원본 이미지를 남자 랜드마크(어깨 반너비 117 등)에 맞춰 `align_top`/`align_bottom`/`align_shoe`로 다시 정렬만 해서 `male_*_opt.png`로 저장. ComfyUI 재생성이 전혀 필요 없었음.

**헤어 흰 반점 잔여 문제 수정**: `extract_hair_layer`의 연결-컴포넌트 필터를 "min_blob 이상이면 전부 유지"에서 "가장 큰 덩어리(=실제 헤어) 또는 그 20% 이상 크기인 덩어리만 유지"로 강화함 -- 이전 필터는 VAE 왕복 노이즈보다 큰(그러나 실제 헤어보다는 훨씬 작은) 우연한 조각(예: 마스크 안에서 모델이 살짝 다시 그린 쇄골 윤곽선)을 걸러내지 못했음. 여자 헤어 11종 전부 이 필터로 재추출.

**shoe_heels 스틸레토 굽 재생성**: "front view"만으로는 굽이 거의 안 보였던 문제 -- 프롬프트에 "slight three-quarter angled view so the heel post is clearly visible"를 추가해서 해결(양쪽 신발이 약간 겹쳐 보이는 구도는 남았지만 굽은 뚜렷해짐).

**`appearGarmentFilterCss(baseHex, targetHex)` 신설**: 헤어는 모든 스타일을 `#3B2A1E` 한 가지 기준색으로 생성해서 단일 기준점이 있었지만, 옷은 에셋마다 원래 디자인 색이 다름(후드=남색, 치파오=빨강 등) -- 그래서 에셋별 기준색 테이블(`APPEAR_RASTER_TOP_BASE`/`APPEAR_RASTER_BOTTOM_BASE`, 실제 렌더된 픽셀에서 칼라/트림을 피해 고정 좌표로 샘플링해 얻음)을 만들고, 그 기준색에서 `ap.topColor`/`ap.bottomColor`로 향하는 상대적 hue-rotate를 적용. 트림/단추 같은 보조색과의 상대적 색 관계는 유지된 채 주조색만 바뀜. **알려진 한계**: 갑옷류(armor/armorLegs)처럼 채도가 거의 0인 회색 기준색은 hue-rotate로 선명한 색을 "만들어낼" 수 없어서 재지정 결과가 탁하게 나옴(헤어 필터도 같은 계열의 한계를 이미 가지고 있었음, CSS filter 방식 자체의 근본적 한계).

**앱에서 재검증**: 실제 브라우저에서 남자 기본 조합·남자 조합+색상 재지정·여자 조합+색상 재지정·bald/none/barefoot 조합·muscular/slim 체형까지 크래시 없이 렌더링되는 것을 canvas dataURL 저장으로 직접 확인.

**남은 다듬을 점** (기능은 아님)
- 일부 헤어(업두)에 아주 작은 쇄골 윤곽선 조각이 여전히 남아있음 -- 상의 레이어가 그 위에 그려지면 대체로 안 보임.
- 갑옷류는 topColor/bottomColor 재지정 결과가 탁함(위 한계 참고).
