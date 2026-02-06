# LangChain_Query_Analysis_Multiple_Retrievers

## Project - Overview
  **Purpose**:사용자의 질문 내요에 따라 서로 다른 데이터 소스를 선택하여 답변을 생성하는 실습을 진행하였다.

## Tech - Stack
  **FrameWork**:LangChain(Classic)

  **LLM**:OpenAI GPT-4o-mini

  **Assist LLM**:Gemini

  **Vector DB**:Chroma DB

  **Libraries**:Langchain-openai, Langchain-chroma, Pydantic


  
## HightLight - Code

  **독립적인 데이터 소스**
  ```python
  # Harrison 데이터용 리트리버
  vectorstore_harrison = Chroma.from_texts(
      ["Harrison worked at Kensho"], 
      embeddings, 
      collection_name="harrison"
  )
  retriever_harrison = vectorstore_harrison.as_retriever(search_kwargs={"k": 1})
  
  # Ankush 데이터용 리트리버
  vectorstore_ankush = Chroma.from_texts(
      ["Ankush worked at Facebook"], 
      embeddings, 
      collection_name="ankush"
  )
  retriever_ankush = vectorstore_ankush.as_retriever(search_kwargs={"k": 1})
```

**라우팅 로직**
```python
from pydantic import BaseModel, Field
from typing import Literal

class RouteQuery(BaseModel):
    """질문을 가장 적절한 데이터 소스로 라우팅하기 위한 스키마"""
    datasource: Literal["harrison", "ankush"] = Field(
        ...,
        description="질문에 따라 'harrison' 또는 'ankush' 중 하나를 선택함"
    )
    query: str = Field(..., description="검색에 최적화된 키워드")

```

## TroubleShooting
  **Pydantic 버전 호환성 문제**
  **해결**: langchain_core.pydantic_v1 대신 pydantic v2 사용

  **리소스 최적화**
  **해결**:OpenAI Embedding 객체를한번만 생성하여 두 개의 Retriever가 공유하도록 최적화하였다


# LangChain_JudgeGpt_Multiple_Retriever

## Project - Overview
  **Purpose** : 방대한 양의 법률 판결문 데이터 셋을 가지고 사용자가 프롬프트를 입력 하였을때 각 법률 도메인에 맞게끔 처리하는 실습 진행

## Tech - Stack
  **FrameWork**:LangChain(Classic)

  **LLM**:OpenAI GPT-4o-mini

  **Assist LLM**:Gemini

  **Vector DB**:Chroma DB

  **Libraries**:Langchain-openai, Langchain-chroma, Pydantic

## HightLight - Code
  **대용량 분할 압축 데이터 처리를 하여 하나로 데이터를 합쳤다**
  ```python
  #분할된 조각들을 하나의 zip 파일로 합침
  !cat TS_1.판결문.zip.part* > TS_1.판결문_combined.zip
  
  #통합된 파일의 압축 해제
  !unzip -q TS_1.판결문_combined.zip -d ./judgement_data
  ```

  **ChromaDB 연동을 위한 SQLite 패치**
  ```python
  import sys
  __import__('pysqlite3')
  sys.modules['sqlite3'] = sys.modules.pop('pysqlite3')
  
  from langchain_chroma import Chroma
  ```


    
