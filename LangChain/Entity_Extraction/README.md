# P1. Entity Extraction

**Assist LLM**:Gemini

## P1-Overview
  **Purpose**:일상적인 문장 데이터에서 인물, 반려동물, 특징 등 특정 정보를 추출하여 시스템이 처리 가능하게끔 JSON/Pydantic객체로 변환

## P1-구현내용
  **스키마 정의**:Pydatic 라이브러리를 사용하여 추출할 데이터의 타입과 제약 조건 정의
  
  **다중 엔티티 처리**:한 문장에서 여러 개의 객체를 동시에 추출하는 로직을 사용하였다

## P1-TroubleShooting
  **문법 마이그레이션**:과거에 사용되었떤 create_extraction_chain을 최신 방식인 .with_structed_output()함수로 변경하였다
  
  **추출 정확도 향상**: Pydantic 필드에 있는 description을 상세히 작성하여 LLM이 모호한 문맥에서도 정확히 필에 값을 매핑하도록 유도하였다


# P2. AutoMatcit Tagging & Sentiment

## P2-Overview
  **Purpose**: 고객 리뷰 데이터셋에서 텍스트를 분석하여 사용 언어, 분위기 등을 자동으로 분류하게끔 실습

## P2-주요 구현 내용
  **분류 체계 설계**: Enum 클래스를 사용하여 감정 등급, 언어 종류를 고정된 선택지로 제한하였다
  
  **다중 속성 태깅**: 한 번의 추론으로 감정, 언어, 점수를 동시에 추출하여 멀티 태깅하였다

## P2-TroubleShooting
  **스키마 제목 제약**:Pydantic 스키마 정의 시 title속성이 무조건 있어야 하는데 기존 실습 코드에는 존재하지 않아서 추가하였다
  
  **일관된 결과 출력**:LLM이 자유로운 문장 대신 사전에 Enum 함수에 정의되어 있는 값 내에서만 답변하도록 프롬프트, 스키마 동기화

  
