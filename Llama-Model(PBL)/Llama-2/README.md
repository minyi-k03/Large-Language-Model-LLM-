# Llama-2 KorQuad Fine-Tuning (with AutoTrain Advanced)

## Project-Overview
  **Purpose** : Hugging Face의 AutoTrain Advanced를 활용하여 Llama-2(7B) 모델을 한국어 질의응답 데이터 셋(KorQuad)에 맞게끔 Fine-Tuning실습을 진행하였다.
  
  **Dataset** : KorQuad, DataAugmentation 기법이 적용된 KorQuad_Prompt_da 사용


## Tech-Stack
 **Langague** : Python

 **FrameWork** : HuggingFace(Transformers, AutoTrain Advanced, PEFT, Bitsandbytes)

 **Base Model** : `TinyPixel/Llama-2-7B-bf16-sharded`

 **Assist LLM** : Gemini

## HighLight-Code
  **1. AutoTrain CLI 기반 학습 시작**

  코드 작성 없이 터미널 명령어(CLI) 만으로 LLM Fine-Tuning을 수행하였다. INT4 양자화 PEFT 옵션을 활성화하여 GPU 메모리 사용량을 최소화하였다.
  ```bash
  autotrain llm --train \
    --project-name "llama2-korquad-finetuning-da" \
    --model "TinyPixel/Llama-2-7B-bf16-sharded" \
    --data-path "korquad_prompt_da" \
    --text-column "text" \
    --peft \
    --quantization "int4" \
    --lr 1e-4 \
    --batch-size 8 \
    --epochs 10 \
    --trainer sft \
    --model_max_length 256
  ```

  **2. 베이스 모델과 학습된 Adapter 병합 및 추론**

  학습이 완료된 후, 원본 Llama-2 모델을 로드하고 Fine-Tuning된 가중치(Adapter)를 병합하여 추론을 진행하였다.

  ```python
  from peft import PeftModel, PeftConfig
  from transformers import AutoModelForCausalLM, AutoTokenizer
  
  # 베이스 모델 로드
  base_model = AutoModelForCausalLM.from_pretrained(
      "TinyPixel/Llama-2-7B-bf16-sharded",
      return_dict=True,
      torch_dtype=torch.float16,
      device_map='auto'
  )
  ```

## TroubleShooting
  **(Problem 1) 패키지 의존성 충돌 및 버전 오류**
  
  **(원인)** : AutoTrain은 최신 버전의 Transformer와 PEFT에 의존하나, 런타임에 구버전이 같이 섞여있다.

  **(해결)** : 강제 uninstall을 통해 기존 라이브러리를 완전히 제거한 후, 필수 라이브러리들을 모두 최신 버전으로 재설치하여 런타임 환경을 초기화 하였다

 **(Problem 2) 과도한 Epoch및 높은 Learning Rate로 인한 문제 발생**

 **(현상)** : Llama-2-7B Model을 KorQuad 데이터 셋에 약 220개 일부를 가지고 Fine-Tuning 실습중 15 Epoch 지점부터 Loss값이 비정상적으로 변하였다, grad_norm 값이 Nan으로 뜨며 Loss값은 0.0이었다.

 **(원인)** : Epoch = 40, Learning_Rate = 2e-4 였다.

 **(해결)** : 기존 Epoch를 40 -> 10으로 변경하여 데이터 셋에 대한 규모를 생각하여 Epoch 횟수를 줄이고, Learning Rate를 2e4 -> 1e-4로 줄여서 Gradient에 대한 보폭을 줄여서 수렴 안정성을 확보하여 해결하였다.

 **(Problem 3) AutoTrain 훈련 중 GPU 라이브러리 경로 인식 실패**

 **(원인)** : Colab 또는 특정 Linux 환경에서 NVIDIA 드라이버 라이브러리 경로가 기본 환경 변수(LD_LIBRARY_PATH)에 매핑되어 있지 않았다.

 **(해결)** : AutoTrain 명령어 실행 직전에 LD_LIBRARY_PATH=/usr/lib64-nvidia:/usr/local/cuda/lib64:$LD_LIBRARY_PATH를 강제로 주입하여 시스템이 GPU 라이브러리를 정상적으로 참조하도록 해결하였다.
 

 
 
