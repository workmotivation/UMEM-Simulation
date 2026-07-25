# Contributing

Thank you for considering a contribution. This is a scientific tool, so the
review emphasis is on correctness and on not overstating what the model can
show.

## Getting set up

```bash
git clone https://github.com/ORGANISATION_HERE/umem-evolutionary-workbench.git
cd umem-evolutionary-workbench
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
pytest
```

## Ground rules

**Every scientific mechanism needs a test that isolates it.** The suite in
`tests/unit/test_mechanisms.py` is the model: one mechanism, one test, in a
parameter regime where the expected effect is unambiguous. A mechanism without
such a test can silently stop working — this has already happened once in this
codebase, when a task's predation multiplier was computed but never applied, and
only a matched-comparison test revealed it.

**Write tests that can actually fail.** A metric that saturates (every death
attributed to one cause in both arms) cannot distinguish anything. Choose a
regime where the contrast is visible, and prefer absolute counts over shares
when shares can saturate.

**Never impose a fitness function.** Fitness in this model is realised lifetime
reproductive success. Adding a survival score, a utility weighting over energy
and health, or truncation selection to the main engine would defeat its purpose.
The legacy AOM model does use imposed fitness, and stays isolated in `legacy/`.

**No user code execution.** Task effects are declarative rules. Do not add
`eval`, `exec`, expression parsing, or plugin loading of arbitrary Python.

**Keep uncertainty visible.** Do not remove confidence intervals, extinction
rates, validation warnings, or documented limitations to make output tidier.

**Reproducibility is not negotiable.** All randomness must derive from the
`RandomStreams` seeded by `(master_seed, replication_index)`. Never call the
global NumPy random functions. Parallel and serial execution must remain
bit-identical; `tests/unit/test_reproducibility.py` enforces this.

## Style

- Follow the surrounding code. Run `ruff check src tests`.
- Use full words in identifiers. `predation_multiplier`, not `pred_mult`.
- Document *why*, not *what*. A comment explaining why density hazard has an
  onset threshold is worth more than one restating the formula.
- Public functions get docstrings including units and the meaning of the return
  value.
- New configuration fields must carry `ui()` metadata (label, help, unit, safe
  range), because both interfaces build their controls from it.

## Adding a new task effect target

1. Add the enum member in `config/enums.py`.
2. Add the target code and its handling in `engine/compiled.py` and
   `tasks/effects.py`.
3. Ensure the state variable is clipped to its configured bounds.
4. Extend `TaskConfig.has_direct_benefit()` if the new target confers a benefit,
   otherwise the role-consistency validator and the misallocation metrics will
   misclassify tasks that use it.
5. Add a mechanism test.

## Pull requests

- One logical change per pull request.
- State what you changed, why, and how you verified it.
- Include the test output.
- If behaviour of the shipped presets changes, say so explicitly and re-verify
  their calibration; the extinction table in `config/presets.py` and in the
  README must be updated to match.

## Reporting bugs

Please include the scenario JSON (or preset name), the configuration hash, the
master seed and replication index, the observed behaviour, and the expected
behaviour. With the hash and seed a maintainer can reproduce your run exactly.
