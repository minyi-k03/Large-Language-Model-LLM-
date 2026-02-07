# LangChain_RAG

## Project - Overview
  **Purpose** : 임의의 PDF(Llamam-1) 논문을 이용하여 사용자의 질문에 관련된 문서를 검색하고 해당 PDF 파일을 기반으로 답변을 생성하는   RAG(Retrieval-Augmented Generation) 실습을 진행하였다.

## Tech - Stack
  **FrameWork** : LangChain(Classic)

  **LLM Model** : OpenAI GPT-40-mini

  **Vector DB** : Chroma DB

  **Assist LLM** : Gemini

  **Embedding** : OpenAI Embedding(text-embedding-3-small)

  **Libraries** : langchain-openai. langchain-community, langchain-chroma, pypdf, langchain-text-splitters


## HightLight - Code
  **전체 문서 인덱식**
  ```python
 from langchain_community.document_loaders import PyPDFLoader
 from langchain_text_splitters import RecursiveCharacterTextSplitter
 from langchain_chroma import Chroma
 from langchain_openai import OpenAIEmbeddings
 
 # PDF 로드 및 텍스트 분할
 loader = PyPDFLoader("./llama1_paper.pdf")
 docs = loader.load()
 text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
 splits = text_splitter.split_documents(docs)
 
 # 벡터 스토어 생성 및 리트리버 설정
 vectorstore = Chroma.from_documents(documents=splits, embedding=OpenAIEmbeddings())
 retriever = vectorstore.as_retriever()
 ```

**LCEL기반 RAG 파이프라인**
```python
 from langchain_core.runnables import RunnablePassthrough
 from langchain_core.output_parsers import StrOutputParser
 
 # 검색된 문서들을 하나의 문자열로 결합하는 함수
 def format_docs(docs):
     return "\n\n".join(doc.page_content for doc in docs)
 
  # RAG 체인 정의
  rag_chain = (
      {"context": retriever | format_docs, "question": RunnablePassthrough()}
      | rag_prompt_custom
      | llm
      | StrOutputParser()
  )
 ```

