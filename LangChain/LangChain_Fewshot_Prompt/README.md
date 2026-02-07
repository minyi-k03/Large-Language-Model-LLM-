# LangChain_Fewshot_Prompt_Template

## Project - Overview
  **Purpose** : LLM에게 몇가지 Few-shot Example를 제공하여 모델이 사용자의 의도와 형식을 잘 이해하도록 유도하는 실습을 진행하였다.


## Tech - Stack
  **FrameWork**:LangChain(Classic)

  **Language** : Python

  **LLM** : OpenAI GPT-4o-mini

  **Assist LLM** : Gemini

  **Embedding Model** : OpenAI Embeddings

  **Libraries** : langchain-openai, langchain-community, langchain-chroma, chroma db


## HightLight - Code
  **SemanticSimilarityExampleSelector**
  ```python
  from langchain_community.vectorstores import Chroma
  from langchain_core.example_selectors import SemanticSimilarityExampleSelector
  from langchain_openai import OpenAIEmbeddings
  
  # 질문과 유사한 예시를 찾기 위한 선택기 구성
  example_selector = SemanticSimilarityExampleSelector.from_examples(
      examples,                   # 예시 리스트
      OpenAIEmbeddings(),         # 유사도 계산을 위한 임베딩
      Chroma,                     # 예시를 저장할 벡터 DB
      k=1                         # 가장 유사한 예시 1개만 선택
  )
  ```

  **FewShotPromptTemplate**
  ```python
  from langchain_core.prompts.few_shot import FewShotPromptTemplate

  prompt = FewShotPromptTemplate(
      example_selector=example_selector, # 동적 선택기 사용
      example_prompt=example_prompt,     # 예시 형식
      suffix="Question: {input}",        # 사용자의 질문이 들어갈 자리
      input_variables=["input"]
  )
  ```

