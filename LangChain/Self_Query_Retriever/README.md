# LangChain_Self_Query_Retriever(Hotel Search)

## Project - Overview
 **Purpose** : 단순한 의미 기반 검색이 아닌 사용자의 자연어 질문에서 필터링 조건을 추출하여 메타데이터 필터링을 자동 수행한 SelfQueryRetriever 실습을 진행하였다. 해당 실습을 통해 호텔 검색시 가격, 도시, 편의시설 등 세부 조건을 반영하는 시스템을 실습하였다.


## Tech - Stack
 **FrameWork** : LangCahin(Classic)

 **Language** : Python

 **LLM** : OpenAI GPT-4o-mini

 **Assist LLM** : Gemini

 **Vector DB** : ChromaDB

 **Embedding** : OpenAI Embeddings

 **Libraries** : langchain-chroma, lark

## HightLight - Code
 **메타데이터 필드 정의**
 ```python
 from langchain.chains.query_constructor.base import AttributeInfo

 # LLM이 필터링할 수 있도록 데이터의 속성을 정의
 metadata_field_info = [
     AttributeInfo(
         name="city", 
         description="The city the hotel is located in", 
         type="string"
     ),
     AttributeInfo(
         name="price", 
         description="The price per night", 
         type="integer"
     ),
     AttributeInfo(
         name="amenities", 
         description="List of amenities like pool, breakfast", 
         type="list"
     ),
 ]
 ```

**Self_Query_Retriver**
 ```python
 from langchain.retrievers.self_query.base import SelfQueryRetriever

 # LLM, 벡터 스토어, 메타데이터 정보를 결합하여 스스로 쿼리를 작성하는 리트리버 생성
 retriever = SelfQueryRetriever.from_llm(
     llm=llm,
     vectorstore=vectorstore,
     document_contents="Brief summary of a hotel review",
     metadata_field_info=metadata_field_info,
     verbose=True  # 내부적으로 생성된 쿼리 확인용
 )
 ```

## Trouble-Shooting
 **(Problem 1) : 라이브러리 경로 및 마이그레이션 오류**

 **(해결)** : langchain-classic 패키지를 활용하여 실행방식을 invoke()함수로 변환하였다.

 **(Problem 2)  : 쿼리 변환 정확도 저하**

 **(해결)** : AttributeInfo의 description을 명확하게 수정하고, verbose = True를 모니터링하며 프롬프트가 메타데이터의 의미를 정확히 이해하도록 변경하였다.
 


  
  
