# LangChain V0.2 Migration #

 ## 1. 실습 내역
  LangChain과 RAG를 기반으로 한 챗봇 만들기 실습 


  ## 2. 오류 발생 내역 ##
    2-1. 패키지 의존성 및 설치 오류
    해결 : LangChain 라이브러리가 비대해짐에 따라 core, community, partner로 분리해서 설치하여 경로를 수정하였다

    2-2. Chain 객체를 함수처럼 호출하거나 run()함수 사용시 오류 발생
    해결 : LangChain의 모든 Runnable 객체를 invoke()함수로 통일하여 사용하였다.

    2-3. 기존 실습에서 사용하는 Conversation, ConversationBufferMemory 라이브러리 충돌
    해결 : 메모리 객체를 체인 내부에 숨기는 대신 RunnableWithMessageHistory를 사용하여 외부에서 관리하도록 변경

    2-4. 


  
