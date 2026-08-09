# AI 캐릭터 외형 — 기본 신체 기준 이미지

향후 헤어·눈썹·눈·입·의상·신발·액세서리 등 외형 에셋을 개별 이미지로 만들어 HTML에서 레이어처럼 합성하기 위한 "신체 기준(reference)" 이미지 세트. 완성 캐릭터가 아니라 향후 모든 에셋이 맞춰야 할 비율·자세·화풍의 기준점.

## 파일 위치

```
assets/character-base/female/seed0_female_base_none.png     (가슴 크기: 없음)
assets/character-base/female/seed0_female_base_small.png    (가슴 크기: 작음)
assets/character-base/female/seed0_female_base_medium.png   (가슴 크기: 보통)
assets/character-base/female/seed0_female_base_big.png      (가슴 크기: 큼)
assets/character-base/female/seed0_female_base_verybig.png  (가슴 크기: 매우 큼)
assets/character-base/male/seed42_male_base.png
assets/character-base/male/seed42_male_base_opt.png  (팔레트 축소판, 앱에 실제 임베드된 버전)
```

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

`app/jipseul-note.html`의 `APPEAR_RASTER_PREVIEW.m`(외형 편집 "얼굴 확인"·"키·체형 비교" 패널의 남자 래스터 미리보기)을 이번에 만든 `seed42_male_base_opt.png`로 교체함. 기존에 박혀 있던 남자 이미지는 짧은 머리 실루엣과 젖꼭지가 그려져 있고 손 모서리가 들쭉날쭉해 이 기준 문서 §공통 기준(대머리·이목구비 없음·화풍 일관성)과 어긋나 있었음 — 여자 쪽(`APPEAR_RASTER_PREVIEW.f`)은 이미 기준에 맞는 이미지가 들어있어 손대지 않음. 색상 팔레트를 48색으로 낮춰 44KB로 최적화해서 임베드(기존 남자 임베드 약 99KB보다 오히려 작음). 헤드리스 Chrome 스크린샷으로 앱이 정상 로드되는지 확인함.

## 다음 단계 (미완료, `jipseul-note-sync.md` §③ 참고)

- "외형 확인"(AI 원화 베타) 패널은 아직 이 기준 이미지들과 연결하지 않음 — 부위별로 분리된 투명 PNG 레이어(헤어/의상/신발 등)가 없으면 의상을 표현할 수 없어서, 기존 안내 문구("AI 원화(베타)는 아직 의상을 표현하지 못해요")를 그대로 유지함.
- 가슴 5단계 이미지는 아직 `bustSize` 선택에 따라 실제로 바뀌지 않음(현재 래스터 미리보기는 성별당 이미지 1장 고정). 체형 변형 방식 비교 데모(아티팩트 `6e233096`)에서 검증된 `buildMesh`/`warpTriangle`/`widthRatioAtY` 메시 워프 코드를 이 기준 이미지들에 적용하면, 이미지 1장을 16종 체형 수치(`BODY_PRESETS`)에 맞게 실시간으로 변형할 수 있음 — 다음 세션에서 이어서 할 만한 작업.
- 부위별 분리 레이어를 받으면 Cozy Human Parser 대신 이 환경에 이미 설치된 mediapipe(`hair_segmenter.tflite`, `selfie_multiclass_256x256.tflite`)로 부위 마스크를 뽑는 방법도 시도해볼 것(무거운 커스텀 노드 설치 없이 재현 가능했음).
