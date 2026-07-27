# 574_gpt5_faktencheck_core

GPT-5 on the Faktencheck core schema, dev set, re-run after the #533 fix (model pinned to
`gpt-5-2025-08-07`, `max_output_tokens` raised from 8192 to 32768). GPT-5 was previously dropped
from the core experiments because roughly 16 to 20 percent of chunks failed with JSONDecodeError or
MissingResponseContentError, which I traced to output-budget truncation (reasoning tokens and the
visible answer share `max_output_tokens`). This dev run is the cheap validation that the larger
budget removes those truncation errors before I spend on the test-set runs.

Best setup (with chunking) taken from
[397_faktencheck_core_v1_for_chunking](../397_faktencheck_core_v1_for_chunking) and
[519_faktencheck_core](../519_faktencheck_core). Single seed, since the random seed does not change
anything on the OpenAI side and I want to keep costs down.

## Prediction

```sh
./run_in_process.sh -t "1-12:00:00" -ng 0 -pa "H100-SLT,H100-Trails,H100,H200,B200,A100-80GB,batch" \
-u "-m kibad_llm.predict \
name=574_gpt5_faktencheck_core \
experiment/predict=faktencheck_core_fields_schema_with_chunking \
pdf_directory=/ds/text/kiba-d/dev-set-100 \
extractor/llm=gpt_5 \
seed=42 \
--multirun"
```

result location: <to be filled after the run>

## Evaluation

### F1, P, R

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_faktencheck_core \
experiment/evaluate=faktencheck_core_f1_micro_flat \
dataset.references.file=../interim/faktencheck-db/faktenscheck_core_corrected.jsonl \
metric.fields=[habitat,biodiversity_level,ecosystem_type.term,ecosystem_type.category,taxa.species_group] \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/574_gpt5_faktencheck_core/predict \
--multirun
```

result location: <to be filled after the run>

### Errors

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_faktencheck_core \
experiment/evaluate=prediction_errors \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/574_gpt5_faktencheck_core/predict \
--multirun
```

result location: <to be filled after the run>

## Outcome

<to be filled after the run: did the larger budget remove the JSONDecode / MissingResponseContent
errors, and how does GPT-5 compare to the other models on the dev set>
