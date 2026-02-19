# LLM Guardrails & Moderation System

## Project - Overview
  **Purpose** : LLM 기반 안전성과 신뢰성을 보장하기 위해, 사용자 입력, 출력을 제어하는 Guardrails 및 Moderation system 실습을 진행하였다.

  
## Tech - Stack
  **Language** : Python

  **FrameWork** : Asyncio(비동기 처리)

  **Techniques** : Input/Output Moderation, Concurrent Task Execution


## HighLight - Code
  **1. Async 기반 Guardrail 검사**

  LLM 응답 속도 저하를 방지하기 위해, User Query를 LLM에 전달하는 동시에 Guardrail을 검사하는 로직을 Asyncio를 활용하여 병렬로 실행하였다.

  ```python
  # LLM 답변 생성과 가드레일 검사를 동시에 비동기로 실행
  chat_task = asyncio.create_task(generate_chat_response(request))
  guardrail_task = asyncio.create_task(check_topical_guardrail(request))
  
  while not (chat_task.done() and guardrail_task.done()):
      if guardrail_task.done():
          if guardrail_task.result() == False:
              # 주제를 벗어난 경우 즉시 방어 메시지 반환
              return "죄송하지만, 동물 품종에 대한 조언은 제공해 드릴 수 없습니다. 일반적인 질문이라면 언제든지 도와드릴 수 있어요."
      await asyncio.sleep(0.1)
  ```

  **2. Output Moderation**

  LLM이 정상적으로 답변을 생성했더라도, 최종 사용자에게 전달하기 직전에 결과물의 유해성 및 정책 위반 여부를 검사하는 단계

  ```python
  chat_response = chat_task.result()
  # 생성된 답변(Output)에 대해 커스텀 모더레이션 진행
  output_moderation_response = await custom_moderation_async(chat_response, parameters)
  
  # 정책 위반 감지 시 안전한 답변으로 대체 (Fallback)
  if output_moderation_response == True:
      print("Moderation flagged for LLM response.")
      return "죄송하지만, 이 질문에는 답변해드릴 수 없습니다. 일반적인 문의 사항이라면 도와드릴 수 있습니다."
  
  return chat_response
  ```


  
