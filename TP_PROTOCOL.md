# Practical workshop protocol

## Assessing climate-change impacts on rainfed wheat in Morocco with AgriScale and DSSAT

**Duration:** 3 hours  
**Study area:** Tensift region, Morocco (31.68° N, 7.5961° W)  
**Crop model:** DSSAT/CERES-Wheat  
**Simulation period:** 30 successive growing seasons  
**Target audience:** participants with no prior crop-modelling experience; statistical experience is welcome

## 1. Purpose

This practical exercise introduces the use of AgriScale to investigate how simple climate perturbations affect simulated wheat production. Participants will prepare four otherwise identical DSSAT input databases, run the simulations, inspect the principal outputs, and quantify changes in grain yield.

The exercise is a **climate-sensitivity experiment**, not a climate projection. Adding 2 °C and multiplying rainfall by 0.8 are controlled perturbations that help isolate model responses. They do not reproduce a complete future climate scenario and do not change solar radiation, atmospheric CO₂, wind, humidity, or rainfall occurrence.

## 2. Learning outcomes

By the end of the workshop, participants should be able to:

1. identify the weather, soil, cultivar, management, and simulation components of an AgriScale `MasterInput` database;
2. explain how one-at-a-time and combined climate perturbations define a 2 × 2 factorial design;
3. generate DSSAT inputs and run successive rainfed wheat simulations with AgriScale;
4. distinguish a distributional comparison from a year-paired comparison;
5. calculate absolute and relative yield changes;
6. interpret confidence intervals, paired tests, and multiplicity-adjusted p-values;
7. state the main limitations of inference from a deterministic crop-model sensitivity experiment.

## 3. Experimental configuration

All factors other than temperature and rainfall must remain identical across the four scenarios.

| Component | Configuration |
|---|---|
| Location | 31.68° N, 7.5961° W, Tensift region |
| Crop | Wheat |
| DSSAT cultivar | `karim` |
| Soil | `MA_TENSIFT_XEROSOL` |
| Initial conditions | `MA_AWC50` |
| Sowing | 15 January of each simulated growing season |
| Plant density | 225 plants m⁻² |
| Water management | Rainfed: `IrrigationPolicyCode = 0` |
| Fertilization | No mineral fertilizer in the current successive treatment: `InoFertiPolicyCode = 0` |
| Simulation mode | Successive seasons, retaining soil-state carry-over |

The management identifier contains the legacy text `IA55`, but the successive treatment used in this exercise has irrigation and inorganic-fertilization policy codes equal to zero. Participants should interpret the database fields, not infer management from the identifier alone.

### Climate treatments

| Scenario | Temperature treatment | Rainfall treatment | Database |
|---|---:|---:|---|
| HIST | unchanged | unchanged | `MasterInput_HIST.db` |
| TPLUS2 | Tmin, Tmax and Tmean +2 °C | unchanged | `MasterInput_TPLUS2.db` |
| RAIN80 | unchanged | daily rainfall × 0.80 | `MasterInput_RAIN80.db` |
| TPLUS2_RAIN80 | Tmin, Tmax and Tmean +2 °C | daily rainfall × 0.80 | `MasterInput_TPLUS2_RAIN80.db` |

This is a paired 2 × 2 design: temperature has levels 0 and +2 °C, while rainfall has levels 100% and 80%. The same historical days and years are retained in all treatments.

## 4. Required material

Before the session, verify that participants have:

- a working local AgriScale environment;
- DSSAT 4.8 available to ModFileGen;
- Jupyter support and Python packages required by the notebook, including `matplotlib` and `scipy`;
- access to this directory and sufficient disk space;
- the source databases `MasterInput.db` and `ModelsDictionaryArise.db`;
- the notebook `../scripts/prepare_climate_databases.ipynb`.

Do not edit the source `MasterInput.db` during the exercise. The notebook creates scenario copies before changing climate values.

## 5. Workshop sequence

### Part A — Understand the modelling experiment (20 minutes)

1. Identify the scientific question: how do +2 °C warming, a 20% rainfall reduction, and their combination affect simulated rainfed wheat yield?
2. Locate the five principal input categories: climate, soil, cultivar, management, and simulation options.
3. Discuss which variables are held constant and which are manipulated.
4. Formulate hypotheses before viewing the results:
   - warming may accelerate phenology and alter water demand;
   - reduced rainfall should increase water limitation;
   - the combined response need not equal the sum of the two separate responses.

### Part B — Prepare and verify the climate scenarios (30 minutes)

Open and execute the database-preparation section of `prepare_climate_databases.ipynb`.

For every scenario, verify:

1. row counts and date coverage are unchanged;
2. identifiers distinguish the scenario outputs;
3. HIST values equal the source values;
4. TPLUS2 temperatures equal HIST +2 °C;
5. RAIN80 rainfall equals HIST ×0.8;
6. the combined database applies both transformations;
7. SQLite connections are closed and journal files do not remain after a successful checkpoint.

Use the monthly climatology plot to confirm the transformations visually. Temperature curves should be shifted upward without changing their seasonal shape. Monthly rainfall totals should be 20% lower while retaining the same temporal pattern.

### Part C — Run DSSAT with AgriScale (35 minutes)

Execute the four DSSAT sections in the notebook in this order:

1. HIST;
2. RAIN80;
3. TPLUS2;
4. TPLUS2_RAIN80.

For each run:

1. confirm that DSSAT finishes without a run-time error;
2. record the generated output directory;
3. verify that a `*_dssat_successive.csv` file was created;
4. confirm that the same harvest years occur in every scenario;
5. inspect at least grain yield, anthesis date, maturity date, rainfall, evapotranspiration, and water-stress indicators when available.

If several CSV files exist in an output directory, the analysis selects the most recently modified successive result. For a reproducible training session, archive or remove obsolete outputs before distributing the exercise dataset.

### Part D — Describe yield distributions (20 minutes)

Run analysis step 1 in the notebook.

1. Compare the four boxplots.
2. Examine medians, means, dispersion, overlap, and extreme years.
3. Explain why overlap between boxplots is not a significance test.
4. Explain why the four samples are not independent: each scenario uses the same sequence of climate years.

**Question:** Which perturbation appears to have the largest effect on central yield and which produces the greatest relative variability?

### Part E — Analyse paired annual changes (25 minutes)

Run analysis step 2. For year \(y\), calculate:

\[
\Delta Y_y = Y_{scenario,y} - Y_{HIST,y}
\]

and:

\[
\Delta Y_{relative,y} = 100 \times
\frac{Y_{scenario,y}-Y_{HIST,y}}{Y_{HIST,y}}.
\]

Interpret negative values as losses relative to HIST and positive values as gains. HIST is the reference and therefore has an absolute and relative difference of zero.

Discuss why percentage changes can become extreme when the HIST yield for a year is small. Use absolute changes for the principal inferential analysis and retain relative changes as a complementary, scale-free description.

**Questions:**

1. How many years show a loss under each scenario?
2. Is the temperature-only response consistent among years?
3. Do rainfall reduction and the combined treatment affect the same years equally strongly?

### Part F — Confidence intervals and paired tests (30 minutes)

Run analysis step 3.

For each non-reference scenario, the notebook:

1. estimates the mean paired yield difference;
2. constructs a two-sided 95% Student confidence interval;
3. applies a paired t-test of a zero mean difference;
4. applies a paired Wilcoxon signed-rank test as a rank-based sensitivity analysis;
5. adjusts the three p-values with Holm's procedure.

Read the forest plot as follows:

- the point is the estimated mean difference;
- the horizontal segment is its 95% confidence interval;
- the vertical zero line is the HIST reference;
- an interval crossing zero is compatible with no mean change at the 5% level;
- the effect magnitude and uncertainty are more informative than the p-value alone.

**Questions:**

1. Do the t-test and Wilcoxon test lead to the same conclusions?
2. Does statistical non-significance demonstrate the absence of an effect?
3. Why is a correction for three comparisons appropriate?

### Part G — Synthesis and scaling up (20 minutes)

Each group prepares a short conclusion containing:

1. the estimated effects in t ha⁻¹ and their uncertainty;
2. the direction and consistency of annual responses;
3. one plausible agronomic mechanism;
4. at least two limitations;
5. one proposal for extending the analysis spatially or experimentally.

Conclude by identifying what changes when the workflow moves from one point to a gridded domain: data volume, number of simulation units, task partitioning, data locality, execution time, failure recovery, and output aggregation. The scientific treatment definitions should remain identical.

## 6. Expected findings for the facilitator

With the currently available outputs (30 paired harvest years), approximate results are:

| Scenario vs HIST | Mean change (t ha⁻¹) | Mean annual relative change | Years with lower yield | 95% CI for mean change |
|---|---:|---:|---:|---:|
| TPLUS2 | −0.19 | −5.8% | 20/30 | [−0.46, +0.08] |
| RAIN80 | −1.80 | −45.6% | 30/30 | [−2.18, −1.43] |
| TPLUS2_RAIN80 | −2.03 | −51.4% | 30/30 | [−2.45, −1.61] |

The temperature-only mean difference is not significant at 5%, whereas rainfall reduction and the combined treatment remain highly significant after Holm correction. These values are checks for the current dataset, not universal conclusions for Moroccan wheat.

## 7. Interpretation limits

Participants must state the following cautions:

- DSSAT outputs are deterministic conditional on the supplied inputs and parameters;
- variation among years represents the sampled historical weather sequence, not uncertainty in cultivar parameters or model structure;
- uniform temperature and rainfall perturbations are sensitivity tests, not bias-corrected climate projections;
- annual differences may be temporally correlated;
- successive simulations retain soil-state memory, so seasons are not necessarily independent;
- conclusions from one location, soil, cultivar, sowing date, and management cannot be generalized to all of Morocco;
- the experiment does not quantify uncertainty from multiple climate models, emission pathways, crop models, soils, cultivars, or adaptation strategies.

If temporal dependence is detected, a later analysis should replace the simple independent-year confidence interval with an appropriate method such as a moving-block bootstrap or a time-series model.

## 8. Participant deliverables

Each group submits:

1. the monthly climate comparison figure;
2. the four-scenario yield boxplot;
3. the annual absolute and relative difference figure;
4. the confidence-interval forest plot and paired-test table;
5. a maximum 200-word interpretation separating effect size, uncertainty, statistical evidence, and model limitations.

## 9. Optional extension

For participants who finish early, use the four scenarios as a 2 × 2 factorial experiment. Estimate the average temperature effect, average rainfall effect, and temperature × rainfall interaction from the four paired annual responses. This distinguishes the combined impact from simple additivity and provides the natural next step after the paired comparisons.
