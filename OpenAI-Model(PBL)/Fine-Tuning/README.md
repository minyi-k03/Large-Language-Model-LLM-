# OpenAI API Based GPT-3.5 Turbo Fine-Tuning

## Project - Overview
  **Purpose** : OpenAI의 공식 APUI를 활용하여 클라우드 환경에서 GPT-3.5-Turbo 모델을 특정 페르소나에 맞게 Fine-Tuning하고 API를 통한 추론 및 작업 관리 방법에 대한 실습을 진행하였다.

  **Dataset** : Mydata.jsonl

## Tech-Stack
  **Language** : Python

  **FrameWork/API** : OpenAI Python SDK

  **Base Model** : GPT-3.5-Turbo

  **Assist LLM** : Gemini

  
## HighLight - Code
  **1. 파인튜닝 데이터셋 안전한 업로드**

  학습 데이터인 JSONL 파일에 OpenAI 서버로 업로드합니다. 이때 파일 누락 방지를 위해 Gemini를 이용하여 try-except 예외 처리를 적용하여 안전성을 높였습니다.

  ```python
  import os
  from openai import OpenAI
  
  client = OpenAI(api_key="YOUR_API_KEY")
  
  try:
      response = client.files.create(
          file=open("mydata.jsonl", "rb"),
          purpose='fine-tune'
      )
      print("업로드 성공! 파일 ID:", response.id)
  except FileNotFoundError:
      print("오류: 'mydata.jsonl' 파일을 찾을 수 없습니다.")
  ```

  **2. Fine-Tuning 작업 생성 및 모델 추론**

  생성된 모델에 다양한 시스템 프롬프트를 주입하여, 학습된 페르소나가 정상적으로 발현되는지 테스트한다.

  ```python
  # 1. 파인튜닝 작업 시작
  job = client.fine_tuning.jobs.create(
      training_file="uploaded_file_id",
      model="gpt-3.5-turbo"
  )
  
  # 2. 페르소나 부여 테스트 (Sarcastic Chatbot)
  completion = client.chat.completions.create(
    model="ft:gpt-3.5-turbo-xxxx",
    messages=[
      {"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."},
      {"role": "user", "content": "What's the capital of France?"}
    ]
  )
  print(completion.choices[0].message.content)
  ```

## TroubleShooting
  **(Problem 1) Fine-Tuning 작업 ID 분실 및 상태 확인 불가 이슈**

  **(현상)** : OpenAI API를 통한 Fine-Tuning을 클라우드 백그라운드에서 진행되므로, 현재 학습이 완료되었는지 또는 실패했는지에 대한 파악이 어려움

  **(해결)** : client.fine_tuning.jobs.list(limit=10)메서를 활용하여 최근 요청된 파인튜닝 작업리스트를 불러오고, job.id, job.status, job.model 값을 파싱하여 실시간으로 상태를 모니터링 하는 형태로 해결하였다.


  
