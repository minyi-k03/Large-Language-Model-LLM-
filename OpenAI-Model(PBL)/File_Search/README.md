# OpenAI Responses API - File Search (RAG) 구현

## Project - Overview
  **Purpose** : OpenAI 최신 Responses API를 활용하여, 모델이 기본적으로 알지 못하는 최신 정보나, 내부 문서를 기반으로 답변을 생성하게 만드는 능력 File Search(with RAG:Retrieval-Augmentation-Generation) 기능을 구현하는 실습을 진행하였다.

  **Dataset/File** : OpenAI Deep Research와 관련 PDF 문서

## Tech - Stack
  **Language** : Python

  **FrameWork** : OpenAI API, Requests, BytesIO

  **Base Model** : GPT-4o-mini

  **Techniques** : Vector Store 구성, File Search Tool, RAG(Retrieval-Augmented-Generation)

  **Assist LLM** : Gemini

## HighLight - Code
  **1. 파일 업로드 및 Vector Store 구성**

  사용자의 문서를 OpenAI 서버에 업로드 하고, 이를 검색 가능한 형태(Vector)로 Embedding 하여 지식 데이터베이스(Vector_Store)를 구축한다.

  ```python
  # Vector Store 생성 및 파일 매핑
  vector_store = client.vector_stores.create(name="knowledge_base")
  client.vector_stores.files.create(
      vector_store_id=vector_store.id,
      file_id=file_id
  )
  ```

  **2. File Search 도구를 활용한 추론(RAG)**
  
    GPT-4o-mini는 OpenAI Deep Research에 대한 최신 지식이 없어 답변하지 못하지만, tools 파라미터, file_search를 지정하면 구축된         Vector Store를 탐색하여 정확한 내요을 도출해낸다.

  ```python
  response = client.responses.create(
    model="gpt-4o-mini",
    input="OpenAI의 deep research가 뭐야?",
    tools=[{
        "type": "file_search",
        "vector_store_ids": [vector_store.id]
    }]
  )
  ```

  **3. 검색 과정 트래킹**

  include = ["file_search_call.results] 파라미터를 추가하여, LLM이 실제로 어떤 문서를 참고했고, 검색된 텍스트 청크(Text-Chunk)의 연관도 점수가 몇 점인지 모델의 내부 탐색 과저을 상세히 모니터링 한다.


## TroubleShooting
  **(Problem 1) 외부 웹 URL 파일의 다이렉트 업로드 처리 방식 개선**

  **(현상)** : OpenAI API의 파일 업로드 기능은 기본적으로 로컬 스토리지에 저장된 파일만 지원하므로, 웹에 있는 문서를 처리하려면 매번 디스크에 다운로드해야 하는 I/O 병목현상 발생

  **(해결)** : requests라이브러리와 io.BytesIO를 결합하여, 메모리 상에서 파일 객체(Byte Stream)을 직접 생성하고 (file_name,file_content) 튜플 형태로 서버에 전송하는 create_file 커스텀 함수를 구축한다. 이를 통해 불필요한 로컬 저장 과정 없이 URL만으로 즉시 Vector Store를 구성하는 방식으로 해결하였다.

  **(Problem 2) 복잡한 API 응답 객체 파싱 및 가독성 문제**

  **(현상)** : API 호출 결과로 반환되는 response 객체가 깊게 중첩된 트리 형태라, File Search 수행 시 검색된 텍스트 내용이나 연관도 점수를 직관적으로 디버깅하기 어렵다

  **(해결)** : Python의 hasattr()함수를 활용하여 응답 객체 내에 queries나 results 속성이 존재하는지 동적으로 안전하게 검사하고, ㅔ이터를 시각적으로 보기 좋게 포맷팅 하여 출력하는 pretty_print_response커스텀 함수를 작성하여 해결하였다.
  
