# PEFT P-Tuning Semantic Similarity

## Project-Overview
  **Purpose** : Hugging Face의 PEFT(Parameter-Efficient Fine-Tuning)라이브러리와 P-Tuning 기법을 활용하여 RoBERTa-large 를 Semantic Similarity(문장 간 의미 유사도 판별)에 맞게끔 효율적인 Fine-Tuning 실습을 진행하였다.

## Tech-Stack
  **Language** : Python

  **FrameWork** : Hugging Face(Transformers, PEFT, Datasets, Evaluate)

  **Base Model** : RoBERTa-large

  **Metrics** : Accuracy, F1 score


## HighLight-Code
  **1. 데이터 전처리 및 동적 패딩(Dynamic Padding)**
  
  두 문장을 한 번에 토크나이징하고, 'DataCollatorWithPadding'을 통해 각 Batch 내에서 가장 긴 문장에 맞춰 패딩을 적용하여 불필요한 연산량을 줄인다.
  
  ```python
  def tokenize_function(examples):
      # 두 문장을 입력받아 토크나이징 진행 (max_length는 모델 기본값 따름)
      outputs = tokenizer(examples["sentence1"], examples["sentence2"], truncation=True, max_length=None)
      return outputs
  
  data_collator = DataCollatorWithPadding(tokenizer=tokenizer, padding="longest")
  ```

  **2. P-Tuning Model Setting**
  
  전체 파라미터를 업데이트하는 대신, 연속적인 가상 프롬프트 토큰(Virtual Toekns)을 입력 시퀀스에 추가하고 해당 프롬프트 인코더의 가중치    만 학습시킨다. 전체 파라미터의 0.67%만 학습에 적용한다
  ```python
  from peft import PromptEncoderConfig, get_peft_model
  
  peft_config = PromptEncoderConfig(
      task_type="SEQ_CLS", 
      num_virtual_tokens=20, 
      encoder_hidden_size=128
  )
  ```

## TroubleShooting
  **(Problem 1) 패키지 의존성 및 호환성 에러**
  
  **(원인)** : 최신 Transformers 라이브러리에서 Trainer API를 정상 구동하기 위해 accelerate 라이브러리가 필수로 요구되며, 최신 datasets라이브러리 버전에서 특정 데이터셋(GLUE) 로드시 구조적 호환성 문제 발생

  **(해결)** : PIP Install 과정에서 accelerate 라이브러리를 명시적으로 추가. datasets 라이브러리 버전을 강제 고정하여 해결하였다.
  

model = get_peft_model(model, peft_config)
model.print_trainable_parameters()
