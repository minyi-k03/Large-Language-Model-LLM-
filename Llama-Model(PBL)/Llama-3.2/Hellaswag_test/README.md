# Llama-3.2 & Gemma-2 SLM Performance Test (Korean Hellaswag)

## Project - Overview
  **Purpose** : 최신 경량화 언어 모델 SLM인 Llama-3.2 (3B, 1B)와 Gemma-2 (2B)모델을 활용하여, 상식 추론 벤치마크인 Hellaswag 데이터 셋에 대한 Zero-Shot Inference Performance를 테스트 하고 상용 LLM과 정확도를 비교 분석하는 실습을 진행하였다.

  **DataSets** : Korean Hellaswag (주어진 문맥 다음에 이어질 가장 적절한 결말을 4가지 선다형으로 고르는 데이터 셋)

  
## Tech-Stack
  **Language** : Python

  **FrameWork** : PyTorch, HuggingFace(Transformers)

  **Models Tested** : Meta-Llama-3.2-1B-Instruct, Meta-Llama-3.2-3B-Instruct, Google-Gemma-2-2B-it

  **Assist LLM** : Gemini


## HighLight-Code
  **1. 프롬프트 엔지니어링 및 챗 템플릿 적용**

  모델이 불필요한 설명 없이 숫자 정답만 출력하도록 시스템 프롬프트를 구성하고, 각 모델의 학습 포맷에 맞게 Chat Template를 적용한다

  ```python
    system_prompt = '너는 주어진 label과 context를 기반으로 4개의 선택지 중에 가장 그럴듯한 ending을 선택하는 임무를 가진 챗봇이야.     ending 후보중에서 가장 그럴듯한 ending을 선택하고, 출력은 다른 말은 하지말고 "[0, 1, 2, 3]" 중에 하나의 값으로 출력해줘.'
  
  # Llama-3.2 포맷 적용
  input_ids = tokenizer.apply_chat_template(
      messages,
      add_generation_prompt=True,
      return_tensors="pt"
  ).to(model.device)
  ```

## Model Perfromance Comparison Results

  총 10개의 샘플 데이터에 대한 모델별 정답률(Accuracy)를 비교한 결과이다

  1. GPT-4O-mini (9/10)
  2. Llama-3.1-8B-Instruct (7/10)
  3. Gemma-2-2B-it (6/10)
  4. Llama-3.2-3B-Instruct(5/10)
  5. Llama-3.2-1B-Instruct(2/10)

## TroubleShooting
  **(Problem 1) Tokenizer Padding Token 미지정 이슈**

  **(현상)** : Llama 계열 모델 로드시 pad_token이 정의되어 있지 않아 에러 발생 가능성 존재

  **(해결)** : Tokenizer 선언부 하단에 if tokenizer.pad_token is None: tokenizer.pad_token = tokenizer.eos_token 방어 코드를 추가하여 문장 종료 토큰일 패딩 토큰으로 사용하도록 조치

  **(Problem 2) Llama Model 생성 종료 제어**

  **(해결)** : 추론 함수 내에 tokenizer.convert_tokens_to_ids("<|eot_id|>")를 추가 설정하여, 모델이 정답 번호를 출력한 직후 문장 생성을 강제로 중단하여 해결하였다.
                        
