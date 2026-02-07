# Langchain_Document_Loaders

## Project - Overview
  **Purpose** : 외부 데이터를 랭체인 시스템으로부터 가져와 DOC 객체로 변환하는 과정을 실습하였다. WebBaseLoader, PyPDFLoader를 통해 비정형 데이터를 정형화 하는 실습을 진행하였다.

## Tech - Stack
  **FrameWork** : LangChain(Classic)
  
  **Langugage** : Python

  **Assist LLM** : Gemini

  **Libraries** : langchain-community, pypdf, beautifulsoup4


## HightLight - Code
  **웹 페이지, PDF 구현**
  ```python
  from langchain_community.document_loaders import WebBaseLoader, PyPDFLoader

  # 웹 문서 로드 (Wikipedia - 대형 언어 모델)
  loader = WebBaseLoader("https://ko.wikipedia.org/wiki/대형_언어_모델")
  docs = loader.load()
  
  # PDF 문서 로드 및 페이지 분할
  loader_pdf = PyPDFLoader("./llama1_paper.pdf")
  pages = loader_pdf.load_and_split()
  ```


## Trouble-Shooting
  **(Problem 1)라이브러리 파편화로 인한 Import 경로 문제 발생**

  **(문제)** : WebBaseLoader 등을 호출할때 langchain 경로에서 해당 모듈을 찾지 못하였다

  **(해결)** : langchain-community 패키지에서 가져와야 한 것을 langchain_classic으로 변경하여 해결하였다.

    
