# LangCahin Self-Query Retriever (with Hotel Search)

## Project-Overview
  **Purpose**: Langchain의 SelfQueryRetriever를 활용하여 단순 검색이 아닌 MetaData를 필터링이 작동으로 적용되게끔 실습해본것이 목표

## Tech Stack
  **Langauge**:Python
  
  **FrameWork**:Langchain(Core,Chroma,OpenAI)
  
  **LLM**:Gpt-4o-mini
  
  **Assist LLM**:Gemini

  **Vector Store**:ChromaDB

  **Embedding**:OpenAI Embeddings



## Main Logic
  '''
  **메타데이터 필드 정의 예시**
  
  metadata_field_info = [
  
    AttributeInfo(name="city", description="호텔이 위치한 도시", type="string"),
    
    AttributeInfo(name="price", description="1박당 숙박 비용", type="integer"),
    
    AttributeInfo(name="amenities", description="수영장, 조식 등 편의시설 목록", type="list"),
    
  ]

  **리트리버 구성**
  
  retriever = SelfQueryRetriever.from_llm(
  
      llm=llm,
      
      vectorstore=vectorstore,
      
      document_contents="호텔 정보 및 리뷰 데이터셋",
      
      metadata_field_info=metadata_field_info,
      
      verbose=True
  )


## TroubleShooting
  **문제1**: 라이브러리 경로 및 마이그레이션 오류
  
  **현상**: 기존 실습 환경에서는 langchain.retriever 경로를 찾지 못하거나 inoke()매서드 호환성 문제 발생
  
  **해결**:langchain-classic 패키지를 이용하여 기존 인터페이스를 유지하면서도 최신 LLM을 사용할 수 있도록 구조 변경


  **문제2**: 쿼리 변환 정확도 떨어짐

  **현상**: 20만원 이하와 같이 수치 조건이 문자열로 인식되는 경우 필터링이 실패하는 경우 발생

  **해결**:AttributeInfo 정의 시 type = "integer"로 명시 후 프롬프트에 데이터 타입을 확실하게 명시하여 인지시키는 가이드를 추가하였다
  

  
  
