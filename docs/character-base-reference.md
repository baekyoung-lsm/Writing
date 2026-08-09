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

## 다음 단계 (미완료, `jipseul-note-sync.md` §③ 참고)

부위별로 분리된 투명 PNG 레이어(헤어/의상/신발 등)를 받기 전까지는 이 기준 이미지들을 실제 앱(`app/jipseul-note.html`)의 렌더링 로직에 아직 연결하지 않음 — 지금은 향후 에셋 제작의 비율·화풍 기준으로만 사용.
