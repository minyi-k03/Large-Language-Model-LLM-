# LangChain_Document_Transformers

## Project - Overview
  **Purpose** : 길이가 긴 문서를 LLM이 처리할 수 있도록 토큰 단위로 분할하는 실습을 진행하였다.

## Tech - Stack
  **FrameWork** : LangChain(Classic)
  
  **Language** : Python

  **Assist LLM** : Gemini
  
  **Libraries** : langchain-text-splitters, tiktoken


## HightLight - Code
  **재귀적인 분할, 토큰 기반 분할 설정**
  ```python
  from langchain_text_splitters import RecursiveCharacterTextSplitter, CharacterTextSplitter

  # 1. 문맥 보존을 위한 재귀적 분할
  text_splitter = RecursiveCharacterTextSplitter(chunk_size=100, chunk_overlap=20)
  splits = text_splitter.split_documents(pages)
  
  # 2. 모델 토큰 제한을 고려한 Tiktoken 기반 분할
  token_splitter = CharacterTextSplitter.from_tiktoken_encoder(chunk_size=100, chunk_overlap=0)
  token_splits = token_splitter.split_documents(pages)
  ```

  
