# Airplane-Domain Turkish-English Translation Assistant

A domain-specific Turkish-English translation assistant for airplane, airport boarding, cabin, passenger, and flight-service communication.

The project fine-tunes `meta-llama/Llama-3.2-1B-Instruct` with LoRA on an airplane-domain translation dataset and evaluates the resulting model against general-purpose and translation-specific baselines. The goal is not to build a general machine translation system. The goal is to build a compact assistant that behaves reliably in a narrow travel context: given one Turkish or English sentence, return only the translated sentence.

## Why This Project Exists

Airplane and airport interactions are short, practical, and often time-sensitive. Passengers may ask about baggage, seats, food, water, toilets, children, pets, delays, boarding, or arrival information. In that setting, a general chatbot can be problematic because it may:

- answer the request instead of translating it,
- add explanations or role labels,
- refuse harmless translation requests,
- copy the source sentence,
- leak prompt text such as `Translate to Turkish`.

This project treats translation as an application behavior problem, not only a language-generation problem. The model should produce a clean translated sentence that can be read aloud or shown directly on a phone.

## Core Behavior

```text
Input:  a Turkish or English airplane-domain sentence
Output: only the translated sentence
```

Example:

```text
Input:  I need to use the toilet. It cannot wait.
Output: Tuvaleti kullanmam gerekiyor. Bekleyemem.
```

## What Is Included

- Scenario-based airplane-domain Turkish-English dataset.
- Dataset generation notebook with batching, checkpoints, and scenario chunks.
- LoRA fine-tuning notebook for Llama 3.2 1B Instruct.
- Trained LoRA adapter.
- Exported merged-model configuration and tokenizer files.
- Evaluation notebook with translation metrics, semantic metrics, behavior checks, and plots.
- Completed evaluation outputs for a 300-example stratified held-out sample.

## Intended App Experience

The model is designed for a phone-first offline translator interface. The intended user flow is simple:

1. Select translation direction: English to Turkish or Turkish to English.
2. Enter or speak a short travel-related sentence.
3. Receive one clean translated sentence.
4. Optionally use quick phrase buttons for common airport and cabin requests.

The interface concept prioritizes fast passenger-to-crew communication. It avoids long chatbot conversations and focuses on compact output that can be copied, shown on screen, or read aloud. The app-facing behavior checks in the evaluation measure this requirement directly by tracking empty outputs, prompt leakage, source copying, extra explanations, and repetition.

## Repository Structure

```text
airplane_translation_with_LLM/
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

## End-to-End Workflow

```mermaid
flowchart LR
  A["Scenario design"] --> B["Batch data generation"]
  B --> C["Merged translation pairs"]
  C --> D["Chat-format fine-tuning data"]
  D --> E["LoRA fine-tuning"]
  E --> F["Adapter export"]
  F --> G["Merged-model export"]
  G --> H["In-domain evaluation"]
  H --> I["Metrics, plots, and reports"]
```

## Dataset

The final training dataset is:

```text
airplane_translation_dataset/final/fine_tune_chat.jsonl
```

It contains 50,256 chat-style instruction examples. The root-level `airplane_translation_dataset/fine_tune_chat.jsonl` is an earlier smaller dataset and should not be used as the final evaluation source.

| Category | Count |
|---|---:|
| Total examples | 50,256 |
| English to Turkish | 24,485 |
| Turkish to English | 25,771 |
| Held-out split | 1,006 |
| Default evaluation sample | 300 |

### Domain Coverage

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

### Record Format

Each final chat record contains:

- a system instruction defining translation-only behavior,
- a user translation request,
- an assistant reference translation,
- domain metadata,
- tone and difficulty labels,
- scenario and speaker/listener metadata.

Direction is encoded in the prompt, for example `Translate to Turkish:` or `Translate to English:`. It is also available explicitly in `airplane_translation_dataset/final/pairs_merged.csv` through `source_language` and `target_language`.

## Data Generation Method

The dataset was generated with a scenario-based batching process. Instead of creating one large generic list of translations, the airplane domain was divided into focused scenario groups, including:

- baggage placement and lost items,
- seat location and cabin movement,
- food, drinks, allergies, and meal choices,
- toilet and lavatory needs,
- children and family requests,
- delays, arrival time, and connecting flights,
- pets and service animals,
- greetings and basic help phrases.

Each scenario group was generated in smaller English-to-Turkish and Turkish-to-English batches. This was done to:

- improve coverage of realistic airplane interactions,
- avoid overusing the same simple generic examples,
- reduce repeated examples through scenario chunks and merged raw-pair files,
- make generation recoverable through checkpoint files if a notebook run stopped.

Intermediate generation outputs are stored in:

```text
airplane_translation_dataset/scenario_chunks/
airplane_translation_dataset/checkpoints/
```

Final merged pair files are stored in:

```text
airplane_translation_dataset/final/pairs_merged.csv
airplane_translation_dataset/final/raw_pairs_merged.jsonl
```

The merged pairs are then converted into chat-style instruction records for supervised fine-tuning:

```text
airplane_translation_dataset/final/fine_tune_chat.jsonl
```

The chat format is important because the model must learn the full interaction contract, not only isolated string-to-string translation.

## Model and Training

Base model:

```text
meta-llama/Llama-3.2-1B-Instruct
```

Fine-tuning method:

```text
LoRA supervised fine-tuning
```

Main training configuration:

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

Training uses the chat-formatted dataset:

```text
system: translation-only assistant rules
user: Translate to Turkish/English: ...
assistant: translated sentence only
```

LoRA was chosen because the project is focused on narrow domain adaptation and instruction-following behavior. The base model weights remain frozen, and only compact adapter weights are trained.

## Model Artifact Policy

The repository includes the trained LoRA adapter:

```text
airplane_translation_model/adapter_lora/adapter_model.safetensors
```

The full merged model weight is intentionally not included in this repository and is not hosted externally. The local merged weight was too large for the normal repository workflow. To recreate it, rerun the merge/export step in `Notebooks/TRAIN_NOTEBOOK.ipynb` with:

- base model: `meta-llama/Llama-3.2-1B-Instruct`
- adapter folder: `airplane_translation_model/adapter_lora/`
- output folder: `airplane_translation_model/merged_final_model/`

The `merged_final_model/` folder in this repository contains exported configuration and tokenizer files, but not the large merged `model.safetensors` binary.

## Evaluation Design

Evaluation is based on a recreated held-out split:

```text
test_size = 0.02
seed = 42
```

This produces 1,006 held-out examples. The completed exported run uses a fixed 300-example stratified sample to keep runtime manageable while preserving direction and domain coverage.

Every evaluated model receives the same input rows. This makes the in-domain leaderboard directly comparable.

### Models Evaluated

- Fine-tuned Llama 3.2 1B
- Base Llama 3.2 1B
- OPUS-MT tc-big
- NLLB-200 distilled 600M
- M2M100 418M
- mBART-50 many-to-many

### Metrics

Translation quality:

- BLEU
- chrF++
- TER
- ROUGE-L F1
- Token F1
- Edit similarity

Semantic quality:

- BERTScore F1
- COMET

Application behavior:

- empty-output rate,
- source-copy rate,
- prompt-leakage rate,
- extra-explanation rate,
- repetition rate,
- length ratio,
- average latency.

chrF++ is treated as the headline metric because Turkish morphology makes character-level matching especially useful.

## Evaluation Results

The completed exported evaluation uses 300 stratified held-out examples.

| Model | BLEU | chrF++ | TER | BERTScore F1 | COMET | Source Copy | Leakage |
|---|---:|---:|---:|---:|---:|---:|---:|
| Fine-tuned Llama 3.2 1B | 51.23 | 68.38 | 37.37 | 0.9200 | 0.9048 | 0.0% | 0.0% |
| OPUS-MT tc-big | 40.44 | 61.19 | 42.19 | 0.9075 | 0.8910 | 0.0% | 0.0% |
| NLLB-200 distilled 600M | 36.56 | 57.36 | 46.69 | 0.8950 | 0.8738 | 0.0% | 0.0% |
| M2M100 418M | 29.68 | 51.63 | 52.98 | 0.8760 | 0.8407 | 0.0% | 0.0% |
| mBART-50 many-to-many | 26.04 | 46.43 | 59.82 | 0.8557 | 0.7895 | 0.0% | 0.0% |
| Base Llama 3.2 1B | 4.82 | 15.69 | 115.71 | 0.6864 | 0.5766 | 23.0% | 16.0% |

Key interpretation:

- Fine-tuning improved chrF++ by 52.69 points over the base Llama model.
- Fine-tuning improved BLEU by 46.42 points over the base Llama model.
- Base Llama showed source-copy and prompt-leakage failures.
- OPUS-MT and NLLB remain strong dedicated translation baselines.
- The result should be interpreted as in-domain airplane validation, not universal Turkish-English translation superiority.

## Paired Comparison

The evaluation also compares the fine-tuned model against each baseline on the exact same examples.

| Comparison | Mean chrF++ Delta | Fine-Tuned Win Rate | Fine-Tuned Loss Rate |
|---|---:|---:|---:|
| Fine-tuned vs Base Llama | +54.20 | 98.7% | 1.3% |
| Fine-tuned vs M2M100 418M | +18.08 | 74.0% | 22.7% |
| Fine-tuned vs mBART-50 | +23.12 | 79.7% | 18.3% |
| Fine-tuned vs NLLB-200 distilled 600M | +11.36 | 63.0% | 30.0% |
| Fine-tuned vs OPUS-MT tc-big | +6.75 | 57.0% | 34.7% |

Paired comparison is stricter than comparing unrelated averages because it checks model behavior on the same input rows.

## Qualitative Example

Example from `evaluation_outputs/qualitative_examples.csv`:

| Field | Text |
|---|---|
| Source | I need to use the toilet. It cannot wait. |
| Reference | Tuvaleti kullanmam gerekiyor. Bekleyemez. |
| Base Llama | I cannot translate to Turkish. |
| Fine-tuned Llama | Tuvaleti kullanmam gerekiyor. Bekleyemem. |
| OPUS-MT | Tuvaleti kullanmam lazım, bekleyemez. |
| NLLB distilled | Tuvalete gitmem gerekiyor. |

This example shows why behavior checks matter. The base model gives a refusal-style response, while the fine-tuned model follows the translation-only instruction.

## Reproduction Guide

The notebooks are designed for Google Colab.

### 1. Project Location

The notebooks were developed with this Google Drive folder:

```text
/content/drive/MyDrive/MIS48B+
```

If your folder has a different name, update `PROJECT_ROOT` in the setup cells before running training or evaluation.

The notebooks expect paths similar to:

```text
/content/drive/MyDrive/MIS48B+/airplane_translation_dataset/final/fine_tune_chat.jsonl
/content/drive/MyDrive/MIS48B+/airplane_translation_model/adapter_lora
/content/drive/MyDrive/MIS48B+/airplane_translation_model/merged_final_model
/content/drive/MyDrive/MIS48B+/evaluation_outputs
```

### 2. Generate or Verify the Dataset

Run:

```text
Notebooks/TR_EN_EN_TR_input_generation.ipynb
```

If the final dataset already exists, verify:

```text
airplane_translation_dataset/final/fine_tune_chat.jsonl
```

### 3. Train or Regenerate the Model

Run:

```text
Notebooks/TRAIN_NOTEBOOK.ipynb
```

Hugging Face access may be required for `meta-llama/Llama-3.2-1B-Instruct`.

The notebook trains the LoRA adapter and can regenerate the merged model weight locally.

### 4. Run Evaluation

Run:

```text
Notebooks/EVALUATION_METRICS.ipynb
```

Default evaluation:

```text
RUN_FULL_TEST_SET = False
EVAL_SAMPLE_SIZE = 300
```

Full held-out evaluation:

```text
RUN_FULL_TEST_SET = True
```

## Output Files

Evaluation outputs are saved under:

```text
evaluation_outputs/
```

| File | Purpose |
|---|---|
| `predictions.csv` | Model outputs for each evaluated example |
| `metrics_summary.csv` | Overall metrics by model |
| `leaderboard.csv` | Presentation-ready model leaderboard |
| `grouped_metrics.csv` | Metrics grouped by direction, domain, difficulty, and tone |
| `paired_model_comparison.csv` | Paired fine-tuned-vs-baseline comparison |
| `qualitative_examples.csv` | Example translations |
| `internet_benchmarks.csv` | External benchmark context only |
| `comet_scores.csv` | Per-example COMET scores |
| `evaluation_report.md` | Markdown evaluation summary |
| `*.png` | Evaluation plots, heatmaps, and comparison charts |

## Scientific Notes

- Direct in-domain results are the main evidence because every model is evaluated on the same airplane-domain examples.
- External benchmark rows are context only because they use different datasets and metric settings.
- The dataset is synthetic, so the model may have a style advantage over general translation systems.
- Automatic metrics can penalize valid paraphrases.
- Human evaluation is needed for stronger claims about adequacy, fluency, and real-world usefulness.

## Limitations

- The held-out split is validation-style, not a blind human-written test set.
- The completed exported run uses 300 examples rather than the full 1,006-example held-out split.
- The full merged model weight is not distributed in this repository.
- Larger baselines require more runtime and GPU memory.
- A production aviation tool would need safety testing, human review, and escalation behavior for urgent or sensitive requests.

## Future Work

- Run the full 1,006-example held-out split.
- Build a blind human-written airplane-domain test set.
- Add human evaluation for adequacy, fluency, and instruction following.
- Create a simple live demo app for passenger-to-crew translation.
- Add safety handling for urgent, ambiguous, or sensitive cabin requests.
