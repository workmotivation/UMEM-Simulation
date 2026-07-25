# Known limitations

Read this before drawing any conclusion from a run. Everything below is a
deliberate design boundary or a known weakness, not a bug.

---

## 1. The model is abstract, and its parameters are invented

No constant in this software was estimated from data. The preset values were
chosen by sweeping seeds until each scenario produced a *usable* demographic
trajectory — one that neither collapsed in the first few seasons nor saturated
immediately at the population ceiling.

The consequence is specific: **absolute numbers carry no meaning.** A mean
lifespan of 3.2 seasons is a property of the numbers that were typed in. Only
*contrasts* under controlled comparison — the same scenario, the same seeds,
one parameter changed — support any inference, and even then the inference is
about the model, not about organisms.

## 2. "Motivation" here is a functional construct

Agents carry heritable positive numbers that bias behavioural allocation. Calling
them motivational propensities is a modelling convenience.

They are not genes. Real motivational variation is polygenic, developmentally
constructed, plastic, and environmentally contingent; here it is a small vector
inherited with lognormal noise. The model says nothing about subjective
experience, felt desire, or awareness, and no result should be described in
those terms.

## 3. Cognition is deliberately impoverished

The three cognitive mechanisms are caricatures chosen to be legible:

- **A, intergenerational experience**, is a *population-level* memory, shared by
  every agent. Real social learning is local, networked, and asymmetric.
- **B, forecasting**, is a single exponentially weighted average per
  environmental variable, and is likewise **shared across the population rather
  than held per agent**. This was a tractability decision. It means the model
  cannot represent variation in forecasting skill, and cannot show selection
  acting on forecasting ability.
- **C, individual learning**, is a simple EWMA over an agent's own outcomes with
  no generalisation between tasks.

None of these is a model of cognition. They are switches that let you ask what
happens when a particular information pathway is added.

## 4. Fitness, and what the model refuses to do

Fitness is realised lifetime reproductive success. There is no fitness function,
no utility weighting over energy and health, and no truncation selection.

This is a strength for the question the tool is built for, but it has a cost:
because nothing is optimised, **the model cannot tell you what the optimal
behaviour would have been.** It shows what happened, not what should have
happened. The `one_step_regret_proxy` diagnostic is explicitly labelled a local
single-step measure and must not be read as a fitness gap.

The legacy AOM model in `legacy/` *does* use imposed fitness with truncation
selection. It is retained for continuity with earlier published work, and its
differences are tabulated in `umem_workbench/legacy/aom_adapter.py`.

## 5. Selection, drift, and short runs

Populations are small (hundreds) and runs are short (tens of seasons). Genetic
drift is therefore strong, and over a short horizon it routinely swamps
selection.

Two practical consequences:

- **A single run demonstrates nothing.** Use replications and report intervals.
- Trait change is not evidence of selection. A propensity can rise across thirty
  seasons purely by drift. Comparing against a matched control with the mechanism
  of interest disabled is the minimum standard.

## 6. Specific metric caveats

**Cognitive amplification ratio.** A ratio of nonfunctional behaviour with
cognition to that without. When the baseline is near zero the ratio explodes,
so the additive difference is always reported alongside it and should be
preferred in that regime. The two runs must be a matched pair — same scenario,
same seeds, cognition the only difference — and the software cannot verify that
you did this.

**Experience-causality mismatch.** The "objective contribution" of a task is a
coarse binary: does any effect rule confer a direct benefit? A task with a small
real benefit and one with a large one are treated identically. The metric is
undefined, and reported as such, when intergenerational memory is disabled.

**Opportunity for selection (Crow's I).** An upper bound on the strength of
selection given the observed variance in reproductive success. It is not a
measure of selection acting on any particular trait, and much of that variance
is luck.

**Aggregated replicate curves.** When runs go extinct at different times, later
points in an aggregate curve average over fewer replicates. The replicate count
per step is reported, and the plots mark where dropout begins. Reading the tail
of such a curve without checking that count is a common error.

## 7. Engine behaviours worth knowing

**Disabled tasks retain a heritable propensity.** A task switched off is never
chosen, and its realised frequency, time, and energy shares are exactly zero. Its
inherited propensity column is still recorded and still drifts, because the trait
exists but is not expressed. Exports therefore contain `taskprop_` columns for
disabled tasks. Analysis code aggregates only over enabled tasks, so summary
metrics are unaffected.

**Density regulation has an onset threshold.** Crowding mortality is zero below
`density_onset_fraction` of carrying capacity (0.6 by default) and rises
linearly above it. Scaling it from zero population instead — as an earlier
version did — penalises small populations and creates an extinction ratchet that
prevents demographic recovery.

**Newborns act from the following step.** They are appended at the end of the
breeding step and take their first action in the next one. A closing snapshot is
recorded after the final breeding event so that the last row of the time series
agrees with the reported final population.

**Mate pairing is a simple matching.** The number of pairs is bounded by the
smaller eligible sex. There is no mating market, no repeated pairing within a
season, and no partner fidelity across seasons.

**The environment is generated in advance.** Environmental trajectories do not
respond to the population: there is no resource depletion, and no
predator–prey feedback. Food abundance is exogenous.

## 8. Scale

The engine is vectorised over agents and handles populations in the low
thousands over tens of seasons comfortably. It is not designed for millions of
agents, spatial structure, or networks — there is no spatial dimension at all.
Agents do not interact except through mating, density, and the shared
intergenerational memory.

Recording every step for a large population produces large files. The recording
interval exists for this reason; the workload estimate shown before a run is
approximate.

## 9. Numerical reproducibility

Results are reproducible for a given `(master_seed, replication_index)` pair and
a given NumPy version, across machines and across serial or parallel execution.

They are **not** guaranteed identical across NumPy major versions, since
generator implementations may change. Record the dependency versions in the
export bundle's `metadata.json` alongside the configuration hash.

## 10. What has been verified, and what has not

Verified in this environment: the engine, configuration and validation layers,
experiments including parallel determinism and cancellation, analysis, exports,
the command-line interface, and the Streamlit interface end to end. 145
automated tests.

**Not executed here:** the PySide6 desktop application, the PyInstaller Windows
build, and the GitHub Actions Windows workflow. These were written against their
documented APIs and are syntactically valid and import-safe, but they have not
been run. Treat them as untested until you have built and launched them on a
Windows machine.
