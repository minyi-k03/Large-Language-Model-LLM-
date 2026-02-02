# LangChain V0.2 Migration #

 ## 1. 실습 내역
  LangChain과 RAG를 기반으로 한 챗봇 만들기 실습 


  ## 2. 오류 발생 내역 ##
    2-1. 패키지 의존성 및 설치 오류
    해결 : LangChain 라이브러리가 비대해짐에 따라 core, community, partner로 분리해서 설치하여 경로를 수정하였다
    

    2-2. Chain 객체를 함수처럼 호출하거나 run()함수 사용시 오류 발생
    해결 : LangChain의 모든 Runnable 객체를 invoke()함수로 통일하여 사용하였다.
    <CODE>
    -------------------------------------------------------------------------------------------------------
    
     # Legacy 방식
      response = chain.run("이 문장을 번역해줘")

     # LCEL 표준 방식
     response = chain.invoke({"input": "이 문장을 번역해줘"})
     print(response.content) 
  
  ---------------------------------------------------------------------------------------------------------
   
    2-3. 기존 실습에서 사용하는 Conversation, ConversationBufferMemory 라이브러리 충돌
    해결 : 메모리 객체를 체인 내부에 숨기는 대신 RunnableWithMessageHistory를 사용하여 외부에서 관리하도록 변경
    <CODE>
    ------------------------------------------------------------------------------------------------------
    # 내부 결합 방식
     memory = ConversationBufferMemory(memory_key="history")
     chain = ConversationChain(llm=llm, memory=memory)
     
     # 외부 주입 방식 (LCEL)
     chain = prompt | llm 
     chain_with_history = RunnableWithMessageHistory(
         chain,
         get_session_history, # 세션 저장소 관리 함수
         input_messages_key="input",
         history_messages_key="history"
     )

    --------------------------------------------------------------------------------------------------------     

    2-4. ConversationRetrievalChain 라이브러리 사용시 정확도 떨어짐
    해결: 두 단계의 LCEL 파이프라인으로 분리하여 구현하였다.
    <CODE>
    ---------------------------------------------------------------------------------------------------------
    # ConversationalRetrievalChain (Legacy)
    qa = ConversationalRetrievalChain.from_llm(llm, retriever, memory=memory)
    
    # LCEL Pipeline
    # 질문 재구성
    history_aware_retriever = rephrase_prompt | llm | parser | retriever #두 단계의 파이프라인으로 구축
    
    # 문서 기반 답변
    rag_chain = (
        RunnablePassthrough.assign(context=history_aware_retriever | format_docs)
        | qa_prompt
        | llm
    )

    -------------------------------------------------------------------------------------------------------

  
