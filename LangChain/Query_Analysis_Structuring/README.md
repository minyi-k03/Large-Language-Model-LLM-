# LangChain_Query_Analysis_Structing

## Project - Overview
 **Purpose** : 자연어로 입력된 Query를 시스템이 정확히 해석하고 실행할 수 있도록, LLM을 활용하여 구조화된 데이터로 변환하는 Query Analysis 실습을 진행하였다.


## Tech - Stack
 **FrameWork** : LangChain(Classic)

 **Language** : Python

 **LLM** : OpenAI GPT-4o-mini

 **Assist LLM** : Gemini

 **Libraries** : langchain-core, pydantic


## HightLight - Code
 **검색 의도 파악을 위핸 Pydantic Model**
 ```python
 from pydantic import BaseModel, Field
 from typing import Optional
 
 class Search(BaseModel):
     """비디오 데이터베이스 검색을 위한 구조화된 쿼리 스키마"""
     
     query: str = Field(
         ..., 
         description="유사도 검색에 사용될 핵심 키워드"
     )
     publish_year: Optional[int] = Field(
         None, 
         description="영상이 게시된 특정 연도"
     )
 ```

## TroubleShooting
 **(Problem 1) Pydantic 버전 호환성 및 경로 문제**

 **(해결)** : langchain 내부 경로로 불러오지 않고, 표준 Pydantic 라이브러리를 직접 임포트 하여 환경에 맞게 변경하였다.
