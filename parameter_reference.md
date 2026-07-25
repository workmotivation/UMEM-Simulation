# Parameter reference

Every configuration parameter, generated directly from the model definitions so this document cannot drift from the code.

Scenario files are JSON or YAML. Paths in sweeps and the API use dotted notation, for example `mortality.base_predation_hazard`. List elements may be addressed by identifier rather than index, for example `tasks.safe_forage.energy_cost`.

> All defaults are illustrative demonstration values. None is calibrated to any real species. See [`known_limitations.md`](known_limitations.md).

---

## Top level

| Parameter | Type | Default | Description |
|---|---|---|---|
| `schema_version` | int | `1` | Config schema version. |
| `name` | str | `Untitled` | Display name. |
| `description` | str | `` | Free text. |
| `preset_origin` | str (optional) | required | Preset this derives from, if any. |
| `master_seed` | int | `12345` | Root seed for reproducibility. |

---

## `population`

**Population**

Starting size, carrying capacity, and a hard population ceiling. Carrying capacity regulates crowding; the ceiling is a safety limit on memory use.

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `initial_population` | int | `200` | agents |  | Starting agents. |
| `carrying_capacity` | int | `500` | agents |  | Soft population ceiling. |
| `max_population` | int | `5000` | agents |  | Hard cap for memory safety. |
| `init_distribution` | enum: `equal`, `gamma`, `lognormal`, `from_file` | `gamma` |  |  | Propensity init distribution. |
| `sex_ratio_type_a` | float | `0.5` | frac | 0 to 1 | Fraction born as reproductive type A. |
| `starting_population_file` | str (optional) | required |  |  | Path for from_file init. |

## `lifecycle`

**Life cycle**

Time structure and metabolic accounting: steps per season, total seasons, maturity age, maximum lifespan, and per-step energy costs.

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `steps_per_season` | int | `40` | steps |  | Time steps in a season. |
| `total_seasons` | int | `30` | seasons |  | Seasons to simulate. |
| `actions_per_step` | int | `1` |  |  | Tasks drawn per step. |
| `initial_energy` | float | `70.0` | 0-100 | 0 to 100 | Newborn energy. |
| `energy_max` | float | `100.0` | 0-100 |  | Energy ceiling. |
| `initial_health` | float | `90.0` | 0-100 | 0 to 100 | Newborn health. |
| `health_max` | float | `100.0` | 0-100 |  | Health ceiling. |
| `initial_repro_capital` | float | `0.0` | 0-100 |  | Newborn reproductive capital. |
| `repro_capital_max` | float | `100.0` | 0-100 |  | Ceiling. |
| `initial_status` | float | `50.0` | 0-100 |  | Newborn status. |
| `status_max` | float | `100.0` | 0-100 |  | Ceiling. |
| `basal_metabolism` | float | `1.0` | energy/step | 0 to 5 | Energy lost per step. |
| `rest_energy_cost` | float | `0.5` | energy | 0 to 5 | Energy for fallback rest. |
| `health_starvation_penalty` | float | `10.0` | health/step | 0 to 50 | Health lost per step at zero energy. |
| `maturity_seasons` | int | `2` | seasons |  | Age of reproductive maturity. |
| `max_lifespan_seasons` | int | `12` | seasons |  | Hard lifespan ceiling. |
| `newborns_act_next_step` | bool | `True` |  |  | Newborns act from the next step, not immediately. |

## `mortality`

**Mortality and hazards**

Hazard sources and their strengths. Hazards combine as independent competing risks, and the cause of each death is attributed in proportion to the contributing hazards.

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `base_predation_hazard` | float | `0.02` | prob | 0 to 0.5 | Per-step baseline predation hazard. |
| `predation_env_driver` | str (optional) | `predator` |  |  | Env variable scaling predation. |
| `predation_env_sensitivity` | float | `1.0` |  | 0 to 5 | Scaling of hazard by driver. |
| `age_hazard_shape` | enum: `gompertz`, `logistic` | `gompertz` |  |  | Gompertz or logistic senescence. |
| `age_hazard_scale` | float | `0.002` |  | 0 to 0.1 | Baseline age hazard. |
| `age_hazard_rate` | float | `0.15` |  | 0 to 2 | Growth of hazard with age (per season). |
| `background_hazard` | float | `0.0` | prob |  | Constant per-step mortality. |
| `juvenile_extra_hazard` | float | `0.0` | prob |  | Extra per-step hazard before maturity. |
| `density_hazard_strength` | float | `0.5` |  | 0 to 5 | Extra hazard as pop approaches K. |
| `density_onset_fraction` | float | `0.6` |  | 0 to 2 | Fraction of carrying capacity at which crowding mortality begins. Below this the density hazard is zero. |
| `density_affects_juveniles_only` | bool | `True` |  |  | Apply density hazard only to juveniles. |

## `reproduction`

**Reproduction**

Breeding thresholds, mate selection, fertility distribution, and juvenile survival. Density-dependent recruitment softens juvenile survival as the population approaches carrying capacity.

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `mode` | enum: `sexual`, `asexual` | `sexual` |  |  | Sexual or asexual reproduction. |
| `breeding_every_seasons` | int | `1` | seasons |  | Seasons between breeding events. |
| `energy_threshold` | float | `40.0` | 0-100 | 0 to 100 | Min energy to breed. |
| `health_threshold` | float | `30.0` | 0-100 | 0 to 100 | Min health to breed. |
| `repro_capital_threshold` | float | `0.0` | 0-100 |  | Min reproductive capital to breed. |
| `mate_selection` | enum: `random`, `weighted_capital`, `competitive`, `assortative` | `weighted_capital` |  |  | How mates are chosen. |
| `mate_availability_driver` | str (optional) | `mate` |  |  | Env variable for mate availability. |
| `max_offspring_per_event` | int | `3` |  |  | Cap per breeding event. |
| `fertility_distribution` | enum: `fixed`, `poisson`, `binomial`, `discrete` | `poisson` |  |  | Distribution of surviving offspring. |
| `fertility_mean` | float | `1.5` | offspring | 0 to 8 | Mean surviving offspring per event. |
| `fertility_binomial_p` | float | `0.5` | prob |  | Success prob for binomial fertility. |
| `fertility_discrete_pmf` | list[float] | required |  |  | Probabilities for 0,1,2,... offspring. |
| `repro_energy_cost` | float | `20.0` | energy | 0 to 100 | Energy spent per breeding. |
| `repro_health_cost` | float | `5.0` | health | 0 to 100 | Health spent per breeding. |
| `gestation_delay_steps` | int | `0` | steps |  | Steps before offspring appear. |
| `juvenile_survival` | float | `0.85` | prob | 0 to 1 | Prob a newborn survives to enter pop. |
| `density_dependent_recruitment` | bool | `True` |  |  | Soft Beverton-Holt decline of juvenile survival as population approaches carrying capacity. |
| `recruitment_strength` | float | `1.0` |  | 0 to 5 | Steepness of density-dependent recruitment. |

## `inheritance`

**Inheritance and mutation**

How offspring propensities derive from parents: per-trait parental selection or blending, plus multiplicative lognormal mutation.

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `mutation_probability` | float | `1.0` | prob | 0 to 1 | Prob a trait mutates. |
| `mutation_log_sd` | float | `0.1` |  | 0 to 1 | Log-normal mutation sd (multiplicative). |
| `recombination_per_trait` | bool | `True` |  |  | Pick each trait from one parent vs. blend. |
| `blend_alpha` | float | `0.5` |  | 0 to 1 | Parent-A weight when blending. |
| `min_propensity` | float | `0.01` |  |  | Floor keeping traits positive. |

## `homeostasis`

**Homeostasis (internal state)**

Internal state urgencies (hunger, threat, reproductive readiness) that shift task preference. This is deliberately *not* cognition: it is unlearned physiological regulation and stays active even with all cognitive mechanisms disabled.

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `enabled` | bool | `True` |  |  | Master switch. |
| `hunger_enabled` | bool | `True` |  |  | Energy urgency on. |
| `hunger_strength` | float | `2.0` |  | 0 to 5 | Logit boost to food tasks when hungry. |
| `safety_enabled` | bool | `True` |  |  | Threat urgency on. |
| `safety_strength` | float | `2.0` |  | 0 to 5 | Logit boost to safety tasks under threat. |
| `reproduction_enabled` | bool | `True` |  |  | Reproductive urgency on. |
| `reproduction_strength` | float | `2.0` |  | 0 to 5 | Logit boost to reproduction when ready. |

## `cognition`

**Cognition (A / B / C)**

Three independent cognitive mechanisms can be enabled, alone or together.

**A - Intergenerational experience.** A population-level memory of which tasks
were associated with successful agents in recent cohorts, which then biases
choice. The association is *correlational*: a task can become well represented
simply because successful agents had the time to do it. This is the primary
route to cognitive misallocation in the model.

**B - Noisy environmental forecasting.** An exponentially weighted forecast of
each environmental variable, which shifts preference toward tasks whose success
depends on that variable. The forecast is built only from past noisy
observations; there is no look-ahead to the true future state.

**C - Individual learning (advanced).** Per-agent reinforcement from an agent's
own recent outcomes. Off by default, because it introduces within-lifetime
adaptation that can mask the evolutionary signal the model is designed to show.

With all three disabled the model is a pure evolutionary baseline: behaviour is
driven only by inherited propensity and homeostatic state.

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `experience_enabled` | bool | `False` |  |  | Intergenerational memory on. |
| `experience_success_definition` | enum: `survived`, `reproduced`, `surviving_offspring`, `top_quantile_lrs` | `surviving_offspring` |  |  | Who counts as successful. |
| `experience_top_quantile` | float | `0.2` |  | 0 to 1 | Quantile for top-LRS definition. |
| `experience_weight_recent` | float | `0.7` |  | 0 to 1 | Weight on the most recent cohort. |
| `experience_smoothing` | float | `1.0` |  |  | Additive smoothing on counts. |
| `experience_noise_sd` | float | `0.3` |  | 0 to 3 | Noise on the memory signal. |
| `experience_strength` | float | `2.5` |  | 0 to 8 | Logit multiplier from memory. |
| `forecast_enabled` | bool | `False` |  |  | Noisy env forecasting on. |
| `forecast_memory` | float | `0.6` |  | 0 to 1 | EWMA lambda on past forecast. |
| `forecast_strength` | float | `1.0` |  | 0 to 5 | Logit shift from forecast surprise. |
| `individual_learning_enabled` | bool | `False` |  |  | Per-agent EWMA reinforcement (advanced). |
| `individual_learning_rate` | float | `0.2` |  | 0 to 1 | EWMA rate on own outcomes. |
| `individual_learning_strength` | float | `1.0` |  | 0 to 5 | Logit shift from own experience. |

## `choice`

**Decision policy**

The decision policy that converts propensities, urgencies, and cognitive signals into task probabilities. Contributions are additive in log-odds and passed through a softmax.

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `softmax_temperature` | float | `1.0` |  | 0.1 to 5 | Higher = more random choice. |
| `decision_noise_sd` | float | `0.1` |  | 0 to 2 | Gumbel-like logit noise sd. |

## `experiment`

**Experiment**

Single run, replications, or a parameter sweep of up to three parameters.

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `mode` | enum: `single`, `replications`, `sweep` | `single` |  |  | single/replications/sweep. |
| `replications` | int | `1` | runs |  | Independent repeats. |
| `max_workers` | int | `1` |  |  | Parallel processes for replicates. |
| `sweep_parameters` | list[SweepParameter] | required |  |  | 1-3 parameters to sweep. |
| `max_sweep_combinations` | int | `500` |  |  | Warn/refuse above this. |
| `common_random_numbers` | bool | `False` |  |  | Reuse seeds across sweep conditions. |

## `output`

**Output and recording**

Recording interval and optional individual-level sampling. Longer intervals reduce memory use at the cost of temporal resolution.

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `record_interval_steps` | int | `10` | steps |  | Steps between snapshots. |
| `store_agent_sample` | bool | `True` |  |  | Store a reproducible agent sample. |
| `agent_sample_size` | int | `50` |  |  | Agents per snapshot sample. |
| `store_event_log` | bool | `False` |  |  | Store detailed events (large!). |

---

## List-valued sections

## `domains[]`

**DomainConfig**

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `domain_id` | str | required |  |  | Immutable key. |
| `name` | str | required |  |  | Display name. |
| `init_mean` | float | `1.0` |  | 0.1 to 5 | Mean initial propensity. |
| `init_var` | float | `0.5` |  | 0 to 5 | Initial variance. |

## `tasks[]`

**TaskConfig**

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `task_id` | str | required |  |  | Immutable unique key. |
| `name` | str | required |  |  | Display name. |
| `description` | str | `` |  |  | Free text. |
| `domain` | str | required |  |  | Behavioural domain id. |
| `enabled` | bool | `True` |  |  | Include in runs. |
| `time_cost` | float | `1.0` | steps | 0 to 5 | Time steps consumed. |
| `energy_cost` | float | `1.0` | energy | 0 to 20 | Direct energy spent. |
| `base_success` | float | `0.2` | prob | 0 to 1 | Baseline success probability. |
| `success_env_driver` | str (optional) | required |  |  | Env variable modulating success. |
| `success_env_sensitivity` | float | `0.0` |  | -8 to 8 | Logit shift per unit env driver. |
| `outcome_noise` | float | `0.3` |  | 0 to 3 | Logit noise sd on success. |
| `predation_multiplier` | float | `1.0` |  | 0 to 3 | Per-step hazard multiplier. |
| `min_age_steps` | int | `0` |  |  | Age gate. |
| `max_age_steps` | int (optional) | required |  |  | Upper age gate (optional). |
| `require_mature` | bool | `False` |  |  | Maturity gate. |
| `sex_restriction` | enum: `any`, `type_a`, `type_b` | `any` |  |  | Reproductive-type gate. |
| `observable_to_actor` | bool | `True` |  |  | Actor sees its own outcome. |
| `public_signal` | bool | `False` |  |  | Non-actors see a weak public signal. |
| `role` | enum: `functional`, `low_cost_neutral`, `costly_nonfunctional` | `functional` |  |  | functional / neutral / costly nonfunctional. |
| `memory_eligible` | bool | `True` |  |  | May enter intergenerational memory. |
| `effects` | list[TaskEffectRule] | required |  |  | Multidimensional effects. |

## `tasks[].effects[]`

**TaskEffectRule**

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `target` | enum: `energy`, `health`, `repro_capital`, `status`, `hazard`, `observation_quality` | required |  |  | State variable this rule changes. |
| `effect_type` | enum: `additive`, `multiplicative`, `probability_modifier`, `bounded_replacement` | `additive` |  |  | How the value combines with state. |
| `base_value` | float | `0.0` | state units | -100 to 100 | Magnitude before env scaling. |
| `condition` | enum: `always`, `on_success`, `on_failure` | `on_success` |  |  | Fires on success, failure, or always. |
| `env_driver` | str (optional) | required |  |  | Env variable id that scales this effect. |
| `env_sensitivity` | float | `0.0` |  | -5 to 5 | Coupling of the effect to the env driver. |
| `noise_std` | float | `0.0` | state units | 0 to 10 | Gaussian noise on the effect. |
| `min_age_steps` | int | `0` |  |  | Rule inactive below this age. |
| `require_mature` | bool | `False` |  |  | Rule only for mature agents. |
| `sex_restriction` | enum: `any`, `type_a`, `type_b` | `any` |  |  | Reproductive type this rule needs. |

## `environment[]`

**EnvVariableConfig**

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `var_id` | str | required |  |  | Immutable unique key. |
| `name` | str | required |  |  | Display name. |
| `description` | str | `` |  |  | Free text. |
| `process` | enum: `constant`, `random_walk`, `ar1`, `seasonal`, `ar1_seasonal`, `regime_switching`, `rare_shocks`, `imported` | `ar1` |  |  | Stochastic process type. |
| `initial_value` | float | `0.5` | norm 0-1 | 0 to 1 | Value at t=0. |
| `mean` | float | `0.5` | norm 0-1 | 0 to 1 | Reversion target. |
| `min_value` | float | `0.0` | norm | 0 to 1 | Lower bound. |
| `max_value` | float | `1.0` | norm | 0 to 1 | Upper bound. |
| `persistence` | float | `0.9` |  | 0 to 1 | AR(1) autocorrelation. |
| `volatility` | float | `0.05` |  | 0 to 1 | Innovation sd. |
| `seasonal_amplitude` | float | `0.0` |  | 0 to 1 | Seasonal amplitude. |
| `seasonal_period_steps` | int | `40` | steps |  | Steps per cycle. |
| `shock_probability` | float | `0.0` | prob | 0 to 1 | Per-step shock probability. |
| `shock_magnitude` | float | `0.0` |  | -1 to 1 | Additive shock magnitude. |
| `shock_duration_steps` | int | `1` | steps |  | Steps a shock persists. |
| `regime_values` | list[float] | required |  |  | Values for regime switching. |
| `regime_switch_prob` | float | `0.02` | prob |  | Per-step switch probability. |
| `imported_series` | list[float] | required |  |  | User time series (process=imported). |
| `observation_noise` | float | `0.05` |  | 0 to 1 | Noise sd on agent observations. |
| `globally_observable` | bool | `True` |  |  | Directly observed by agents. |

## `experiment.sweep_parameters[]`

**SweepParameter**

| Parameter | Type | Default | Unit | Safe range | Description |
|---|---|---|---|---|---|
| `path` | str | required |  |  | Dotted config path, e.g. mortality.base_predation_hazard. |
| `values` | list[float] | required |  |  | Explicit values. |
| `start` | float (optional) | required |  |  | Range start. |
| `stop` | float (optional) | required |  |  | Range stop. |
| `step` | float (optional) | required |  |  | Range step. |
| `logarithmic` | bool | `False` |  |  | Log-spaced range. |

