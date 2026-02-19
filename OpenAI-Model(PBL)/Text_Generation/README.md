# OpenAI Text Generation API Deep Dive

## Project - Overview
  **Purpose** : OpenAI Text Generation API가 제공하는 핵심 파라미터(seed, temperature, penalty 등)을 조작하여 LLM의 생성 패턴을 제어하는 방법을 익히고, tiktoken을 활용한 토큰 관리 및 JSON Mode를 통한 구조화된 출력 실습을 진행하였다.


## Tech - Stack
  **Language** : Python

  **FrameWork** : OpenAI API(Async Clinet), Tiktoken

  **Base Model** : GPT-4o-mini, GPT-3.5-Turbo

  **Techniques** : Prompt-Engineering, HyperParameter-Tuning, Asynchronous Programming


## HighLight
  **1. 결정론적 결과 생성(Seed Parameter)**

  LLM의 무작위성을 통제하기 위해 seed 파라미터를 고정하여, 동일한 프롬프트에 대해 일관된 답변을 생성할 수 있는지 difflib을 통해 시각적으로 검증하였다.

  **2. 토큰 관리 및 비용 예측**

  API 요청 존 tiktoken 라이브러리를 사용하여 메세지의 토큰 수를 미리 계산(num_token_from_messages)와, 실제 API 응답의 Usage 데이터와 비교하여 정확한 비용 산정 로직을 구현하였다.

  **3. HyperParameter Tuning을 통한 생성 제어**

  - **3.1 Temperature** : 0.0(사실적/반복적) ~ 2.0(창의적/무작위)범위 조절을 통해 답변의 다양성을 체크하였다.

  - **3.2 JSON Mode** : 시스템 프롬프트와 response_format 설정을 통해 출력을 JSON 객체로 강제하여 파싱 용이성을 확보하였다.

  - **3.3 Penalty(Frequency/Presence)** : 단어의 재등장 빈도에 패널티를 부여하여 문장의 반복을 줄이거나(양수), 같은 단어를 반복(음수)하게 유도하였다.

## TroubleShooting
  **(Problem 1)Async 클라이언트 호출 시 코루틴 실행 불가**
  
  **(현상)** : AsyncOpenAI 클라이언트를 사용했으나 응답값이 오지 않고 코르틴 객체만 반환되는 현상 발생

  **(원인)** : Python의 Async 함수 내에서 API 호출 시 awit 키워드를 누락하면 비동기 작업이 스케쥴링이 되지 않는다

  **(해결)** : API 호출부를 response = await client.chat.completions.create(...)형태로 수정하여 해결하였다.

  **(Problem 2) : 최신 SDK 에서의 Response 데이터 접근 오류**

  **(현상)** : API응답에서 토큰 사용량(Usage)를 가져올 때 TypeError 등 접근 오류 발생

  **(원인)** : OpenAI SDK가 업데이트되면서 응답 객체가 딕셔너리가 아닌 Pydantic 모델로 변경되었다

  **(해결)** : 기존 딕셔너리 접근법 response['usage'] -> 객체 속성 접근법(response.usage.prompt_tokens)형태로 변경하여 해결하였다.
  
  
  
