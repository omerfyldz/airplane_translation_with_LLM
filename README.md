# Airplane-Domain Turkish-English Translation Assistant

This project contains a domain-specific Turkish-English translation assistant for airplane, airport boarding, cabin, passenger, and flight-service situations. It was developed for MIS 48B: Generative AI and Deep Learning, Spring 2025.

The system fine-tunes `meta-llama/Llama-3.2-1B-Instruct` with LoRA on an airplane-domain translation dataset and evaluates the fine-tuned model against general and translation-specific baselines.

## Team Members

| Student ID | Name |
|---|---|
| 2022300318 | Umut Bülbül |
| 2019301183 | Ahmet Mirza Duru |
| 2021300162 | Ömer Faruk Yıldız |

## Project Goal

Airplane cabins involve short, practical, and sometimes time-sensitive multilingual interactions. Passengers may ask about baggage, seat location, water, food restrictions, toilets, children, pets, delays, or arrival information.

The goal of this project is to build a translation assistant that follows a strict application behavior:

```text
Given a Turkish or English airplane-domain sentence, return only the translated sentence.
```

This is different from a general chatbot. The model should not answer the passenger request, add explanations, copy the source sentence, or include prompt text.

The intended product form is phone-based. A passenger or crew member should be able to type or paste a short sentence on a mobile device and immediately receive a clean translated sentence that can be read or shown to another person. This is why the project emphasizes concise outputs, no extra explanation, low latency, and behavior checks such as source-copy and prompt-leakage detection.

## Main Contributions

- A domain-specific Turkish-English airplane translation dataset.
- LoRA fine-tuning of Llama 3.2 1B Instruct.
- A merged final model and separate LoRA adapter output.
- A reproducible evaluation notebook.
- Direct in-domain comparison against several baseline models.
- Automatic translation metrics and application-behavior checks.
- A phone-first application direction for passenger-to-crew translation.
- Reproducible notebooks and exported evaluation outputs.

## Project Structure

```text
MIS48B+/
  README.md

  Notebooks/
    TR_EN_EN_TR_input_generation.ipynb
    TRAIN_NOTEBOOK.ipynb
    EVALUATION_METRICS.ipynb
    EVALUATION_METRICS_original_backup.ipynb
    EVALUATION_METRICS_runned (1).ipynb

  airplane_translation_dataset/
    final/
      fine_tune_chat.jsonl
      pairs_merged.csv
      raw_pairs_merged.jsonl
    checkpoints/
    scenario_chunks/
    fine_tune_chat.jsonl
    pairs.csv
    raw_pairs.jsonl
    generation_progress.json

  airplane_translation_model/
    adapter_lora/
      adapter_config.json
      adapter_model.safetensors
      tokenizer.json
      tokenizer_config.json
      chat_template.jinja
      README.md
    merged_final_model/
      config.json
      generation_config.json
      tokenizer.json
      tokenizer_config.json
      chat_template.jinja

  evaluation_outputs/
    predictions.csv
    metrics_summary.csv
    leaderboard.csv
    grouped_metrics.csv
    paired_model_comparison.csv
    qualitative_examples.csv
    internet_benchmarks.csv
    comet_scores.csv
    evaluation_report.md
    *.png

```

## Dataset

The final training dataset is:

```text
airplane_translation_dataset/final/fine_tune_chat.jsonl
```

It contains 50,256 chat-style instruction examples. The root-level `airplane_translation_dataset/fine_tune_chat.jsonl` is a smaller earlier dataset and should not be used as the final evaluation source.

| Category | Count |
|---|---:|
| Total examples | 50,256 |
| English to Turkish | 24,485 |
| Turkish to English | 25,771 |
| Held-out split | 1,006 |
| Default evaluation sample | 300 |

The dataset covers these airplane-domain categories:

| Domain | Examples |
|---|---:|
| Food and drink | 8,055 |
| Flight time, delay, and arrival | 7,377 |
| Seat and cabin location | 7,253 |
| Baggage and personal items | 6,634 |
| Comfort and cabin needs | 6,403 |
| Children and family | 5,615 |
| Toilet and lavatory | 4,491 |
| Greetings and basic conversation | 2,693 |
| Pets and service animals | 1,735 |

Each record contains:

- system instruction,
- translation prompt,
- assistant reference translation,
- domain metadata,
- tone and difficulty labels,
- scenario and speaker/listener information.

Direction is encoded in the user prompt, for example `Translate to Turkish:` or `Translate to English:`, and is also stored explicitly in `airplane_translation_dataset/final/pairs_merged.csv` through `source_language` and `target_language`.

## Data Generation Workflow

The dataset was generated with a scenario-based batching process. Instead of creating one large generic translation list, the project divided airplane communication into focused scenario groups such as:

- baggage placement and lost items,
- seat location and cabin movement,
- food, drinks, allergies, and meal choices,
- toilet and lavatory needs,
- children and family requests,
- delays, arrival time, and connecting flights,
- pets and service animals,
- greetings and basic help phrases.

Each scenario group was generated in smaller batches for both English-to-Turkish and Turkish-to-English. This batching approach had three practical goals:

- improve coverage of real airplane situations rather than repeating only simple generic phrases,
- reduce repeated examples by organizing generation around scenario chunks and merged raw-pair files,
- make generation recoverable in Colab through checkpoint files if a run stopped or failed.

The generation notebook stores intermediate files under:

```text
airplane_translation_dataset/scenario_chunks/
airplane_translation_dataset/checkpoints/
```

After generation, raw examples are merged into:

```text
airplane_translation_dataset/final/pairs_merged.csv
airplane_translation_dataset/final/raw_pairs_merged.jsonl
```

The final step converts the merged translation pairs into chat-style instruction records:

```text
airplane_translation_dataset/final/fine_tune_chat.jsonl
```

This chat format matters because the model is trained to follow an application instruction, not only to translate isolated strings. Each training example contains a system instruction, a user translation request, and an assistant response that contains only the translated sentence.

## Model and Training

Base model:

```text
meta-llama/Llama-3.2-1B-Instruct
```

Fine-tuning method:

```text
LoRA supervised fine-tuning
```

Main training settings:

| Setting | Value |
|---|---|
| LoRA rank | 32 |
| LoRA alpha | 64 |
| LoRA dropout | 0.05 |
| Target modules | q, k, v, o, gate, up, down projections |
| Max sequence length | 256 |
| Epochs | 1 |
| Learning rate | 2e-4 |
| Batch size | 4 |
| Gradient accumulation | 4 |
| Train/test split | 98% / 2% |
| Split seed | 42 |

Model output folders:

```text
airplane_translation_model/adapter_lora/
airplane_translation_model/merged_final_model/
```

The adapter folder contains the trained LoRA adapter. The merged final model folder contains the exported configuration and tokenizer files. The full merged model weight can be recreated by rerunning the training notebook's merge/export step using the base Llama model and the saved LoRA adapter.

The `.safetensors` model files are stored with Git LFS because they are binary model artifacts. Install Git LFS before cloning or pulling the repository if the model weights are needed locally.

Training uses the chat-formatted dataset so the model learns the full interaction pattern:

```text
system: translation-only assistant rules
user: Translate to Turkish/English: ...
assistant: translated sentence only
```

LoRA was used because the goal is narrow domain and behavior adaptation rather than training a new translation model from scratch. The base model weights stay frozen, and only small adapter weights are trained. This makes the experiment practical in Colab while still teaching the model to follow the airplane-domain translation-only behavior.

After training, the project keeps two model-output folders:

- `adapter_lora/`: compact LoRA adapter and tokenizer files,
- `merged_final_model/`: exported merged-model configuration and tokenizer files; the merged model weight can be regenerated from the adapter and base model.

## Notebooks

### Dataset Generation

```text
Notebooks/TR_EN_EN_TR_input_generation.ipynb
```

Generates airplane-domain Turkish-English and English-Turkish translation examples. It creates scenario chunks, checkpoint files, raw pairs, merged pairs, and the final chat-format fine-tuning dataset.

The generation process is intentionally batched by scenario so the dataset covers many airplane situations and avoids depending on a small number of repeated generic examples.

### Training

```text
Notebooks/TRAIN_NOTEBOOK.ipynb
```

Fine-tunes Llama 3.2 1B Instruct with LoRA and exports the adapter and merged model.

Hugging Face access may be required for `meta-llama/Llama-3.2-1B-Instruct`.

### Evaluation

```text
Notebooks/EVALUATION_METRICS.ipynb
```

Runs direct in-domain evaluation and creates all metric outputs under:

```text
evaluation_outputs/
```

The notebook recreates the original held-out split with:

```text
test_size = 0.02
seed = 42
```

The default evaluation uses a fixed stratified 300-example sample. The notebook also includes a switch for evaluating the full held-out split.

The file `Notebooks/EVALUATION_METRICS_runned (1).ipynb` is the completed evaluation run copy. The file `Notebooks/EVALUATION_METRICS_original_backup.ipynb` is the original backup.

## Evaluation Metrics

The evaluation uses both translation-quality metrics and application-behavior checks.

Translation metrics:

- BLEU
- chrF++
- TER
- BERTScore F1
- COMET
- ROUGE-L F1
- Token F1
- Edit similarity

Behavior and diagnostic metrics:

- Empty output rate
- Source-copy rate
- Prompt-leakage rate
- Extra-explanation rate
- Repetition rate
- Length ratio
- Average latency

chrF++ is treated as the headline metric because Turkish morphology makes character-level matching especially useful.

## Current Evaluation Results

The current completed exported evaluation uses 300 stratified held-out examples.

| Model | BLEU | chrF++ | TER | BERTScore F1 | COMET | Source Copy | Leakage |
|---|---:|---:|---:|---:|---:|---:|---:|
| Fine-tuned Llama 3.2 1B | 51.23 | 68.38 | 37.37 | 0.9200 | 0.9048 | 0.0% | 0.0% |
| OPUS-MT tc-big | 40.44 | 61.19 | 42.19 | 0.9075 | 0.8910 | 0.0% | 0.0% |
| NLLB-200 distilled 600M | 36.56 | 57.36 | 46.69 | 0.8950 | 0.8738 | 0.0% | 0.0% |
| M2M100 418M | 29.68 | 51.63 | 52.98 | 0.8760 | 0.8407 | 0.0% | 0.0% |
| mBART-50 many-to-many | 26.04 | 46.43 | 59.82 | 0.8557 | 0.7895 | 0.0% | 0.0% |
| Base Llama 3.2 1B | 4.82 | 15.69 | 115.71 | 0.6864 | 0.5766 | 23.0% | 16.0% |

Interpretation:

- Fine-tuning improved chrF++ by 52.69 points over the base Llama model.
- Fine-tuning improved BLEU by 46.42 points over the base Llama model.
- Base Llama had source-copy and prompt-leakage problems.
- OPUS-MT and NLLB are strong dedicated translation baselines.
- The correct claim is in-domain airplane validation, not universal Turkish-English translation superiority.

## Paired Comparison

The evaluation also compares the fine-tuned model with each baseline on the exact same examples.

| Comparison | Mean chrF++ Delta | Fine-Tuned Win Rate | Fine-Tuned Loss Rate |
|---|---:|---:|---:|
| Fine-tuned vs Base Llama | +54.20 | 98.7% | 1.3% |
| Fine-tuned vs M2M100 418M | +18.08 | 74.0% | 22.7% |
| Fine-tuned vs mBART-50 | +23.12 | 79.7% | 18.3% |
| Fine-tuned vs NLLB-200 distilled 600M | +11.36 | 63.0% | 30.0% |
| Fine-tuned vs OPUS-MT tc-big | +6.75 | 57.0% | 34.7% |

Paired comparison is important because it tests models on the same input rows rather than comparing unrelated averages.

## Example Output

Example from `evaluation_outputs/qualitative_examples.csv`:

| Field | Text |
|---|---|
| Source | I need to use the toilet. It cannot wait. |
| Reference | Tuvaleti kullanmam gerekiyor. Bekleyemez. |
| Base Llama | I cannot translate to Turkish. |
| Fine-tuned Llama | Tuvaleti kullanmam gerekiyor. Bekleyemem. |
| OPUS-MT | Tuvaleti kullanmam lazım, bekleyemez. |
| NLLB distilled | Tuvalete gitmem gerekiyor. |

This example shows why application behavior matters. The base model gives a refusal-style response, while the fine-tuned model follows the translation-only instruction.

## How to Reproduce

The recommended workflow is to run the notebooks in Google Colab.

### Step 1: Prepare the project in Google Drive

Place the project folder in:

```text
/content/drive/MyDrive/MIS48B+
```

The evaluation notebook expects paths similar to:

```text
/content/drive/MyDrive/MIS48B+/airplane_translation_dataset/final/fine_tune_chat.jsonl
/content/drive/MyDrive/MIS48B+/airplane_translation_model/merged_final_model
/content/drive/MyDrive/MIS48B+/evaluation_outputs
```

### Step 2: Generate or verify the dataset

Run:

```text
Notebooks/TR_EN_EN_TR_input_generation.ipynb
```

If the final dataset already exists, verify:

```text
airplane_translation_dataset/final/fine_tune_chat.jsonl
```

### Step 3: Train the model

Run:

```text
Notebooks/TRAIN_NOTEBOOK.ipynb
```

If Hugging Face asks for access to Llama, log in with a Hugging Face token in Colab.

### Step 4: Run evaluation

Run:

```text
Notebooks/EVALUATION_METRICS.ipynb
```

For faster testing, use the default 300-example evaluation sample. For the full held-out set, enable:

```text
RUN_FULL_TEST_SET = True
```

## Outputs

Evaluation outputs are saved under:

```text
evaluation_outputs/
```

Important files:

| File | Purpose |
|---|---|
| `predictions.csv` | Model outputs for each evaluated example |
| `metrics_summary.csv` | Overall metrics by model |
| `leaderboard.csv` | Presentation-ready model leaderboard |
| `grouped_metrics.csv` | Metrics grouped by direction, domain, difficulty, and tone |
| `paired_model_comparison.csv` | Paired fine-tuned-vs-baseline comparison |
| `qualitative_examples.csv` | Example translations for report/demo |
| `internet_benchmarks.csv` | External benchmark context only |
| `comet_scores.csv` | Per-example COMET scores |
| `evaluation_report.md` | Markdown summary of the evaluation |
| `*.png` | Evaluation plots, heatmaps, and comparison charts |

## Limitations

- The dataset is synthetic.
- The held-out split is validation-style, not a blind human-written test set.
- The fine-tuned model may have a style advantage because references come from the same data-generation pipeline.
- Automatic metrics can penalize valid paraphrases.
- Human evaluation is needed for stronger scientific claims.
- Larger baselines require more compute.
- A production aviation tool would need safety testing, human review, and clear escalation behavior.

## Future Work

- Run the full 1,006-example held-out split.
- Build a blind human-written airplane-domain test set.
- Add human evaluation for adequacy, fluency, and instruction following.
- Create a simple live demo app for passenger-to-crew translation.
- Add safety handling for urgent, ambiguous, or sensitive cabin requests.

## Course Context

This project was created for:

```text
MIS 48B: Generative AI & Deep Learning, Spring 2025
```

It aligns with the course themes of:

- Large language models
- LLM fine-tuning
- AI-powered assistants
- Domain-specific generative AI applications
- Evaluation of generative AI systems
