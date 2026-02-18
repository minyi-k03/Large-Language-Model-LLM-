# Llama-3.1-8B Multilingual Performance Text (MGSM)

## Project-Overview
  **Purpose** : Meta-Llama-3.1-8B-Instruct 모델을 대상으로 MGSM 벤치마크 데이터를 활용하여 다국어 논리 추론 능력을 평가하고, 이전 버전 및 타 상용 LLM 성능들과 비교 분석하는 실습을 진행하였다.

  **DataSet** : MGSM

  **Language Tested** : 한국어, 영어, 일본어, 중국어, 태국어, 벵골어, 독일어, 스페인어, 프랑스어, 러시아어, 스와힐리어, 텔루구어

  
## HighLight-Code
  **1. 메모리 최적화 모델 로드**

  제한된 GPU 환경(T4, 16GB)에서 8B 모델을 원활하게 테스트 하기 위해 BitsAndBytes를 활용하여 4-bit Quantization Load를 수행하였다.

  ```python
  from transformers import BitsAndBytesConfig

  bnb_config = BitsAndBytesConfig(
      load_in_4bit=True,
      bnb_4bit_quant_type="nf4",
      bnb_4bit_use_double_quant=True,
      bnb_4bit_compute_dtype=torch.float16  # T4 환경에 맞춘 float16 설정
  )
  
  llama3_1_model = AutoModelForCausalLM.from_pretrained(
      "meta-llama/Meta-Llama-3.1-8B-Instruct",
      quantization_config=bnb_config,
      device_map="auto",
  )
  ```

## Model Comparison Result

  Llama-3.1 다국어 능력 향상: 정답률 자체는 8/12로 이전 버전인 Llama-3와 동일했으나, Llama-3가 질문 언어와 무관하게 영어로 답변을 생성하던 문제를 극복하고 입력된 언어에 맞춰 자연스럽게 답변을 생성하는 개선된 다국어 지원 능력을 보여주었다.

  **모델 별 순위**

  1. GPT-4o (12/12)

  2. GPT-4o-mini (11/12) and Gemini (11/12)

  3. Llama-3.1-8B-Instruct(8/12) and Llama-3-8B-Instruct (8/12)

  4. Clova X (3/12)


  
