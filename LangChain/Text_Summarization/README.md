# LangChain_Text_Summarization

## Project - Overview
  **Purpose** : LLM에서 Context Window 제한을 극복하기 위해 대량의 문서를 효율적으로 처리하기 위한 두가지 방식을 사용하였다(Stuff, Map-Reduce) 위키피디아에 있는 예시 문서 "대형 언어 모델" 문서를 대상으로 실습을 진행하였다.


 ## Tech - Stack
  **FrameWork**:LangChain(Classic)
  
  **Language**:Python
  
  **LLM**:OpenAI GPT-4o-mini

  **Assist LLM**:Gemini

  **Libraries**: Langchain-openai, Langchain-community, langchain-classic, tiktoken


  ## HighLight - Code
   **Stuff 방식**

   ```python
   from langchain_classic.chains.summarize import load_summarize_chain
   
   chain = load_summarize_chain(llm, chain_type="stuff")
   result = chain.invoke(docs)
   ```

 **Map-Reduce방식**
 ```python
 map_reduce_chain = MapReduceDocumentsChain(
     llm_chain=map_chain, # 각 문서 요약
     reduce_documents_chain=reduce_documents_chain, # 요약본들 통합
     document_variable_name="docs",
     return_intermediate_steps=False,
 )
 text_splitter = CharacterTextSplitter.from_tiktoken_encoder(chunk_size=1000, chunk_overlap=0)
 split_docs = text_splitter.split_documents(docs)
 ```


## Trouble-Shooting
 **(Problem 1) 라이브러리 구조 변경에 따른 NameError 해결**
 
 **문제**: 기존 ChatPromptTemplate, StuffDocumentChain 호출 시 문제 발생

 **해결**: langchain_core, langchain_classic 처럼 세분화하여 경로에 직접 import하여 해결하였다

 **(Problem 2) 토큰 제한 초과 문제**

 **문제** : 문서의 양이 너무 많아 LLM이 한번에 처리하지 못했다.

 **해결**: MapReduceDocumentsChain을 사용하여 1000토큰 단위로 쪼개서 개별 요약 후, 이를 다시 합치는 재귀적인 방식으로 해결하였다.
 
   
     
