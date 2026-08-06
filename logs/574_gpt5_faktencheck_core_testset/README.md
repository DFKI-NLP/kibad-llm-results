# 574_gpt5_faktencheck_core_testset

GPT-5 on the Faktencheck core schema, true test set (`/ds/text/kiba-d/splits/test`), same setup
(with chunking) as in
[525_faktencheck_core_bestconfig_testset](../525_faktencheck_core_bestconfig_testset), where the
`gpt_5` slot was deleted for too many errors. Model pinned to `gpt-5-2025-08-07` and
`max_output_tokens` raised from 8192 to 32768, see
[#533](https://github.com/DFKI-NLP/kibad-llm/issues/533) and
[#574](https://github.com/DFKI-NLP/kibad-llm/pull/574). Test set counterpart of
[574_gpt5_faktencheck_core](../574_gpt5_faktencheck_core), where the share of chunks with an error
went down from about 20% to 3.7%. We use a single seed to limit cost.

## Prediction

```sh
./run_in_process.sh -t "3-00:00:00" -ng 0 -pa "H100-SLT,H100-Trails,H100,H200,B200,A100-80GB,batch" \
-u "-m kibad_llm.predict \
name=574_gpt5_faktencheck_core_testset \
experiment/predict=faktencheck_core_fields_schema_with_chunking \
pdf_directory=/ds/text/kiba-d/splits/test \
extractor/llm=gpt_5 \
seed=42 \
--multirun"
```

result location: `logs/574_gpt5_faktencheck_core_testset/predict/multiruns/2026-08-02_19-46-20`

## Evaluation

Reference is `faktencheck-db-converted_2025-11-05.jsonl`, the default of
`dataset/references/faktencheck_db_converted` and the same file that
[525_faktencheck_core_bestconfig_testset](../525_faktencheck_core_bestconfig_testset) uses for this
test set. The corrected reference `faktenscheck_core_corrected.jsonl` used in
[574_gpt5_faktencheck_core](../574_gpt5_faktencheck_core) holds the 100 dev documents only and has
no entry for any of the 500 test documents, so it does not apply here. The five-field subset is the
same as in 525.

### F1, P, R

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_faktencheck_core_testset \
experiment/evaluate=faktencheck_core_f1_micro_flat \
metric.fields=[habitat,biodiversity_level,ecosystem_type.term,ecosystem_type.category,taxa.species_group] \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/574_gpt5_faktencheck_core_testset/predict \
--multirun
```

result location: `logs/574_gpt5_faktencheck_core_testset/evaluate/multiruns/2026-08-06_11-55-50`

| field                   | precision | recall |    f1 | support |
|:------------------------|----------:|-------:|------:|--------:|
| habitat                 |     0.493 |  0.913 | 0.640 |     564 |
| taxa.species_group      |     0.344 |  0.796 | 0.480 |     555 |
| ecosystem_type.category |     0.299 |  0.951 | 0.455 |     243 |
| biodiversity_level      |     0.316 |  0.689 | 0.433 |     399 |
| ecosystem_type.term     |     0.158 |  0.775 | 0.262 |     280 |
| ALL                     |     0.314 |  0.823 | 0.455 |    2041 |

### Errors

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_faktencheck_core_testset \
experiment/evaluate=prediction_errors \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/574_gpt5_faktencheck_core_testset/predict \
--multirun
```

result location: `logs/574_gpt5_faktencheck_core_testset/evaluate/multiruns/2026-08-06_11-55-04`

```
{
  "no_error": 2662,
  "with_error": 61,
  "JSONDecodeError": 32,
  "ReasoningExtractionError": 29
}
```

## Outcome

- All 500 test set documents processed. Extraction took 62.3 h, so it stayed inside the 3 day
  limit. 525 notes 59 h for its GPT-5 pass over the same 500 documents
- Errors down from 449 to 61 of 2723 chunks. The 449 comes from
  [#533](https://github.com/DFKI-NLP/kibad-llm/issues/533), the only source left since the 525 run
  was deleted. No breakdown by error type there
- Of the 61: 32 `JSONDecodeError`, 29 `ReasoningExtractionError`, no `MissingResponseContentError`.
  Same picture as on the dev set, see
  [574_gpt5_faktencheck_core](../574_gpt5_faktencheck_core)
- `JSONDecodeError` is the error that matters, a broken result JSON is unusable. We did not look
  into the 32 cases
- `ReasoningExtractionError` is non-breaking, all 29 still have their output. Tracked in
  [#575](https://github.com/DFKI-NLP/kibad-llm/issues/575)
- No GPT-5 f1 to compare against here, the 525 run was deleted without keeping its scores. The
  tables below compare with the other four models instead

### Error counts

Base for the other models is the same `experiment/evaluate=prediction_errors` command run in
[525_faktencheck_core_bestconfig_testset](../525_faktencheck_core_bestconfig_testset), result
location `logs/525_faktencheck_core_bestconfig_testset/evaluate/multiruns/2026-07-13_15-08-46`.
Three seeds per model, all runs process 2723 chunks.

| model               | with_error of 2723 | error types                                     |
|:--------------------|-------------------:|:------------------------------------------------|
| gpt-5 (this run)    |                 61 | `JSONDecodeError`, `ReasoningExtractionError`    |
| gemma3_27b          |           44 to 61 | `JSONDecodeError`                                |
| mistral_small_3_24b |           47 to 56 | `JSONDecodeError`                                |
| qwen3_30b           |           26 to 39 | `JSONDecodeError`, `MissingResponseContentError` |
| gpt_oss_20b         |             5 to 8 | `JSONDecodeError`, `MissingResponseContentError` |

Notes
- GPT-5 at 61 is level with the worst Gemma3 seed (44 to 61) and above the other three models
- On `JSONDecodeError` alone GPT-5 has 32, below every Gemma3 and Mistral seed (44 to 61 and 47 to
  56), above the Qwen3 seeds (21 to 27)
- `ReasoningExtractionError` does not appear for the other models in 525

### F1, P, R (ALL)

Base for the other models is the same `experiment/evaluate=faktencheck_core_f1_micro_flat` command
run in [525_faktencheck_core_bestconfig_testset](../525_faktencheck_core_bestconfig_testset), result
location `logs/525_faktencheck_core_bestconfig_testset/evaluate/multiruns/2026-07-13_15-08-10`.
Support is 2041 in all cases.

| model               | precision      | recall         | f1             |
|:--------------------|:---------------|:---------------|:---------------|
| gpt_oss_20b         | 0.384 to 0.391 | 0.721 to 0.724 | 0.501 to 0.507 |
| qwen3_30b           | 0.382 to 0.393 | 0.706 to 0.714 | 0.495 to 0.505 |
| gpt-5 (this run)    | 0.314          | 0.823          | 0.455          |
| gemma3_27b          | 0.298 to 0.307 | 0.730 to 0.738 | 0.423 to 0.434 |
| mistral_small_3_24b | 0.288 to 0.296 | 0.794 to 0.797 | 0.423 to 0.432 |

Notes
- GPT-5 third on ALL f1 at 0.455, below GPT OSS and Qwen3 (0.495 to 0.507), above Gemma3 and
  Mistral (0.423 to 0.434). One seed against three per model
- Best recall of all models at 0.823, next is Mistral at 0.794 to 0.797
- Precision third at 0.314, below GPT OSS and Qwen3 (0.382 to 0.393), above Gemma3 and Mistral
  (0.288 to 0.307)
- Same high recall, low precision profile as on the dev set, see
  [574_gpt5_faktencheck_core](../574_gpt5_faktencheck_core)

This run fills the `gpt_5` slot deleted from 525. GPT OSS and Qwen3 stay ahead on f1.
