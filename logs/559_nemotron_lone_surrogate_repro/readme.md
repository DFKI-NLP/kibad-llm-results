# 559_nemotron_lone_surrogate_repro

Attempts to reproduce the `UnicodeEncodeError` lone-surrogate crash from
[issue #555](https://github.com/DFKI-NLP/kibad-llm/issues/555) live on the cluster, on top of the
fix in [PR #559](https://github.com/DFKI-NLP/kibad-llm/pull/559). Both prior occurrences of the bug
happened with `nemotron_nano_3_30b_in_process` and `seed=7331` on the 100-PDF dev set, so both
attempts here reuse that exact setup, pinned to the fix commit.

## Prediction

**Attempt 1: single seed, matching the original standalone rerun that first hit the bug.**

```sh
./run_in_process.sh -t "2-00:00:00" -pa "H100-SLT,H100-Trails,H100,H200,B200" -ng 2 -sr \
  -r 865104bf111404cd728b94505579e8377a92616d \
  -u "-m kibad_llm.predict \
  name=559_nemotron_lone_surrogate_repro \
  experiment/predict=faktencheck_core_fields_schema_with_chunking \
  pdf_directory=/ds/text/kiba-d/dev-set-100 \
  extractor/llm=nemotron_nano_3_30b_in_process \
  extractor.llm.vllm_kwargs.tensor_parallel_size=2 \
  pdf_reader_num_proc=200 \
  seed=7331"
```

Result location: `logs/559_nemotron_lone_surrogate_repro/predict/runs/2026-07-14_14-25-40` (this is **not persisted**)

Completed cleanly after ~11h35m. No `UnicodeEncodeError` was raised. This run did a fresh, uncached
PDF conversion (`time_pdf_conversion` ≈ 1228 s ≈ 20 min), so the cold-cache condition was covered
here.

**Attempt 2: full multirun with all three original seeds, matching the original
251_nemotron_faktencheck_core conditions.**

```sh
./run_in_process.sh -t "2-00:00:00" -pa "H100-SLT,H100-Trails,H100,H200,B200" -ng 2 -sr \
  -r 865104bf111404cd728b94505579e8377a92616d \
  -u "-m kibad_llm.predict \
  name=559_nemotron_lone_surrogate_repro \
  experiment/predict=faktencheck_core_fields_schema_with_chunking \
  pdf_directory=/ds/text/kiba-d/dev-set-100 \
  extractor/llm=nemotron_nano_3_30b_in_process \
  extractor.llm.vllm_kwargs.tensor_parallel_size=2 \
  pdf_reader_num_proc=200 \
  seed=42,1337,7331 \
  --multirun"
```

Result location: `logs/559_nemotron_lone_surrogate_repro/predict/multiruns/2026-07-16_14-22-34` (this is **not persisted**)

All three seeds completed cleanly (~11.4-11.6h extraction time each). No `UnicodeEncodeError` was
raised for any seed. These runs reused the PDF conversion cached by Attempt 1 (`time_pdf_conversion`
< 1 s each), so they exercised the warm-cache condition.

## Outcome

Neither attempt reproduced the bug, even though both reused the exact model, dataset, and seed(s)
that triggered the two prior occurrences. This is consistent with the bug being a rare,
nondeterministic event tied to specific generation output, not something reliably reproducible on
demand with a fixed seed.

Between the two attempts, both PDF-conversion cache states were covered: Attempt 1 ran a fresh,
uncached conversion and Attempt 2 ran fully cached. Neither triggered the crash, so a warm
conversion cache does not, on its own, explain the non-reproduction.

Note that [PR #559](https://github.com/DFKI-NLP/kibad-llm/pull/559) fixes the crash, not the root
cause. A lone surrogate is now surfaced per document instead of taking down the whole job, but
Nemotron can still occasionally emit a malformed `\uXXXX` escape in its raw output. That model-level
behavior is unchanged.

No prediction outputs from either run were copied into `data/results/predictions/`, since this
experiment's purpose was crash reproduction, not F1 evaluation.

## References

- Issue: https://github.com/DFKI-NLP/kibad-llm/issues/555
- Fix PR: https://github.com/DFKI-NLP/kibad-llm/pull/559
- Prior occurrences: [251_nemotron_faktencheck_core](../251_nemotron_faktencheck_core)
- Minimal synthetic reproduction: posted directly in
  [issue #555](https://github.com/DFKI-NLP/kibad-llm/issues/555) (`json.loads` accepting an
  unpaired `\uXXXX` escape, then `Dataset.map` failing on the resulting lone surrogate). Used as
  the reference reproduction for this bug instead of a live cluster capture.
