# LangChain_Patent_Gpt

## 1.Overview
  **Purpose**:JSON파일 형식의 특허 데이터셋에서 필요한 메타데이터(출원번호, 요약문, 출원인 등)을 추출 후 효율적인 RAG 시스템에 대한 실습 진행

## 2. Tech Stack
  **FrameWork**:LangChain 사용
  **Data Parsing**:jq(JSON Processor), JSONLoader 사용
  **Pre-Processing**:unstructed 라이브러리 사용
  **Vector DB**:ChromDB 사용
  **Assist LLM**:Gemini

## 3.TroubleShooting으로 인한 로컬 환경 변경점
  **3-1**: 기존 강의 실습에서 JSONLoader사용시 jq라이브러리 부재로 인한 에러 발생

  **해결**:Python 패키지 뿐만 아니라 시스템 레벨의 라이브러리 설치를 통해 해결

  '''
  
  !apt-get install -y jq
  
  !pip install -qU unstructured

  '''

  **3-2**: 특허 데이터 셋에 대해 전체를 임베딩하면 토큰 소모가 너무 크고 검색 품질이 떨어진다

  **문제점**: 특허 데이터 셋 내에는 기술적 내용과 권리 정보가 섞여 있기에 필요한 데이터만 골라내는 작업이 필수적이다

  **해결**: jq_schema를 활용하여 필요한 필드만 추출하도록 기존 실습에 있는 로직을 변경하였다.

## 4.주요 Learning Point
  **정형 데이터에 대한 전처리 중요성** : LLM에 모든 데이터를 넣기보다는 필요한 데이터만 추출하는 것이 비용적인 측면에서 유리하다

  **외부 라이브러리 통합** : 시스템 수준의 라이브러리를 파이썬 환경과 통합하여 사용하는 법이 중요하다

  **특허 도메인 지식** : 특허 데이터의 구조에 대해서 알게 되었다
