# LangChain_Product_Recommendataion_GPT

## Project - Overview
 **Purpose** : 임의의 데이터 셋 내에 IT 기기 데이터셋을 기반으로 사용자의 Query를 분석하여, 가격, 카테고리, 평점 등 메타데이터를 자동으로 필터링 하여 Query 맞게끔 최적의 상품을 추천하는 실습을 진행하였다.


## Tech - Stack
 **FrameWork** : LangChain(Classic)

 **Language** : Python

 **LLM** : OpenAI-GPT-4o-mini

 **Assist LLM** : Gemini

 **Embedding** : ko-sroberta-multitask

 **Vector Store** : Chroma DB

 **Parsing** : lark


## HightLight - Code

 **상품 속성 정의**
 ```python
 from langchain.chains.query_constructor.base import AttributeInfo

 # LLM이 자연어 질문에서 추출할 상품의 속성 정보 정의
 metadata_field_info = [
     AttributeInfo(
         name="ProductName",
         description="상품의 전체 이름 (예: 갤럭시 북4 Pro)",
         type="string",
     ),
     AttributeInfo(
         name="MainCategory",
         description="상품의 대분류 카테고리 (예: 노트북, 휴대폰)",
         type="string",
     ),
     AttributeInfo(
         name="Price",
         description="상품의 현재 판매 가격 (단위: 원)",
         type="integer",
     ),
 ]
 ```

**Self-Query Retriver**
```python
from langchain.retrievers.self_query.base import SelfQueryRetriever

 # 자연어를 구조화된 필터(예: Price < 1000000 AND MainCategory == '노트북')로 변환
 retriever = SelfQueryRetriever.from_llm(
     llm=llm,
     vectorstore=vectorstore,
     document_contents="IT 기기 제품 정보 및 사양",
     metadata_field_info=metadata_field_info,
     query_constructor=query_constructor,
     verbose=True # 내부 필터 생성 과정 확인
 )
 ```

