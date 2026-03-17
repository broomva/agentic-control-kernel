# EGRI for Controllers

Apply Autoany's Evaluator-Governed Recursive Improvement to controller tuning,
world-model learning, and safety parameter optimization.

## Core Mapping

| EGRI concept | Controller domain |
|---|---|
| **Artifact** | Controller parameters, cost weights, model structure, safety thresholds, policy code |
| **Harness** | Simulator/digital twin + scenario library + regression suite |
| **Evaluator** | Cost + constraint violations + robustness + latency |
| **Mutation surface** | MPC weights, CBF margins, Koopman lifts, RL hyperparams |
| **Promotion** | Deploy controller version if it improves metrics and passes constraints |
| **Ledger** | Trace of all trials with full state/action/score records |

## Problem-Spec Template for Controller Optimization

Use `assets/templates/problem-spec.control.yaml` as starting point:

```yaml
name: "controller-optimization"
objective:
  metric: "closed_loop_cost"
  direction: minimize
  baseline: null  # Filled after first eval

constraints:
  - "constraint_violation_count == 0"
  - "shield_intervention_rate <= 0.05"
  - "solve_time_ms <= 100"

artifacts:
  mutable:
    - path: "control/mpc/weights.yaml"
      type: config
      description: "MPC cost weights and horizon"
    - path: "control/shield/cbf_params.yaml"
      type: config
      description: "CBF barrier parameters and margins"
  immutable:
    - path: "evals/scenario_library/"
      reason: "Evaluator scenarios — must not change during trials"
    - path: "evals/run_eval.sh"
      reason: "Evaluator script"

evaluator:
  script: "evals/run_eval.sh"
  inputs: ["control/mpc/weights.yaml", "evals/scenario_library/"]
  outputs:
    closed_loop_cost: float
    constraint_violations: int
    shield_intervention_rate: float
    solve_time_p99_ms: float
  trusted: true
  baseline_score: null

execution:
  backend: simulator
  command: "python3 twin/run_scenarios.py --config {{artifact}}"
  timeout_s: 300
  sandbox: true

budget:
  max_trials: 30
  time_per_trial_s: 300

promotion:
  policy: keep_if_improves
  require_constraint_check: true

autonomy:
  mode: sandbox
  escalation_triggers:
    - "constraint_violation_detected"
    - "shield_intervention_rate > 0.10"
    - "budget_75_percent_exhausted"
```

## Mutation Surfaces by Control Method

### MPC Weight Tuning
- **What mutates**: Q, R matrices, prediction horizon N, constraint tightening
- **Mutation operators**: scale, perturb, restructure (diagonal → full)
- **Evaluator**: closed-loop cost + constraint satisfaction over scenario library

### CBF Parameter Tuning
- **What mutates**: barrier function parameters, class-K function gains, margins
- **Mutation operators**: scale margins, adjust gains, swap barrier formulations
- **Evaluator**: safety margin utilization + nominal performance degradation

### Koopman Lift Learning
- **What mutates**: observable functions, dictionary size, regularization
- **Mutation operators**: add/remove observables, adjust regularization, retrain
- **Evaluator**: multi-step prediction error + closed-loop stability indicators

### DeePC Configuration
- **What mutates**: data window, regularization weights, horizon, constraint sets
- **Mutation operators**: adjust parameters, refresh data, modify constraints
- **Evaluator**: tracking error + robustness under distribution shift scenarios

## Safety Rules for Controller EGRI

1. **Never bypass the safety shield during trials** — shield is part of the immutable harness
2. **Scenario library must include adversarial cases** — not just nominal operation
3. **Constraint violations are hard failures** — no "soft" constraint violations in promotion
4. **Holdout scenarios for anti-gaming** — evaluator uses scenarios not visible to the mutator
5. **Shield intervention rate is a first-class metric** — rising rate signals degrading controller
6. **Rollback to last known-good** — if promoted controller fails in deployment, immediate revert

## Nesting: EGRI Over EGRI

Level 0: Optimize controller parameters (MPC weights, CBF margins)
Level 1: Optimize the mutation strategy (which parameters to tune, search heuristics)
Level 2: Optimize the evaluation (which scenarios matter most, budget allocation)

Start with Level 0. Only nest when Level 0 converges and you need more signal.
