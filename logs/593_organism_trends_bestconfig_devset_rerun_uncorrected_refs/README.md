# 593_organism_trends_bestconfig_devset_rerun_uncorred_refs

Evaluation of the best setup (with chunking) from [593_organism_trends_bestconfig_devset_rerun_uncorrected_refs](../593_organism_trends_bestconfig_devset_rerun_uncorrected_refs), 
but with the full set of input PDFs (59 files were missing) and the uncorrected (!) reference file.

## Evaluation

### F1, P, R
Base for the command is https://github.com/DFKI-NLP/kibad-llm/tree/main/data/prediction_results/logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs

##### flattened
```sh
uv run -m kibad_llm.evaluate \
name=593_organism_trends_bestconfig_devset_rerun_uncorrected_refs \
experiment/evaluate=organism_trends_f1_micro_flat \
prediction_logs=logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Wald Literatur - Sheet1.csv" \
--multirun
```

Saved to `logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/evaluate/multiruns/XXX`

##### full compounds

```sh
uv run -m kibad_llm.evaluate \
name=593_organism_trends_bestconfig_devset_rerun_uncorrected_refs \
experiment/evaluate=organism_trends_f1_micro \
prediction_logs=logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Wald Literatur - Sheet1.csv" \
--multirun
```

Saved to `logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/evaluate/multiruns/XXX`

##### base elements

```sh
uv run -m kibad_llm.evaluate \
name=593_organism_trends_bestconfig_devset_rerun_uncorrected_refs \
experiment/evaluate=organism_trends_f1_micro_base_entries \
prediction_logs=logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Wald Literatur - Sheet1.csv" \
--multirun
```

Saved to `logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/evaluate/multiruns/XXX`

##### `Antwortvariable` conditioned on base elements

```sh
uv run -m kibad_llm.evaluate \
name=593_organism_trends_bestconfig_devset_rerun_uncorrected_refs \
experiment/evaluate=organism_trends_f1_micro_conditional_variable_only \
prediction_logs=logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Wald Literatur - Sheet1.csv" \
--multirun
```

Saved to `logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/evaluate/multiruns/XXX`

##### `Antwortvariable` & `Trend` conditioned on base elements

```sh
uv run -m kibad_llm.evaluate \
name=593_organism_trends_bestconfig_devset_rerun_uncorrected_refs \
experiment/evaluate=organism_trends_f1_micro_conditional_variable_and_trend \
prediction_logs=logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/predict \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
dataset.references.file="../external/organism_trends/Weighted Vote Count Wald Literatur - Sheet1.csv" \
--multirun
```

Saved to `logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/evaluate/multiruns/XXX`

### Errors

```sh
uv run -m kibad_llm.evaluate \
name=593_organism_trends_bestconfig_devset_rerun_uncorrected_refs \
experiment/evaluate=prediction_errors \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
prediction_logs=[\
logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/predict \
] \
--multirun
```

Saved to `logs/593_organism_trends_bestconfig_devset_rerun_uncorrected_refs/evaluate/multiruns/XXX`

## Outcome

### F1, P, R

The results in this folder can serve as a basis for the [Journal experiments](https://github.com/DFKI-NLP/kibad-llm/issues/521),
namely for the Organism Trend schema plots:

#### flattened

Legend

![legend.svg](figures/organism_trends_f1_micro_flat-ALL/legend.svg)

Micro-F1 (ALL.f1)

![Micro F1, flattened evaluation](figures/organism_trends_f1_micro_flat-ALL/f1.svg) 

Micro-Precision (ALL.precision)

![Micro Precision, flattened evaluation](figures/organism_trends_f1_micro_flat-ALL/precision.svg)

Micro-Recall (ALL.recall)

![Micro Recall, flattened evaluation](figures/organism_trends_f1_micro_flat-ALL/recall.svg)

TODO Notes (to update!)
- Micro-F1 on the flatted schema (per-field evaluation)  - Qwen best with 0.436
- Qwen has very good precision at 0.447, all other models much lower
- Recall is similar across models (0.53-0.56), except for Qwen (0.43)
- Flattened results are approx 7-15% (Gemma) lower than the results on the core schema, which had a best 0.504 F1 for GPT OSS 20B and Qwen3, and a low of 0.438 for Gemma and Mistral
- Compared to dev set results [428_organism_trends_with_chunking](../428_organism_trends_with_chunking), F1 is better for
  GPT OSS at 0.378 (vs 0.33), worse for Qwen3 (0.436 now vs approx 0.47 then), and better for Mistral (0.308 vs 0.23) and Gemma (0.271 vs 0.24)

#### full compounds

Legend

![legend.svg](figures/organism_trends_f1_micro-ALL/legend.svg)

Micro-F1 (ALL.f1)

![Figure/Table 1 "main pipeline results": F1 scores for the best configuration (prompt+chunking+...)](figures/organism_trends_f1_micro-ALL/f1.svg) 

Micro-Precision (ALL.precision)

![Figure/Table 2: "detail results - precision and recall" - same plots as above, but with precision scores instead of F1](figures/organism_trends_f1_micro-ALL/precision.svg)

Micro-Recall (ALL.recall)

![Figure/Table 2: "detail results - precision and recall" - same plots as above, but with recall scores instead of F1](figures/organism_trends_f1_micro-ALL/recall.svg)

TODO Notes (to update)
- F1 scores range from 0.17 (Qwen3) to 0.018 (Gemma)
- Compared to the dev set results, results are worse by 2-7% - GPT OSS (0.111 vs 0.135), Qwen (0.172 vs 0.24), Mistral (0.06 vs 0.06), Gemma (0.018 vs 0.075)

#### base elements

Legend

![legend.svg](figures/organism_trends_f1_micro_base_entries-ALL/legend.svg)

Micro-F1 (ALL.f1)

![Base elements Micro-F1](figures/organism_trends_f1_micro_base_entries-ALL/f1.svg) 

Micro-Precision (ALL.precision)

![Base elements Micro-Precision](figures/organism_trends_f1_micro_base_entries-ALL/precision.svg)

Micro-Recall (ALL.recall)

![Base elements, Micro-Recall](figures/organism_trends_f1_micro_base_entries-ALL/recall.svg)

TODO Notes (to update)
- Gemma3 performs much worse than the other models on detecting base elements, not sure why this happens since in the core schema, 
  it performs much better for the 2 variables habitat and species group (0.67 F1 for habitat, 0.57 for species group, 
  see https://github.com/DFKI-NLP/kibad-llm/blob/main/data/prediction_results/logs/397_faktencheck_core_v1_for_chunking/f1_per_class.png)
- Compared to the dev set results, where Gemma achieved about 0.2 F1, the 0.06 here are a 14% drop
- For Qwen and GPT OSS, the drop from the dev set is approx 14% (Qwen) and 5% (GPT OSS)

#### `Antwortvariable` conditioned on base elements

Legend

![legend.svg](figures/organism_trends_f1_micro_conditional_variable_only-ALL/legend.svg)

F1

![Micro-F1](figures/organism_trends_f1_micro_conditional_variable_only-ALL/f1.svg) 

Precision

![Micro-Precision](figures/organism_trends_f1_micro_conditional_variable_only-ALL/precision.svg)

Recall

![Micro-Recall](figures/organism_trends_f1_micro_conditional_variable_only-ALL/recall.svg)

TODO Notes (to update)
- F1 scores are 3-5% lower than on the dev set
- Recall is better, precision lower than on the dev set

#### `Antwortvariable` & `Trend` conditioned on base elements

Legend

![legend.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-ALL/legend.svg)

F1

![Micro-F1](figures/organism_trends_f1_micro_conditional_variable_and_trend-ALL/f1.svg) 

Precision

![Micro-Precision](figures/organism_trends_f1_micro_conditional_variable_and_trend-ALL/precision.svg)

Recall

![Micro-Recall](figures/organism_trends_f1_micro_conditional_variable_and_trend-ALL/recall.svg)

TODO Notes (to update)
- F1 is better than on the dev set by 5-10%
- Qwen3 is best at 0.5 F1, 0.47 precision and 0.54 recall

### Errors

![legend.svg](figures/prediction_errors-total/legend.svg)

### no error

![no_error.svg](figures/prediction_errors-total/no_error.svg)

### with error

![with_error.svg](figures/prediction_errors-total/with_error.svg)

TODO Notes (to update)
- Mistral has the most errors (approx 64), but this is still negligible compared to the approx 2600 chunks processed.


