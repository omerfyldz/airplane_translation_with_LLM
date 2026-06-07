# Airplane Translation - Evaluation Report

Generated: 2026-06-06

## 1. Setup
The project is evaluated as a Turkish<->English airplane-domain translation system. The held-out split was
recreated from `fine_tune_chat.jsonl` with the training settings `test_size=0.02`, `seed=42`.

- Full held-out size: **1006** examples
- Evaluated in this run: **300** examples (RUN_FULL_TEST_SET = False)
- Models evaluated: Fine-tuned Llama 3.2 1B, OPUS-MT tc-big, NLLB-200 distilled 600M, M2M100 418M, mBART-50 many-to-many, Base Llama 3.2 1B
- COMET model: Unbabel/wmt22-comet-da

## 2. Headline result
The best in-domain model by chrF++ is **Fine-tuned Llama 3.2 1B** (chrF++ = 68.4). Fine-tuning improved chrF++ by **+52.7** and BLEU by **+46.4** points over the base Llama-3.2-1B.

### Leaderboard (sorted by chrF++)
| model_name              |   n |   BLEU |   chrF++ | chrF++_95CI   |    TER |   BERTScore_F1 |   COMET |   ROUGE_L_F1 |   Token_F1 |   Edit_similarity |   Length_ratio |   Length_error |   Repetition_rate |   src_copy% |   leak% |   latency_s |
|:------------------------|----:|-------:|---------:|:--------------|-------:|---------------:|--------:|-------------:|-----------:|------------------:|---------------:|---------------:|------------------:|------------:|--------:|------------:|
| Fine-tuned Llama 3.2 1B | 300 |  51.23 |    68.38 | [65.6, 70.7]  |  37.37 |         0.92   |  0.9048 |       0.717  |     0.7302 |            0.8207 |          1.015 |          0.096 |            0.0005 |           0 |       0 |       0.522 |
| OPUS-MT tc-big          | 300 |  40.44 |    61.19 | [58.9, 63.3]  |  42.19 |         0.9075 |  0.891  |       0.6609 |     0.6767 |            0.7911 |          0.977 |          0.087 |            0.001  |           0 |       0 |       0.119 |
| NLLB-200 distilled 600M | 300 |  36.56 |    57.36 | [54.8, 59.7]  |  46.69 |         0.895  |  0.8738 |       0.6109 |     0.6251 |            0.7642 |          0.98  |          0.099 |            0.0006 |           0 |       0 |       0.19  |
| M2M100 418M             | 300 |  29.68 |    51.63 | [49.2, 53.9]  |  52.98 |         0.876  |  0.8407 |       0.5525 |     0.5668 |            0.7113 |          0.965 |          0.127 |            0.0011 |           0 |       0 |       0.205 |
| mBART-50 many-to-many   | 300 |  26.04 |    46.43 | [44.3, 48.7]  |  59.82 |         0.8557 |  0.7895 |       0.4835 |     0.4975 |            0.6682 |          1.002 |          0.13  |            0.0013 |           0 |       0 |       0.26  |
| Base Llama 3.2 1B       | 300 |   4.82 |    15.69 | [14.0, 17.6]  | 115.71 |         0.6864 |  0.5766 |       0.1044 |     0.1123 |            0.3051 |          1.17  |          0.526 |            0.0058 |          23 |      16 |       0.471 |

## 3. Metric definitions
- **BLEU** - word n-gram overlap (0-100, higher better).
- **chrF++** - character + word n-gram F-score (0-100, higher better); primary metric for morphologically rich Turkish.
- **TER** - translation edit rate (lower better).
- **BERTScore F1** - multilingual semantic similarity (0-1, higher better).
- **COMET** - learned neural quality estimate, if enabled (higher better).
- **ROUGE-L / Token F1 / Edit similarity** - additional lexical and character-level similarity checks (higher better).
- **Length ratio / length error / repetition rate** - diagnostics for too-short, too-long, or repetitive outputs.
- **Behavior rates** - source-copy / prompt-leakage / empty / extra-explanation (all lower better).

## 4. Overall in-domain metrics
| model_name              |   n |    bleu |   chrfpp |      ter |   bertscore_f1 |   comet |   avg_rouge_l_f1 |   avg_token_f1 |   avg_edit_similarity |   avg_length_ratio |   avg_abs_length_ratio_error |   avg_repetition_rate |   normalized_exact_match_rate |   empty_output_rate |   source_copy_rate |   prompt_leakage_rate |   extra_explanation_rate |   avg_latency_seconds |
|:------------------------|----:|--------:|---------:|---------:|---------------:|--------:|-----------------:|---------------:|----------------------:|-------------------:|-----------------------------:|----------------------:|------------------------------:|--------------------:|-------------------:|----------------------:|-------------------------:|----------------------:|
| Fine-tuned Llama 3.2 1B | 300 | 51.2343 |  68.3761 |  37.3652 |         0.92   |  0.9048 |           0.717  |         0.7302 |                0.8207 |             1.0154 |                       0.0965 |                0.0005 |                        0.2233 |                   0 |               0    |                  0    |                   0      |                0.5223 |
| OPUS-MT tc-big          | 300 | 40.4411 |  61.1946 |  42.1941 |         0.9075 |  0.891  |           0.6609 |         0.6767 |                0.7911 |             0.9773 |                       0.0872 |                0.001  |                        0.14   |                   0 |               0    |                  0    |                   0      |                0.1192 |
| NLLB-200 distilled 600M | 300 | 36.5572 |  57.3641 |  46.6948 |         0.895  |  0.8738 |           0.6109 |         0.6251 |                0.7642 |             0.9797 |                       0.0987 |                0.0006 |                        0.11   |                   0 |               0    |                  0    |                   0      |                0.1903 |
| M2M100 418M             | 300 | 29.6755 |  51.6276 |  52.977  |         0.876  |  0.8407 |           0.5525 |         0.5668 |                0.7113 |             0.965  |                       0.1266 |                0.0011 |                        0.0633 |                   0 |               0    |                  0    |                   0      |                0.2051 |
| mBART-50 many-to-many   | 300 | 26.0355 |  46.4315 |  59.8218 |         0.8557 |  0.7895 |           0.4835 |         0.4975 |                0.6682 |             1.0022 |                       0.1305 |                0.0013 |                        0.0367 |                   0 |               0    |                  0    |                   0      |                0.2596 |
| Base Llama 3.2 1B       | 300 |  4.8165 |  15.6879 | 115.706  |         0.6864 |  0.5766 |           0.1044 |         0.1123 |                0.3051 |             1.1697 |                       0.5257 |                0.0058 |                        0.0033 |                   0 |               0.23 |                  0.16 |                   0.0567 |                0.4714 |

## 5. Paired fine-tuned-vs-baseline comparison
Positive deltas mean the fine-tuned model scored higher than the baseline on the exact same examples. Win rate is the share of examples where the fine-tuned model scored higher.

| comparison                                         |   n_paired_examples |   mean_delta_sentence_chrfpp |   win_rate_sentence_chrfpp |   mean_delta_token_f1 |   win_rate_token_f1 |   mean_delta_bertscore_f1 |   win_rate_bertscore_f1 |   mean_delta_comet |   win_rate_comet |
|:---------------------------------------------------|--------------------:|-----------------------------:|---------------------------:|----------------------:|--------------------:|--------------------------:|------------------------:|-------------------:|-----------------:|
| Fine-tuned Llama 3.2 1B vs Base Llama 3.2 1B       |                 300 |                      54.2041 |                     0.9867 |                0.6179 |              0.9667 |                    0.2336 |                  0.9933 |             0.3282 |           0.9767 |
| Fine-tuned Llama 3.2 1B vs M2M100 418M             |                 300 |                      18.0758 |                     0.74   |                0.1634 |              0.69   |                    0.0441 |                  0.7167 |             0.0641 |           0.72   |
| Fine-tuned Llama 3.2 1B vs mBART-50 many-to-many   |                 300 |                      23.1248 |                     0.7967 |                0.2327 |              0.77   |                    0.0643 |                  0.78   |             0.1153 |           0.7967 |
| Fine-tuned Llama 3.2 1B vs NLLB-200 distilled 600M |                 300 |                      11.3554 |                     0.63   |                0.1051 |              0.5867 |                    0.0251 |                  0.6167 |             0.0309 |           0.6267 |
| Fine-tuned Llama 3.2 1B vs OPUS-MT tc-big          |                 300 |                       6.751  |                     0.57   |                0.0536 |              0.52   |                    0.0125 |                  0.56   |             0.0138 |           0.5467 |

## 6. By direction
| model_name              | group_value      |   n |    bleu |   chrfpp |      ter |   bertscore_f1 |   avg_token_f1 |   avg_rouge_l_f1 |
|:------------------------|:-----------------|----:|--------:|---------:|---------:|---------------:|---------------:|-----------------:|
| Fine-tuned Llama 3.2 1B | English->Turkish | 149 | 42.5702 |  66.3225 |  42.3488 |         0.9061 |         0.6665 |           0.6548 |
| OPUS-MT tc-big          | English->Turkish | 149 | 35.6977 |  62.5137 |  45.1957 |         0.8993 |         0.6339 |           0.6233 |
| NLLB-200 distilled 600M | English->Turkish | 149 | 28.9442 |  56.7202 |  53.2622 |         0.8802 |         0.5584 |           0.5468 |
| M2M100 418M             | English->Turkish | 149 | 21.9519 |  50.2282 |  62.2776 |         0.8557 |         0.4811 |           0.4698 |
| mBART-50 many-to-many   | English->Turkish | 149 | 14.133  |  40.8135 |  72.9537 |         0.8228 |         0.3735 |           0.3634 |
| Base Llama 3.2 1B       | English->Turkish | 149 |  0.4537 |  12.4078 | 136.418  |         0.6672 |         0.042  |           0.0391 |
| Fine-tuned Llama 3.2 1B | Turkish->English | 151 | 56.1871 |  69.9295 |  34.1085 |         0.9337 |         0.7931 |           0.7785 |
| OPUS-MT tc-big          | Turkish->English | 151 | 43.2432 |  59.429  |  40.2326 |         0.9157 |         0.7189 |           0.6981 |
| NLLB-200 distilled 600M | Turkish->English | 151 | 41.0021 |  57.4397 |  42.4031 |         0.9096 |         0.6909 |           0.6743 |
| M2M100 418M             | Turkish->English | 151 | 34.2189 |  52.3517 |  46.8992 |         0.8959 |         0.6514 |           0.6342 |
| mBART-50 many-to-many   | Turkish->English | 151 | 32.8352 |  51.2057 |  51.2403 |         0.8883 |         0.6199 |           0.6019 |
| Base Llama 3.2 1B       | Turkish->English | 151 |  7.7878 |  18.8157 | 102.171  |         0.7054 |         0.1817 |           0.1689 |

## 7. External published benchmarks (context only)
These are from model cards / papers on **other** datasets and are **not** directly comparable to the
in-domain numbers above. They show OPUS-MT is a strong general EN<->TR system on public benchmarks.

| model                             | direction        | dataset                  | metric   |    value | source_url                                               | notes                                               | retrieval_date   |
|:----------------------------------|:-----------------|:-------------------------|:---------|---------:|:---------------------------------------------------------|:----------------------------------------------------|:-----------------|
| Helsinki-NLP/opus-mt-tc-big-en-tr | English->Turkish | tatoeba-test-v2021-08-07 | BLEU     | 42.3     | https://huggingface.co/Helsinki-NLP/opus-mt-tc-big-en-tr | Model-card score; external dataset.                 | 2026-06-06       |
| Helsinki-NLP/opus-mt-tc-big-en-tr | English->Turkish | flores101-devtest        | BLEU     | 31.4     | https://huggingface.co/Helsinki-NLP/opus-mt-tc-big-en-tr | Model-card score; external dataset.                 | 2026-06-06       |
| Helsinki-NLP/opus-mt-tc-big-en-tr | English->Turkish | flores101-devtest        | chr-F    |  0.62829 | https://huggingface.co/Helsinki-NLP/opus-mt-tc-big-en-tr | Model-card chr-F (different scale than our chrF++). | 2026-06-06       |
| Helsinki-NLP/opus-mt-tc-big-tr-en | Turkish->English | tatoeba-test-v2021-08-07 | BLEU     | 57.6     | https://huggingface.co/Helsinki-NLP/opus-mt-tc-big-tr-en | Model-card score; external dataset.                 | 2026-06-06       |
| Helsinki-NLP/opus-mt-tc-big-tr-en | Turkish->English | flores101-devtest        | BLEU     | 37.6     | https://huggingface.co/Helsinki-NLP/opus-mt-tc-big-tr-en | Model-card score; external dataset.                 | 2026-06-06       |
| Helsinki-NLP/opus-mt-tc-big-tr-en | Turkish->English | flores101-devtest        | chr-F    |  0.64152 | https://huggingface.co/Helsinki-NLP/opus-mt-tc-big-tr-en | Model-card chr-F (different scale than our chrF++). | 2026-06-06       |
| JAIST WMT17 phrase-based system   | Turkish->English | newstest2017             | BLEU     | 13.1     | https://www.statmt.org/wmt17/pdf/WMT41.pdf               | Historical WMT17 news-domain; not comparable.       | 2026-06-06       |
| JAIST WMT17 phrase-based system   | English->Turkish | newstest2017             | BLEU     | 10.4     | https://www.statmt.org/wmt17/pdf/WMT41.pdf               | Historical WMT17 news-domain; not comparable.       | 2026-06-06       |

## 8. How to use this in the report
Use the **in-domain leaderboard (Section 2)** as the primary result. The fine-tuned-vs-base comparison is
the key evidence that fine-tuning helped. Keep the external benchmarks (Section 7) in a separate
"related work / context" paragraph - never mix them into the same ranking.

## 9. Limitations
- The held-out split was the training notebook's eval split, so it is validation-style, not a fully blind test set.
- The dataset is synthetic (LLM-generated), so automatic metrics should be paired with a small human spot-check.
- Because the references come from the same project data pipeline, the fine-tuned model may receive some style advantage over general translation baselines.
- BLEU / exact-match penalize valid paraphrases; chrF++, BERTScore, and COMET reduce but do not remove this.
- External benchmarks use different tokenization, datasets, and metric settings.

## 10. Output files
`predictions.csv`, `metrics_summary.csv`, `grouped_metrics.csv`, `leaderboard.csv`,
`paired_model_comparison.csv`, `qualitative_examples.csv`, `internet_benchmarks.csv`,
and the PNG charts in this folder.
