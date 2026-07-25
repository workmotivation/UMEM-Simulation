# UMEM Evolutionary Simulation Workbench

An open-source research workbench for studying how heritable, motivation-like
**behavioural allocation propensities** might evolve in abstract biological
agents that have finite lives, internal physiological state, uncertain
environments, survival hazards, reproduction, and optional limited cognition.

> **This is an abstract theoretical model.** Every default value is an
> illustrative demonstration setting, not a parameter estimate for any real
> species. The heritable "motivational propensity" is a functional modelling
> construct and does not correspond to any identified genetic locus. The model
> makes no claim about consciousness or subjective experience.

---

## What question is this built for?

Under what ecological conditions does behaviour that has **no modelled benefit**
nevertheless persist — and when does adding cognition make that *worse* rather
than better?

To make that question tractable the workbench:

- tracks low-cost neutral and costly nonfunctional behaviour separately and
  explicitly;
- reports several transparent misallocation diagnostics instead of one opaque
  index;
- never imposes a fitness function. Fitness is **realised lifetime reproductive
  success**: an outcome of birth and death in the simulated ecology, not a score
  agents are optimised against.

That last point is the central design commitment. A model that scores agents
against a fitness function has already decided what counts as success, and so
cannot show misallocation *emerging*. The legacy AOM model bundled in
`legacy/` does use imposed fitness with truncation selection, and is retained
purely for continuity and comparison.

---

## Installation

Requires Python 3.10 or newer.

```bash
git clone https://github.com/ORGANISATION_HERE/umem-evolutionary-workbench.git
cd umem-evolutionary-workbench

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

pip install -e ".[streamlit]"      # engine + web interface
pip install -e ".[all]"            # everything, including the desktop app
pip install -e ".[dev]"            # everything plus test and build tooling
```

The engine itself depends only on NumPy, pandas, SciPy, matplotlib, Pydantic,
PyYAML, and openpyxl. Streamlit and PySide6 are optional extras: the package
imports and runs headless without either.

---

## Quick start

### Web interface

```bash
streamlit run streamlit_app.py
```

Choose a preset in **Simple mode**, press **Apply settings**, then go to
**Run**. Results appear under **Results**, and **Exports** builds a complete
reproducibility bundle.

### Desktop application

```bash
python desktop_app.py            # requires: pip install -e ".[desktop]"
```

### Command line

```bash
umem-workbench presets
umem-workbench run --preset stable_abundant --replications 10 --output ./results
umem-workbench validate presets/scarce_low_fecundity.json
umem-workbench benchmark
```

### Python API

```python
from umem_workbench.config.presets import load_preset
from umem_workbench.experiments import run_single, run_replications
from umem_workbench.analysis import summarise_run

scenario = load_preset("stable_abundant")
scenario.lifecycle.total_seasons = 30

result = run_single(scenario, replication_index=0)
print(summarise_run(result)["final_population"])

batch = run_replications(scenario, n_replications=20, parallel=True, max_workers=4)
print(batch.extinction())
print(batch.report_text())
```

---

## The model in one page

Time is nested: **time steps** inside **seasons** inside **lifetimes**.

At each time step every living agent:

1. observes the environment **noisily**, with no look-ahead;
2. computes homeostatic urgencies (hunger, threat, reproductive readiness);
3. filters the task library down to what it is eligible to do;
4. converts inherited propensities, urgencies, and any enabled cognition into
   task probabilities — additively in log-odds, through a softmax;
5. draws and performs a task, paying its time and energy cost;
6. succeeds or fails stochastically, depending on the environment;
7. receives the task's configured effects on energy, health, reproductive
   capital, status, observation quality, and hazard exposure;
8. faces predation, injury, background, senescence, crowding, and juvenile
   hazards, which combine as independent competing risks.

At season boundaries, agents meeting the reproductive thresholds pair and breed.
Offspring inherit recombined, mutated propensities and enter the population
subject to juvenile survival and density-dependent recruitment.

Propensities are **hierarchical**: one per motivational domain (food, safety,
reproduction, neutral), plus one per task within a domain.

Three cognitive mechanisms can be enabled independently:

| | Mechanism | Note |
|---|---|---|
| **A** | Intergenerational experience | Population memory of which tasks were *associated* with successful agents. The association is correlational, which is the main route to misallocation. |
| **B** | Noisy environmental forecasting | EWMA forecast built only from past noisy observations. |
| **C** | Individual learning | Per-agent reinforcement from own outcomes. Off by default. |

Homeostasis is deliberately **not** cognition: it is unlearned physiological
regulation and remains active with every cognitive module disabled.

---

## Built-in presets

| Preset | Ecology | Behaviour at shipped values (12 seeds) |
|---|---|---|
| `stable_abundant` | Abundant, stable | 0/12 extinct, final ~230–390 (K=500) |
| `scarce_low_fecundity` | Scarce, low fecundity | 0/12 extinct, final ~230–380, dips to ~20 |
| `volatile_high_fecundity` | Volatile boom-bust | 0/12 extinct, final ~520–680 (K=700) |
| `legacy_aom_reference` | AOM 2.1 continuity | 0/12 extinct, final ~290–380 (K=400) |

These constants were chosen by seed sweeps until each preset produced a *usable*
demographic trajectory — neither immediate extinction nor instant saturation.
They are numerical demonstration settings and carry no biological
interpretation. The qualitative contrasts between presets are the intended
object of study; the absolute numbers are arbitrary.

---

## Reproducibility

Every random stream derives from a NumPy `SeedSequence` spawned from
`(master_seed, replication_index)`. Consequences:

- the same pair always reproduces the same trajectory;
- replications are statistically independent, not merely offset;
- **serial and parallel execution produce identical results** — worker count
  never affects output;
- the environment preview draws from a separate branch, so previewing never
  perturbs a subsequent run.

Each scenario has a **configuration hash** covering every scientifically
relevant setting and deliberately excluding cosmetic fields, so renaming a
scenario does not change its identity. Matching hashes mean the same model.

See [`docs/reproducibility.md`](docs/reproducibility.md).

---

## Exports

**Export All** produces a ZIP bundle containing the scenario JSON, run metadata,
a formatted Excel workbook, CSV tables, figures (PNG/SVG/PDF at up to 300 DPI),
a warning log, and short reproducibility instructions. Tables that would exceed
Excel's row limit are written as companion CSV files and referenced from the
workbook rather than silently truncated.

---

## Testing

```bash
pip install -e ".[test]"
pytest                    # full suite
pytest -m "not ui"        # skip the slower Streamlit tests
```

145 tests cover configuration round-tripping and validation, reproducibility and
parallel determinism, the scientific mechanisms individually, births/deaths
reconciliation, exports, and an end-to-end walk through the web interface.

---

## Documentation

| Document | Contents |
|---|---|
| [`docs/quick_start.md`](docs/quick_start.md) | First run, walkthrough |
| [`docs/model_reference.md`](docs/model_reference.md) | Full model specification |
| [`docs/parameter_reference.md`](docs/parameter_reference.md) | Every parameter |
| [`docs/reproducibility.md`](docs/reproducibility.md) | Seeding and hashing |
| [`docs/known_limitations.md`](docs/known_limitations.md) | What this model cannot do |
| [`docs/packaging.md`](docs/packaging.md) | Building a Windows executable |

**Please read [`docs/known_limitations.md`](docs/known_limitations.md) before
drawing any conclusion from a run.**

---

## Project layout

```
src/umem_workbench/
    config/         Pydantic models, validation, presets, serialisation
    engine/         Simulation core: state, choice, mortality, reproduction
    environment/    Stochastic environmental processes and noisy observation
    tasks/          Task library and the effect-rule system
    cognition/      Mechanisms A, B, and C
    experiments/    Single runs, replications, sweeps, cancellation
    analysis/       Metrics, misallocation diagnostics, aggregation, plots
    export/         Excel, CSV, figures, ZIP bundles
    ui_common/      Toolkit-independent state and labels
    ui_streamlit/   Web interface
    ui_desktop/     PySide6 desktop interface
    legacy/         AOM 2.1 reference model
```

There is no user-supplied code execution anywhere: task effects are declarative
rules, never evaluated expressions.

---

## Citing

See [`CITATION.cff`](CITATION.cff).

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md). Contributions that add scientific
mechanisms should come with a test that isolates the mechanism.

## Licence

MIT — see [`LICENSE`](LICENSE).
