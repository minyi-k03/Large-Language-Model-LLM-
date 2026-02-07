# LangChain_Embedding_Vector_Stores

## Project - Overview
  **Purpose** : 임의의 텍스트 데이터를 컴퓨터가 이해할 수 있게끔 임베딩 작업을 진행하고 변환된 데이터를 효율적인 저장 및 검색을 위하여 벡트 스토어(Chroma DB)에 넣는 실습을 진행하였다.


## Tech - Stack
  **FrameWork** : LangChain(Classic)

  **Language** : Python

  **Embedding Models** : HuggingFace(jhgan/ko-sroberta-multitask)

  **OpenAI** : text-embedding-3-small

  **Assist LLM** : Gemini

  **Vector DB** : Chroma DB

  **Libraries** : langchain-huggingface, langchain-openai, langchain-chroma, sentence-tranformers


## HightLight - Code
  **HuggingFace vs OpenAI Embedding Code 비교**
  ```python
  # 1. 오픈소스 모델 (HuggingFace) 활용
  from langchain_huggingface import HuggingFaceEmbeddings
  hf_embeddings = HuggingFaceEmbeddings(model_name="jhgan/ko-sroberta-multitask")
  db_hf = Chroma.from_texts(texts, hf_embeddings)
  
  # 2. 상용 API (OpenAI) 활용
  from langchain_openai import OpenAIEmbeddings
  oa_embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
  db_oa = Chroma.from_texts(texts, oa_embeddings)
  ```
  
