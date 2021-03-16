# [Solutions](@id SolutionAPI)

More information can be found in the [Solutions](@ref) section of the manual.

## Basic utilities

```@docs
JuMP.optimize!
NoOptimizer
OptimizeNotCalled
```

## Termination status

```@docs
termination_status
raw_status
result_count
```

## Primal solutions

```@docs
primal_status
has_values
value
```

## Dual solutions

```@docs
dual_status
has_duals
dual
shadow_price
reduced_cost
```

## Basic attributes

```@docs
objective_value
objective_bound
dual_objective_value
solve_time
relative_gap
simplex_iterations
barrier_iterations
node_count
```

## [Conflicts](@id ref_conflicts)

```@docs
compute_conflict!
copy_conflict
```

## Sensitivity

```@docs
lp_sensitivity_report
SensitivityReport
lp_objective_perturbation_range
lp_rhs_perturbation_range
```
