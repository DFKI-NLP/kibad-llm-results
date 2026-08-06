# 574_gpt5_faktencheck_core

GPT-5 on the Faktencheck core schema, 100-PDF dev set, using
`faktencheck_core_fields_schema_with_chunking`, as in
[397_faktencheck_core_v1_for_chunking](../397_faktencheck_core_v1_for_chunking) and
[519_faktencheck_core](../519_faktencheck_core).

**Motivation**: GPT-5 was dropped from the core experiments because about 20% of the chunks failed
with `JSONDecodeError` or `MissingResponseContentError` (see
[519_faktencheck_core](../519_faktencheck_core)). Reasoning tokens and the visible answer share
`max_output_tokens`, so the budget was raised from 8192 to 32768, the same change as for Nemotron in
[#523](https://github.com/DFKI-NLP/kibad-llm/issues/523). The model was also pinned to
`gpt-5-2025-08-07`. See [#533](https://github.com/DFKI-NLP/kibad-llm/issues/533) and
[#574](https://github.com/DFKI-NLP/kibad-llm/pull/574). This run measures the effect on the dev set
before we spend on the test set.

We use a single seed to limit cost.

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

The job finished on all 100 dev PDFs. The tables below compare it with the two GPT-5 runs in
[519_faktencheck_core](../519_faktencheck_core), which use the same dev set and setup with
`max_output_tokens: 8192`. The error counts come from
`logs/519_faktencheck_core/evaluate/multiruns/2026-06-16_14-57-41`, the f1, precision and recall
numbers from `.../2026-06-16_15-19-32`, jobs 3 and 4 in both cases. Support is the same in all three
runs, since they use the same reference.

### Error counts

|                             | 519, seed 42 | 519, seed 1337 | 574, seed 42 |
|:----------------------------|-------------:|---------------:|-------------:|
| chunks                      |         1654 |           1654 |         1655 |
| with_error                  |          336 |            350 |           61 |
| JSONDecodeError             |          193 |            195 |           36 |
| MissingResponseContentError |          123 |            127 |            0 |
| ReasoningExtractionError    |           26 |             30 |           27 |

### F1 per field

| field                   | 519, seed 42 | 519, seed 1337 | 574, seed 42 |
|:------------------------|-------------:|---------------:|-------------:|
| habitat                 |        0.810 |          0.846 |        0.833 |
| ecosystem_type.category |        0.810 |          0.799 |        0.783 |
| biodiversity_level      |        0.705 |          0.689 |        0.700 |
| taxa.species_group      |        0.678 |          0.678 |        0.657 |
| ecosystem_type.term     |        0.623 |          0.605 |        0.592 |
| ALL                     |        0.712 |          0.711 |        0.696 |

### Precision per field

| field                   | 519, seed 42 | 519, seed 1337 | 574, seed 42 |
|:------------------------|-------------:|---------------:|-------------:|
| habitat                 |        0.736 |          0.767 |        0.741 |
| ecosystem_type.category |        0.747 |          0.713 |        0.677 |
| biodiversity_level      |        0.633 |          0.612 |        0.596 |
| taxa.species_group      |        0.573 |          0.573 |        0.545 |
| ecosystem_type.term     |        0.542 |          0.523 |        0.469 |
| ALL                     |        0.628 |          0.622 |        0.583 |

### Recall per field

| field                   | 519, seed 42 | 519, seed 1337 | 574, seed 42 |
|:------------------------|-------------:|---------------:|-------------:|
| habitat                 |        0.899 |          0.942 |        0.952 |
| ecosystem_type.category |        0.886 |          0.907 |        0.929 |
| biodiversity_level      |        0.795 |          0.788 |        0.848 |
| taxa.species_group      |        0.831 |          0.831 |        0.826 |
| ecosystem_type.term     |        0.733 |          0.717 |        0.800 |
| ALL                     |        0.823 |          0.829 |        0.864 |

Errors drop from 336 and 350 to 61, while ALL f1 goes from 0.712 and 0.711 to 0.696: recall is
higher (0.864 vs 0.823 and 0.829) and precision lower (0.583 vs 0.628 and 0.622). Per field, recall
is higher than in both 519 runs except for `taxa.species_group`, and precision is lower than in both
except for `habitat`, where it falls between the two 519 seeds.

`JSONDecodeError` is the remaining error that matters, since a broken result JSON cannot be used. We
did not look into the 36 remaining cases. `ReasoningExtractionError` is non-breaking: the entry is
counted as "with error" in the overview figures and we lose the reasoning for analysis, but the
output can still be used. It is tracked in
[#575](https://github.com/DFKI-NLP/kibad-llm/issues/575).

The budget increase removed `MissingResponseContentError` (123 and 127 in 519, 0 here) and reduced
`JSONDecodeError` from 193 and 195 to 36, which is what the change was meant to do. The test set and
organism-trends runs follow with the same config.
