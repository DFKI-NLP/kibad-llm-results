# 544_organism_trends_wald_corrected

uses same predictions as [428_organism_trends_with_chunking](../428_organism_trends_with_chunking) , but evaluated against the new (corrected) reference data ([Referenz_Wald_korrigiert_angepasst.csv](../../../external/organism_trends/Referenz_Wald_korrigiert_angepasst.csv))

## Evaluation
 - based on [428_organism_trends_with_chunking](../428_organism_trends_with_chunking)

### F1
 - all without `Untergruppe_RoteListen`

#### flattened
```Bash
uv run -m kibad_llm.evaluate \
name=544_organism_trends_wald_corrected \
experiment/evaluate=organism_trends_f1_micro_flat \
prediction_logs=logs/428_organism_trends_with_chunking/predict \
dataset.references.file=../external/organism_trends/Referenz_Wald_korrigiert_angepasst.csv \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
--multirun
```

result location: `logs/544_organism_trends_wald_corrected/evaluate/multiruns/2026-07-16_13-43-56`

#### full compounds

```Bash
uv run -m kibad_llm.evaluate \
name=544_organism_trends_wald_corrected \
experiment/evaluate=organism_trends_f1_micro \
prediction_logs=logs/428_organism_trends_with_chunking/predict \
dataset.references.file=../external/organism_trends/Referenz_Wald_korrigiert_angepasst.csv \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
--multirun
```

result location: `logs/544_organism_trends_wald_corrected/evaluate/multiruns/2026-07-16_13-44-13`

#### base elements

```Bash
uv run -m kibad_llm.evaluate \
name=544_organism_trends_wald_corrected \
experiment/evaluate=organism_trends_f1_micro_base_entries \
prediction_logs=logs/428_organism_trends_with_chunking/predict \
dataset.references.file=../external/organism_trends/Referenz_Wald_korrigiert_angepasst.csv \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
--multirun
```

result location: `logs/544_organism_trends_wald_corrected/evaluate/multiruns/2026-07-16_13-44-31`

#### `Antwortvariable` conditioned on base elements

```Bash
uv run -m kibad_llm.evaluate \
name=544_organism_trends_wald_corrected \
experiment/evaluate=organism_trends_f1_micro_conditional_variable_only \
prediction_logs=logs/428_organism_trends_with_chunking/predict \
dataset.references.file=../external/organism_trends/Referenz_Wald_korrigiert_angepasst.csv \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
--multirun
```

result location: `logs/544_organism_trends_wald_corrected/evaluate/multiruns/2026-07-16_13-45-02`

#### `Antwortvariable` & `Trend` conditioned on base elements

```Bash
uv run -m kibad_llm.evaluate \
name=544_organism_trends_wald_corrected \
experiment/evaluate=organism_trends_f1_micro_conditional_variable_and_trend \
prediction_logs=logs/428_organism_trends_with_chunking/predict \
dataset.references.file=../external/organism_trends/Referenz_Wald_korrigiert_angepasst.csv \
hydra.callbacks.save_job_return.multirun_show_file_contents=null \
--multirun
```

result location: `logs/544_organism_trends_wald_corrected/evaluate/multiruns/2026-07-16_13-45-22`

## Comparison with old (not corrected) reference data

- comparison data from [428_organism_trends_with_chunking](../428_organism_trends_with_chunking) (`file=`)

![legend.svg](figures/organism_trends_f1_micro-ALL/legend.svg)

### organism_trends_f1_micro-ALL

#### f1

![f1.svg](figures/organism_trends_f1_micro-ALL/f1.svg)

#### precision

![precision.svg](figures/organism_trends_f1_micro-ALL/precision.svg)

#### recall

![recall.svg](figures/organism_trends_f1_micro-ALL/recall.svg)

#### support

![support.svg](figures/organism_trends_f1_micro-ALL/support.svg)

### organism_trends_f1_micro_base_entries-ALL

#### f1

![f1.svg](figures/organism_trends_f1_micro_base_entries-ALL/f1.svg)

#### precision

![precision.svg](figures/organism_trends_f1_micro_base_entries-ALL/precision.svg)

#### recall

![recall.svg](figures/organism_trends_f1_micro_base_entries-ALL/recall.svg)

#### support

![support.svg](figures/organism_trends_f1_micro_base_entries-ALL/support.svg)

### organism_trends_f1_micro_conditional_variable_and_trend-ALL

#### f1

![f1.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-ALL/f1.svg)

<details>
<summary>click to see plots per `Hauptgruppe_RoteListen` & `Habitat`</summary>

##### Pflanzen&Wald

![organism_trends.Pflanzen&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-f1/organism_trends.Pflanzen%26Wald.svg)

#### Pilze_Flechten&Wald

![organism_trends.Pilze_Flechten&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-f1/organism_trends.Pilze_Flechten%26Wald.svg)

#### Wirbellose&Wald

![organism_trends.Wirbellose&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-f1/organism_trends.Wirbellose%26Wald.svg)

#### Wirbeltiere&Wald

![organism_trends.Wirbeltiere&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-f1/organism_trends.Wirbeltiere%26Wald.svg)

</details>

#### precision

![precision.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-ALL/precision.svg)

<details>
<summary>click to see plots per `Hauptgruppe_RoteListen` & `Habitat`</summary>

#### Pflanzen&Wald

![organism_trends.Pflanzen&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-precision/organism_trends.Pflanzen%26Wald.svg)

#### Pilze_Flechten&Wald

![organism_trends.Pilze_Flechten&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-precision/organism_trends.Pilze_Flechten%26Wald.svg)

#### Wirbellose&Wald

![organism_trends.Wirbellose&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-precision/organism_trends.Wirbellose%26Wald.svg)

#### Wirbeltiere&Wald

![organism_trends.Wirbeltiere&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-precision/organism_trends.Wirbeltiere%26Wald.svg)

</details>

#### recall

![recall.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-ALL/recall.svg)

<details>
<summary>click to see plots per `Hauptgruppe_RoteListen` & `Habitat`</summary>

#### Pflanzen&Wald

![organism_trends.Pflanzen&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-recall/organism_trends.Pflanzen%26Wald.svg)

#### Pilze_Flechten&Wald

![organism_trends.Pilze_Flechten&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-recall/organism_trends.Pilze_Flechten%26Wald.svg)

#### Wirbellose&Wald

![organism_trends.Wirbellose&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-recall/organism_trends.Wirbellose%26Wald.svg)

#### Wirbeltiere&Wald

![organism_trends.Wirbeltiere&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-recall/organism_trends.Wirbeltiere%26Wald.svg)

</details>

#### support

![support.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-ALL/support.svg)

<details>
<summary>click to see plots per `Hauptgruppe_RoteListen` & `Habitat`</summary>

#### Pflanzen&Wald

![organism_trends.Pflanzen&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-support/organism_trends.Pflanzen%26Wald.svg)

#### Pilze_Flechten&Wald

![organism_trends.Pilze_Flechten&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-support/organism_trends.Pilze_Flechten%26Wald.svg)

#### Wirbellose&Wald

![organism_trends.Wirbellose&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-support/organism_trends.Wirbellose%26Wald.svg)

#### Wirbeltiere&Wald

![organism_trends.Wirbeltiere&Wald.svg](figures/organism_trends_f1_micro_conditional_variable_and_trend-support/organism_trends.Wirbeltiere%26Wald.svg)

</details>