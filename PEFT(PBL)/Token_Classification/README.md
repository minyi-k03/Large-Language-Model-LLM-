# PEFT LoRA Token Classification(BioNER)

## Project - Overview
  **Purpose**: Hugging Face의 PEFT(Parameter-Efficient Fine-Tuning) 라이브러리와 LoRA 기법을 적용하여 RoBERTa를 바이오 의학 분야 토큰 분류 실습을 진행하였다.


## Tech-Stack
  **Langague** : Python
  **FrameWork** : Hugging Face(Transformer, PEFT, Dataset, Evaluate)
  **Base Model** : roberta-base
  **Dataset** : ncbi-disease (BioNLP 대체 데이터 셋)
  **Metrics** : seqeval(Precision, Recall, F1, Accuracy)


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



