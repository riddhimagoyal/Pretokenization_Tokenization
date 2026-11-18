# Pretokenization and Tokenization of Indian Languages  
### Using Model-Guided Pretokenization + Cohesion-Weighted BPE

This project explores tokenization strategies tailored for Indian languages, focusing on combining **model-driven pretokenization signals** with a modified **Cohesion-Weighted BPE** algorithm. The goal is to preserve the linguistic structure of Indic scripts and improve downstream tasks like **Machine Translation (MT)**.

---

## 🔥 Motivation

Standard BPE works well for English but performs poorly for Indian languages because:

- Indic scripts are **akshara-based**, not character-based.  
- Words contain **matras, halant, conjunct consonants**, and rich morphology.  
- Vanilla BPE often **breaks meaningful akshara units**.  
- Token fragmentation directly hurts **translation quality**.

This project designs a tokenization pipeline that respects Indian linguistic structure.

---

## 🧠 Overview of Our Approach

We perform two major steps:

### **1. Model-Based Pretokenization**
We train multiple NLP models on Indic datasets and extract **characterwise accuracies** to understand:

- which character combinations are stable,
- which pairs should be merged,
- where natural boundaries should exist.

### **2. Cohesion-Weighted BPE + Machine Translation**
Using model signals + Unicode rules, we create cohesive pretokenized units and apply a modified BPE:


Score(A,B) = Frequency(A,B) * W(A,B)

where the weight **W(A, B)** boosts or penalizes merges based on character categories.

Finally, we evaluate these tokenizers using a **Sanskrit → English** MT task.


## 📚 Datasets

We experiment with three datasets:

| Dataset | Description |
|--------|-------------|
| **Sanskrit** | Monolingual and parallel Sanskrit corpora |
| **Hindi** | Standard Hindi monolingual corpus |
| **Mixed (Sanskrit + Hindi)** | Combined dataset to evaluate multilingual generalization |

---

## 🧩 Step 1: Training Models to Extract Character Accuracies

We trained the following models:

- **IndicBERT v2** (encoder-only)
- **ByT5-small** (byte-level transformer)
- **BiLSTM character model**

From each model, we collected:

- Character prediction accuracy  
- Confusion matrix  
- Misclassification hotspots  
- Stable character-group patterns  

These accuracies were later used to infer **which characters should stay together** during tokenization.

---

## 🧩 Step 2: Model-Guided Pretokenization

Using character-level accuracy:

### **Boost weights (W > 1.0)**
- Consonant + Matra  
- Consonant + Halant  

These combinations form valid **aksharas** and must not be split.

### **Penalty weights (W < 1.0)**
- Vowel → Consonant transitions  
- Boundaries likely representing syllable breaks  

This prevents merging across natural linguistic boundaries.

---

## 🧩 Step 3: Cohesion-Weighted BPE

Modified BPE scoring:


**Why this works:**
- Encourages linguistically-valid merges  
- Reduces harmful splits  
- Maintains akshara structure  
- Makes tokenization consistent across models and languages  

---

## 🧩 Step 4: Machine Translation Evaluation

We test tokenizers via:

### **Task**
Sanskrit → English translation

### **Models**
- Seq2Seq Transformer

### **Metrics**
- BLEU  


---

## 📊 Results (Example Structure)

### **BLEU Score Comparison**
| Tokenizer | BLEU ↑ |
|----------|--------|
| Vanilla BPE | 14 |
| Unicode Pretokenization + BPE | 69 |
| **Cohesion-Weighted BPE (Ours)** | **18** |



---

## 📂 Project Structure
```
NLP_2025/
│
├── dataset/
│   ├── machinetranslation/
│   │   ├── mix_test.csv
│   │   ├── mix_train.csv
│   │   └── mix_val.csv
│   │
│   ├── pretok/
│   │   ├── hindi/
│   │   │   ├── hindi_test.csv
│   │   │   ├── hindi_train.csv
│   │   │   └── hindi_val.csv
│   │   ├── mix/
│   │   │   ├── test_mixed.csv
│   │   │   ├── train_mixed.csv
│   │   │   └── val_mixed.csv
│   │   └── sanskrit/
│   │       ├── test.csv
│   │       ├── train.csv
│   │       └── val.csv
│   │
│   └── tok/
│       └── token_train.csv
│
├── machine translation/
│   ├── cohesive/
│   │   └── output/
│   │       ├── merges.txt_1 … merges.txt_5
│   │       ├── tokenizer.model_1 … tokenizer.model_5
│   │       └── tokenizer.vocab_1 … tokenizer.vocab_5
│   │
│   ├── output_10/
│   │   ├── merges.txt
│   │   ├── tokenizer.model
│   │   └── tokenizer.vocab
│   │
│   ├── mt_normal.py
│   ├── mt_pretokenization.py
│
│   ├── standard bpe/
│   │   ├── bpe/
│   │   ├── mt_bpe/
│   │   ├── seq2seq_out/
│   │   ├── seq2seq_out_no_sandhi/
│   │   ├── bpe_mixed.model
│   │   ├── bpe_mixed.vocab
│   │   ├── mt_bpe_normal.py
│   │   ├── mt_bpe_withoutpretok.py
│   │   └── mt_bpe.py
│
│   ├── unigram/
│   │   ├── mt_unigram/
│   │   ├── seq2seq_out/
│   │   ├── seq2seq_out_no_sandhi/
│   │   ├── seq2seq_out_norm/
│   │   ├── mt_normalization_uni.py
│   │   ├── mt_withoutpre_uni.py
│   │   ├── mt_withpre_uni.py
│   │   └── unigram_mixed.vocab
│
├── pretokenization/
│   ├── bilstm_attention_model/
│   ├── byt5_best_model_hindi/
│   ├── byt5_best_model_mixed/
│   ├── byt5_best_model_sanskrit/
│   ├── byt5_env/
│   ├── best_sandhi_model_hindi.pt
│   ├── best_sandhi_model_mixed.pt
│   ├── best_sandhi_model_sanskrit.pt
│   ├── bilstm.py
│   ├── byt5_hindi.py
│   ├── byt5_mixed.py
│   ├── byt5_sanskrit.py
│   ├── indictbert_hindi.py
│   ├── indictbert_mixed.py
│   ├── indictbert_sanskrit.py
│   ├── indictbert_new.py
│   └── test_predictions.csv
│
├── Tokenization/
│   ├── sentencepiece_to_huggingface.py
│   ├── train_bpe.py
│   ├── train_tokenizer_cohesive.py
│   └── train_unigram.py
│
└── README.md

```




---

# 🚀 Instructions to Run the Project

Below are all the commands to execute the full pipeline.

---


# 1️⃣ Pretokenization

## **IndicBERT-v2**
```bash
python3 indicbert.py
```
## **ByT5-Small**
```bash
python3 byt5-small.py
```

## **BiLSTM Character Model**
```bash
python3 bilstm.py
```


# **2️⃣ Tokenization**

## **Unigram Model**
```bash
python3 train_uni.py
```

## **Standard BPE**
```bash
python3 train_bpe.py
```
## **Cohesive BPE (Proposed Model)**
```bash
python3 train_tokenizer.py
```




# **3️⃣ Machine Translation Experiments**

We evaluated tokenizers for:
✔ Normalised text
✔ Without pretoke
✔ With pretoke

Separated for Unigram, Standard BPE, and Cohesive BPE.

## 🌐 Machine Translation — Unigram
### Normalised
```bash
python3 mt_normalisation_uni.py
```

### Without Pretokenization
```bash
python3 mt_withoutpre_uni.py
```

### With Pretokenization
```bash
python3 mt_withpre_uni.py
```

## 📘 Machine Translation — Standard BPE
### Normalised
```bash
python3 mt_normalisation_bpe.py
```

### Without Pretokenization
```bash
python3 mt_withoutpre_bpe.py
```

### With Pretokenization
```bash
python3 mt_withpre_bpe.py
```

## 🟣 Machine Translation — Cohesive BPE (Proposed Method)
### Normalised
```bash
python3 mt_normalisation_cohesive_bpe.py
```

### Without Pretokenization
```bash
python3 mt_withoutpre_cohesive_bpe.py
```

### With Pretokenization
```bash
python3 mt_withpre_cohesive_bpe.py
```



# 🔮 Future Work

Extend to more Indic languages (Marathi, Tamil, Telugu, Odia)

Explore neural morphological analyzers for dynamic cohesion scoring

Evaluate on ASR, QA, summarization, and LLM finetuning tasks

# ✨ Contributors
```
Riddhima Goyal
Diksha Sharma
Ishika Agarwal
Mehak
Zainab
Manya Gupta
```

---


