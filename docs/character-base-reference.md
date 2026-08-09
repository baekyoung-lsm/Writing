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

## 조사했지만 보류한 것 — 체형(bodyType) 16종 메시 워프

체형 변형 방식 비교 데모(아티팩트 `6e233096`)의 `buildMesh`/`warpTriangle`/`widthRatioAtY`를 이 기준 이미지에 그대로 적용해보려고 실제 픽셀 좌표를 분석했는데, VectorDoll의 랜드마크 Y좌표(`VY`: shoulder=84, waist=164, hip=190 등, 총 302유닛)를 기준 이미지에 비율로 매핑해보니 **A/T-pose로 뻗은 팔이 waist·hip 랜드마크 Y 밴드를 가로질러**, 그 지점의 실루콘 반너비를 측정하면 팔 폭까지 같이 잡혀버림(예: medium 이미지 waist 지점 반너비가 314px로 실제 허리보다 훨씬 넓게 나옴). 팔 영역을 제외한 몸통만 분리하지 않고는 이 랜드마크 기반 워프를 안전하게 못 씀 — 그대로 적용하면 체형을 바꿀 때마다 몸통이 부자연스럽게 부풀거나 팔이 뒤틀리는 식으로 보일 위험이 커서 이번엔 넣지 않음. 시도할 거면:
1. mediapipe `selfie_multiclass_256x256.tflite`로 팔/몸통 마스크를 먼저 분리하고 몸통만의 실제 랜드마크 반너비를 다시 측정하거나,
2. 손으로 각 기준 이미지의 어깨/허리/골반/무릎/발목 Y좌표와 반너비를 직접 재서(`assets/character-base/*/landmarks.json` 같은 파일로) 하드코딩하는 방법이 더 안전함.

현재는 raster 모드의 체형 차이는 기존처럼 `bodyRatio`(수동 슬라이더, `scaleX`)로만 근사되고, `bodyType`(16종) 자체는 raster 미리보기에 반영되지 않음 — 이건 이번 변경 이전과 동일한 동작이라 회귀는 아님.

## 다음 단계 (미완료, `jipseul-note-sync.md` §③ 참고)

- "외형 확인"(AI 원화 베타) 패널은 아직 이 기준 이미지들과 연결하지 않음 — 부위별로 분리된 투명 PNG 레이어(헤어/의상/신발 등)가 없으면 의상을 표현할 수 없어서, 기존 안내 문구("AI 원화(베타)는 아직 의상을 표현하지 못해요")를 그대로 유지함.
- 체형(bodyType) 16종 메시 워프는 위 "조사했지만 보류한 것" 참고.
- 부위별 분리 레이어를 받으면 Cozy Human Parser 대신 이 환경에 이미 설치된 mediapipe(`hair_segmenter.tflite`, `selfie_multiclass_256x256.tflite`)로 부위 마스크를 뽑는 방법도 시도해볼 것(무거운 커스텀 노드 설치 없이 재현 가능했음).
