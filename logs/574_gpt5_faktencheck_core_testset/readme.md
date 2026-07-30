# 574_gpt5_faktencheck_core_testset

GPT-5 on the Faktencheck core schema, true test set (`/ds/text/kiba-d/splits/test`). Re-run after the
[#533](https://github.com/DFKI-NLP/kibad-llm/issues/533) fix: model pinned to `gpt-5-2025-08-07`,
`max_output_tokens` raised from 8192 to 32768. This is the test-set counterpart of the dev run
[574_gpt5_faktencheck_core](../574_gpt5_faktencheck_core), which already confirmed the truncation
errors are gone. It re-fills the GPT-5 slot that was deleted from
[525_faktencheck_core_bestconfig_testset](../525_faktencheck_core_bestconfig_testset) for too many
errors (the first seed alone cost about $195 and took about 59h). Single seed, since the seed does
not change anything on the OpenAI side.

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

result location: <to be filled after the run>

## Evaluation

### F1, P, R

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_faktencheck_core_testset \
experiment/evaluate=faktencheck_core_f1_micro_flat \
dataset.references.file=../interim/faktencheck-db/faktenscheck_core_corrected.jsonl \
metric.fields=[habitat,biodiversity_level,ecosystem_type.term,ecosystem_type.category,taxa.species_group] \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/574_gpt5_faktencheck_core_testset/predict \
--multirun
```

result location: <to be filled after the run>

### Errors

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_faktencheck_core_testset \
experiment/evaluate=prediction_errors \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/574_gpt5_faktencheck_core_testset/predict \
--multirun
```

result location: <to be filled after the run>

## Outcome

<to be filled after the run>
