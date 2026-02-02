# LangChain Fewshot Prompting Template 로컬 환경 변경점

## 1. 개요
  학습목표 : LLM에게 임의의 예시 제공을 통한 답변 품질 향상

## 2. 주요변경점
  2-1: **프롬프트 템플릿 라이브러리 패키지 경로 변경**
  
  변경점 : 기존에 langchain.prompt에서 FewShotPromptTemplate등을 가져오던 방식은 현재 colab환경 버젼과 맞지 않아 수정하였다.

  python'''
  
  #[Legacy]
  
  from langchain.prompts import PromptTemplate, FewShotPromptTemplate

  #[Modern (v0.2+)]
  from langchain_core.prompts import PromptTemplate, FewShotPromptTemplate
  
  '''

  2-2 : **Semantic Selector 의존성 분리**
  
  문제점 : SemanticSimilarityExampleSelector 사용시 내부에서 사용하느 VectorDB 와 Embedding Model의 경로 변경으로 인한 ImportError가 연쇄적으로 발생
  해결책 : LangChain단일 패키지 설치가 아닌 기능별 전용 패키치들 (Langchain_chroma, Langchain_openai)를 설치해서 연결하였다

  python'''
  
  #Legacy Code
  
  from langchain.prompts.example_selector import SemanticSimilarityExampleSelector
  from langchain.vectorstores import Chroma
  from langchain.embeddings import OpenAIEmbeddings

  selector = SemanticSimilarityExampleSelector.from_examples(
      examples,
      OpenAIEmbeddings(), # 구형 임베딩 클래스
      Chroma,             # 구형 벡터스토어 클래스
      k=1
  )
  
  '''

  python'''
  
  #Modern Code
  
  #패키지 세분화 적용
  from langchain_core.example_selectors import SemanticSimilarityExampleSelector
  from langchain_chroma import Chroma            # pip install langchain-chroma
  from langchain_openai import OpenAIEmbeddings  # pip install langchain-openai


  selector = SemanticSimilarityExampleSelector.from_examples(
      examples,
      OpenAIEmbeddings(api_key=KEY), # 최신 임베딩 클래스
      Chroma,                        # 최신 Chroma 클래스
      k=1
  )
  
  '''
  

  
  
  
