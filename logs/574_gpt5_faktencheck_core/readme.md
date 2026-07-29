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

result location: `logs/574_gpt5_faktencheck_core/predict/multiruns/2026-07-27_12-23-00`

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

result location: `logs/574_gpt5_faktencheck_core/evaluate/multiruns/2026-07-29_12-15-58`

| field                   | precision | recall |    f1 | support |
|:------------------------|----------:|-------:|------:|--------:|
| habitat                 |     0.741 |  0.952 | 0.833 |     189 |
| ecosystem_type.category |     0.677 |  0.929 | 0.783 |     140 |
| biodiversity_level      |     0.596 |  0.848 | 0.700 |     132 |
| taxa.species_group      |     0.545 |  0.826 | 0.657 |     213 |
| ecosystem_type.term     |     0.469 |  0.800 | 0.592 |     240 |
| ALL                     |     0.583 |  0.864 | 0.696 |     914 |

### Errors

```sh
uv run -m kibad_llm.evaluate \
name=574_gpt5_faktencheck_core \
experiment/evaluate=prediction_errors \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=logs/574_gpt5_faktencheck_core/predict \
--multirun
```

result location: `logs/574_gpt5_faktencheck_core/evaluate/multiruns/2026-07-29_12-08-22`

```
{
  "no_error": 1594,
  "with_error": 61,
  "JSONDecodeError": 36,
  "ReasoningExtractionError": 27
}
```

## Outcome

The larger output budget resolves the truncation failures that #533 was about. The run completed
cleanly on all 100 dev documents with no job crash, whereas the earlier GPT-5 core runs died partway
through.

Errors dropped sharply. Across 1655 chunks, 1594 were clean and 61 carried an error, an error rate of
3.7 percent, down from the roughly 16 to 20 percent seen before the fix. Most importantly,
`MissingResponseContentError`, the pure truncation failure where reasoning consumed the whole budget
and left no visible answer, no longer occurs at all (0 occurrences).

The remaining 61 errors split into two groups:

- `JSONDecodeError` (36): a small number of chunks with very large outputs (tens of thousands of
  characters) still overrun even the 32768 budget and produce truncated JSON. This is residual
  truncation on outlier documents, not the systemic failure from before.
- `ReasoningExtractionError` (27): these are not a truncation problem. GPT-5 sometimes returns a
  valid answer without a reasoning summary even though `reasoning_options.summary=auto` is set, and
  the extractor currently treats a missing summary as a hard failure. Raising the token budget does
  not affect this. I am reporting it separately as a new, unrelated finding rather than handling it
  here, since #533 is scoped to the truncation fix.

For completeness, F1 on the corrected reference over the five evaluated fields (flattened, micro) is
ALL F1 0.696 (precision 0.583, recall 0.864). I include these numbers only for the record. The goal
of this run was to validate the error fix, not to benchmark GPT-5 as a model.

Recommendation: the fix in #574 removes the truncation crashes and cuts the error rate by roughly
five times, so GPT-5 is safe to re-enable for the core experiments. The residual
`ReasoningExtractionError` should be tracked as a separate issue.
