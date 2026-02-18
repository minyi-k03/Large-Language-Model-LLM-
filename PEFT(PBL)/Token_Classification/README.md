# PEFT LoRA Token Classification(BioNER)

## Project - Overview
  **Purpose**: Hugging Face의 PEFT(Parameter-Efficient Fine-Tuning) 라이브러리와 LoRA 기법을 적용하여 RoBERTa를 바이오 의학 분야 토큰 분류 실습을 진행하였다.


## Tech-Stack
 **Langague** : Python
  
 **FrameWork** : Hugging Face(Transformer, PEFT, Dataset, Evaluate)
  
 **Base Model** : roberta-base
  
 **Dataset** : ncbi-disease (BioNLP 대체 데이터 셋)
  
 **Metrics** : seqeval(Precision, Recall, F1, Accuracy)
 
 **Assist LLM** : Gemini
  

## HighLight-Code
  **1. 레이블 정렬 및 토크나이징**
  ```python
  def tokenize_and_align_labels(examples):
    tokenized_inputs = tokenizer(examples["tokens"], truncation=True, is_split_into_words=True)
    labels = []
    
    for i, label in enumerate(examples["ner_tags"]): 
        word_ids = tokenized_inputs.word_ids(batch_index=i)
        previous_word_idx = None
        label_ids = []
        for word_idx in word_ids:
            if word_idx is None:
                label_ids.append(-100)
            elif word_idx != previous_word_idx:
                label_ids.append(label[word_idx])
            else:
                label_ids.append(-100)
            previous_word_idx = word_idx
        labels.append(label_ids)

    tokenized_inputs["labels"] = labels
    return tokenized_inputs
  ```

  **2. LoRA 설정 적용**
  ```python
  from peft import get_peft_model, LoraConfig, TaskType
  
  peft_config = LoraConfig(
      task_type=TaskType.TOKEN_CLS, 
      inference_mode=False, 
      r=16, 
      lora_alpha=16, 
      lora_dropout=0.1, 
      bias="all"
  )
  
  model = get_peft_model(model, peft_config)
  model.print_trainable_parameters()
  ```

## TroubleShooting 
 **(Problem 1) 데이터셋 구조 변경으로 인한 KeyError** 
  
 **(원인)** : 기존에 사용하던 tner/bionlp2004 데이터 셋의 지원이 종료되어 ncbi-disease 데이터 셋으로 대체하였으나, 타겟 레이블 컬럼명이 tags에서 ner_tags로 구성되어 있었음
 
 **(해결)** : 기존 태그명 tags가 아닌 ner_tags로 변경하여 레이블 매핑을 하였다.

 **(Problem2)**:모델 훈련 파라미터 세팅 시 evaluation_strategy 관련 Deprecation Warning 혹은 오류 발생
 
 **(원인)**: Hugging Face Transformers 라이브러리가 업데이트되면서 기존의 evaluation_strategy인자명이 변경됨
 
 **(해결)**:TrainingArguments 선언 시 인자명을 최신 API 명세에 맞추어 eval_strategy = "epoch" 형태로 변경하였다.


