# OpenAI API Function Calling 실습

## Project - Overview
  **Purpose** : OpenAI API의 Function Calling 기능을 활용하여, LLM이 외부 도구를 사용하기 위해 매개변수(Arguments)를 스스로 추론하고 구조화된 JSON 형태로 반환하는 실습을 진행하였다

  
## Tech - Stack
  **Language** : Python

  **FrameWork** : OpenAI API (Rest HTTP Request), Tenacity (Retry Logic)

  **Base Model** : GPT-4o-mini

  **Techniques** : Function Calling, Parallel Tool Use, Prompt-Engineering

  **Assist LLM** : Gemini

## HighLight - Code
  **1. Defined Tools Schema** 

  LLM이 이해할 수 있도록 JSON 형태로 사용할 수 있는 함수들의 이름, 설명, 필수 매개변수들을 정의한다.

  ```python
  tools = [
    {
        "type": "function",
        "function": {
            "name": "get_n_day_weather_forecast",
            "description": "Get an N-day weather forecast",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string", "description": "The city and state"},
                    "format": {"type": "string", "enum": ["celsius", "fahrenheit"]},
                    "num_days": {"type": "integer"}
                },
                "required": ["location", "format", "num_days"]
            }
        }
    }
  ]
  ```
  **2. Tool Choice 및 병렬 호출 제어**

  사용자의 질문과 상황에 따라 모델이 함수를 어떻게 사용할지 통제한다

    1. tool_choice = 특정 함수를 강제로 사용하거나 {"name": " ..."}, 아예 사용하지 못하게 none로 제어 가능

    2. Parallel Calling : 한 번의 프롬프트에 대해 여러개의 함수 호출 값 배열을 동시에 반환하는 기능 검증.


## TroubleShooting
  **(Problem 1) API 통신 불안정 및 TimeOut 에러 방지**

  **(현상)** : OpenAI API 호출 시 간헐적인 네트워크 지연, 502/504 타임아웃 오류로 인해 스크립트가 강제 종료되는 문제 발생 위험

  **(해결)** : OpenAI SDK 내장 매서드를 맹신하지 않고, requests.post 기반의 커스텀 통신 함수 chat_completion_request를 별도로 작성하고 tenacity 라이브러리의 @retry 데코레이터를 결합하여 통신 실패 시 지수 백오프 방식으로 최대 3회 자동 재시도하도록 변경하여 해결하였다.

  **(Problem 2) : Function Calling 시 모델의 임의 값 주입을 통해 Hallucination 방지**

  **(현상)**: 필수 인자 num_days가 누락된 모호한 요청이 들어올 시, 모델이 임의의 숫자를 추측하여 강제로 함수를 호출하는 문제 발생

  **(해결)** : 방어적 시스템 프롬프트를 주입하여, 정보가 부족할 경우 모델이 멋대로 함수를 실행하지 않도록 사용자에게 다시 되묻도록 통제하였다.

  
