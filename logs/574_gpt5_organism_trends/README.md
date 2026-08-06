# 574_gpt5_organism_trends

GPT-5 on the organism trends dev set (`/ds/text/kiba-d/dev-set-Wald-WVC`), same setup (with
chunking) as in [428_organism_trends_with_chunking](../428_organism_trends_with_chunking), which
does not include GPT-5. Model pinned to `gpt-5-2025-08-07` and `max_output_tokens` raised from 8192
to 32768, see [#533](https://github.com/DFKI-NLP/kibad-llm/issues/533) and
[#574](https://github.com/DFKI-NLP/kibad-llm/pull/574). Dev counterpart of
[574_gpt5_organism_trends_testset](../574_gpt5_organism_trends_testset). We use a single seed to
limit cost.

## Prediction

```sh
./run_in_process.sh -t "2-00:00:00" -ng 0 -pa "H100-SLT,H100-Trails,H100,H200,B200,A100-80GB,batch" \
-u "-m kibad_llm.predict \
name=574_gpt5_organism_trends \
experiment/predict=organism_trends_with_chunking \
pdf_directory=/ds/text/kiba-d/dev-set-Wald-WVC \
extractor/llm=gpt_5 \
seed=42 \
--multirun"
```

result location: `logs/574_gpt5_organism_trends/predict/multiruns/2026-08-03_02-30-39`

## Evaluation

Reference is `Referenz_Wald_korrigiert_angepasst.csv`, the corrected Wald reference that
[544_organism_trends_wald_corrected](../544_organism_trends_wald_corrected) uses for the same dev
set.

### F1, P, R

##### flattened

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=organism_trends_f1_micro_flat \
prediction_logs=logs/574_gpt5_organism_trends/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file=../external/organism_trends/Referenz_Wald_korrigiert_angepasst.csv \
--multirun
```

result location: `logs/574_gpt5_organism_trends/evaluate/multiruns/2026-08-04_14-02-35`

| field                                  | precision | recall |    f1 | support |
|:---------------------------------------|----------:|-------:|------:|--------:|
| organism_trends.Antwortvariable        |     0.298 |  0.789 | 0.433 |     128 |
| organism_trends.Hauptgruppe_RoteListen |     0.282 |  0.813 | 0.419 |     107 |
| organism_trends.Lebensraum             |     0.258 |  0.794 | 0.389 |     102 |
| organism_trends.Trend                  |     0.216 |  0.712 | 0.332 |     132 |
| AVG                                    |     0.264 |  0.777 | 0.393 |  117.25 |
| ALL                                    |     0.260 |  0.774 | 0.389 |     469 |

##### full compounds

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=organism_trends_f1_micro \
prediction_logs=logs/574_gpt5_organism_trends/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file=../external/organism_trends/Referenz_Wald_korrigiert_angepasst.csv \
--multirun
```

result location: `logs/574_gpt5_organism_trends/evaluate/multiruns/2026-08-04_14-02-36`

| field           | precision | recall |    f1 | support |
|:----------------|----------:|-------:|------:|--------:|
| organism_trends |     0.132 |  0.675 | 0.221 |     163 |
| AVG             |     0.132 |  0.675 | 0.221 |     163 |
| ALL             |     0.132 |  0.675 | 0.221 |     163 |

##### base elements

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=organism_trends_f1_micro_base_entries \
prediction_logs=logs/574_gpt5_organism_trends/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file=../external/organism_trends/Referenz_Wald_korrigiert_angepasst.csv \
--multirun
```

result location: `logs/574_gpt5_organism_trends/evaluate/multiruns/2026-08-04_14-02-38`

| field           | precision | recall |    f1 | support |
|:----------------|----------:|-------:|------:|--------:|
| organism_trends |     0.204 |  0.794 | 0.325 |     107 |
| AVG             |     0.204 |  0.794 | 0.325 |     107 |
| ALL             |     0.204 |  0.794 | 0.325 |     107 |

##### `Antwortvariable` conditioned on base elements

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=organism_trends_f1_micro_conditional_variable_only \
prediction_logs=logs/574_gpt5_organism_trends/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file=../external/organism_trends/Referenz_Wald_korrigiert_angepasst.csv \
--multirun
```

result location: `logs/574_gpt5_organism_trends/evaluate/multiruns/2026-08-04_14-02-39`

| field                               | precision | recall |    f1 | support |
|:------------------------------------|----------:|-------:|------:|--------:|
| organism_trends.Pflanzen&Wald       |     0.828 |  0.946 | 0.883 |      56 |
| organism_trends.Pilze_Flechten&Wald |     0.625 |  0.714 | 0.667 |       7 |
| organism_trends.Wirbellose&Wald     |     0.720 |  0.818 | 0.766 |      22 |
| organism_trends.Wirbeltiere&Wald    |     0.966 |  1.000 | 0.982 |      28 |
| AVG                                 |     0.785 |  0.870 | 0.825 |   28.25 |
| ALL                                 |     0.825 |  0.920 | 0.870 |     113 |

##### `Antwortvariable` & `Trend` conditioned on base elements

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=organism_trends_f1_micro_conditional_variable_and_trend \
prediction_logs=logs/574_gpt5_organism_trends/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file=../external/organism_trends/Referenz_Wald_korrigiert_angepasst.csv \
--multirun
```

result location: `logs/574_gpt5_organism_trends/evaluate/multiruns/2026-08-04_14-02-41`

| field                               | precision | recall |    f1 | support |
|:------------------------------------|----------:|-------:|------:|--------:|
| organism_trends.Pflanzen&Wald       |     0.554 |  0.859 | 0.673 |      78 |
| organism_trends.Pilze_Flechten&Wald |     0.455 |  0.714 | 0.556 |       7 |
| organism_trends.Wirbellose&Wald     |     0.318 |  0.636 | 0.424 |      22 |
| organism_trends.Wirbeltiere&Wald    |     0.545 |  0.750 | 0.632 |      32 |
| AVG                                 |     0.468 |  0.740 | 0.571 |   34.75 |
| ALL                                 |     0.500 |  0.791 | 0.613 |     139 |

### Errors

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=prediction_errors \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/574_gpt5_organism_trends/predict \
--multirun
```

result location: `logs/574_gpt5_organism_trends/evaluate/multiruns/2026-08-04_13-38-48`

```
{
  "no_error": 2191,
  "with_error": 246,
  "ReasoningExtractionError": 246
}
```

## Outcome

- All 409 dev documents processed
- No `JSONDecodeError`, no `MissingResponseContentError`. All 246 chunks with an error are
  `ReasoningExtractionError`, which is non-breaking, so no chunk lost its result. Tracked in
  [#575](https://github.com/DFKI-NLP/kibad-llm/issues/575)
- This is not a before and after. [#533](https://github.com/DFKI-NLP/kibad-llm/issues/533) covers
  the Faktencheck core schema only (343 of 1654 chunks on the dev set, 449 of 2723 on the test set,
  in 519 and 525). No GPT-5 run exists on this dev set with chunking and the old budget
- The `gpt_5` slot in
  [549_organism_trends_bestconfig_testset](../549_organism_trends_bestconfig_testset) was deleted
  for too many errors without keeping the numbers, so the organism trends error rate before the
  change is unknown
- Closest earlier GPT-5 run here is
  [255_organism_trend_baseline_no_evi](../255_organism_trend_baseline_no_evi), without chunking
  (`experiment/predict=organism_trends`), one entry per document instead of one per chunk. Over
  three seeds already 0 `JSONDecodeError`, 0 `MissingResponseContentError`, 13 to 22
  `ReasoningExtractionError` and 8 `ValueError` of 409 entries. Different base than the 246 here,
  same error types

### Error counts

Base for the other models is the same `experiment/evaluate=prediction_errors` command run in
[428_organism_trends_with_chunking](../428_organism_trends_with_chunking), result location
`logs/428_organism_trends_with_chunking/evaluate/multiruns/2026-05-04_13-01-11`. Three seeds per
model, all runs process 2437 chunks.

| model               | with_error of 2437 | error types                                      |
|:--------------------|-------------------:|:-------------------------------------------------|
| gpt-5 (this run)    |                246 | `ReasoningExtractionError`                        |
| mistral_small_3_24b |           47 to 50 | `JSONDecodeError`                                 |
| qwen3_30b           |            6 to 16 | `JSONDecodeError`, `MissingResponseContentError`  |
| gemma3_27b          |              2 to 4 | `JSONDecodeError`                                 |
| gpt_oss_20b         |              1 to 2 | `JSONDecodeError`                                 |

Notes
- GPT-5 has more entries with an error than any model in 428 (246 vs. 1 to 50), but none of the two
  types that break the result JSON
- `ReasoningExtractionError` does not appear for the other models in 428

### F1, P, R (flattened, ALL)

Base for the other models is the same `experiment/evaluate=organism_trends_f1_micro_flat` command
run in [544_organism_trends_wald_corrected](../544_organism_trends_wald_corrected), result location
`logs/544_organism_trends_wald_corrected/evaluate/multiruns/2026-07-16_13-43-56`. 544 re-evaluates
the predictions from 428 against the corrected reference, so the same reference file as here.
Support is 469 in all cases.

| model               | precision      | recall         | f1             |
|:--------------------|:---------------|:---------------|:---------------|
| qwen3_30b           | 0.336 to 0.345 | 0.689 to 0.719 | 0.452 to 0.466 |
| gpt-5 (this run)    | 0.260          | 0.774          | 0.389          |
| gpt_oss_20b         | 0.198 to 0.204 | 0.744 to 0.755 | 0.313 to 0.321 |
| gemma3_27b          | 0.137 to 0.140 | 0.819 to 0.840 | 0.235 to 0.240 |
| mistral_small_3_24b | 0.130 to 0.131 | 0.751 to 0.763 | 0.221 to 0.224 |

Notes
- GPT-5 second on flat ALL f1 at 0.389, Qwen3 best at 0.452 to 0.466. One seed against three per
  model
- Precision second after Qwen3 (0.260 vs. 0.336 to 0.345), recall second after Gemma3 (0.774 vs.
  0.819 to 0.840)

This adds a GPT-5 number on the organism trends dev set with chunking, which did not exist before.
Qwen3 stays the best model here.
