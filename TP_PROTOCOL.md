# Practical workshop protocol

## Assessing climate-change impacts on rainfed wheat in Morocco with AgriScale and DSSAT

**Duration:** 3 hours
**Study area:** Tensift region, Morocco (31.68° N, 7.5961° W)
**Crop model:** DSSAT/CERES-Wheat
**Simulation period:** 30 successive growing seasons
**Audience:** no crop-modelling experience required; some statistics background is useful

## 1. Purpose

Participants build four DSSAT input databases that are identical except for temperature and rainfall, run them through AgriScale, and compare the resulting wheat yields.

This is a sensitivity experiment, not a climate projection. Adding 2°C and cutting rainfall by 20% are controlled perturbations meant to isolate the model's response — they do not represent a real future scenario, and radiation, CO₂, wind, humidity and rainfall timing are untouched.

## 2. What this covers

By the end of the session, participants should be able to:

- read the weather, soil, cultivar, management and simulation tables of a `MasterInput` database;
- build a 2×2 climate perturbation and run it as successive seasons in AgriScale;
- tell a distributional comparison apart from a year-paired one;
- read a confidence interval and a p-value without over-claiming what they show;
- list what a deterministic, single-site sensitivity experiment cannot tell you.

## 3. Experimental setup

Keep every factor identical across the four scenarios except temperature and rainfall.

| Component | Value |
|---|---|
| Location | 31.68° N, 7.5961° W, Tensift region |
| Crop | Wheat |
| DSSAT cultivar | `karim` |
| Soil | `MA_TENSIFT_XEROSOL` |
| Initial conditions | `MA_AWC50` |
| Sowing | 15 January, every season |
| Plant density | 225 plants m⁻² |
| Water | Rainfed (`IrrigationPolicyCode = 0`) |
| Fertilizer | None (`InoFertiPolicyCode = 0`) |
| Mode | Successive seasons, soil state carried over |

The management identifier still contains the legacy text `IA55`. Ignore it — check the policy codes in the database, not the label.

### Climate scenarios

| Scenario | Temperature | Rainfall | Database |
|---|---:|---:|---|
| HIST | unchanged | unchanged | `MasterInput_HIST.db` |
| TPLUS2 | Tmin/Tmax/Tmean +2°C | unchanged | `MasterInput_TPLUS2.db` |
| RAIN80 | unchanged | ×0.80 daily | `MasterInput_RAIN80.db` |
| TPLUS2_RAIN80 | +2°C | ×0.80 daily | `MasterInput_TPLUS2_RAIN80.db` |

Same years, same days, in all four databases — only temperature and rainfall levels change.

## 4. Before the session, check that participants have

- a working AgriScale environment;
- DSSAT 4.8 reachable from ModFileGen;
- Jupyter, `matplotlib` and `scipy` installed;
- read access to this directory with enough free disk space;
- the source files `MasterInput.db` and `ModelsDictionaryArise.db`;
- the notebook `../scripts/prepare_climate_databases.ipynb`.

Do not touch the source `MasterInput.db` — the notebook copies it before applying any climate change.

## 5. Workshop sequence

### Part A — Frame the experiment

1. State the question: how do +2°C, −20% rainfall, and their combination change simulated rainfed wheat yield?
2. Locate the five input categories in the database: climate, soil, cultivar, management, simulation options.
3. Identify what stays fixed and what is manipulated.
4. Before looking at any output, write down a prediction for each scenario and for the combined one.

### Part B — Build and check the climate scenarios

Open `prepare_climate_databases.ipynb` and run the database-preparation cells.

For each of the four databases, check:

1. row counts and date ranges match the source;
2. scenario identifiers are distinct;
3. HIST values equal the source values;
4. TPLUS2 temperatures equal HIST + 2°C;
5. RAIN80 rainfall equals HIST × 0.8;
6. the combined database applies both changes;
7. no leftover SQLite journal file after the checkpoint.

Then look at the monthly climatology plot: temperature curves shift up without changing shape, rainfall drops by a fifth without changing the seasonal pattern.

### Part C — Run DSSAT through AgriScale

Run the four DSSAT sections in this order: HIST, RAIN80, TPLUS2, TPLUS2_RAIN80.

For each run:

1. check DSSAT finished without error;
2. note the output directory;
3. confirm a `*_dssat_successive.csv` file exists;
4. confirm the same 30 harvest years appear in every scenario;
5. look at grain yield, anthesis and maturity dates, rainfall, evapotranspiration, and water stress where available.

If an output directory holds more than one CSV, the notebook picks the most recently modified one — clear out old runs before the session so nobody analyses stale results.

### Part D — Compare yield distributions

Run analysis step 1.

1. Compare the four boxplots: medians, spread, overlap, outlier years.
2. For each scenario, call it from the boxplot alone: significant difference from HIST or not? Write the call down — you'll check it against the confidence intervals in Part F.
3. Explain why the four samples are not independent of each other.

**Ask the group:** which perturbation shifts central yield the most, and which one spreads the results the most?

### Part E — Look at paired annual changes

Run analysis step 2. For each year, the notebook computes the absolute change from HIST and the same change as a percentage.

Discuss why a percentage change can blow up when the HIST yield for that year is small, and why the absolute change is the safer number to reason with.

**Ask the group:**

1. In how many years does each scenario lose yield?
2. Is the temperature-only response the same sign every year, or does it flip?
3. Do the rainfall and combined scenarios hit the same years, or different ones?

### Part F — Test the paired differences

Run analysis step 3. Before reading the notebook's output, have the group decide: given that the four samples share the same years and are not independent, what kind of test or interval is appropriate here, and why would a plain two-sample test be wrong?

Then read what the notebook actually reports — including the forest plot, where the point is the estimated mean difference, the bar is its confidence interval, and the vertical line marks zero.

**Ask the group:**

1. Do the different tests the notebook runs agree with each other? If not, why might that be?
2. Given only 30 simulated years, do you have enough power to detect an effect the size of TPLUS2's? What would change your answer — more years, a larger perturbation, or a different test?
3. Compare the raw and Holm-adjusted p-values. Does the adjustment change which scenarios you'd call significant, or only the margin?
4. Check your Part D calls against these intervals — where did visual intuition and the formal test disagree, and why?

### Part G — Synthesise and scale up

Each group writes a short conclusion with:

1. the estimated effect in t ha⁻¹ for each scenario, with its uncertainty;
2. whether the yearly response is consistent or scenario-dependent;
3. one agronomic explanation for the pattern;
4. two limitations of this experiment;
5. one way to extend it — spatially or experimentally.

Close with: what changes when this moves from one point to a gridded domain — data volume, number of simulation units, task partitioning, data locality, run time, failure recovery, output aggregation. The scientific design itself does not change.

## 6. State these limits explicitly in your conclusion

- DSSAT outputs are deterministic given the inputs and parameters supplied.
- Year-to-year variation here reflects the historical weather sequence used, not uncertainty in cultivar parameters or model structure.
- Uniform +2°C / ×0.8 perturbations are sensitivity tests, not bias-corrected climate projections.
- Annual differences may be correlated over time.
- Successive simulations carry soil state from one season to the next, so seasons are not fully independent.
- Results from one site, soil, cultivar, sowing date and management scheme do not generalise to Morocco as a whole.
- This experiment says nothing about uncertainty coming from different climate models, emission pathways, crop models, soils, cultivars or adaptation options.

If the annual differences turn out to be temporally correlated, the independent-year confidence interval used here is the wrong tool — flag it, and consider a block bootstrap or a time-series approach instead.

## 7. Deliverables

Each group submits:

1. the monthly climate comparison figure;
2. the four-scenario yield boxplot;
3. the annual absolute and relative difference figure;
4. the confidence-interval forest plot and test results table;
5. a 200-word maximum write-up that separates effect size, uncertainty, statistical evidence, and model limitations.

## 8. If you finish early

Treat the four scenarios as a 2×2 factorial design. From the four paired annual responses, estimate the average temperature effect, the average rainfall effect, and the temperature × rainfall interaction. This tells you whether the combined impact is additive or not — the natural next question after the paired comparisons.
