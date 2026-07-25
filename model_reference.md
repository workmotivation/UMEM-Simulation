# Model reference

A precise description of what the simulation computes. Notation: `t` is a time
step, `s` a season. A season contains `steps_per_season` steps; a run contains
`total_seasons` seasons.

---

## 1. State

Each agent carries:

| Variable | Range | Meaning |
|---|---|---|
| `energy` | 0–100 | Depleted by metabolism and action costs, restored by food tasks. Reaching zero causes health loss. |
| `health` | 0–100 | Reduced by starvation and injury. Reaching zero is fatal. |
| `repro_capital` | 0–100 | Accumulated readiness to breed. |
| `status` | 0–100 | Social standing; influences mate choice under some modes. |
| `age_steps` | ≥ 0 | Age; maturity and senescence derive from it. |

Heritable traits are strictly positive and hierarchical:

- one **domain propensity** per motivational domain (food, safety, reproduction,
  neutral);
- one **task propensity** per task within its domain.

The effective weight of a task is the product of the two, entering the decision
in log-odds.

---

## 2. Time step

The order below is fixed and matches the specification.

1. **Environment updates.** Trajectories are generated in advance from the
   environment stream, so the environment is exogenous: it does not respond to
   the population.
2. **Noisy observation.** Each agent perceives `env[t]` plus Gaussian noise
   scaled by `observation_noise`. There is no look-ahead: `env[t+1]` is never
   visible to any decision at `t`.
3. **Cognitive updates.** Forecast (B) advances on the current observation;
   experience memory (A) refreshes at season boundaries.
4. **Homeostatic urgencies.**
   - hunger `= 1 − energy / energy_max`
   - threat `=` blend of low health and perceived predator level
   - reproductive readiness `=` maturity gate × energy and health gates
5. **Eligibility.** Tasks are filtered by minimum age, maturity requirement, and
   sex restriction.
6. **Logits.** For each eligible task:

   ```
   logit = log(domain_propensity)
         + log(task_propensity)
         + homeostatic_urgency  × channel_weight
         + experience_modifier                    (cognition A)
         + forecast_modifier                      (cognition B)
         + individual_preference                  (cognition C)
         + decision_noise
   ```

   All contributions are additive in log-odds. Ineligible tasks receive `−inf`.
7. **Choice.** A softmax over the logits with `softmax_temperature`, sampled by
   inverse CDF. `actions_per_step` draws are made.
8. **Costs.** The chosen task's `energy_cost`; agents with no eligible task pay
   `rest_energy_cost`. Basal metabolism is charged once per step.
9. **Success.** `P(success) = sigmoid(base_logit + env_sensitivity × driver +
   noise)`.
10. **Effects.** Each effect rule applies to its target
    (energy, health, reproductive capital, status, observation quality, hazard)
    with a type (additive, multiplicative, probabilistic, bounded replacement)
    and a condition (on success, on failure, always). Rules are **declarative
    data**; no user expression is ever evaluated.
11. **Hazards.** Combined as independent competing risks:

    ```
    P(death) = 1 − Π (1 − hazard_i)
    ```

    Sources: predation, senescence, background, juvenile, crowding, plus the
    deterministic state deaths (starvation, health failure, maximum lifespan).

    Predation is scaled by the environment, by any hazard effects, **and by the
    chosen task's `predation_multiplier`** — sheltering is safer than open
    foraging. Exposure applies whether or not the task succeeded.
12. **Death and attribution.** Deterministic causes are resolved first
    (starvation if energy is also zero, otherwise health failure; then maximum
    lifespan). For probabilistic death, the cause is sampled in proportion to
    the contributing hazards, so every death carries an attributed cause.
13. **Ageing and recording.** Age advances, maturity is updated, and a snapshot
    is written at the configured interval.

---

## 3. Season boundary

1. **Eligibility.** Mature agents meeting the energy, health, and reproductive
   capital thresholds, and past their breeding interval.
2. **Pairing.** One of four modes: random, weighted by capital, competitive
   (assortative on quality), or assortative on domain propensity. The number of
   pairs is bounded by the smaller eligible sex.
3. **Fertility.** Drawn from a fixed, Poisson, binomial, or discrete
   distribution, capped by `max_offspring_per_event`.
4. **Inheritance.** Per trait, either a parent is selected or the parental values
   are blended, then multiplicative lognormal mutation is applied with
   probability `mutation_probability` and magnitude `mutation_log_sd`. Values are
   clipped at `min_propensity` so propensities stay strictly positive.
5. **Recruitment.** Juveniles survive with probability `juvenile_survival`,
   optionally reduced by Beverton–Holt density dependence:

   ```
   effective_survival = juvenile_survival / (1 + recruitment_strength × N / K)
   ```

6. **Memory update (A).** The success distribution of the successful cohort is
   folded into the two-cohort memory. "Successful" is configurable: survived,
   reproduced, left surviving offspring, or top quantile by lifetime
   reproductive success.
7. **Birth.** Offspring are appended and act from the **next** step. With a
   gestation delay they enter a pending queue and appear later.

A closing snapshot is recorded after the final breeding event, so the last row
of the time series agrees with the reported final population.

---

## 4. Fitness

There is no fitness function. Fitness is **realised lifetime reproductive
success**: the number of surviving offspring an agent actually produced,
recorded when it dies.

Selection is whatever emerges from differential birth and death. Nothing scores
agents, ranks them, or optimises them. This is the model's central design
commitment, and the reason it can show misallocation arising rather than
assuming it away.

---

## 5. Cognition

**A — Intergenerational experience.** A population-level memory over
memory-eligible tasks, built from the success distribution of the two most
recent successful cohorts, smoothed, noised, and applied as an additive logit
shift. The association it encodes is *correlational*: a task can become well
represented simply because successful agents had time to perform it. This is the
main route to cognitive misallocation.

**B — Forecasting.** An exponentially weighted moving average per environmental
variable, built only from past noisy observations:

```
forecast[t] = λ · forecast[t−1] + (1 − λ) · observation[t]
```

The shift applied to a task is proportional to that task's environmental
sensitivity times the forecast's departure from the long-run mean. The forecast
is **shared across the population**, not held per agent — a documented
simplification.

**C — Individual learning.** Per-agent EWMA over the agent's own outcomes
(+0.5 on success, −0.5 on failure). Off by default.

**Homeostasis is not cognition.** It is unlearned physiological regulation and
stays active with all three mechanisms disabled.

---

## 6. Behavioural categories

| Role | Direct benefit | Cost |
|---|---|---|
| Functional | Yes | Any |
| Low-cost neutral | No | Minimal — persists by drift |
| Costly nonfunctional | No | Meaningful time, energy, or hazard cost |

The role is what the user asserted; the effect rules are what the model does.
The validator warns whenever the two disagree, because a "neutral" task that
grants energy would silently corrupt every misallocation metric.

---

## 7. Misallocation metrics

1. Nonfunctional action rate
2. Nonfunctional time allocation
3. Nonfunctional energy allocation
4. Cognitive amplification ratio — nonfunctional behaviour with cognition over a
   matched no-cognition baseline, reported with its raw components and an
   additive difference because the ratio is unstable near a zero denominator
5. Forecast error — forecast against truth, distinct from observation error
6. Experience–causality mismatch — a task's share of success memory minus its
   configured objective contribution

Several transparent measures, deliberately never combined into one index.

---

## 8. Environment

Eight process types are available, including AR(1), random walk, cyclical, and
rare-shock regimes. Each variable has a mean, persistence, volatility, optional
shock probability and magnitude, and an observation noise level.

Variables drive task success, predation pressure, and mate availability. They are
generated in advance and are not affected by the population: there is no resource
depletion and no predator–prey feedback.
