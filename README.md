# DKTC Threat Detection — HuntrixAI

> AIFFEL DLthon | 한국어 위협 대화 5클래스 분류
>
> Hunt the hidden threats in the Matrix of conversations.

[![Kaggle](https://img.shields.io/badge/Kaggle-Competition-20BEFF?logo=kaggle)](https://www.kaggle.com/competitions/aiffel-d-lthon-dktc-online-16)

## Overview

온라인 대화에서 협박 / 갈취 / 직장 내 괴롭힘 / 기타 괴롭힘 / 일반 대화를 자동 분류하는 프로젝트입니다.

핵심 난제: train 데이터에 일반 대화가 0개이기 때문에 직접 합성해야 합니다.

| 항목 | 내용 |
|---|---|
| 평가 지표 | Macro F1 Score |
| 모델 | KcELECTRA + KcBERT 앙상블 |
| 최종 성과 | Kaggle F1 0.79 |


## Solution: 가설 → 검증 구조

### 가설 1: 일반대화를 못 잡는 건 데이터 부재 때문이다
- 가정: train에 일반대화 0개 → 모델이 패턴 자체를 모름
- 검증: 합성데이터 0개 → 1,000개 → 3,000개로 점진 증가
- 결론: 데이터 양도 중요하지만 품질(Hard Negative)이 더 결정적

### 가설 2: 오분류는 경계 케이스에서 발생한다
- 가정: "야 죽을래 ㅋㅋ"(농담)을 협박으로 분류하는 게 핵심 병목
- 검증: Hard Negative 200개 추가 전/후 비교 → 경계 정확도 향상
- 결론: 어려운 샘플을 의도적으로 학습시키는 게 범용 데이터 증가보다 효과적

### 가설 3: train과 test의 분포가 다르면 보정이 필요하다
- 가정: train(일반대화 ~20%) vs test(일반대화 ~75%)
- 검증: Prior Calibration 적용 전/후 비교
- 결론: 모델 성능과 별개로 분포 보정이 F1에 직접적 영향


## Pipeline

```
1. EDA           → 일반대화 부재 발견, 클래스별 텍스트 패턴 분석
2. 데이터 구축    → 공개 데이터 4종 + Hard Negative 200개 합성
3. 데이터 정제    → Quality Filter + Cosine Similarity 중복 제거
4. 모델 학습     → KcELECTRA + KcBERT, 5-Fold, 7가지 학습 기법
5. 앙상블 + 보정  → Weighted Ensemble + Prior Calibration + Threshold
6. Hard Sample   → Confidence 기반 Pseudo Labeling + 재학습
7. Ablation      → 각 기법의 기여도 정량 증명
```


## 데이터 구축 전략

### 기본 합성 (~3,000개)

4개 공개 데이터셋에서 일반 대화를 수집하고, 위협 키워드 포함 문장을 사전 필터링:

| 소스 | 설명 |
|---|---|
| SmileStyle | 한국어 스타일 대화 |
| kor_unsmile | 혐오 표현 데이터 (일반 라벨만 사용) |
| KakaoChat | 카카오톡 스타일 대화 |
| NSMC | 네이버 영화 리뷰 (중립 라벨) |

### Hard Negative 200개 (핵심 차별점)

임베딩 공간에서 괴롭힘과 가까운 일반 대화를 학습해야 경계 구분력이 올라간다는 가정 하에, 4개 경계면에서 각 50개씩 설계:

| 경계면 | 예시 | 수량 |
|---|---|---|
| 협박 ↔ 비유/관용 표현 | "죽여버린다 ㅋㅋ 이 게임 보스가~" | 50개 |
| 갈취 ↔ 호혜적 금전 거래 | "돈 내놔 ㅋㅋ 밥값 네가 쏜다며" | 50개 |
| 직장괴롭힘 ↔ 정당한 업무 피드백 | "이거 다시 해와. 퀄리티가 안 나와" | 50개 |
| 기타괴롭힘 ↔ 친밀한 놀림 | "야 뚱뚱이 ㅋㅋ 아 배고프다 밥 먹자" | 50개 |

### 데이터 정제

- Quality Filter: 15자 미만 / 500자 초과 / 특수문자 30%+ / 같은 문자 5회 반복 → 제거
- Cosine Similarity 중복 제거: KcELECTRA 임베딩 유사도 0.95+ → 제거


## 모델

### 선정 이유

| 모델 | 특징 | 역할 |
|---|---|---|
| KcELECTRA | 한국어 댓글 기반, 토큰 대체 탐지 | 세밀한 문맥 구분 |
| KcBERT | 한국어 댓글 기반, 마스크 예측 | 전반적 의미 이해 |

서로 다른 학습 방식이기 때문에 앙상블 시 상호 보완 효과를 기대할 수 있음. 거대 모델(GPT 등)을 쓰지 않은 이유는 Colab T4 GPU 환경 제약 때문이며, 앙상블 + 학습 기법으로 base 모델의 한계를 극복하는 전략을 택함.

### 7가지 학습 기법

| 기법 | 목적 | 근거 논문 |
|---|---|---|
| Focal Loss | 소수 클래스 집중 학습 | Lin et al., 2017 |
| R-Drop | 소규모 데이터 과적합 방지 | Liang et al., 2021 |
| FGM | 경계 케이스 강건성 확보 | Miyato et al., 2017 |
| LLRD | 사전학습 지식 보존 + 분류 특화 | Zhang et al., 2021 |
| EMA | 학습 후반 진동 안정화 | Tarvainen & Valpola, 2017 |
| Label Smoothing | 과도한 확신 방지, 일반화 향상 | Szegedy et al., 2016 |
| Dynamic Class Weight | train/test 분포 불일치 보정 | Saerens et al., 2002 |


## Ablation Study

각 기법이 실제로 효과가 있었는지 하나씩 빼면서 검증:

| 실험 | 구성 | 목적 |
|---|---|---|
| Exp1 | CE Loss only | Baseline 기준점 |
| Exp2 | + Focal Loss | 클래스 불균형 해결 효과 |
| Exp3 | + Focal + R-Drop | 과적합 방지 효과 |
| Exp4 | 합성 데이터 500개로 축소 | 데이터 양의 영향 |


## Repository Structure

```
DLthon_DKTC_Detection/
├── README.md
├── experiments/
│   ├── v1_baseline.ipynb
│   ├── v2_focal_rdrop.ipynb
│   ├── v3_hard_negative_mining.ipynb
│   └── v4_tapt_ensemble.ipynb
├── data/
│   ├── train.csv
│   ├── test.csv
│   └── synthetic/
│       ├── hard_negatives_200.csv
│       └── normal_conversation.csv
├── models/
├── submissions/
│   ├── submission_v1.csv
│   └── submission_final.csv
└── docs/
    └── presentation.md
```


## Quick Start

```bash
git clone https://github.com/HuntrixAI/DLthon_DKTC_Detection.git
cd DLthon_DKTC_Detection

pip install torch transformers datasets scikit-learn matplotlib seaborn

jupyter notebook experiments/v3_hard_negative_mining.ipynb
```

### 하이퍼파라미터

```python
MODEL_CONFIGS = [
    {'name': 'beomi/KcELECTRA-base-v2022', 'max_len': 384},
    {'name': 'beomi/kcbert-base',            'max_len': 300},
]
BATCH_SIZE    = 16
EPOCHS        = 5
LR            = 2e-5
N_FOLDS       = 5
WARMUP_RATIO  = 0.1
LLRD_FACTOR   = 0.95
FGM_EPSILON   = 1.0
EMA_DECAY     = 0.999
LABEL_SMOOTH  = 0.05
```


## Results

| Version | 전략 | Kaggle F1 |
|---|---|---|
| v1 | KcELECTRA + 합성 40개 | 0.72 |
| v2 | + 합성 1,000개 + Focal + R-Drop | 0.74 |
| v3 | + Hard Negative 200개 + 7기법 | 0.79 |
| v4 | + Prior Calibration + Pseudo Label | 0.72 |


## Team HuntrixAI

| 이름 | 역할 |
|---|---|
| 팀원 전원 | End-to-End (데이터 → 모델 → 제출) 각자 풀 파이프라인 경험 |

매일 회고를 통해 Lesson Learned → 4개 버전 반복 개선


## References

- [DKTC Dataset](https://github.com/tunib-ai/DKTC) — Cho et al., 2022
- [Focal Loss](https://arxiv.org/abs/1708.02002) — Lin et al., 2017
- [R-Drop](https://arxiv.org/abs/2106.14448) — Liang et al., 2021
- [FGM](https://arxiv.org/abs/1605.07725) — Miyato et al., 2017
- [Don't Stop Pretraining](https://arxiv.org/abs/2004.10964) — Gururangan et al., 2020
- [LLRD](https://arxiv.org/abs/2006.09462) — Zhang et al., 2021
- [EMA / Mean Teachers](https://arxiv.org/abs/1703.01780) — Tarvainen & Valpola, 2017
- [Label Smoothing](https://arxiv.org/abs/1512.00567) — Szegedy et al., 2016
- [Prior Calibration](https://doi.org/10.1162/089976602753284446) — Saerens et al., 2002


## License

MIT License
