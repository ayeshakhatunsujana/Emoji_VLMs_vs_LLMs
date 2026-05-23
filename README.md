# Emoji_VLMs_vs_LLMs
#QWEN
# Local Qwen Emoji Visual-Semantic Description Pipeline

This is the Qwen version of the local emoji description and sentiment/sarcasm experiment pipeline.

It generates the same three description fields used in the Llama version:

1. **C_v**: Plain visual description from the Qwen vision model.
2. **C_v_st**: Smart visual-emotion / visual-communication description from the Qwen vision model.
3. **C_t**: Textual/social emoji meaning from the Qwen text model.

The final emoji description CSV has this format:

```text
file_name, code_hex, unicode_char, unicode_name, official_names, C_v, C_v_st, C_t
```

## 1. Folder structure

```text
emoji_qwen_project/
│
├── emoji/
│   ├── 1F600.png
│   ├── 1F602.png
│   ├── 1F62D.png
│   └── ...
│
├── data/
│   ├── train_split_70.csv
│   ├── sarcasm_full_with_emoji.csv
│   └── tweet_eval_train_last250.csv
│
├── outputs/
│   ├── results_qwen_10.csv
│   ├── results_qwen_full.csv
│   ├── sentiment3_predictions_qwen_full.csv
│   └── sentiment3_results_qwen_full.json
│
├── requirements.txt
└── src/
    ├── __init__.py
    ├── run_all_qwen.py
    ├── vlm_caption_qwen.py
    ├── text_llm_describe_qwen.py
    └── run_experiments_qwen_captions.py
```

## 2. Install Ollama

### macOS / Linux

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Start Ollama:

```bash
ollama serve
```

If Ollama is already running, this may say the port is already in use. That is okay.

## 3. Pull Qwen models

Recommended local setup:

```bash
ollama pull qwen2.5vl:7b
ollama pull qwen2.5:3b
```

For a smaller/slower computer, try:

```bash
ollama pull qwen2.5vl:3b
ollama pull qwen2.5:1.5b
```

For stronger GPU/server, try:

```bash
ollama pull qwen2.5vl:32b
ollama pull qwen2.5:7b
```

## 4. Create Python environment

```bash
python -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt
```

## 5. Put emoji images in `emoji/`

Example filenames:

```text
emoji/1F600.png
emoji/1F602.png
emoji/1F62D.png
emoji/1F44D.png
emoji/1F469_200D_2764_FE0F_200D_1F468.png
```

The filename is important because the code converts the hex filename into the actual emoji character and Unicode information.

## 6. Test with 10 emoji images

```bash
python -m src.run_all_qwen \
  --emoji_dir emoji \
  --out_csv outputs/results_qwen_10.csv \
  --vlm_model qwen2.5vl:7b \
  --llm_model qwen2.5:3b \
  --max_images 10
```

## 7. Run full emoji description generation

```bash
python -m src.run_all_qwen \
  --emoji_dir emoji \
  --out_csv outputs/results_qwen_full.csv \
  --vlm_model qwen2.5vl:7b \
  --llm_model qwen2.5:3b
```

Resume if interrupted:

```bash
python -m src.run_all_qwen \
  --emoji_dir emoji \
  --out_csv outputs/results_qwen_full.csv \
  --vlm_model qwen2.5vl:7b \
  --llm_model qwen2.5:3b \
  --resume
```

## 8. Run 3-class sentiment experiment

```bash
python -m src.run_experiments_qwen_captions \
  --train_csv data/train_split_70.csv \
  --emoji_csv outputs/results_qwen_full.csv \
  --task_type sentiment3 \
  --model_name qwen2.5:3b \
  --out_csv outputs/sentiment3_predictions_qwen_full.csv \
  --out_xlsx outputs/sentiment3_predictions_qwen_full.xlsx \
  --out_json outputs/sentiment3_results_qwen_full.json \
  --skip_missing_descriptions
```

## 9. Run sarcasm experiment

```bash
python -m src.run_experiments_qwen_captions \
  --train_csv data/sarcasm_full_with_emoji.csv \
  --emoji_csv outputs/results_qwen_full.csv \
  --task_type sarcasm2 \
  --model_name qwen2.5:3b \
  --out_csv outputs/sarcasm_predictions_qwen_full.csv \
  --out_xlsx outputs/sarcasm_predictions_qwen_full.xlsx \
  --out_json outputs/sarcasm_results_qwen_full.json \
  --skip_missing_descriptions
```

## 10. Run 2-class sentiment experiment

```bash
python -m src.run_experiments_qwen_captions \
  --train_csv data/tweet_eval_train_last250.csv \
  --emoji_csv outputs/results_qwen_full.csv \
  --task_type sentiment2 \
  --model_name qwen2.5:3b \
  --out_csv outputs/sentiment2_predictions_qwen_full.csv \
  --out_xlsx outputs/sentiment2_predictions_qwen_full.xlsx \
  --out_json outputs/sentiment2_results_qwen_full.json \
  --skip_missing_descriptions
```

## 11. Experiment conditions

The script evaluates the same eight conditions:

```text
a_text_only
b_text_plus_emoji
c_text_plus_emoji_Cv
d_text_plus_emoji_Cvst
e_text_plus_emoji_Ct
f_text_plus_emoji_Cv_Ct
g_text_plus_emoji_Cvst_Ct
h_text_plus_emoji_Cv_Cvst_Ct
```

## 12. Main outputs

For each experiment, the code saves:

```text
CSV:  all predictions and condition inputs
XLSX: predictions sheet + summary sheet
JSON: metric details, confusion matrix, classification report
```

Main metric:

```text
macro_f1
```

Use macro-F1 for your paper table because your sentiment/sarcasm labels may be imbalanced.

## 13. Suggested fair comparison with Llama

Run the same files with Llama and Qwen:

```text
Llama emoji CSV: outputs/results_llama_full.csv
Qwen emoji CSV:  outputs/results_qwen_full.csv
```

Then compare:

```text
C_v quality
C_v_st quality
C_t quality
sentiment3 macro-F1
sarcasm2 macro-F1
sentiment2 macro-F1
confusion matrix
which emojis improve only in Qwen
which emojis improve only in Llama
```

## 14. Troubleshooting

### Problem: model not found

Run:

```bash
ollama list
```

Then pull the missing model:

```bash
ollama pull qwen2.5vl:7b
ollama pull qwen2.5:3b
```

### Problem: Ollama connection error

Start Ollama:

```bash
ollama serve
```

### Problem: Qwen vision model is slow

Use smaller model:

```bash
ollama pull qwen2.5vl:3b
```

Then run:

```bash
python -m src.run_all_qwen \
  --emoji_dir emoji \
  --out_csv outputs/results_qwen_full.csv \
  --vlm_model qwen2.5vl:3b \
  --llm_model qwen2.5:1.5b
```
