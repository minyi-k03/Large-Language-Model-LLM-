# LangChain_Patent_GPT

## Project - Overview
 **Purpose** : JSON 형식의 임의의 데이터셋에서 출원번호, 요약문, 출원인 등 핵심 메타데이터를 추출하여 검색 효율석을 극대화한 특허 분석 RAG 실습을 진행하였다.

## Tech - Stack
 **FrameWork** : LangChain(Classic)

 **Assist LLM** : Gemini

 **Vector DB** : Chroma DB

 **Data Parsing** : jq, JSONLoader

 **Pre-Processing** : unstructured


## HightLight - Code
 **jq_schema를 활용한 데이터 필터링**
 ```python
 from langchain_community.document_loaders import JSONLoader

 # 전체 JSON을 다 읽지 않고, 요약문(abstract)과 출원인(applicant) 정보만 추출
 loader = JSONLoader(
     file_path='./patent_data.json',
     jq_schema='.[] | {abstract: .abstract, applicant: .applicant}',
     text_content=False
 )
 docs = loader.load()
 ```

