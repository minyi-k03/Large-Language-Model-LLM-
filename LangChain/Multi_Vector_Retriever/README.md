# LangCahin_Multi_Vector_Retriever 

## Project - Overview
  **Purpose** : 대규모 문서를 통째로 벡터화 할때 발생하는 검색 성능 저하를 해결하기 위해 Summary 형태로 검색하고 원본 본문을 반환하는 실습을 진행하였다. 


## Tech - Stack
  **FrameWork**: Langchain(classic)
  
  **Language**: python
  
  **LLM** : OpenAI GPT-4o-mini

  **Assist LLM** : Gemini

  **Vector DB**: Chroma DB

  **Doc Store** : InMemoryStore(부모 청크 저장용)

  **Libraries** : langchain-openai, langchain-community, uuid, tiktoken


## HightLight - Code
  **1. 부모청크, 자식청크를 이어주는 MultiVectorRetriever 설정**
  ```python
  from langchain.retrievers.multi_vector import MultiVectorRetriever
  from langchain.storage import InMemoryStore
  
  vectorstore = Chroma(collection_name="summaries", embedding_function=OpenAIEmbeddings())
  store = InMemoryStore()
  id_key = "doc_id"
  
  retriever = MultiVectorRetriever(
      vectorstore=vectorstore,
      docstore=store,
      id_key=id_key,
  )
  ```

  **2. UUID를 활용한 데이터 무결성 인덱싱**
  ```python
  import uuid
  
  # 부모 문서마다 고유 ID 생성
  doc_ids = [str(uuid.uuid4()) for _ in docs]
  
  # 자식 청크(요약본) 메타데이터에 부모 ID를 매핑하여 저장
  summary_docs = [
      Document(page_content=s, metadata={id_key: doc_ids[i]})
      for i, s in enumerate(summaries)
  ]
  
  # 벡터 DB에는 요약본을, 스토어에는 원본 본문을 각각 저장
  retriever.vectorstore.add_documents(summary_docs)
  retriever.docstore.mset(list(zip(doc_ids, docs)))
  ```

## Trouble - Shooting
  **(Problem1) 패키지 경로 및 모율 임포트 에러**

  **문제** : 표준 langchain경로의 MultiVectorRetriever를 불러올 때 모듈을 찾을 수 없다고 에러 발생

  **해결** : langchain_classic.retrievers 경로로 직접 임포트 하여 해결하였다.


