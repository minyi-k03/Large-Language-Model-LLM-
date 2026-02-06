# Langchain 상품추천 GPT

## Project - Overview
  **Purpose**: IT 데이터 셋에서 사용자가 원하는 상품을 입력하면 상품을 추천하여 찾아주는 방식으로 실습 진행


## Tech-Stack
  **FrameWork**:Langchain(Classic)

  **Language**:Python 3.12

  **Assist LLM**:Gemini

  **LLM**:OpenAI GPT-4o-mini

  **Embedding**:ko-sroberta-multitask(한국어 특화 임베딩 모델)

  **Parsing**:Lark(Structured Query Parser)


## MetaData-Schema 
  '''
  
  from langchain.chains.query_constructor.base import AttributeInfo

  **1. 각 메타데이터 필드의 의미와 데이터 타입을 정의합니다.**
  **이 정보는 LLM이 쿼리를 생성할 때 '어떤 필드로 필터링할지' 결정하는 근거가 됩니다.**
  metadata_field_info = [
  
      AttributeInfo(
      
          name="ProductName",
          
          description="상품의 전체 이름 (예: 갤럭시 북4 Pro, 로지텍 MX Master 3S)",
          
          type="string",
          
      ),
      AttributeInfo(
      
          name="MainCategory",
          
          description="상품의 대분류 카테고리 (예: 노트북, 휴대폰, 키보드, 마우스)",
          
          type="string",
      ),
      
      AttributeInfo(
      
          name="Price",
          
          description="상품의 현재 판매 가격 (단위: 원)",
          
          type="integer",
          
      ),
      AttributeInfo(
      
          name="ReviewScore",
          
          description="사용자들이 남긴 평점의 평균 (1점에서 5점 사이)",
          
          type="integer",
          
      ),
  ]
  
  **2. 데이터셋 전체에 대한 설명을 정의합니다.**
  
  document_content_description = "다양한 IT 기기의 제품명, 카테고리, 가격 및 사용자 평점 정보가 담긴 데이터셋"

  '''

## Main-Tech
  **Query Constructor and Self-Query**
  
  '''
  **자연어를 구조화된 필터로 변환**
  
  retriever = SelfQueryRetriever(
  
      query_constructor=query_constructor,
      
      vectorstore=vecstore,
      
      verbose=True
  )
  '''

## TroubleShooting

  **문제1**:복합 조건 검색시 Unexpected toke Token 에러 발생

  **원인**: LLM의 논리 연산자(and)를 파이썬 스타일인 Infix 방식으로 생성했지만, Langchain Parser는 Prefix 방식으로 요구하여 문제 발생

  **해결**: Few-shot Example을 프롬프트에 같이 넣어 출력 형식을 표준 랭체인 표준 문법에 맞게끔 수정하였다.
  
