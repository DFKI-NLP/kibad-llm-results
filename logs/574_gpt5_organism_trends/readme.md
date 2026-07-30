# 574_gpt5_organism_trends

GPT-5 on the Organism trends schema, dev set (`/ds/text/kiba-d/dev-set-Wald-WVC`). Re-run after the
[#533](https://github.com/DFKI-NLP/kibad-llm/issues/533) fix: model pinned to `gpt-5-2025-08-07`,
`max_output_tokens` raised from 8192 to 32768. This is the dev counterpart of
[574_gpt5_organism_trends_testset](../574_gpt5_organism_trends_testset) on the organism-trends
schema, a cheaper check before the test-set run. Best setup (with chunking) as in
[428_organism_trends_with_chunking](../428_organism_trends_with_chunking). Single seed, since the
seed does not change anything on the OpenAI side.

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

result location: <to be filled after the run>

## Evaluation

Reference is the Weighted Vote Count for the Forest (Wald) chapter, matching the dev set.

### F1, P, R

##### flattened

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=organism_trends_f1_micro_flat \
prediction_logs=logs/574_gpt5_organism_trends/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Wald Literatur - Sheet1.csv" \
--multirun
```

##### full compounds

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=organism_trends_f1_micro \
prediction_logs=logs/574_gpt5_organism_trends/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Wald Literatur - Sheet1.csv" \
--multirun
```

##### base elements

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=organism_trends_f1_micro_base_entries \
prediction_logs=logs/574_gpt5_organism_trends/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Wald Literatur - Sheet1.csv" \
--multirun
```

##### `Antwortvariable` conditioned on base elements

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=organism_trends_f1_micro_conditional_variable_only \
prediction_logs=logs/574_gpt5_organism_trends/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Wald Literatur - Sheet1.csv" \
--multirun
```

##### `Antwortvariable` & `Trend` conditioned on base elements

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=organism_trends_f1_micro_conditional_variable_and_trend \
prediction_logs=logs/574_gpt5_organism_trends/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Wald Literatur - Sheet1.csv" \
--multirun
```

result locations: <to be filled after the runs>

### Errors

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_organism_trends \
experiment/evaluate=prediction_errors \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/574_gpt5_organism_trends/predict \
--multirun
```

result location: <to be filled after the run>

## Outcome

<to be filled after the run>
