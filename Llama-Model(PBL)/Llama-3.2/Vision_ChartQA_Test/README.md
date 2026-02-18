# Llama-3.2-Vision-11B ChartQA Performance Test

## Project - Overview
  **Purpose** : Meta의 최신 멀티모달 모델인 Llama-3.2-11B-Vision-Instruct를 활용하여, 복잡한 차트 및 그래프 이미지를 분석하고 질의응답을 수행하는 ChartQA 실습을 진행하였다. 한국어 프롬프트를 각각 적용하여 모델의 다국어 비전 추론 능력을 비교 평가하였다.

  **DataSets** : ahmed-masry/ChartQA (차트 이미지 및 Query-Label 데이터 셋)

## Tech-Stack
  **Language** : Python

  **FrameWork** : PyTorch, HuggingFace(Transformers, BitsAndBytes)

  **Model** : Meta-Llama3.2-11B-Vision-Instruct

  **Techniques** : 4-bit Quantization, Multimodal Prompt-Engineering(VQA)

  **Assist LLM** : Gemini

## HightLight-Code
  **1. MLlama Architecture 및 4-bit Quantization Model Load**

  11B 파라미터의 무거운 비전 모델을 T4 GPU에서 구동하기 위해 `MllamaForConditionalGeneration` 클래스와 4-bit Quantization 설정을 결합하여 로드하였다.

  ```python
  bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_use_double_quant=True,
    bnb_4bit_compute_dtype=torch.float16
  )
  
  model = MllamaForConditionalGeneration.from_pretrained(
      model_id,
      quantization_config=bnb_config,
      device_map="auto",
      torch_dtype=torch.float16
  )
  processor = AutoProcessor.from_pretrained(model_id)
  ```

  **2. MultiModal Prompt Template Composed**

  텍스트만 입력받던 기존 LLM과 달리, 이미지와 텍스트를 명시적으로 구분하여 Processor에 전달한다.

  ```python
  messages = [
    {"role": "user", "content": [
        {"type": "image"},
        {"type": "text", "text": query} # 영어 또는 번역된 한국어 쿼리
    ]}
  ]
  input_text = processor.apply_chat_template(messages, add_generation_prompt=True)
  ```

## Performance Results

  총 10개의 테스트 샘플 이미지에 대한 English Query와 Korean Query를 각각 입력하여 Inference Accuracy를 비교하였다.

  **English** : 80% (차타의 수치, 범례, 색상, 구조를 명확히 이해하고 연산을 수행하였다.)

  **Korean** : 50% (기본적인 차트 이해는 가능하나, 특정 한국어 표현이나 복합 추론에서 영문 대비 성능 하락이 발생하였다)


## TroubleShooting
  **(Problem 1) 11B MultiModal 모델의 OOM 발생**

  **(현상)** : Vision Model 특성상 모델 가중치 뿐만 아니라 이미지를 프로세싱하는 과정에서 VRAM 16GB을 가진 T4 GPU에서는 메모리가 쉽게 고갈된다

  **(해결)** : BitsAndBtyes를 통한 4-bit Quantization Load를 필수적으로 적용하고, 연산 데이터 타입을 torch.float16으로 고정하여 안정적으로 추론 환경을 확보하였다.

  **(Problem 2)** : Llama-3.2-Vision 규격에 맞추어 message 리스트 내 딕셔너리 구조를 세분화하여, 토크나이저가 이미지 텐서와 텍스트 토큰을 올바르게 매핑하도록 수정하였다.
  
