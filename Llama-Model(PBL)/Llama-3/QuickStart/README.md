# Meta Llama-3 (8B) Instruct Model Inference

## Project-Overview
  **Purpose** : Hugging Face를 통해 Meta의 최신 오픈소스인 LLM 'Meta-Llama-8B-Instruct' 모델을 로드하고, 시스템 프롬프트 및 Chat Template을 적용하여 텍스트 생성, 번역, 코드 작성 등 다양한 자연어 처리에 대한 실습을 진행하였다.

  
## Tech-Stack
  **Language** : Python

  **FrameWork** : Pytorch, HuggingFace(Transformer, Accelerate)

  **Model** : `meta-llama/Meta-Llama-3-8B-Instruct`

  **Assit LLM** : Gemini


## HighLight-Code
  **1. 메모리 효율화 모델 로드**

  8B 파라미터 크기의 모델을 로드하기 위해 데이터 타입을 bfloat 16으로 설정하고, Accelerate 활용하여 GPU 모델을 자동으로 할당(`device_map="auto"`)한다.
  ```python
  model = AutoModelForCausalLM.from_pretrained(
      "meta-llama/Meta-Llama-3-8B-Instruct",
      torch_dtype=torch.bfloat16,
      device_map="auto",
  )
  ```

  **2. Chat Template 및 Terminators 설정**

  Llama 3 모델이 대화형 문맥을 정확히 인지할 수 있도록 apply_chat_template를 적용한다. 특히, 모델이 끝 없이 텍스트를 생성하는 것을 방지하기 위해 특수 토큰 <|eot_id|> 를 정지 신호로 추가하였다.

  ```python
  # Llama 3 대화형 템플릿 적용
  input_ids = tokenizer.apply_chat_template(
      messages,
      add_generation_prompt=True,
      return_tensors="pt"
  ).to(model.device)
  
  # 모델 생성 중단 토큰 지정
  terminators = [
      tokenizer.eos_token_id,
      tokenizer.convert_tokens_to_ids("<|eot_id|>") # 대화 턴 종료 토큰
  ]
  
  outputs = model.generate(
      input_ids,
      max_new_tokens=256,
      eos_token_id=terminators,
      do_sample=True,
      temperature=0.6,
      top_p=0.9
  )
  ```

## TroubleShooting
  **(Problem 1) Llama-3 모델의 무한 텍스트 생성 및 사용자 흉내 현상 방지**

  **(원인)** : Llama-3 Instruct 모델은 대화의 턴을 구분하기 위해 특별한 토큰 <|eot_id|> 을 사용하는데, 이를 생성 중단 조건으로 명시하지 않으면 모델이 대화가 끝났음을 인지하지 못한다.

  **(해결)** : tokenizer.convert_tokens_to_ids(<|eot_id|>))를 추출하여 eos_token_id 파라미터의 리스트에 포함시킴으로써, AI가 답변의 끝맺음을 스스로 통제할 수 있도록 해결하였다.
  
