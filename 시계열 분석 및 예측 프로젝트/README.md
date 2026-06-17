# Model Training: NLI 기반 가짜뉴스 판독 모델 파인튜닝

## Project-Overview
* **Purpose** : 한국어 기사 본문(Premise)과 제목(Hypothesis) 간의 논리적 모순을 판별하여 가짜뉴스를 탐지하기 위해, 최대 4096 토큰을 지원하는 `monologg/kobigbird-bert-base` 모델을 이진 분류(참/거짓) Task에 맞게 파인튜닝하는 실습 진행.
* **Dataset** : 가짜뉴스 판별용 NLI 학습 데이터 총 10,000개 (Label 0과 1의 1:1 클래스 비율을 유지하도록 EDA 기반 교체 증강 적용).

## Tech-Stack
* **Language** : Python
* **FrameWork** : PyTorch, HuggingFace (Transformers)
* **Base Model** : `monologg/kobigbird-bert-base`
* **Techniques** : EDA(Random Deletion, Random Swap), Dynamic Padding, AMP(Automatic Mixed Precision)

## HighLight-Code

**1. 1:1 클래스 불균형을 방지하는 원본 데이터 교체 증강(EDA)**
데이터를 단순히 추가하여 비율이 무너지는 것을 막기 위해, 랜덤 삭제(Random Deletion) 및 위치 변경(Random Swap)을 적용한 변형 데이터를 생성한 후 기존 원본 데이터와 교체하여 1:1 비율을 유지한다.
```python
df_true_all = df[df['label'] == 0]
df_true_to_augment = df_true_all.sample(n=2000, random_state=42).copy()
df_true_to_augment['hypothesis'] = df_true_to_augment['hypothesis'].apply(augment_text)

# 원본 데이터 프레임에서 증강할 대상이었던 원본 2000개를 삭제
df_remaining = df.drop(df_true_to_augment.index)

# 남은 데이터 8000개 + 변형된 데이터 2000개 결합 (총 10,000개, 5000:5000 완벽 유지)
df_augmented = pd.concat([df_remaining, df_true_to_augment], ignore_index=True)


**2. 메모리 최적화를 위한 동적 패딩 및 혼합 정밀도(AMP) 학습 **


