# 574_gpt5_faktencheck_core

GPT-5 on the Faktencheck core schema, dev set, re-run after the #533 fix (model pinned to
`gpt-5-2025-08-07`, `max_output_tokens` raised from 8192 to 32768). GPT-5 was previously dropped
from the core experiments because about 20 percent of chunks failed with JSONDecodeError or
MissingResponseContentError (see [519_faktencheck_core](../519_faktencheck_core)), which I traced to
output-budget truncation (reasoning tokens and the visible answer share `max_output_tokens`). This
dev run is the cheap validation that the larger
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

The larger output budget resolves the truncation failures that #533 was about. The run completed on
all 100 dev documents with no job crash. The earlier GPT-5 problem was never the whole job dying;
individual requests for certain PDFs failed and were logged as per-chunk errors, and it is those
per-chunk errors that I compare below.

I compare against [519_faktencheck_core](../519_faktencheck_core), which ran GPT-5 on the same dev
set with the same chunking config before the fix (floating `gpt-5` alias, `max_output_tokens` 8192).
Its two GPT-5 seeds logged 336 and 350 errors out of 1654 chunks, an error rate of about 20 percent.
After the fix, this run logged 61 errors out of 1655 chunks, 3.7 percent.

By error type, before (519) to after (this run):

- `MissingResponseContentError`: 123 to 127 before, **0 after**. This is the pure truncation failure
  where reasoning consumed the whole budget and left no visible answer. The larger budget removes it
  completely.
- `JSONDecodeError`: about 194 before, **36 after**. This is the important remaining error, because
  we cannot recover from it: if the result JSON is broken we cannot use the output at all. The larger
  budget cuts it by roughly 80 percent, but a few chunks with very large outputs (tens of thousands
  of characters) still overrun even the 32768 budget and produce truncated JSON. This is residual
  truncation on outlier documents, not the systemic failure from before.
- `ReasoningExtractionError`: about 28 before, 27 after. The budget does not affect this, as
  expected, because it is not a truncation problem. GPT-5 sometimes returns a valid answer without a
  reasoning summary even though `reasoning_options.summary=auto` is set. This is a non-breaking
  error: the extractor logs it and moves the entry for that PDF from "without error" to "with error"
  in the overview figures, and we lose the reasoning for analysis, but the extracted output is still
  usable. I am reporting it separately as a new issue rather than handling it here, since #533 is
  scoped to the truncation fix.

For F1, this run scores ALL F1 0.696 (precision 0.583, recall 0.864) on the corrected reference over
the five evaluated fields (flattened, micro, support 914). 519 scored ALL F1 0.712 and 0.711 for its
two GPT-5 seeds on the same 914-support reference, so the fix gives a slightly lower F1, about 1.5
points. The likely reason is that the fix recovers chunks that previously failed and contributed no
predictions, which raises recall (0.864 vs 0.823) but lowers precision (0.583 vs 0.628) as the
recovered chunks add some wrong predictions. 519 also used a different, floating model snapshot, so
part of the gap can be version drift. I include these numbers for the record; the goal of this run
was to validate the error fix, not to benchmark GPT-5 as a model.

Recommendation: the fix in #574 removes the `MissingResponseContentError` truncation failures and
cuts the total error rate from about 20 percent to 3.7 percent, so GPT-5 is safe to re-enable for the
core experiments. The residual to track is the `JSONDecodeError` on outlier documents, since it is
the one error we cannot recover from. The `ReasoningExtractionError` is non-breaking and lower
priority.
