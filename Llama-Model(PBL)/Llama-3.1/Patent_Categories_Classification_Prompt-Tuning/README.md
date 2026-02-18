# Llama-3.1-8B Patent Categories Classification (with Prompt-Tuning)

## Project - Overview
  **Purpose** : Meta-Llama-3.1-8B-Instruct 모델을 활용한 특허 데이터 분류에서, 모델 파라미터를 업데이트 하는 Fine-Tuning없이 Prompt-Tuning만으로 분류 정확도를 극대화 하는 실습을 진행하였다.

  **DataSet** : AI Hub 특허 분야 자동분류 데이터 


## Tech-Stack
  **Language** : Python

  **FrameWork** : PyTorch, HuggingFace(Transformers)

  **Base Model** : Meta-Llama-3.1-8B-Instruct 

  **Techniques** : 4-bit Quantization, Prompt Engineering, Error Analysis(Confusion Matrix)

  **Assist LLM** : Gemini

## HighLight-Code
  **1. Base모델의 오분류 원인 분석**

  기본 프롬프트를 사용했을 때 낮은 정확도(42.68%)의 원인을 찾기 위해, 정답 카테고리별 모델의 예측 분포를 분석하였다. 그 결과 모델이 농업, 임업 등의 텍스트 특징을 뚜렷하게 구분하지 못해 혼동하는 현상 확인

  **2. 제약 조건 및 정의를 포함하는 Prompt-Tuning**

  오류를 교정하기 위해 모델의 스스템 프롬프트를 고도화했다. 혼동이 잦은 카테고리에 대한 명확한 사전적 정의와 제약 조건을 주입하였다.
  ```python
  # 수정된 시스템 프롬프트 (Prompt Tuning)
  system_prompt_ver2 = f"""너는 특허 카테고리를 분류하는 전문가야. 
  아래 내용을 다음 특허 카테고리 중 하나로 분류해줘. 가능한 특허 카테고리 : {특허_카테고리_list}. 
  '농업' 카테고리를 '임업' 카테고리로 분류하지 않도록 주의해. '임업'은 '삼림에서 주로 나무를 벌채하고 목재를 생산하는 산업'을 의미해. 
  최종 출력 결과는 다른말은 하지말고 분류한 카테고리만 출력해줘."""
  ```

## Performance - Results
  **Base Prompt Accuracy** : 42.68%

  **After Prompt-Tuning Accuracy** : 70.09%

## TroubleShooting
  **(Problem 1) 비슷한 도메인에 대한 모델의 분류 혼동**

  **(현상)** : Base Prompt를 사용한 Zero-Shot 추론 시 정답이 '농업'인 특허를 '임업'으로 혹은 그 반대로 분류하는 오류 발생

  **(원인)** : LLM이 농림어업이라는 큰 범주 안에서 각 세부 산업의 텍스트적 특성 차이를 일반적인 프롬프트 지시만으로는 명확하게 잡지 못하였다.

  **(해결)** : 오류 케이스를 분석한 뒤, 가장 오답률이 높음 임업에 대한 명확한 정의를 프롬프트에 추가하였다. 이를 통해 추가적인 모델 재학습 비용 없이 모델의 추론 능력을 향상시켰다.

  
  
