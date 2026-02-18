# PEFT Prefix-Tuning Sentiment Classification

## Project-Overview
  **Purpose** : Hugging Face의 PEFT 라이브러리와 Prefix-Tuning 기법을 활용하여, Seq2Seq 구조의 T5-large를 금융 뉴스 감정 분류(Sentiment Classification) 에 맞게끔 Fine-Tuning 실습을 진행하였다.

  **DataSet** : 'Financial_phrasebank' 사용

## Tech-Stack
 **Language** : Python

 **FrameWork** : PyTorch, Hugging Face(Transformers, PEFT, Datasets)

 **Base Model** : T5-Large (Text-to-Text Transfer Transforemr)

 **Metrics** : Accuracy, Perplexity (PPL)

 **Assist LLM** : Gemini 

## HighLight-Code
  **1. Seq2Seq 모델을 위한 데이터 매핑**
  
  T5 Model은 모드 문제를 텍스트 생성 방식으로 풀기 때문에, 분류 문제의 정답(0,1,2)을 직접적인 텍스트(Negative, Netral, Positive)로 변환하여 학습 타겟으로 설정
  ```python
  # 정답 레이블을 텍스트로 변환
  classes = dataset["train"].features["label"].names
  dataset = dataset.map(
      lambda x: {"text_label": [classes[label] for label in x["label"]]},
      batched=True,
      num_proc=1,
  )
  ```

  **2. Loss Optimization을 위한 패딩 처리**
  
  정답(Label) 시퀀스의 패딩 토큰을 -100으로 변환하여,  Pytorch의 손실 함수가 불필요한 패딩 영역의 오차를 계산하지 않도록 설정
  ```python
  # pad_token_id를 -100으로 변경하여 Loss 계산에서 제외
  labels = tokenizer(targets, max_length=2, padding="max_length", truncation=True, return_tensors="pt")
  labels = labels["input_ids"]
  labels[labels == tokenizer.pad_token_id] = -100
  model_inputs["labels"] = labels
  ```

  **3. Prefix-Tuning Architecture**
  
  모델의 모든 파라미터를 동결하고, 각 Transformer Layer 앞에 가상의 프롬프트 토큰(Prefix) 20개를 덧붙여서 해당 파라미턴만 학습
  ```python
  from peft import PrefixTuningConfig, TaskType

  peft_config = PrefixTuningConfig(
      task_type=TaskType.SEQ_2_SEQ_LM, 
      inference_mode=False, 
      num_virtual_tokens=20
  )
  model = get_peft_model(model, peft_config)
  model.print_trainable_parameters()
  ```


## TroubleShooting
  **(Problem 1) 최신 라이브러리 설치 과정 혹은 데이터셋 로딩 중, 런타임의 파이썬 버전과 의존성 패키지 충돌로 에러 발생**

  **(원인)** : datasets의 라이브러리와 일부 PEFT 관련 모듈의 버전 불일치

  **(해결)** : 구동 스크립트 버전을 고정시킨채로 관련된 Transformers, Accelerate, PEFT는 최신 버전으로 강제 업데이터 하였다.
  
