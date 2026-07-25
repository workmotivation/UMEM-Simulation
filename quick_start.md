# Quick start

## Install

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -e ".[streamlit]"
```

## Your first run, in the web interface

```bash
streamlit run streamlit_app.py
```

1. **Simple mode.** Leave the preset as *Stable abundant*. Set **Number of
   seasons** to 10 so the first run is quick. Press **Apply these settings**.
2. **Run.** Press **Run experiment** and watch the progress bar.
3. **Results.** Population, survival, reproduction, behaviour, environment,
   misallocation, and diversity each have a tab.
4. **Exports.** Press **Build export bundle**, then download the ZIP.

That bundle contains everything needed to reproduce what you just saw.

## Your first run, in Python

```python
from umem_workbench.config.presets import load_preset
from umem_workbench.experiments import run_single
from umem_workbench.analysis import summarise_run

scenario = load_preset("stable_abundant")
scenario.lifecycle.total_seasons = 20

result = run_single(scenario, replication_index=0)
summary = summarise_run(result)

print(f"final population : {summary['final_population']}")
print(f"mean lifespan    : {summary['mean_lifespan_seasons']:.2f} seasons")
print(f"mean LRS         : {summary['mean_lrs_completed_lives']:.3f}")
```

## A controlled comparison

A single run shows nothing. The useful unit is a matched pair of replicated
experiments differing in exactly one thing.

Here is the question the tool is built for: **does cognition increase costly
pointless behaviour?**

```python
from umem_workbench.config.presets import load_preset
from umem_workbench.experiments import run_replications
from umem_workbench.analysis import (aggregate_summaries, summary_table,
                                     cognitive_amplification_ratio)

def build(with_cognition: bool):
    scenario = load_preset("stable_abundant")
    scenario.lifecycle.total_seasons = 30
    scenario.cognition.experience_enabled = with_cognition
    return scenario

baseline = run_replications(build(False), 20, parallel=True, max_workers=4)
cognitive = run_replications(build(True), 20, parallel=True, max_workers=4)

metric = "final_nonfunctional_action_rate"
for label, batch in (("no cognition", baseline), ("cognition A", cognitive)):
    stats = aggregate_summaries(summary_table(batch.runs))
    row = stats[stats.metric == metric].iloc[0]
    print(f"{label:14} {row['mean']:.4f}  "
          f"[{row['ci_lower']:.4f}, {row['ci_upper']:.4f}]")

print(cognitive_amplification_ratio(cognitive.runs[0], baseline.runs[0]))
```

Both arms share the master seed, so replication *i* faces the same
environmental sequence in each. The remaining difference is the mechanism.

Note the amplification ratio also reports an additive difference: when the
baseline rate is near zero the ratio is unstable and the difference is the
sounder summary.

## A parameter sweep

```python
from umem_workbench.config.models import SweepParameter
from umem_workbench.experiments import run_sweep

scenario = load_preset("stable_abundant")
scenario.lifecycle.total_seasons = 20

sweep = run_sweep(
    scenario,
    [SweepParameter(path="mortality.base_predation_hazard",
                    values=[0.0005, 0.001, 0.002, 0.004]),
     SweepParameter(path="reproduction.fertility_mean",
                    values=[1.0, 1.5, 2.0])],
    replications_per_cell=5,
    parallel=True, max_workers=4)

print(sweep.response_curve("final_population"))
```

Parameters are addressed by dotted path. List elements may be addressed by
identifier rather than index — `tasks.safe_forage.energy_cost` — so inserting a
task does not silently change what a saved sweep refers to.

## Command line

```bash
umem-workbench presets
umem-workbench run --preset scarce_low_fecundity --replications 10 --output ./results
umem-workbench validate presets/stable_abundant.json
umem-workbench benchmark
```

## Where to go next

- [`model_reference.md`](model_reference.md) — what the model actually computes
- [`parameter_reference.md`](parameter_reference.md) — every parameter
- [`reproducibility.md`](reproducibility.md) — seeding and hashing
- [`known_limitations.md`](known_limitations.md) — **read before interpreting**
