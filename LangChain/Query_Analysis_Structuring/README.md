# LangChain_Query_Analysis_Structuring

## Project - Overview

  **Purpose**:자연어로 된 사용자의 Query를 시스템이 해석 가능한 구조화로 변환하는 실습을 진행하였다.


## Tech-Stack
  **Langauge**:Python
  
  **FrameWork**:LangChain(Classic)
  
  **LLM**:GPT-4o-mini

  **Assist LLM**:Gemini


## Schema
  ```python
  from pydantic import BaseModel, Field
  from typing import Optional

  class Search(BaseModel):
    """비디오 데이터베이스 검색을 위한 구조화된 쿼리 스키마"""
    query: str = Field(..., description="유사도 검색에 사용될 핵심 키워드")
    publish_year: Optional[int] = Field(None, description="영상이 게시된 특정 연도")
  ```

## TroubleShooting
  **문제1**: langchain_core.pydantice_v1 버젼 문제
  
  **현상**: 기존 랭체인 예제에서 사용하는 v1 경로가 최신 환경에서는 경고 발생

  **해결**: 표준 pydantic 라이브러리로 경로를 수정하였다.

  **문제2**:다중 쿼리 생성시 문제 발생

  **현상**: 사용자가 한 번에 여러 질문을 할 경우, 단일 객체 반환 시에는 데이터 손실이 발생하였다.

  **해결**: List[Search]형태로 Wrapper 클래스를 따로 정의하여 각각 구조화된 쿼리로 분리하여 생성하도록 수정하였다.

