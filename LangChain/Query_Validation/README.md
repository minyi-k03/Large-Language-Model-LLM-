# LangChain_SQL_Query_Validation

## Project - Overview
  **Purpose** : 사용자의 입력 프롬프트를 SQLite 쿼리로 변환하고, 생성된 쿼리에서 문법적 오류, 논리적 실수가 ㅇ벗는지에 대한 검증 및 수정하는 실습을 진행하였다.


## Tech - Stack
  **FrameWork** : LangChain(Classic)

  **LLM** : OpenAI GPT-4o-mini

  **Language** : Python

  **Assist LLM** : Gemini

  **DataBase** : SQLite(Chinook Sample DB)

  **Libraries** : langchain-openai, langchain-community, langchain-core


## HightLight - Core
  **자가 수정 프롬프트**
  ```python
  system_template = """당신은 {dialect} 전문가입니다. 
  쿼리의 초안을 작성하세요. 그런 다음, {dialect} 쿼리에서 흔한 실수를 다시 확인하고 수정하세요.
  아래 형식을 엄격히 준수하세요:
  
  First draft: <<FIRST_DRAFT_QUERY>>
  Final answer: <<FINAL_ANSWER_QUERY>>
  """
  
  # 프롬프트 구조 시각화 확인
  prompt.pretty_print()
  ```

  **최종 SQL 추출을 위한 커스텀 Parser Function**
  ```python
  def parse_final_answer(output: str) -> str:
    try:
        # "Final answer:" 기준으로 자르고 뒷부분 가져오기
        text = output.split("Final answer:")[-1].strip()
        # 마크다운 태그(```sql) 제거
        return text.replace("```sql", "").replace("```", "").strip()
    except IndexError:
        return output

  # 체인 마지막 단계에 파서 연결
  chain = (
      # ... 이전 단계 ...
      | llm
      | StrOutputParser()
      | parse_final_answer 
  )
  ```

## Trouble Shooting
  **(Problem 1)실행 불가능한 마크다운 태그 포함 문제**
  
  **(문제)** : LLM이 SQL로 쿼리 반환시 마크다운 형식을 포함하여 전달하여 db.run() 실행시 문법 오류 발생

  **(해결)** : 당시 Gemini를 이용하여 커스텀 함수 parse_final_answer함수내에서 마크다운 형식을 제거하여 순수 문자열만 추출하여 해결
  
  

   
