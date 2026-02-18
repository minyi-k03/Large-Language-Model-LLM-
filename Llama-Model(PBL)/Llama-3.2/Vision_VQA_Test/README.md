# Llama-3.2-Vision-11B Korean VQA Performance Test

## Project-Overview
  **Purpose** : Meta-Llama3.2-11B-Vision-Instruct를 활용하여, 이미지 내의 객체, 색상, 상황을 인지하고 한국어 질문에 답변하는 VQA 추론 성능을 테스트 하는 실습을 진행하였다.

  **DataSet** : AI Hub 시각정보 기반 질의응답 데이터 셋 일부

## Tech-Stack
  **Language** : Python

  **FrameWork** : PyTorch, HuggingFace(Transformers, BitsAndBytes)

  **Model** : Meta-Llama3.2-11B-Vision-Instruct

  **Techniques** : 4-bit Quantization(NF4), Multimodal Prompt Engineering

  **Assist LLM** : Gemini

## HighLight-Code
  **1. Vision 모델 로드 및 메모리 최적화**

  11B 파라미터의 거대 멀티모달 모델을 T4 GPU 환경에서 구동하기 위해 `MllamaForConditionalGeneration` 아키텍쳐와 4-bit Quantization을   적용하였다
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
  ```

**2. Image-Text 복합 프롬프트 추론 함수**

이미지 데이터(PIL.Image)와 Korean Query를 Llama-3.2 Vision 규격에 맞게끔 결합하여 텍스트를 생성하는 모듈 구성

```python
def get_llama_response(image_path, query):
    image = Image.open(image_path).convert('RGB')
    messages = [
        {"role": "user", "content": [
            {"type": "image"},
            {"type": "text", "text": query}
        ]}
    ]
    input_text = processor.apply_chat_template(messages, add_generation_prompt=True)
    inputs = processor(image, input_text, add_special_tokens=False, return_tensors="pt").to(model.device)
    
    output = model.generate(**inputs, max_new_tokens=100, do_sample=False)
    return processor.decode(output[0]).split("<|start_header_id|>assistant<|end_header_id|>")[1].strip()

  processor = AutoProcessor.from_pretrained(model_id)
  ```

## TroubleShooting
  **(Problem 1) T4 GPU 환경에서 11B MultiModal 모델 OOM 현상**

  **(현상)** : 모델 가중치와 이미지 프로세싱 텐서를 동시에 GPU에 올릴 때 OOM 발생

  **(해결)** : BitsAndBytes를 활용하여 4-bit로 경량화 하여 로드하고, torch.float16연산으로 고정하여 성능 손실 없이 16GB VRAM내에서 구동 하도록 해결하였다.

  **(Problem 2) 텍스트 전용 프롬프트 방식 적용 시 발생한 텐서 병합 에러**

  **(현상)** : 단일 텍스트 문자열에 이미지를 전달하려 할 때 텐서 형태가 일치하지 않아서 모델이 추론을 거부하였다.

  **(해결)** : message 딕셔너리의 content 블록 내에 image타입과 text 타입을 분리하는 Mllama 전용 탬플릿 구조를 적용하고,   Processor.apply_chat_template를 활용하여 입력 텐서를 규격화 하여 해결하였다.
