# 574_gpt5_faktencheck_core

GPT-5 on the Faktencheck core schema, dev set. Re-run after the
[#533](https://github.com/DFKI-NLP/kibad-llm/issues/533) fix: model pinned to `gpt-5-2025-08-07` and
`max_output_tokens` raised from 8192 to 32768.

GPT-5 was dropped from the core experiments in [519_faktencheck_core](../519_faktencheck_core)
because about 20% of the chunks failed (JSONDecodeError or MissingResponseContentError). These come
from output-budget truncation: reasoning tokens and the visible answer share `max_output_tokens`.
This run checks on the dev set that the larger budget removes them before we spend on the test set.

Best setup (with chunking) as in
[397_faktencheck_core_v1_for_chunking](../397_faktencheck_core_v1_for_chunking) and
[519_faktencheck_core](../519_faktencheck_core). Single seed, since the seed does not change anything
on the OpenAI side.

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

## Insights

- The run went through for all 100 dev PDFs. As before the fix, the job itself never crashed; what
  failed were single requests for some PDFs, which get logged as per-chunk errors.
- The error rate dropped from about 20% to 3.7%. The same dev set and config in
  [519_faktencheck_core](../519_faktencheck_core) (old 8192 budget) had 336 and 350 errors over 1654
  chunks for its two GPT-5 seeds; here it is 61 over 1655.
- `MissingResponseContentError` is gone: 123-127 before, 0 now. This was the pure truncation case,
  where reasoning used up the whole budget and left no answer, and the larger budget removes it.
- `JSONDecodeError` is the error that actually matters, since we cannot recover from it: a truncated
  result JSON is unusable. It went from ~194 to 36. The remaining ones are outlier PDFs whose output
  is large enough to still overrun even the 32768 budget.
- `ReasoningExtractionError` stayed the same (~28 vs 27), as expected since it has nothing to do with
  the budget: GPT-5 sometimes returns an answer with no reasoning summary although
  `reasoning_options.summary=auto` is set. It is non-breaking: the entry is logged and moved from
  "without error" to "with error" in the overview figures, and we lose the reasoning for analysis,
  but the output stays usable. Tracked in [#575](https://github.com/DFKI-NLP/kibad-llm/issues/575).
- Performance is slightly lower than before the fix: ALL F1 0.696 vs 0.712 and 0.711 for the two
  GPT-5 seeds in 519 (same 914 support, corrected reference, flat micro). The model is identical, so
  the only change is the budget: the fix recovers chunks that used to error out and return nothing,
  which pushes recall up (0.864 vs 0.823) and precision down (0.583 vs 0.628).
- Overall the fix removes the MissingResponseContentError failures and cuts the error rate about 5x,
  so GPT-5 can go back into the core experiments. The one thing left to watch is the JSONDecodeError
  on outlier PDFs.
