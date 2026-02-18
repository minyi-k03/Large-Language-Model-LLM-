# Llama-3.1 (8B) Patent Categories Auto Classification

## Project-Overview
  **Purpose** : Meta의 최신 오픈 소스 LLM인 Meta-Llama-3.1-8B-Instruct 모델을 활용하여, 복잡한 특허 문서의 텍스트(발명명칭, 요약, 청구항)울 분석하고 알맞은 산업 카테고리(농업, 임업, 어업)로 자동 분류(Zero-Shot Classification)하는 실습 진행.

  **Dataset** : AI Hub 특허 분야 자동분류 데이터

## Tech-Stack
  **Language** : Python

  **FrameWork** : PyTorch, HuggingFace(Transformers, Accelerate)

  **Base Model** : Meta-Llama-3.1-8B-Instruct

  **Techniques** : 4-bit Quantization(BitsAndBytes), Prompt Engineering

  **Assist LLM** : Gemini


## HighLight-Code
  **1. 특허 데이터 정제 및 병합**

  JSON 형태의 원천 데이터에서 모델 분류에 필요한 핵심 필드만 추출하여 하나의 프롬프트 문자열 결합한다. 이후 출원번호를 기준으로 정답 데이터와 병합 및 중복을 제거한다.

  ```python
  def extract_fields(row):
    invention_title = row.get('invention_title', '')
    abstract = row.get('abstract', '')
    claims = row.get('claims', '')
    return f"invention_title: {invention_title} abstract: {abstract} claims: {claims}"

  merged_df['combined_string'] = merged_df.apply(extract_fields, axis=1)
  ```

  **2. 전문가 페르소나 및 프롬프트 주입**

  시스템 프롬프트에 특허 카테고리 분류 전문가라는 역할을 부여하고, 분류할 수 있는 카테고리 리스트를 제한하여 모델이 불필요한 말을 생성하지 않고 정확학 카테고리 명칭만 출력하도록 통제하였다.

  ```python
  system_prompt = f"너는 특허 카테고리를 분류하는 전문가야. \
  아래 내용을 다음 특허 카테고리 중 하나로 분류해줘. 가능한 특허 카테고리 : {특허_카테고리_list}. \
  최종 출력 결과는 다른말은 하지말고 분류한 카테고리만 출력해줘."
  ```

## TroubleShooting
  **(Problem 1) T4 GPU에서 OOM및 데이터 타입 호환성 문제**

  **(현상)** : 8B 크기의 Llama-3.1 모델을 Colab 무료 버전 T4 GPU 16GB VRAM에 로드시 메모리가 부족하여 커널이 다운된다, 또한 기존 Llama-3 코드에 있던 bfloat16 데이터 타입이 T4 Architecture에서 완벽히 호환되지 않았다.

  **(해결)** : BitsAndBytesConfig를 적용하여 모델을 4-bit Quantization하여 OOM 문제를 해결하였다. 추가로 bnb_4bit_compute_dtype 및 모델 로드시의 torch_dtype를 T4와 완벽히 호환되는 torch.float16으로 명시적으로 수정하여 해결하였다.

  **(Problem 2) Tokenizer Padding Token 누락 에러**

  **(현상)** : 데이터 전처리 및 생성 단계에서 모델 토크나이저에 pad_token이 설정되지 않아 문제 발생

  **(해결)** : tokenizer.pad_token = tokenizer.eos_token 코드를 삽입하여 문장 종료 토큰(EOS)을 패딩 토큰으로 사용하도록 예외처리하여 해결하였다.

  
  
  
  
