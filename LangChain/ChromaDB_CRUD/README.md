# LangChain_ChromaDB_CRUD

## Project - Overview
  **Purpose** : Chroma DB를 대상으로 데이터를 삽입, 유사도 검색, 메타데이터 업데이트, 특정 데이터 삭제등의 CRUD 전 과정 실습을 진행하였다.


## Tech - Stack
  **FrameWork** : Langchain(Classic)
  
  **LLM Embedding** : OpenAI(text-embedding-3-small)

  **Assist LLM** : Gemini

  **Vector DB** : Chroma DB

  **Libraries** : langchain-openai, langchain-chroma, langchain-community
  

## HightLight - Code
  **메타데이터 수정을 통한 데이터 갱신**
  ```python
  # 1. 수정할 문서 선택 및 메타데이터 변경
  docs = example_db.get(ids=[ids[0]])
  docs[0].metadata = {
      "source": "../../출산율.pdf",
      "new_value": "hello world",
  }
  
  # 2. ID를 지정하여 해당 문서의 정보 업데이트
  example_db.update_document(ids[0], docs[0])
  ```

  **특정 데이터 삭제**
  ```python
  # 문서 ID를 리스트 형태로 전달하여 삭제 수행
  example_db.delete(ids=[ids[0]])
  
  # 삭제 후 컬렉션에서 해당 데이터가 사라졌는지 확인
  print(example_db._collection.count())
  ```

## Trouble-Shooting
  **(Problem 1)로컬 환경에 맞게끔 import 경로 변경**
  
  **(문제)** : 표준 langchain 패키지 경로에서 Chroma나 다른 관련된 모듈 로드시 경로를 찾지 못하는 문제 발생

  **(해결)** : langchain_chroma나 langchain_classic으로 경로를 변경하여 import 하였다.

  **(Problem 2)Chroma DB 연동을 위한 SQLite 버전 호환성**

  **(문제)** : 로컬 환경에서 Chroma DB 실행시 SQLite3버전이 낮아 작동하지 않는 문제 발생

  **(해결)** : pysqlite3-binary를 설치하여 sys.modules 패치하는 코드를 작성하여 해결하였다.
  
  
