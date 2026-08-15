# LangChain V0.2 Migration #

 ## Project - Overview
  **Purpose** : LangChain V0.2 업데이트에 맞춰 기존 RAG기반 챗봇 실습 코드를 최신 표준 LCEL 방식으로 마이그레이션을 진행하였다.


## Tech - Stack
 **FrameWork** : LangChain V0.2

 **Language** : Python

 **Assist LLM** : Gemini

 **Libraries** : langchain-core, langchain-community

 **Methodology** : LCEL, RAG


## HighLight - Code
 **Runnable Interface Integrated**
 ```python
 # Legacy: 객체를 함수처럼 호출하거나 run() 사용 시 오류 발생 가능
 # response = chain.run("이 문장을 번역해줘")
 
 # LCEL Standard: 모든 Runnable 객체를 invoke()로 통일하여 안정성 확보
 response = chain.invoke({"input": "이 문장을 번역해줘"})
 print(response.content)
 ```

 **외부 세션 관리를 통한 메모리 구조 개선**
 ```python
  # RunnableWithMessageHistory를 사용하여 메모리를 체인 외부에서 관리
  chain = prompt | llm 
  chain_with_history = RunnableWithMessageHistory(
      chain,
      get_session_history,          # 세션 ID별 기록 관리 함수
      input_messages_key="input",
      history_messages_key="history"
  )
  ```

 **정확도 향상을 위해 2-step RAG**
 ```python
 # 1단계: 대화 기록을 반영하여 검색에 최적화된 질문으로 재구성
 history_aware_retriever = rephrase_prompt | llm | parser | retriever
 
 # 2단계: 재구성된 질문과 검색된 문서를 바탕으로 최종 답변 생성
 rag_chain = (
     RunnablePassthrough.assign(context=history_aware_retriever | format_docs)
     | qa_prompt
     | llm
 )
 ```

## TroubleShooting
 * **(Problem 1)패키치 의존성 및 설치 경로 오류**
 
 * **(해결)** : langchain 단일 설치 대신 각각 langchain-core, langchain-community, langchain-openai등 모듈 세분화 설치 후 임포트 경로를 수정하여 해결하였다.

 * **(Problem 2) Legacy Chain 실행 방식 호환성 문제 발생**

 * **(해결)** : LangChain V0.2에 맞춰서 기존 run()함수 대신 invoke(), stream(), batch()로 실행 방식을 통일하여 해결하였다.

 * **(Problem 3) ConversationBufferMemory 라이브러리 충돌**

 * **(해결)** : 메모리를 체인 내부에 결합하는 대신, RunnableWithMessageHistory Wrapper를 사용하여 대화 기록을 체인 외부에서 주입하는 방식으로 변경하여 해결하였다.

 * **(Problem 4) CoversationRetrievalChain의 낮은 정확도**

 * **(해결)** : 질문 재구성(RePhrase)와 문서 기반 답변(QA) 두 단계 LCEL 파이프라인으로 명시적으로 분리하여 각 단계의 프롬프트와 로직을 제어하였다.
 
  
