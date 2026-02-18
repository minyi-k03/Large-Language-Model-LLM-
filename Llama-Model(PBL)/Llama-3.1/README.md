# Llama-3.1 Local Setting

## 1. 기존 실습 영상에서는 고사양 GPU (A100)을 가지고 실습하였지만, 금전적 한계로 인하여 기존 Colab에서 제공하는 Tesla T4 GPU를 이용하여 실습 진행한다.
  **1-1**: Gemini 도움을 받아 4-bit Quantization 기법을 적용하여 메모리 부족 문제를 해결하였다

  **1-2** : Transformers, bitsandbytes 라이브러리를 활용하여 LLM 추론을 실습하였다.

## 2. 환경, 기술 스택

  **문제점** : Llama-3.1-8B 모델을 표준으로 16bit로 로드할 경우 가중치만으로 약 16GB VRAM 이 소모되며, 현재 가진 GPU자원으로는 한계가 있다.

  **해결책** : BitsAndBytesConfig를 사용하여 4-bit씩 묶어서 양자화 적용

  **결과** : 모델 메모리 점유율을 약 5.5GB 수준으로 절감하고, 남은 VRAM공간을 추론 연산에 사용하였다.

