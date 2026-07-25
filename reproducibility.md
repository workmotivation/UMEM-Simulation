# Reproducibility

Every result this software produces is reproducible from two numbers and a
configuration file.

## The three identifiers

| Identifier | What it fixes |
|---|---|
| **Configuration hash** | Every scientifically relevant setting |
| **Master seed** | The root of all randomness |
| **Replication index** | Which independent substream is used |

Given all three, any machine running the same NumPy version reproduces the same
trajectory exactly.

## How seeding works

All randomness derives from a NumPy `SeedSequence`:

```
SeedSequence([master_seed, replication_index])
        |
        +-- spawn one named child stream per subsystem
              environment, initial_population, task_choice, task_outcome,
              mortality, mate_choice, fertility, inheritance, observation,
              cognition
```

Named substreams matter. If every subsystem drew from one generator, adding a
single extra random call anywhere — a new hazard, an extra diagnostic — would
shift every subsequent draw and silently change results that ought to be
unrelated. Separate streams confine that risk to the subsystem that changed.

Consequences, each covered by a test:

- **Same seed, same replication index, same trajectory.** Always.
- **Replications are independent**, not offset. Replication 3 is the same run
  whether you request 4 replications or 40.
- **Serial and parallel execution are bit-identical.** Worker count never
  affects results. Results are re-sorted by replication index after collection,
  because completion order is not deterministic but output must be.
- **Previewing the environment does not perturb the run.** The preview draws
  from a separate high branch of the seed sequence.

## The configuration hash

A SHA-256 over the canonical serialisation of the scenario, deliberately
**excluding** `name`, `description`, and `preset_origin`.

Renaming a scenario does not change its hash. Changing any parameter does. Two
scenarios with the same hash are the same model, whatever they are called.

```python
from umem_workbench.config.serialization import config_hash, load_scenario_json

scenario = load_scenario_json("scenario.json")
assert config_hash(scenario) == "the hash from your bundle"
```

The interfaces label a scenario **Custom** as soon as its hash stops matching
the preset it came from, so a modified preset is never presented as the
original.

## Reproducing a run from a bundle

Every export bundle contains `scenario.json`, `metadata.json`, and
`REPRODUCIBILITY.txt`. To reproduce:

```python
from umem_workbench.config.serialization import load_scenario_json
from umem_workbench.experiments import run_single

scenario = load_scenario_json("scenario.json")
result = run_single(scenario, replication_index=0)   # index from metadata.json
```

Or from the command line:

```bash
umem-workbench run --scenario scenario.json --seed 12345
```

## What is *not* guaranteed

- **Across NumPy major versions.** Generator implementations may change. Record
  the dependency versions from `metadata.json`.
- **Floating-point summaries across architectures.** Reduction order can differ
  slightly on different hardware. Population counts and event sequences are
  integer and stable; means and variances may differ in the last bits.
- **Across schema versions**, unless migrated. `config/migrations.py` upgrades
  older scenario documents; the migration is idempotent and tested.

## Reporting results

Report at minimum: the software version, the configuration hash, the master
seed, the number of replications, and the NumPy version. Attach the export
bundle where possible — it contains all of these plus the scenario itself.

Report uncertainty. A single trajectory is one draw from a stochastic process
and can show almost any pattern by chance. Use replications, and quote the
intervals the analysis layer produces.
