# Model Training: NLI 기반 가짜뉴스 판독 모델 파인튜닝

## Project-Overview
* **Purpose** : 한국어 기사 본문(Premise)과 제목(Hypothesis) 간의 논리적 모순을 판별하여 가짜뉴스를 탐지하기 위해, 최대 4096 토큰을 지원하는 `monologg/kobigbird-bert-base` 모델을 이진 분류(참/거짓) Task에 맞게 파인튜닝하는 실습 진행.
* **Dataset** : 가짜뉴스 판별용 NLI 학습 데이터 총 10,000개 (Label 0과 1의 1:1 클래스 비율을 유지하도록 EDA 기반 교체 증강 적용).

## Tech-Stack
* **Language** : Python
* **FrameWork** : PyTorch, HuggingFace (Transformers)
* **Base Model** : `monologg/kobigbird-bert-base`
* **Techniques** : EDA(Random Deletion, Random Swap), Dynamic Padding, AMP(Automatic Mixed Precision)

## HighLight-Code

**1. 1:1 클래스 불균형을 방지하는 원본 데이터 교체 증강(EDA)**
데이터를 단순히 추가하여 비율이 무너지는 것을 막기 위해, 랜덤 삭제(Random Deletion) 및 위치 변경(Random Swap)을 적용한 변형 데이터를 생성한 후 기존 원본 데이터와 교체하여 1:1 비율을 유지한다.
```python
df_true_all = df[df['label'] == 0]
df_true_to_augment = df_true_all.sample(n=2000, random_state=42).copy()
df_true_to_augment['hypothesis'] = df_true_to_augment['hypothesis'].apply(augment_text)

# 원본 데이터 프레임에서 증강할 대상이었던 원본 2000개를 삭제
df_remaining = df.drop(df_true_to_augment.index)

# 남은 데이터 8000개 + 변형된 데이터 2000개 결합 (총 10,000개, 5000:5000 완벽 유지)
df_augmented = pd.concat([df_remaining, df_true_to_augment], ignore_index=True)

```


**2. 1:1 메모리 최적화를 위한 동적 패딩 및 혼합 정밀도(AMP) 학습**
고정 길이 패딩으로 인한 메모리 낭비를 막기 위해 DataCollatorWithPadding을 사용하고, 최신 autocast와 GradScaler API를 사용하여 T4 GPU 환경에서 학습 속도와 메모리를 최적화하였다.
```python
data_collator = DataCollatorWithPadding(tokenizer=tokenizer)

with autocast('cuda'):
    outputs = model(
        input_ids=input_ids,
        attention_mask=attention_mask,
        token_type_ids=token_type_ids,
        labels=labels
    )
    loss = outputs.loss / ACCUMULATION_STEPS

scaler.scale(loss).backward()
```

## TroubleShooting
**(Problem 1) Colab T4 GPU VRAM OOM 현상**

**(현상)**: 기사 전문을 커버하기 위해 1024의 max_length를 설정할 시 고정 길이 패딩과 일반적인 모델 구조에서는 Colab T4 GPU 환경에서 OOM 에러가 발생

**(해결)**: 최대 4096 토큰을 지원하는 BigBird 아키텍처 모델(monologg/kobigbird-bert-base)을 채택하고, DataCollatorWithPadding을 통해 배치 내 최대 길이에 맞춰 동적으로 패딩을 조절하였다. 추가적으로 GradScaler('cuda')를 이용한 혼합 정밀도 연산을 적용하여 메모리 사용량을 획기적으로 줄여 해결하였다.


**(Problem 2) 데이터 증강(Data Augmentation) 시 클래스 불균형 문제 발생**

**(현상)** : 모델의 어휘적 과적합을 방지하고자 문장 변형을 적용할 때, 단순히 데이터를 추가하면 참(0)과 거짓(1) 레이블 간의 1:1 비율이 무너짐.

**(해결)** : random_deletion 및 random_swap 기법을 적용하여 EDA를 수행하되, 증강 대상으로 샘플링된 원본 2000개의 인덱스를 기존 데이터 프레임에서 찾아 drop 시킨 후 변형된 데이터를 concat하여 추가가 아닌 교체(Replace) 방식으로 병합함으로써 완벽히 10,000개 데이터의 균형을 유지하였다.

**(Problem 3) Device-side assert 에러 발생 및 모델 학습 중단**

**(현상)** : 모델 학습 루프 중 텐서 인덱스가 단어 사전 크기를 벗어나거나 타겟 라벨 범위가 맞지 않아 커널이 중단되는 문제 발생 우려.

**(해결)** : torch.clamp 함수를 사용하여 input_ids를 모델의 vocab_size - 1 이내로 강제하고, labels 역시 min=0, max=1로 고정하는 강력한 방어 코드를 학습 및 검증 루프 내에 삽입하여 에러를 원천 차단하였다.


## FrontEnd & Serving: SLM-LLM 하이브리드 가짜뉴스 팩트체크 웹 서비스

## Project-Overview

* **Purpose** : 파인튜닝된 소형 언어 모델(Ko-BigBird)을 1차 필터로, 대형 언어 모델(Llama 3.3)을 2차 교차 검증용으로 결합한 'Cascade 아키텍처'를 구현하고, Streamlit 웹 기반 UI와 Cloudflare 터널링을 통해 사용자가 직접 테스트해 볼 수 있는 실시간 서빙 환경을 구축한다Project.ipynb].

* **Architecture** : SLM Filter (Ko-BigBird) -> LLM Routing (Llama 3.3 via Groq API)Project.ipynb].

## Tech-Stack
* **Language** : Python

* **FrameWork & UI** : PyTorch, HuggingFace (Transformers), Streamlit

* **LLM / API** : Llama-3.3-70b-versatile, Groq API

* **Deployment** : Cloudflare Tunnel

## HighLight-Code
**1. 비용 및 속도를 최적화하는 SLM-LLM 하이브리드 라우팅 (Cascade)**
1차적으로 Ko-BigBird 모델이 판독을 수행하고, 확신도(Confidence)가 사용자가 설정한 임계값(Threshold, 기본 권장값 0.80) 미만일 때만 Llama 3.3 모델로 연산을 넘겨 속도와 API 과금 비용을 동시에 최적화한다Project.ipynb].

```python
if slm_confidence >= threshold:
    # 1차 연산 완료: 확신도가 높아 SLM 단독 처리 (LLM 비용 발생 X)
    if pred_class == 0:
        st.success(f"✅ 신뢰 가능한 정보 확정 (확신도: {slm_confidence * 100:.1f}%)")
    else:
        st.error(f"🚨 가짜뉴스 판단 확정 (확신도: {slm_confidence * 100:.1f}%)")
else:
    # 2차 연산 라우팅: 확신도가 낮아 LLM으로 교차 검증 요청
    with st.spinner("🧠 Llama 3.3 MaaS로 라우팅 중..."):
        llama_result = ask_llama_factcheck(refined_premise, refined_hypothesis)
```

**2. Few-Shot 및 CoT(Chain of Thought) 기반 프롬프트 튜닝 (Prompt Engineering)**
LLM이 언론의 관행(따옴표, 축약어)을 오해하여 가짜뉴스로 오판하는 현상을 막기 위해, '수석 에디터' 페르소나를 부여하고 상세한 판정 가이드라인, 3가지 시나리오별 예시(Few-Shot) 및 최종 출력 형식을 지정하여 추론의 일관성과 정확도를 확보했다.

```python
def ask_llama_factcheck(premise, hypothesis):
    system_prompt =(
        "[1. 역할 (Role)]\n"
        "당신은 대한민국 최고 수준의 팩트체크 전문 AI 수석 에디터입니다. "
        "객관적인 논리와 문맥 이해력을 바탕으로 가짜뉴스와 낚시성 기사를 정확하게 판별해야 합니다.\n\n"

        "[2. 맥락 (Context)]\n"
        "당신에게는 1차 AI 필터가 판독을 보류한, 다소 교묘하고 판단하기 까다로운 '기사 본문(Evidence)'과 '기사 제목(Claim)'이 주어집니다. "
        "주의: 기사 제목에 사용된 따옴표(' ', \" \")나 축약어, 비유적 표현은 언론의 정상적인 편집 관행일 수 있습니다. "
        "단순한 '어휘의 불일치'가 아니라, 실제 팩트가 충돌하는 '논리적 모순'이나 '과장(낚시)'이 있는지를 파악해야 합니다.\n\n"

        "[3. 수행 작업 (Task)]\n"
        "주어진 '기사 본문'과 '기사 제목'을 꼼꼼히 대조하여 다음 세 가지 중 하나로 판별하십시오.\n"
        "1. 진짜 뉴스: 제목이 본문의 내용을 논리적으로 정확히 반영함.\n"
        "2. 가짜뉴스: 제목이 본문의 내용과 정면으로 충돌하거나, 없는 사실을 낚시성으로 과장함.\n"
        "3. 판단 불가: 주어진 본문의 내용만으로는 제목의 진위 여부를 도저히 판단할 수 없거나 아예 무관한 내용임.\n\n"

        "[4. 출력 형식 및 제약 사항 (Format/Constraints)]\n"
        "- 절대 억지로 추론하여 정답을 끼워 맞추지 마십시오. 본문에 근거가 없다면 반드시 '판단 불가'로 판정해야 합니다.\n"
        "- 높은 정답률을 위해 판정 전에 반드시 논리적 분석(사고 과정)을 먼저 거치십시오.\n"
        "- 답변은 반드시 아래의 지정된 포맷을 엄격히 준수하여 출력하십시오.\n\n"

        "[판단 예시 1]\n"
        "본문: 박명수가 라디오에서 선거에 대해 이야기했다.\n"
        "제목: 박명수 '사람 잘못 뽑으면 큰일나'\n"
        "[사고 과정]: 본문에 구체적인 인용구는 없으나, 선거 관련 이야기라는 문맥상 제목의 인용구는 언론의 정상적인 축약 및 인용 관행으로 볼 수 있어 모순되지 않음.\n"
        "[판정]: 진짜 뉴스\n\n"

        "[판단 예시 2]\n"
        "본문: 경찰 조사 결과, 해당 유명인의 횡령 의혹은 사실무근으로 밝혀졌으며 무혐의 처분을 받았다.\n"
        "제목: [단독] 유명인 A씨 수백억 횡령 혐의 인정… 결국 구속되나\n"
        "[사고 과정]: 본문에서는 명확히 '사실무근' 및 '무혐의'라고 밝혔으나, 제목은 혐의를 인정하고 구속될 것처럼 정반대의 내용을 서술하여 논리적 모순이 발생함.\n"
        "[판정]: 가짜뉴스\n\n"

        "[판단 예시 3]\n"
        "본문: 서울에 비가 내린다.\n"
        "제목: 전국 부동산 가격 폭락\n"
        "[사고 과정]: 주어진 본문의 '비'라는 기상 정보와 제목의 '부동산 가격 폭락' 사이에는 어떠한 논리적 연관성도 없음.\n"
        "[판정]: 판단 불가\n\n"

        "[최종 출력 포맷]\n"
        "[사고 과정]: (한 줄로 간결하고 명확한 논리적 근거 제시)\n"
        "[판정]: ('진짜 뉴스', '가짜뉴스', '판단 불가' 중 택 1)"
    )
```

**3. 정규표현식(Regex)을 활용한 Cloudflare URL 자동 추출 배포**
Streamlit 로컬 서버를 외부망으로 자동 배포하기 위해 백그라운드에서 Cloudflare 터널을 구동하고, 로그 파일(tunnel.log)에서 정규표현식을 사용해 접근 가능한 외부 URL만 깔끔하게 파싱하여 출력한다.

```python
!nohup ./cloudflared-linux-amd64 tunnel --url http://localhost:8501 > tunnel.log 2>&1 &

with open("tunnel.log", "r") as f:
    # https://로 시작하고 .trycloudflare.com으로 끝나는 외부망 접속 URL 패턴 자동 추출
    match = re.search(r"https://[a-zA-Z0-9-]+\.trycloudflare\.com", f.read())
    if match: print(f"👉 {match.group(0)}")
```

## TroubleShooting
**(Problem 1) 긴 뉴스 기사 처리 시 문맥 유실(Truncation) 문제**

**(현상)** : 긴 기사 본문을 입력할 때 과거 모델이 지원하던 짧은 토큰 한계(512 길이)로 인해 후반부 문맥이 잘려나가 정확한 팩트체크가 불가능함.

**(해결)**: 본문 일부를 임의로 잘라내는 기존 텍스트 절삭 함수를 제거하고, 최대 4096 토큰을 지원하는 Ko-BigBird 모델의 이점을 100% 살려 추론 시 max_length를 1024로 확장함으로써 기사 전문을 훼손 없이 검증하도록 해결하였다. (Colab환경을 고려해서 일부러 1024토큰으로 제한 설정)

**(Problem 2) LLM API 출력 포맷 불일치 및 할루시네이션(Hallucination)**

**(현상)** : Llama 3.3 모델이 가끔 지정된 출력 포맷([판정]:)을 누락하거나 불필요한 부연 설명을 덧붙이는 현상 발생.

**(해결)** : 프롬프트 튜닝을 통해 3가지 판정 예시(Few-Shot)를 주입하고, max_tokens를 256으로 타이트하게 제한하였다. 추가로 출력 텍스트에 필수 지정 키워드가 누락될 경우를 대비해, 문자열 포함 여부(예: "가짜", "모순")에 따라 강제로 폼을 재생성해주는 간단한 텍스트 후처리 방어 로직(Fallback)을 구현하여 에러율을 0%로 만들었다.

**(problem 3) Colab 환경에서의 UI 데모 접속 및 배포 불가**

**(현상)** : 구글 Colab 환경에서 Streamlit을 구동하면 로컬 포트(8501)가 외부에 노출되지 않아 브라우저에서 UI 접속이 불가능함.

**(해결)** : cloudflared 바이너리를 다운받아 터널링(Tunneling) 인프라를 구축하였다. 특히 기존 프로세스와의 포트 충돌을 막기 위해 pkill 명령어로 찌꺼기 프로세스를 정화하고, Streamlit 백그라운드 구동 후 터널 로그에서 정규표현식으로 접속 URL만 파싱하는 자동화 배포 스크립트를 작성하여 해결하였다.
