# Large Language Model (LLM) Practice & Fine-Tuning

LLM(대형 언어 모델)의 이론적 배경을 학습하고, 다양한 오픈소스 모델 및 프레임워크를 활용하여 직접 코드를 구현한 실습 저장소입니다. 
단순한 API 호출을 넘어, **데이터 정제, 모델 성능 평가, PEFT 기반 파인튜닝, 그리고 로컬 환경으로의 코드 마이그레이션** 경험을 담고 있습니다.

## 🛠 Tech Stack
* **Frameworks:** LangChain, HuggingFace (Transformers, PEFT)
* **Models:** LLaMA, Nexus Raven, OpenAI API
* **Assist LLM** : Gemini

---

## 📂 Directory Index (실습 목차)

### 1. `llama-model` 
* LLaMA 아키텍처 기반 모델의 로드 및 추론 실습
* 로컬 환경에 맞춘 모델 최적화 및 실행 환경 세팅 경험

### 2. `peft` 
* PEFT(Parameter-Efficient Fine-Tuning) 기법 실습
* LoRA(Low-Rank Adaptation) 등을 활용한 자원 효율적인 모델 미세조정 및 가중치 업데이트 과정 구현

### 3. `langchain`
* LangChain 프레임워크를 활용한 프롬프트 엔지니어링 및 체인(Chain) 구성 실습
* 외부 데이터 연동(RAG 등) 기초 구조 파악

### 4. `nexus_raven`
* 함수 호출(Function Calling)에 특화된 Nexus Raven 모델 활용 실습

### 5. `openai-model`
* OpenAI API를 활용한 기본적인 LLM 기능 연동 및 성능 평가 실습

---
> *본 저장소의 코드 중 일부는 클라우드 환경의 코드를 개인 로컬 환경에 맞게 직접 마이그레이션 및 디버깅하여 구동한 결과물입니다.*
