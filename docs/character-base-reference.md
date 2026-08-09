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

## 조사했지만 보류한 것 — 헤어 레이어 생성 (Z-Image Turbo omni 편집 방식)

기준 이미지(image1)를 그대로 두고 "대머리에 단발머리만 추가"하도록 `TextEncodeZImageOmni`에 편집 지시를 내려 3번 시도함:
1. 프레이밍 지시 없이 시도 → 인물이 두상 클로즈업으로 줌인되어 전신 캔버스 비율이 완전히 깨짐.
2. "전신 프레이밍 유지" 지시 추가 → 프레이밍은 고쳐졌지만, 이번엔 몸통 충실도가 무너짐(다리가 하나로 뭉개짐, 손가락 디테일 소실, 팔다리 비율이 기준 이미지와 어긋남).

즉 이 모델·이 방식(이미지 1장 조건 + 텍스트 편집 지시)으로는 "몸은 완전히 고정하고 머리카락만 추가"가 안정적으로 안 됨 — `jipseul-note-sync.md` §2-1~2-2에서 이미 두 차례 실패로 기록된 것과 같은 종류의 문제(레이어 분리·화풍 유지가 동시에 잘 안 되는 것)가 이번에도 재현됨. 실험 결과물은 저장소에 커밋하지 않음(완성도가 기준 미달이라 자산으로 쓸 수 없음).

다음에 시도해볼 만한 방향:
- 헤어를 "편집"이 아니라 완전히 새 이미지로 생성(같은 seed·같은 style reference, 몸 대신 헤어스타일만 다른 프롬프트로 처음부터 생성)한 뒤, mediapipe `hair_segmenter.tflite`로 머리 영역만 마스킹해서 몸 부분 픽셀 자체를 버리는 방식 — 몸 충실도를 신경 쓸 필요가 없어짐.
- 또는 ControlNet(설치돼 있는 `flux1-dev-controlnet-union-pro-2.0`)으로 기준 이미지의 외곽선을 하드 제약으로 걸어 몸 형태 자체가 픽셀 조건이 아니라 구조적 제약이 되게 하는 방법(다만 이 환경은 현재 Flux 체크포인트 없이 Z-Image Turbo만 로드돼 있어 별도 unet 다운로드가 필요함).

## 다음 단계 (미완료, `jipseul-note-sync.md` §③ 참고)

- "외형 확인"(AI 원화 베타) 패널은 아직 이 기준 이미지들과 연결하지 않음 — 부위별로 분리된 투명 PNG 레이어(헤어/의상/신발 등)가 없으면 의상을 표현할 수 없어서, 기존 안내 문구("AI 원화(베타)는 아직 의상을 표현하지 못해요")를 그대로 유지함.
- 헤어 레이어 생성은 위 "조사했지만 보류한 것 — 헤어 레이어 생성" 참고 — 부위별 분리 레이어(헤어/의상/신발) 자체가 아직 없는 것이 이 프로젝트의 가장 큰 미해결 병목임.
- 의상은 VTON 계열(외부 유료 API·실사풍 학습이라 화풍 불일치 위험) 대신, 지금까지와 같은 방식(Z-Image Turbo + 기준 이미지 스타일 참조)으로 의상 1개를 흰 배경에 단독으로 생성한 뒤 이미 설치된 `rembg`로 배경만 투명화하는 방향으로 다음에 시도할 것.
- 부위별 분리 레이어를 받으면 Cozy Human Parser 대신 이 환경에 이미 설치된 mediapipe(`hair_segmenter.tflite`, `selfie_multiclass_256x256.tflite`)로 부위 마스크를 뽑는 방법도 시도해볼 것(무거운 커스텀 노드 설치 없이 재현 가능했음).
