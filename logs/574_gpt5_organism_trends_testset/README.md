# 574_gpt5_organism_trends_testset

GPT-5 on the true test set (`/ds/text/kiba-d/test-set-AuO-WVC`), same setup (with chunking) as in
[428_organism_trends_with_chunking](../428_organism_trends_with_chunking) and
[549_organism_trends_bestconfig_testset](../549_organism_trends_bestconfig_testset), where the
`gpt_5` slot was deleted for too many errors. Model pinned to `gpt-5-2025-08-07` and
`max_output_tokens` raised from 8192 to 32768, see
[#533](https://github.com/DFKI-NLP/kibad-llm/issues/533) and
[#574](https://github.com/DFKI-NLP/kibad-llm/pull/574). Test set counterpart of
[574_gpt5_organism_trends](../574_gpt5_organism_trends). We use a single seed to limit cost.

## Prediction

```sh
./run_in_process.sh -t "3-00:00:00" -ng 0 -pa "H100-SLT,H100-Trails,H100,H200,B200,A100-80GB,batch" \
-u "-m kibad_llm.predict \
name=574_gpt5_organism_trends_testset \
experiment/predict=organism_trends_with_chunking \
pdf_directory=/ds/text/kiba-d/test-set-AuO-WVC \
extractor/llm=gpt_5 \
seed=42 \
--multirun"
```

result location: `logs/574_gpt5_organism_trends_testset/predict/multiruns/2026-08-03_09-54-45`

## Evaluation

Reference is the Weighted Vote Count for the Agrarian and Open Landscapes (AuO) chapter, matching
the test set, and the same file that
[549_organism_trends_bestconfig_testset](../549_organism_trends_bestconfig_testset) uses. The
corrected reference introduced in
[544_organism_trends_wald_corrected](../544_organism_trends_wald_corrected) covers the Wald chapter
only, so it does not apply here.

### F1, P, R

##### flattened

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends_testset \
experiment/evaluate=organism_trends_f1_micro_flat \
prediction_logs=logs/574_gpt5_organism_trends_testset/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Agrar- und Offenland Literatur - Sheet1.csv" \
--multirun
```

result location: `logs/574_gpt5_organism_trends_testset/evaluate/multiruns/2026-08-04_13-39-26`

| field                                  | precision | recall |    f1 | support |
|:---------------------------------------|----------:|-------:|------:|--------:|
| organism_trends.Antwortvariable        |     0.363 |  0.592 | 0.450 |     238 |
| organism_trends.Hauptgruppe_RoteListen |     0.406 |  0.603 | 0.485 |     214 |
| organism_trends.Lebensraum             |     0.338 |  0.536 | 0.414 |     194 |
| organism_trends.Trend                  |     0.264 |  0.555 | 0.358 |     238 |
| AVG                                    |     0.343 |  0.571 | 0.427 |     221 |
| ALL                                    |     0.334 |  0.572 | 0.422 |     884 |

##### full compounds

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends_testset \
experiment/evaluate=organism_trends_f1_micro \
prediction_logs=logs/574_gpt5_organism_trends_testset/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Agrar- und Offenland Literatur - Sheet1.csv" \
--multirun
```

result location: `logs/574_gpt5_organism_trends_testset/evaluate/multiruns/2026-08-04_13-39-27`

| field           | precision | recall |    f1 | support |
|:----------------|----------:|-------:|------:|--------:|
| organism_trends |     0.128 |  0.402 | 0.194 |     281 |
| AVG             |     0.128 |  0.402 | 0.194 |     281 |
| ALL             |     0.128 |  0.402 | 0.194 |     281 |

##### base elements

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends_testset \
experiment/evaluate=organism_trends_f1_micro_base_entries \
prediction_logs=logs/574_gpt5_organism_trends_testset/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Agrar- und Offenland Literatur - Sheet1.csv" \
--multirun
```

result location: `logs/574_gpt5_organism_trends_testset/evaluate/multiruns/2026-08-04_13-39-29`

| field           | precision | recall |    f1 | support |
|:----------------|----------:|-------:|------:|--------:|
| organism_trends |     0.267 |  0.495 | 0.347 |     214 |
| AVG             |     0.267 |  0.495 | 0.347 |     214 |
| ALL             |     0.267 |  0.495 | 0.347 |     214 |

##### `Antwortvariable` conditioned on base elements

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends_testset \
experiment/evaluate=organism_trends_f1_micro_conditional_variable_only \
prediction_logs=logs/574_gpt5_organism_trends_testset/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Agrar- und Offenland Literatur - Sheet1.csv" \
--multirun
```

result location: `logs/574_gpt5_organism_trends_testset/evaluate/multiruns/2026-08-04_13-39-30`

| field                                  | precision | recall |    f1 | support |
|:---------------------------------------|----------:|-------:|------:|--------:|
| organism_trends.Pflanzen&AgrarOffen    |     0.660 |  0.833 | 0.737 |      84 |
| organism_trends.Wirbellose&AgrarOffen  |     0.784 |  0.952 | 0.860 |      42 |
| organism_trends.Wirbeltiere&AgrarOffen |     0.857 |  0.857 | 0.857 |      14 |
| AVG                                    |     0.767 |  0.881 | 0.818 |  46.667 |
| ALL                                    |     0.713 |  0.871 | 0.785 |     140 |

##### `Antwortvariable` & `Trend` conditioned on base elements

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends_testset \
experiment/evaluate=organism_trends_f1_micro_conditional_variable_and_trend \
prediction_logs=logs/574_gpt5_organism_trends_testset/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Agrar- und Offenland Literatur - Sheet1.csv" \
--multirun
```

result location: `logs/574_gpt5_organism_trends_testset/evaluate/multiruns/2026-08-04_13-39-32`

| field                                  | precision | recall |    f1 | support |
|:---------------------------------------|----------:|-------:|------:|--------:|
| organism_trends.Pflanzen&AgrarOffen    |     0.339 |  0.688 | 0.454 |      93 |
| organism_trends.Wirbellose&AgrarOffen  |     0.500 |  0.830 | 0.624 |      47 |
| organism_trends.Wirbeltiere&AgrarOffen |     0.455 |  0.714 | 0.556 |      14 |
| AVG                                    |     0.431 |  0.744 | 0.544 |  51.333 |
| ALL                                    |     0.391 |  0.734 | 0.510 |     154 |

### Errors

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends_testset \
experiment/evaluate=prediction_errors \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/574_gpt5_organism_trends_testset/predict \
--multirun
```

result location: `logs/574_gpt5_organism_trends_testset/evaluate/multiruns/2026-08-04_13-39-25`

```
{
  "no_error": 2281,
  "with_error": 265,
  "ReasoningExtractionError": 265
}
```

## Outcome

- All 419 test set documents processed
- No `JSONDecodeError`, no `MissingResponseContentError`. All 265 chunks with an error are
  `ReasoningExtractionError`, which is non-breaking, so no chunk lost its result. Tracked in
  [#575](https://github.com/DFKI-NLP/kibad-llm/issues/575)
- This is not a before and after. The `gpt_5` slot in
  [549_organism_trends_bestconfig_testset](../549_organism_trends_bestconfig_testset) was deleted
  for too many errors without keeping the numbers, and
  [#533](https://github.com/DFKI-NLP/kibad-llm/issues/533) covers the Faktencheck core schema only.
  Same situation on the dev set, see
  [574_gpt5_organism_trends](../574_gpt5_organism_trends)

### Error counts

Base for the other models is the same `experiment/evaluate=prediction_errors` command run in
[549_organism_trends_bestconfig_testset](../549_organism_trends_bestconfig_testset), result location
`logs/549_organism_trends_bestconfig_testset/evaluate/multiruns/2026-07-13_15-00-08`. Three seeds
per model, all runs process 2546 chunks.

| model               | with_error of 2546 | error types                                      |
|:--------------------|-------------------:|:-------------------------------------------------|
| gpt-5 (this run)    |                265 | `ReasoningExtractionError`                        |
| mistral_small_3_24b |           59 to 65 | `JSONDecodeError`                                 |
| qwen3_30b           |           12 to 15 | `JSONDecodeError`, `MissingResponseContentError`  |
| gemma3_27b          |             6 to 7 | `JSONDecodeError`                                 |
| gpt_oss_20b         |             0 to 2 | `JSONDecodeError`                                 |

Notes
- GPT-5 has more entries with an error than any model in 549 (265 vs. 0 to 65), but none of the two
  types that break the result JSON
- `ReasoningExtractionError` does not appear for the other models in 549

### F1, P, R

Both variants below are the compound based ones, see
[#422](https://github.com/DFKI-NLP/kibad-llm/pull/422) for the individual metric variants.

#### full compounds

Base for the other models is the same `experiment/evaluate=organism_trends_f1_micro` command run in
[549_organism_trends_bestconfig_testset](../549_organism_trends_bestconfig_testset), result location
`logs/549_organism_trends_bestconfig_testset/evaluate/multiruns/2026-07-13_14-59-03`. Support is 281
in all cases.

| model               | precision      | recall         | f1             |
|:--------------------|:---------------|:---------------|:---------------|
| gpt-5 (this run)    | 0.128          | 0.402          | 0.194          |
| qwen3_30b           | 0.134 to 0.164 | 0.181 to 0.224 | 0.154 to 0.190 |
| gpt_oss_20b         | 0.066 to 0.080 | 0.214 to 0.267 | 0.100 to 0.123 |
| mistral_small_3_24b | 0.033 to 0.039 | 0.185 to 0.221 | 0.056 to 0.066 |
| gemma3_27b          | 0.009 to 0.011 | 0.053 to 0.068 | 0.016 to 0.019 |

Notes
- Highest full compound ALL f1 at 0.194, above the three Qwen3 seeds at 0.154 to 0.190. One seed
  against three per model
- Best recall of all models at 0.402, next is GPT OSS at 0.214 to 0.267
- Precision second after Qwen3 (0.128 vs. 0.134 to 0.164)
- On the dev set GPT-5 is second behind Qwen3 on the same metric, see
  [574_gpt5_organism_trends](../574_gpt5_organism_trends)
- On the flattened metric GPT-5 is second here instead, 0.422 against 0.426 to 0.450 for Qwen3
  (`experiment/evaluate=organism_trends_f1_micro_flat` in 549, result location
  `logs/549_organism_trends_bestconfig_testset/evaluate/multiruns/2026-07-13_14-58-27`), so the two
  metrics order GPT-5 and Qwen3 differently

#### `Antwortvariable` & `Trend` conditioned on base elements

Base for the other models is the same
`experiment/evaluate=organism_trends_f1_micro_conditional_variable_and_trend` command run in
[549_organism_trends_bestconfig_testset](../549_organism_trends_bestconfig_testset), result location
`logs/549_organism_trends_bestconfig_testset/evaluate/multiruns/2026-07-13_14-59-52`. Support varies
per model, so it is listed in the table.

| model               | precision      | recall         | f1             |    support |
|:--------------------|:---------------|:---------------|:---------------|-----------:|
| qwen3_30b           | 0.447 to 0.500 | 0.505 to 0.558 | 0.483 to 0.527 | 101 to 113 |
| gemma3_27b          | 0.388 to 0.500 | 0.385 to 0.529 | 0.390 to 0.514 |   34 to 44 |
| gpt-5 (this run)    | 0.391          | 0.734          | 0.510          |        154 |
| gpt_oss_20b         | 0.386 to 0.414 | 0.566 to 0.586 | 0.466 to 0.484 | 106 to 129 |
| mistral_small_3_24b | 0.323 to 0.344 | 0.525 to 0.590 | 0.400 to 0.435 |  99 to 107 |

Notes
- Support differs per model, this metric only scores base pairs that are in both prediction and
  reference (`ignore_missing_entries: true`)
- Spread is wide here, 34 to 154, GPT-5 at 154 and Gemma3 at 34 to 44, so the f1 values are not on
  a common base and the order should be read with that in mind
- GPT-5 f1 0.510 sits inside the Qwen3 spread of 0.483 to 0.527, on 154 entries against 101 to 113
- Best recall of all models at 0.734, next are Mistral (0.525 to 0.590) and GPT OSS (0.566 to
  0.586), whose seed ranges overlap
- Precision 0.391 below Qwen3 (0.447 to 0.500), level with GPT OSS (0.386 to 0.414)
- On the base elements GPT-5 and Qwen3 are level here, 0.347 against 0.325 to 0.365
  (`experiment/evaluate=organism_trends_f1_micro_base_entries` in 549, result location
  `logs/549_organism_trends_bestconfig_testset/evaluate/multiruns/2026-07-13_14-59-18`), unlike on
  the dev set where GPT-5 is behind, see [574_gpt5_organism_trends](../574_gpt5_organism_trends)

The `gpt_5` slot deleted from 549 should be filled with this run. GPT-5 and Qwen3 end up close on
full compound ALL f1 from opposite directions, GPT-5 on recall and Qwen3 on precision, and they stay
close once the base elements match.
