# 574_gpt5_faktencheck_core

GPT-5 on the Faktencheck core schema, 100-PDF dev set, using
`faktencheck_core_fields_schema_with_chunking`, as in
[397_faktencheck_core_v1_for_chunking](../397_faktencheck_core_v1_for_chunking) and
[519_faktencheck_core](../519_faktencheck_core).

**Motivation**: in [519_faktencheck_core](../519_faktencheck_core) about 20% of the GPT-5 chunks
failed with `JSONDecodeError` or `MissingResponseContentError` (316 and 322 of 1654 for the two
seeds), see [#533](https://github.com/DFKI-NLP/kibad-llm/issues/533). The `gpt_5` slot in
[525_faktencheck_core_bestconfig_testset](../525_faktencheck_core_bestconfig_testset) was deleted
for the same reason. Reasoning tokens and the visible answer share `max_output_tokens`, so the
budget was raised from 8192 to 32768, the same raise that was made for Nemotron in
[#523](https://github.com/DFKI-NLP/kibad-llm/issues/523), and the model was pinned to
`gpt-5-2025-08-07`, see [#574](https://github.com/DFKI-NLP/kibad-llm/pull/574).

We use a single seed to limit cost. The pin is for future runs, it does not change the model here:
the `gpt-5` identifier used in 519 already resolved to `gpt-5-2025-08-07`.

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

All 100 dev PDFs processed.

Comparing this run with the two GPT-5 runs in [519_faktencheck_core](../519_faktencheck_core), same
dev set and setup, `max_output_tokens: 8192`. Error counts from
`logs/519_faktencheck_core/evaluate/multiruns/2026-06-16_14-57-41`, f1, precision and recall from
`.../2026-06-16_15-19-32`, jobs 3 and 4 in both. Same reference, so support is identical.

### Error counts

|                             | 519, seed 42 | 519, seed 1337 | 574, seed 42 |
|:----------------------------|-------------:|---------------:|-------------:|
| chunks                      |         1654 |           1654 |         1655 |
| with_error                  |          336 |            350 |           61 |
| JSONDecodeError             |          193 |            195 |           36 |
| MissingResponseContentError |          123 |            127 |            0 |
| ReasoningExtractionError    |           26 |             30 |           27 |

Chunk totals differ by one, eight of the 100 documents chunk differently between the runs.

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

Notes
- Errors down from 336 and 350 to 61
- `MissingResponseContentError` gone (123 and 127 before, 0 now), `JSONDecodeError` down from 193
  and 195 to 36. That is what the budget raise was for
- ALL f1 slightly lower at 0.696 vs 0.712 and 0.711, on one seed against two
- Recall up (0.864 vs 0.823 and 0.829), precision down (0.583 vs 0.628 and 0.622)
- Per field, recall up against both 519 seeds except `taxa.species_group`, precision down against
  both except `habitat`
- `JSONDecodeError` is the error that matters, a broken result JSON is unusable. We did not look
  into the 36 cases
- `ReasoningExtractionError` is non-breaking, output still there for 25 of the 27. Tracked in
  [#575](https://github.com/DFKI-NLP/kibad-llm/issues/575)

The two exceptions are chunk 223 of `3WEEGFGW.pdf` and chunk 40 of `84QQ9F5S.pdf`, both from
`dev-set-100`. They raised a `JSONDecodeError` on top of the `ReasoningExtractionError` and have no
output, which is why 36 and not 34 chunks end up unusable. Found by scanning the committed
predictions:

```sh
python3 -c "
import json
for l in open('predictions/574_gpt5_faktencheck_core/2026-07-27_12-23-00/2026-07-27_12-23-04_547890/predictions.jsonl'):
    d = json.loads(l)
    for i, (err, out) in enumerate(zip(d['errors_list'], d['structured_with_metadata_list'])):
        if err and 'ReasoningExtractionError' in str(err) and not out:
            print(d['file_name'], i, err)
"
```

The error rate is now at the level of the other models, so GPT-5 goes back into the test set
experiments: [574_gpt5_faktencheck_core_testset](../574_gpt5_faktencheck_core_testset),
[574_gpt5_organism_trends](../574_gpt5_organism_trends) and
[574_gpt5_organism_trends_testset](../574_gpt5_organism_trends_testset).
