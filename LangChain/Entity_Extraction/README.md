# Entity Extraction

## Project - Overview
 **Purpose** : 임의의 문장 데이터에서 인물 , 반려동물, 특징 등 특정 정보를 추출하여 시스템이 처리 가능한 JSON/Pydantic 객체로 변환하는 실습을 진행하였다.

 ## Tech - Stack
  **FrameWork** : Langchain(Classic)

  **Assist LLM** : Gemini

  **Language** : Python

  **Libraries** : langchain-core, pydantic


## HighLight - Code
 **Pydantic을 활용한 스키마 정의 및 추출**
 ```python
 from langchain_core.pydantic_v1 import BaseModel, Field
 from typing import List
 
 # 추출할 데이터의 타입과 제약 조건 정의
 class Person(BaseModel):
     name: str = Field(..., description="The name of the person")
     hair_color: str = Field(..., description="The color of the person's hair")
 
 # 최신 방식인 .with_structured_output() 활용
 structured_llm = llm.with_structured_output(Person)
 result = structured_llm.invoke("Alan Smith has blond hair.")
 ```

## TroubleShooting
 **(Problem 1) Legacy to Moder**

 **(해결)** : 기존에 사용된 create_extraction_chain 방식에서 .with_structured_output()함수 코드를 이용하여 해결하였다.

 **(Problem 2) Prompt Engineering via Pydantic**

 **(해결)** : Pydantic 필드의 description 속성을 상세하게 작성하여 별도의 프롬프트 튜닝 없이 스키마의 정의만으로 LLM Entity Extraction 정확도를 높였다.


 

  
