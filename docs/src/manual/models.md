```@meta
CurrentModule = JuMP
DocTestSetup = quote
    using JuMP, GLPK
end
DocTestFilters = [r"≤|<=", r"≥|>=", r" == | = ", r" ∈ | in ", r"MathOptInterface|MOI"]
```

# Models

## Create a model

Create a model by passing an optimizer to [`Model`](@ref):
```jldoctest
julia> model = Model(GLPK.Optimizer)
A JuMP Model
Feasibility problem with:
Variables: 0
Model mode: AUTOMATIC
CachingOptimizer state: EMPTY_OPTIMIZER
Solver name: GLPK
```
or by calling [`set_optimizer`](@ref) on an empty [`Model`](@ref):
```jldoctest
julia> model = Model()
A JuMP Model
Feasibility problem with:
Variables: 0
Model mode: AUTOMATIC
CachingOptimizer state: NO_OPTIMIZER
Solver name: No optimizer attached.

julia> set_optimizer(model, GLPK.Optimizer)
```

!!! info
    JuMP uses "optimizer" as a synonym for "solver." Our convention is to use
    "solver" to refer to the underlying software, and use "optimizer" to refer
    to the Julia object that wraps the solver. For example, `GLPK` is a solver,
    and `GLPK.Optimizer` is an optimizer.

!!! tip
    Don't know what the fields `Model mode`, `CachingOptimizer state` mean? Read
    the [Backends](@ref) section.

Use [`optimizer_with_attributes`](@ref) to create an optimizer with some
attributes initialized:
```jldoctest
julia> model = Model(optimizer_with_attributes(GLPK.Optimizer, "msg_lev" => 0))
A JuMP Model
Feasibility problem with:
Variables: 0
Model mode: AUTOMATIC
CachingOptimizer state: EMPTY_OPTIMIZER
Solver name: GLPK
```

Alternatively, you can create a function which takes no arguments and returns
an initialized `Optimizer` object:
```jldoctest
julia> function my_optimizer()
           model = GLPK.Optimizer()
           MOI.set(model, MOI.RawParameter("msg_lev"), 0)
           return model
       end
my_optimizer (generic function with 1 method)

julia> model = Model(my_optimizer)
A JuMP Model
Feasibility problem with:
Variables: 0
Model mode: AUTOMATIC
CachingOptimizer state: EMPTY_OPTIMIZER
Solver name: GLPK
```

## Print the model

By default, `show(model)` will print a summary of the problem.
```jldoctest model_print
julia> model = Model(); @variable(model, x >= 0); @objective(model, Max, x);

julia> model
A JuMP Model
Maximization problem with:
Variable: 1
Objective function type: VariableRef
`VariableRef`-in-`MathOptInterface.GreaterThan{Float64}`: 1 constraint
Model mode: AUTOMATIC
CachingOptimizer state: NO_OPTIMIZER
Solver name: No optimizer attached.
Names registered in the model: x
```

Use `print` to print the formulation of the model (in IJulia, this will render
as LaTeX.
```jldoctest model_print
julia> print(model)
Max x
Subject to
 x ≥ 0.0
```
!!! warning
    This format is specific to JuMP. To write the model to a file, use
    [`write_to_file`](@ref) instead.


Use [`latex_formulation`](@ref) to display the model in LaTeX form.
```jldoctest model_print
julia> latex_formulation(model)
$$ \begin{aligned}
\max\quad & x\\
\text{Subject to} \quad & x \geq 0.0\\
\end{aligned} $$
```

In IJulia (and Documenter), ending a cell in with [`latex_formulation`](@ref)
will render the model in LaTeX!
```@example
using JuMP                # hide
model = Model()           # hide
@variable(model, x >= 0)  # hide
@objective(model, Max, x) # hide
latex_formulation(model)
```

## Turn off output

Use [`set_silent`](@ref) and [`unset_silent`](@ref) to disable or enable
printing output from the solver.
```jldoctest
julia> model = Model(GLPK.Optimizer);

julia> set_silent(model)
true

julia> unset_silent(model)
false
```

## Set a time limit

Use [`set_time_limit_sec`](@ref), [`unset_time_limit_sec`](@ref), and
[`time_limit_sec`](@ref) to manage time limits.
```jldoctest
julia> model = Model(GLPK.Optimizer);

julia> set_time_limit_sec(model, 60.0)
60.0

julia> time_limit_sec(model)
60.0

julia> unset_time_limit_sec(model)

julia> time_limit_sec(model)
2.147483647e6
```

## Write a model to file

JuMP can write models to a variety of file-formats using [`write_to_file`](@ref)
and [`Base.write`](@ref).

```jldoctest file_formats; setup=:(model = Model(); io = IOBuffer())
julia> write_to_file(model, "model.mps")

julia> write(io, model; format = MOI.FileFormats.FORMAT_MPS)
```

!!! info
    The supported file formats are defined by the [MOI.FileFormats.FileFormat](https://jump.dev/MathOptInterface.jl/v0.9/apireference/#MathOptInterface.FileFormats.FileFormat)
    enum.
    ```jldoctest
    julia> MOI.FileFormats.FileFormat
    Enum MathOptInterface.FileFormats.FileFormat:
    FORMAT_AUTOMATIC = 0
    FORMAT_CBF = 1
    FORMAT_LP = 2
    FORMAT_MOF = 3
    FORMAT_MPS = 4
    FORMAT_SDPA = 5
    ```

## Read a model from file

JuMP models can be created from file formats using [`read_from_file`](@ref) and
[`Base.read`](@ref).

```jldoctest file_formats
julia> model = read_from_file("model.mps")
A JuMP Model
Minimization problem with:
Variables: 0
Objective function type: GenericAffExpr{Float64,VariableRef}
Model mode: AUTOMATIC
CachingOptimizer state: NO_OPTIMIZER
Solver name: No optimizer attached.

julia> seekstart(io);

julia> model2 = read(io, Model; format = MOI.FileFormats.FORMAT_MPS)
A JuMP Model
Minimization problem with:
Variables: 0
Objective function type: GenericAffExpr{Float64,VariableRef}
Model mode: AUTOMATIC
CachingOptimizer state: NO_OPTIMIZER
Solver name: No optimizer attached.
```

## Backends

A JuMP [`Model`](@ref) is a thin layer around a *backend* of type [`MOI.ModelLike`](https://jump.dev/MathOptInterface.jl/v0.9/apireference/#Model-Interface)
that stores the optimization problem and acts as the optimization solver.

From JuMP, the MOI backend can be accessed using the [`backend`](@ref) function.
Let's see what the [`backend`](@ref) of a JuMP [`Model`](@ref) is:
```jldoctest models_backends
julia> model = Model(GLPK.Optimizer)
A JuMP Model
Feasibility problem with:
Variables: 0
Model mode: AUTOMATIC
CachingOptimizer state: EMPTY_OPTIMIZER
Solver name: GLPK

julia> b = backend(model)
MOIU.CachingOptimizer{MOI.AbstractOptimizer,MOIU.UniversalFallback{MOIU.Model{Float64}}}
in state EMPTY_OPTIMIZER
in mode AUTOMATIC
with model cache MOIU.UniversalFallback{MOIU.Model{Float64}}
  fallback for MOIU.Model{Float64}
with optimizer MOIB.LazyBridgeOptimizer{GLPK.Optimizer}
  with 0 variable bridges
  with 0 constraint bridges
  with 0 objective bridges
  with inner model A GLPK model
```

The backend is a `MOIU.CachingOptimizer` in the state `EMPTY_OPTIMIZER` and mode
`AUTOMATIC`.

### CachingOptimizer

A `MOIU.CachingOptimizer` is an MOI layer that abstracts the difference between
solvers that support incremental modification (e.g., they support adding
variables one-by-one), and solvers that require the entire problem in a single
API call (e.g., they only accept the `A`, `b` and `c` matrices of a linear
program).

It has two parts:

 1. A cache, where the model can be built and modified incrementally
    ```jldoctest models_backends
    julia> b.model_cache
    MOIU.UniversalFallback{MOIU.Model{Float64}}
    fallback for MOIU.Model{Float64}
    ```
 2. An optimizer, which is used to solve the problem
    ```jldoctest models_backends
    julia> b.optimizer
    MOIB.LazyBridgeOptimizer{GLPK.Optimizer}
    with 0 variable bridges
    with 0 constraint bridges
    with 0 objective bridges
    with inner model A GLPK model
    ```

!!! info
    The [LazyBridgeOptimizer](@ref) section explains what a
    `LazyBridgeOptimizer` is.

The `CachingOptimizer` has logic to decide when to copy the problem from the
cache to the optimizer, and when it can efficiently update the optimizer
in-place.

A `CachingOptimizer` may be in one of three possible states:

* `NO_OPTIMIZER`: The CachingOptimizer does not have any optimizer.
* `EMPTY_OPTIMIZER`: The CachingOptimizer has an empty optimizer, and it is not
  synchronized with the cached model.
* `ATTACHED_OPTIMIZER`: The CachingOptimizer has an optimizer, and it is
  synchronized with the cached model.

A `CachingOptimizer` has two modes of operation:

* `AUTOMATIC`: The `CachingOptimizer` changes its state when necessary. For
  example, [`optimize!`](@ref) will automatically call `attach_optimizer` (an
  optimizer must have been previously set). Attempting to add a constraint or
  perform a modification not supported by the optimizer results in a drop to
  `EMPTY_OPTIMIZER` mode.
* `MANUAL`: The user must change the state of the `CachingOptimizer` using
  [`MOIU.reset_optimizer(::JuMP.Model)`](@ref),
  [`MOIU.drop_optimizer(::JuMP.Model)`](@ref), and
  [`MOIU.attach_optimizer(::JuMP.Model)`](@ref). Attempting to perform
  an operation in the incorrect state results in an error.

By default [`Model`](@ref) will create a `CachingOptimizer` in `AUTOMATIC` mode.
Use the `caching_mode` keyword to create a model in `MANUAL` mode:
```jldoctest
julia> Model(GLPK.Optimizer; caching_mode = MOI.Utilities.MANUAL)
A JuMP Model
Feasibility problem with:
Variables: 0
Model mode: MANUAL
CachingOptimizer state: EMPTY_OPTIMIZER
Solver name: GLPK
```

!!! tip
    Only use `MANUAL` mode if you have a very good reason. If you want to reduce
    the overhead between JuMP and the underlying solver, consider
    [Direct mode](@ref) instead.

### LazyBridgeOptimizer

The second layer that JuMP applies automatically is a `LazyBridgeOptimizer`. A
`LazyBridgeOptimizer` is an MOI layer that attempts to transform constraints
added by the user into constraints supported by the solver. This may involve
adding new variables and constraints to the optimizer. The transformations are
selected from a set of known recipes called _bridges_.

A common example of a bridge is one that splits an interval constrait like
`@constraint(model, 1 <= x + y <= 2)` into two constraints,
`@constraint(model, x + y >= 1)` and `@constraint(model, x + y <= 2)`.

Use the `bridge_constraints=false` keyword to remove the bridging layer:
```jldoctest
julia> model = Model(GLPK.Optimizer; bridge_constraints = false)
A JuMP Model
Feasibility problem with:
Variables: 0
Model mode: AUTOMATIC
CachingOptimizer state: EMPTY_OPTIMIZER
Solver name: GLPK

julia> backend(model)
MOIU.CachingOptimizer{MOI.AbstractOptimizer,MOIU.UniversalFallback{MOIU.Model{Float64}}}
in state EMPTY_OPTIMIZER
in mode AUTOMATIC
with model cache MOIU.UniversalFallback{MOIU.Model{Float64}}
  fallback for MOIU.Model{Float64}
with optimizer A GLPK model
```

!!! tip
    Only disable bridges if you have a very good reason. If you want to reduce
    the overhead between JuMP and the underlying solver, consider
    [Direct mode](@ref) instead.

## Direct mode

Using a `CachingOptimizer` results in an additional copy of the model being
stored by JuMP in the `.model_cache` field. To avoid this overhead, create a
JuMP model using [`direct_model`](@ref):
```jldoctest direct_mode
julia> model = direct_model(GLPK.Optimizer())
A JuMP Model
Feasibility problem with:
Variables: 0
Model mode: DIRECT
Solver name: GLPK
```

!!! warning
    Solvers that do not support incremental modification do not support
    `direct_model`. An error will be thrown, telling you to use a
    `CachingOptimizer` instead.

The benefit of using [`direct_model`](@ref) is that there are no extra layers
(e.g., `Cachingoptimizer` or `LazyBridgeOptimizer`) between `model` and the
provided optimizer:
```jldoctest direct_mode
julia> typeof(backend(model))
GLPK.Optimizer
```

A downside of direct mode is that there is no bridging layer. Therefore, only
constraints which are natively supported by the solver are supported. For
example, `GLPK.jl` does not implement constraints of the form `l <= a' x <= u`.
```julia direct_mode
julia> @variable(model, x[1:2]);

julia> @constraint(model, 1 <= x[1] + x[2] <= 2)
ERROR: Constraints of type MathOptInterface.ScalarAffineFunction{Float64}-in-MathOptInterface.Interval{Float64} are not supported by the solver.
[...]
```
