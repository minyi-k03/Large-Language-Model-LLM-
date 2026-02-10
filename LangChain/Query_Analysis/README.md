# Query Analysis

## Project-Overview
  **Purpose**:검색 성능 향상을 위해 쿼리 분해 및 변형 실습 진행

## Tech-Stack
  **Langauge**:Python
  
  **FrameWork**:LangChain(Classic)
  
  **LLM**:GPT-4o-mini

  **Assist LLM**:Gemini

  **Data Ingestion, Loading**:YoutubeLoader, youtube-transcript-api, pytubefix 

  **Query Analysis**:Query Decomposition, Metadata Filtering, Pydantic

## HighLight - Code
  ```python
  from pydantic import BaseModel, Field
  from typing import Optional

  class Search(BaseModel):
    """비디오 데이터베이스 검색을 위한 구조화된 쿼리 스키마"""
    query: str = Field(..., description="유사도 검색에 사용될 핵심 키워드")
    publish_year: Optional[int] = Field(None, description="영상이 게시된 특정 연도")

  ```
## TroubleShooting

  **문제1: Pydantic 버젼 문제 발생**
  **해결**: Pydantic v2 라이브러리로 교체하여 안전성을 높였다
