# PEFT Prompt Tuning (Casual LM)

## Project - Overview
  **Purpose** : Hugging Face의 PEFT 라이브러리와 Prompt-Tuning 기법을 활용하여, Casual Langugage(bloomz-560m)을 텍스트 분류(트위터 불만 여부 판별)에 맞게끔 극도로 가벼운 연산량을 Fine-Tuning하는 실습을 진행하였다.


## Tech-Stack
  **Laguage** : Python

  **FrameWork** : PyTorch, Hugging Face(Transformers, PEFT, Datasets)

  **Base Model** : bigscience/bloomz-560m (Casual LM)

  **Metrics** : Loss, Perplexity (PPL)


## HightLight-Code
  **1. 프롬프트 템플릿 화 및 토크나이징**
   Casual LM이 분류 작업을 문장 생성 태스크로 이해할 수 있도록, 레이블을 하나의 프롬프트 문장(예: `Tweet text : {내용} Label : {정답}`)으로 결합하였다.

   모델이 오직 정답 토큰을 예측하는 부분에서만 Loss를 계산하도록 입력 텍스트 부분을 -100으로 마스킹 처리하였다.
  ```python
  # 입력 텍스트와 레이블을 프롬프트 형태로 병합
  inputs = [f"{text_column} : {x} Label : " for x in examples[text_column]]
  targets = [str(x) for x in examples[label_column]]
  
  # ... (중략) 입력 길이에 맞춰 패딩 처리 및 타겟 영역 외 -100 처리 ...
  labels["input_ids"][i] = [-100] * (max_length - len(sample_input_ids)) + label_input_ids
  ```

  **2. Prompt-Tuning Initializing**
  분류 태스크를 명확하게 지시하는 초기 텍스트(Prompt_tuning_init_text)를 제공하여 가상의 프롬프트 임베딩을 생성한다. 기존 모델의 가중치는 완전히 동결하고, 프롬프트 임베딩만을 업데이트한다.

  ```python
  from peft import PromptTuningInit, PromptTuningConfig, TaskType

  peft_config = PromptTuningConfig(
      task_type=TaskType.CAUSAL_LM,
      prompt_tuning_init=PromptTuningInit.TEXT,
      num_virtual_tokens=8, 
      prompt_tuning_init_text="Classify if the tweet is a complaint or not:", 
      tokenizer_name_or_path=model_name_or_path,
  )
  
  model = get_peft_model(model, peft_config)
  model.print_trainable_parameters()
  # 전체 5억 6천만 파라미터 중 약 8,192 파라미터(0.0014%)만 학습.
  ```

  **3. Model Inference**
  학습된 모델에 새로운 트위터 문장을 입력하여, 텍스트의 성격이 Complaint인지 No Complaint인지 텍스트 생성 방식으로 예측한다.
  ```python
  inputs = tokenizer(
      f'{text_column} : {"@nationalgridus I have no water and the bill is current and paid. Can you do something about this?"} Label : ',
      return_tensors="pt",
  )
  outputs = model.generate(
      input_ids=inputs["input_ids"], attention_mask=inputs["attention_mask"], max_new_tokens=10, eos_token_id=3
  )
  ```






     
