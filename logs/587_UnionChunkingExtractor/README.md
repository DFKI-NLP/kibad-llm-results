# 587_UnionChunkingExtractor

TODO: goal, motivation, hypothesis.

## Prediction

```sh
./run_in_process.sh -t "1-00:00:00" -pa "H100-SLT,H100-Trails,H100,H200,B200,A100-80GB" -sr \
-r a2cb440dc80ff2e4bb72f63124dc7e1c0d873668 \
-u "-m kibad_llm.predict \
name=587_UnionChunkingExtractor \
experiment/predict=faktencheck_core_fields_schema_with_union \
pdf_directory=/ds/text/kiba-d/dev-set-100/ \
extractor/llm=gpt_oss_20b_in_process \
seed=42,1337,7331 \
--multirun"
```

result location: `logs/587_UnionChunkingExtractor/predict/multiruns/2026-08-20_15-17-50`

## Evaluation

### F1, P, R

```sh
uv run -m kibad_llm.evaluate \
name=587_UnionChunkingExtractor \
experiment/evaluate=faktencheck_core_f1_micro_flat \
dataset.references.file=../interim/faktencheck-db/faktenscheck_core_corrected.jsonl \
metric.fields=[habitat,biodiversity_level,ecosystem_type.term,ecosystem_type.category,taxa.species_group] \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/587_UnionChunkingExtractor/predict \
--multirun
```

result location: TODO

TODO: result table.

### Errors

```sh
uv run -m kibad_llm.evaluate \
name=587_UnionChunkingExtractor \
experiment/evaluate=prediction_errors \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/587_UnionChunkingExtractor/predict \
--multirun
```

result location: TODO

TODO: error counts.

## Outcome

TODO: result analysis, hypothesis evaluation, unexpected outcomes, recommendations.
