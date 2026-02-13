# DLthon DKTC Detection
아이펠(AIFFEL)의 인공지능 리서치 교육 과정의 일환으로 시행된 DLthon 진행 결과를 기록한 팀 레포지토리입니다.

<br>
<br/>

## Team HuntrixAI
Hunt the hidden threats in the Matrix of conversations.
| 이름 | 역할 |
|---|---|
| 팀원 전원 | End-to-End (데이터 → 모델 → 제출) 각자 풀 파이프라인 경험 |

매일 회고를 통해 Lesson Learned → 4개 버전 반복 개선

<br>
<br/>

## DLthon

아이펠에서 진행하는 캐글 형식의 딥러닝 경진 대회입니다.

### **DKTC (Dataset of Korean Threatening Conversations)**

이번 DLthon 주제는 [2021 인공지능 그랜드 챌린지](https://www.ai-challenge.kr/) 대회에 참가하기 위해 [TUNiB](http://tunib.ai/)이 자체적으로 제작한 한국어 위협 대화 데이터셋을 활용하여 대화의 성격을 **위협 세부 클래스 4개 또는 일반 대화 중 하나**로 분류하고, 모델의 분류 성능을 높일 수 있는 방안을 찾는 것을 목표로 합니다.

#### 데이터 개요

- 학습 데이터: '협박', '갈취', '직장 내 괴롭힘', '기타 괴롭힘' 등 4개 클래스 각 1,000개 내외로 구성

- 테스트 데이터: '협박', '갈취', '직장 내 괴롭힘', '기타 괴롭힘', '일반 대화' 등 5개 클래스 각 100개로 구성

#### 대회 규칙

- 4개의 위협 세부 클래스는 Augmentation만 가능 (새로운 데이터 추가/생성 불가)

-   일반대화 클래스는 합성데이터로 구성

    * 다양한 프롬프트로 문장을 생성하고 학습에 활용
        
    * 합성 데이터 기반의 성능만 최종 결과물로 제출 가능

      * 합성데이터 생성 및 활용(필수)
     
      * 기 확보된 데이터 활용(AI hub 등 활용, 추가실험)

- 평가지표는 F1 Score

- 실험 결과를 Ablation study형식으로 기록
<br>
<br/>

## Folders

- `output/`⭐: 발표 자료와 최종 버전의 코드 파일, 실험 결과 그래프 이미지 등 최종 결과물 저장

  
- `data/`: 최종 모델 학습에 사용된 학습 데이터 저장
  
    - `provided/`: 제공된 훈련 및 검증 (테스트) 데이터
      
    - `external/`: 모델 학습을 위해 수집한 외부 데이터 (하단 '사용 데이터 출처' 참고)



- `eda/`: 제공된 훈련 데이터 분포를 탐색한 exploration data analysis 코드 저장



- `experiments/`: 이전 버전의 실험 코드 파일 저장

* 여러 실험 중 최종 버전은 리더보드 최고 성적 기준으로 결정 (`experiments/v3`)

<br>
<br/>

```
├── LICENSE
├── README.md
├── data
│   ├── external
│   │   ├── KakaoData.csv
│   │   ├── nsmc_train.txt
│   │   └── smilestyle_dataset.tsv
│   └── provided
│       ├── test.csv
│       └── train.csv
├── eda
│   └── data_exploration.ipynb
├── experiments
│   ├── v1_baseline.ipynb
│   ├── v2_synthesized_normal_data.ipynb
│   ├── v3_hard_negative_mining.ipynb
│   └── v4_tapt_applied.ipynb
└── output⭐
    ├── README.md
    ├── DLthonHuntrixAI_DKTC_Threat_Detection.pdf
    ├── figures
    │   ├── ablation_confusion_matrices.png
    │   └── ablation_learning_curves.png
    └── final_src_code.ipynb
```

<br>
<br/>

## Data Sources

- **TUNiB**
  - [DKTC Dataset](https://github.com/tunib-ai/DKTC) — Cho et al., 2022

- **Smilegate AI**
  
  - [`SmileStyle (.tsv)`](https://github.com/smilegate-ai/korean_smile_style_dataset?tab=readme-ov-file): 한국어 문체 스타일 변환 데이터셋

  - [`kor_unsmile`](https://huggingface.co/datasets/smilegate-ai/kor_unsmile): 혐오 표현 데이터셋

-  **et9**
   - [`Naver Sentiment Movie Corpus v1.0 (NSMC)`](https://github.com/e9t/nsmc): 네이버 영화 리뷰 말뭉치

<br>
<br/>

## References

- [Focal Loss](https://arxiv.org/abs/1708.02002) — Lin et al., 2017
- [R-Drop](https://arxiv.org/abs/2106.14448) — Liang et al., 2021
- [FGM](https://arxiv.org/abs/1605.07725) — Miyato et al., 2017
- [Don't Stop Pretraining](https://arxiv.org/abs/2004.10964) — Gururangan et al., 2020
- [LLRD](https://arxiv.org/abs/2006.09462) — Zhang et al., 2021
- [EMA / Mean Teachers](https://arxiv.org/abs/1703.01780) — Tarvainen & Valpola, 2017
- [Label Smoothing](https://arxiv.org/abs/1512.00567) — Szegedy et al., 2016
- [Prior Calibration](https://doi.org/10.1162/089976602753284446) — Saerens et al., 2002
- [Hard Negative Mining](https://sjkoding.tistory.com/102)

<br>
<br/>

------

## Version Log

- v1.0 - 레포지토리 생성 (2026.02.12)

- v1.1 - 레포지토리 세팅 + README.md 작성 (2026.02.13)
