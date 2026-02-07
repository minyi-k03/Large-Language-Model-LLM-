# LangChain_Semi-Structured-RAG

## Project - Overview
  **Purpose**: LLaVA 논문에 있는 텍스트와 표가 복잡하게 섞여 있는 PDF문서에서 Text와 Table을 분히라여 효과적으로 검색 답변을 할 수 있도록 Semi-Structured RAG 실습을 진행하였다.


## Tech - Stack
  **FrameWork**:LangChain(Classic)

  **Language**:Python

  **LLM**:OpenAI GPT-4o-mini

  **Assist LLM**:Gemini

  **PDF Parsing** : Unstructured

  **Vector DB** : Chroma DB

  **Libraries** : unstructured[all-docs], langchain-openai, pydantic, lxml


## HightLight - Code
  **복합 데이터 처리를 위한 모델 정의 및 분류**
  ```python
  from pydantic import BaseModel
  from typing import Any
  
  class Element(BaseModel):
      type: str
      text: Any
  
  # 표와 텍스트 분리 로직
  table_elements = [e for e in raw_pdf_elements if e.category == "Table"]
  text_elements = [e for e in raw_pdf_elements if e.category == "CompositeElement"]
  ```

  **표와 텍스트를 동시에 고려하는 RAG Chain**
  ```python
  from langchain_core.runnables import RunnablePassthrough

  # 검색된 컨텍스트(표+텍스트)를 프롬프트에 전달
  chain = (
      {"context": retriever, "question": RunnablePassthrough()}
      | prompt
      | model
      | StrOutputParser()
  )
  ```

## Trouble - Shooting
  **(Problem 1) 라이브러리 경로 인식 불가**

  **(문제)**: MultiVectorRetriever 임포트 시 표준 경로에서 모듈을 찾지 못해서 오류 발생

  **(해결)**: langchain_classic.retrievers import MultiVectorRetriever로 코드 마이그레이션

  **(Problem 2) PDF Chunking 과정에서 Table 데이터 유실**

  **(문제)** : chunk_by_title = True 설정시 Table을 일반 텍스트 덩어리로 강제 흡수하여 Table 데이터가 유실되었다

  **(해결)** : 데이터 무결성을 위해 먼저 Table 데이터를 별도 리스트로 추출하였다. 이후 텍스트에 대해서만 청킹을 수행하였고, 텍스트 청킹후 테이블 데이터와 텍스트 데이터를 합치는 방향으로 해결하였다.

  **(Problem 3) 예상과 다른 추출 데이터 개수**

  **(문제)** : 추출된 텍스트 조각이 예상한 개수와 값이 달랐다

  **(해결)**: combine_text_under_n_chars HyperParameter Tuning을 통해 기존에 2000이었던 임계값을 800으로 조정하여 실습에서 예상한 텍스트 조각 개수를 맞췃다.
  
  
