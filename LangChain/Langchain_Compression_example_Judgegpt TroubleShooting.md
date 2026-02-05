# Langchain_Compression_example_Judgegpt 

## 1. Project Overview ##
   **Purpose**:AI HUB 사이트에서 판례 관련 데이터 셋을 가지고 가벼운 실습 진행

## 2. Tech Stack
  **FrameWork** : LangChain
  
  **LLM**:OpenAI Gpt-4o-mini
  
  **Support LLM**:Gemini
  
  **Vector DB**:Chroma DB
  
  **Embedding**:OpenAIEmbeddings

## 3. Trouble Shooting
  **3-1**:SQLite3 버전 호환성 문제

  **원인**: Colab버젼과 현재 SQLite 버전 호환성 문제 발생

  **해결**: Gemini를 통해 pysqlite-3 binary를 설치하여 런타임시 강제로 모듈을 교체하였다.

  '''
  
  __import__('pysqlite3')
  import sys
  sys.modules['sqlite3'] = sys.modules.pop('pysqlite3')
  
  '''



