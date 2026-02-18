# Llama-3 (8B) KorQuad Fine-Tuning wiht Unsloth

## Project-Overview
  **Purpose** : 최신 오픈소스 LLM인 Llama-3 (8B) 모델을 한국어 질의응답 데이터셋인 KorQuad에 맞게 Fine-Tuning한 실습을 진행하였다.

  **Dataaset** : `korquad_prompt_da`


## Tech - Stack
  **Language** : Python

  **FrameWork** : Unsloth, HuggingFace(TRL, PEFT, Transformers, Datasets)

  **Base Model** : `unsloth/llama-3-8b-bnb-4bit`

  **Techniques** : 4-bit Quantization (Bitsandbytes), LoRA, SFT(Supervised Fine-Tuning)

  **Assit LLM** : Gemini


## HighLight-Code
  **1. Llama-3 전용 Chat Template Formating.**

  Llama-3 모델이 질문과 문맥을 명확히 구분할 수 있도록 시스템 프롬프트와 `<|start_header_id|>`, `<|eot_id|>` 등의 특수 태그를 매핑하여   학습 데이터를 재구성하였다

  ```python
  def formatting_prompts_func(row):
    ctx = row.get('context')
    qst = row.get('question')
    ans = row.get('answer')
    sys_msg = "You are a helpful AI assistant. Answer the question based on the context."

    # Llama-3 Instruct 포맷에 맞춘 프롬프트 구조화
    return f"""<|begin_of_text|><|start_header_id|>system<|end_header_id|>
  {sys_msg}<|eot_id|><|start_header_id|>user<|end_header_id|>
  Context: {ctx}
  Question: {qst}<|eot_id|><|start_header_id|>assistant<|end_header_id|>
  {ans}<|eot_id|>"""
  ```

  **2. Unsloth 기반 모델 로드 및 LoRA 기법 적용**

  Colab T4 GPU 환경에서 8B 사이즈 모델이 원활히 학습될 수 있도록 4-bit Quantization 상태로 로드하고, Unsloth의 최적화된 Gradient CheckPointing을 적용하였다.

  ```python
  from unsloth import FastLanguageModel

  model, tokenizer = FastLanguageModel.from_pretrained(
      model_name = "unsloth/llama-3-8b-bnb-4bit",
      max_seq_length = 2048,
      load_in_4bit = True,
  )
  
  model = FastLanguageModel.get_peft_model(
      model,
      r = 16,
      target_modules = ["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"],
      lora_alpha = 16,
      use_gradient_checkpointing = "unsloth", # Unsloth 전용 메모리 최적화
  )
  ```

## TroubleShooting
  **(Problem 1) AutoTrain Advanced 실행시 의존성 충돌 및 학습 중단 이슈**

  **(현상)** : Hugging Face의 AutoTrain 라이브러리를 사용하여 Llama-3 Fine-Tuning을 시도했으나, 내부 패키지 버전 충돌 및 T4 GPU에서의 OOM(Out Of Memory)현상으로 인하여 정상적인 학습이 되지 않았다.

  **(원인)** : AutoTrain의 무거운 추상화 레이어와 Colab 기본 환경 간의 라이브러리 불일치

  **(해결)** : AutoTrain을 배제하고, 모델 로드 및 학습 속도가 2배가량 빠른 VRAM 사용량이 대폭 최적화된 Unsloth FrameWork 기반의 SFTTrainer 코드로 전면 코드 마이그레이션을 진행하여 해결하였다.

  
