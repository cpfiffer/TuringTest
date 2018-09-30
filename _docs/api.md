---
title: Public API
permalink: /docs/library/
excerpt: "How to quickly install and setup Minimal Mistakes for use with GitHub Pages."
last_modified_at: 2018-07-19T11:31:38-04:00
redirect_from:
  - /theme-setup/
toc: true
---


<a id='Function-Documentation-1'></a>

# Function Documentation





<a id='Modelling-1'></a>

## Modelling

<a id='Turing.@model' href='#Turing.@model'>#</a>
### `Turing.@model` &mdash; *Macro*.



```
@model(name, fbody)
```

Macro to specify a probabilistic model.

Example:

```julia
@model Gaussian(x) = begin
    s ~ InverseGamma(2,3)
    m ~ Normal(0,sqrt.(s))
    for i in 1:length(x)
        x[i] ~ Normal(m, sqrt.(s))
    end
    return (s, m)
end
```

Compiler design: `sample(fname(x,y), sampler)`.

```julia
fname(x=nothing,y=nothing; compiler=compiler) = begin
    ex = quote
        # Pour in kwargs for those args where value != nothing.
        fname_model(vi::VarInfo, sampler::Sampler; x = x, y = y) = begin
            vi.logp = zero(Real)

            # Pour in model definition.
            x ~ Normal(0,1)
            y ~ Normal(x, 1)
            return x, y
        end
    end
    return Main.eval(ex)
end
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/core/compiler.jl#L177-L212' class='documenter-source'>source</a><br>

<a id='Turing.@~' href='#Turing.@~'>#</a>
**`Turing.@~`** &mdash; *Macro*.



```
macro: @~ var Distribution()
```

Tilde notation macro. This macro constructs Turing.observe or Turing.assume calls depending on the left-hand argument. Note that the macro is interconnected with the @model macro and assumes that a `compiler` struct is available.

Example:

```julia
@~ x Normal()
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/core/compiler.jl#L97-L109' class='documenter-source'>source</a><br>


<a id='Samplers-1'></a>

## Samplers

<a id='Turing.Sampler' href='#Turing.Sampler'>#</a>
**`Turing.Sampler`** &mdash; *Type*.



```
Sampler{T}
```

Generic interface for implementing inference algorithms. An implementation of an algorithm should include the following:

1. A type specifying the algorithm and its parameters, derived from InferenceAlgorithm
2. A method of `sample` function that produces results of inference, which is where actual inference happens.

Turing translates models to chunks that call the modelling functions at specified points. The dispatch is based on the value of a `sampler` variable. To include a new inference algorithm implements the requirements mentioned above in a separate file, then include that file at the end of this one.


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/Turing.jl#L91-L102' class='documenter-source'>source</a><br>

<a id='Turing.Gibbs' href='#Turing.Gibbs'>#</a>
**`Turing.Gibbs`** &mdash; *Type*.



```
Gibbs(n_iters, alg_1, alg_2)
```

Compositional MCMC interface.

Example:

```julia
alg = Gibbs(1000, HMC(1, 0.2, 3, :v1), PG(20, 1, :v2))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/gibbs.jl#L1-L10' class='documenter-source'>source</a><br>

<a id='Turing.HMC' href='#Turing.HMC'>#</a>
**`Turing.HMC`** &mdash; *Type*.



```
HMC(n_iters::Int, epsilon::Float64, tau::Int)
```

Hamiltonian Monte Carlo sampler.

Usage:

```julia
HMC(1000, 0.05, 10)
```

Example:

```julia
# Define a simple Normal model with unknown mean and variance.
@model gdemo(x) = begin
    s ~ InverseGamma(2,3)
    m ~ Normal(0, sqrt(s))
    x[1] ~ Normal(m, sqrt(s))
    x[2] ~ Normal(m, sqrt(s))
    return s, m
end

sample(gdemo([1.5, 2]), HMC(1000, 0.05, 10))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/hmc.jl#L1-L26' class='documenter-source'>source</a><br>

<a id='Turing.HMCDA' href='#Turing.HMCDA'>#</a>
**`Turing.HMCDA`** &mdash; *Type*.



```
HMCDA(n_iters::Int, n_adapt::Int, delta::Float64, lambda::Float64)
```

Hamiltonian Monte Carlo sampler wiht Dual Averaging algorithm.

Usage:

```julia
HMCDA(1000, 200, 0.65, 0.3)
```

Example:

```julia
# Define a simple Normal model with unknown mean and variance.
@model gdemo(x) = begin
  s ~ InverseGamma(2,3)
  m ~ Normal(0, sqrt(s))
  x[1] ~ Normal(m, sqrt(s))
  x[2] ~ Normal(m, sqrt(s))
  return s, m
end

sample(gdemo([1.5, 2]), HMCDA(1000, 200, 0.65, 0.3))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/hmcda.jl#L1-L26' class='documenter-source'>source</a><br>

<a id='Turing.IPMCMC' href='#Turing.IPMCMC'>#</a>
**`Turing.IPMCMC`** &mdash; *Type*.



```
IPMCMC(n_particles::Int, n_iters::Int, n_nodes::Int, n_csmc_nodes::Int)
```

Particle Gibbs sampler.

Usage:

```julia
IPMCMC(100, 100, 4, 2)
```

Example:

```julia
# Define a simple Normal model with unknown mean and variance.
@model gdemo(x) = begin
  s ~ InverseGamma(2,3)
  m ~ Normal(0,sqrt(s))
  x[1] ~ Normal(m, sqrt(s))
  x[2] ~ Normal(m, sqrt(s))
  return s, m
end

sample(gdemo([1.5, 2]), IPMCMC(100, 100, 4, 2))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/ipmcmc.jl#L1-L26' class='documenter-source'>source</a><br>

<a id='Turing.IS' href='#Turing.IS'>#</a>
**`Turing.IS`** &mdash; *Type*.



```
IS(n_particles::Int)
```

Importance sampling algorithm object.

  * `n_particles` is the number of particles to use

Usage:

```julia
IS(1000)
```

Example:

```julia
# Define a simple Normal model with unknown mean and variance.
@model gdemo(x) = begin
    s ~ InverseGamma(2,3)
    m ~ Normal(0,sqrt.(s))
    x[1] ~ Normal(m, sqrt.(s))
    x[2] ~ Normal(m, sqrt.(s))
    return s, m
end

sample(gdemo([1.5, 2]), IS(1000))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/is.jl#L1-L28' class='documenter-source'>source</a><br>

<a id='Turing.MH' href='#Turing.MH'>#</a>
**`Turing.MH`** &mdash; *Type*.



```
MH(n_iters::Int)
```

Metropolis-Hasting sampler.

Usage:

```julia
MH(100, (:m, (x) -> Normal(x, 0.1)))
```

Example:

```julia
# Define a simple Normal model with unknown mean and variance.
@model gdemo(x) = begin
  s ~ InverseGamma(2,3)
  m ~ Normal(0,sqrt(s))
  x[1] ~ Normal(m, sqrt(s))
  x[2] ~ Normal(m, sqrt(s))
  return s, m
end

sample(gdemo([1.5, 2]), MH(1000, (:m, (x) -> Normal(x, 0.1)), :s)))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/mh.jl#L1-L26' class='documenter-source'>source</a><br>

<a id='Turing.NUTS' href='#Turing.NUTS'>#</a>
**`Turing.NUTS`** &mdash; *Type*.



```
NUTS(n_iters::Int, n_adapt::Int, delta::Float64)
```

No-U-Turn Sampler (NUTS) sampler.

Usage:

```julia
NUTS(1000, 200, 0.6j_max)
```

Example:

```julia
# Define a simple Normal model with unknown mean and variance.
@model gdemo(x) = begin
  s ~ InverseGamma(2,3)
  m ~ Normal(0, sqrt(s))
  x[1] ~ Normal(m, sqrt(s))
  x[2] ~ Normal(m, sqrt(s))
  return s, m
end

sample(gdemo([1.j_max, 2]), NUTS(1000, 200, 0.6j_max))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/nuts.jl#L1-L26' class='documenter-source'>source</a><br>

<a id='Turing.PG' href='#Turing.PG'>#</a>
**`Turing.PG`** &mdash; *Type*.



```
PG(n_particles::Int, n_iters::Int)
```

Particle Gibbs sampler.

Usage:

```julia
PG(100, 100)
```

Example:

```julia
# Define a simple Normal model with unknown mean and variance.
@model gdemo(x) = begin
  s ~ InverseGamma(2,3)
  m ~ Normal(0, sqrt(s))
  x[1] ~ Normal(m, sqrt(s))
  x[2] ~ Normal(m, sqrt(s))
  return s, m
end

sample(gdemo([1.5, 2]), PG(100, 100))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/pgibbs.jl#L1-L26' class='documenter-source'>source</a><br>

<a id='Turing.PMMH' href='#Turing.PMMH'>#</a>
**`Turing.PMMH`** &mdash; *Type*.



```
PMMH(n_iters::Int, smc_alg:::SMC, parameters_algs::Tuple{MH})
```

Particle independant Metropolis–Hastings and Particle marginal Metropolis–Hastings samplers.

Usage:

```julia
alg = PMMH(100, SMC(20, :v1), MH(1,:v2))
alg = PMMH(100, SMC(20, :v1), MH(1,(:v2, (x) -> Normal(x, 1))))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/pmmh.jl#L1-L13' class='documenter-source'>source</a><br>

<a id='Turing.SGHMC' href='#Turing.SGHMC'>#</a>
**`Turing.SGHMC`** &mdash; *Type*.



```
SGHMC(n_iters::Int, learning_rate::Float64, momentum_decay::Float64)
```

Stochastic Gradient Hamiltonian Monte Carlo sampler.

Usage:

```julia
SGHMC(1000, 0.01, 0.1)
```

Example:

```julia
@model example begin
  ...
end

sample(example, SGHMC(1000, 0.01, 0.1))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/sghmc.jl#L1-L21' class='documenter-source'>source</a><br>

<a id='Turing.SGLD' href='#Turing.SGLD'>#</a>
**`Turing.SGLD`** &mdash; *Type*.



```
SGLD(n_iters::Int, step_size::Float64)
```

Stochastic Gradient Langevin Dynamics sampler.

Usage:

```julia
SGLD(1000, 0.5)
```

Example:

```julia
@model example begin
  ...
end

sample(example, SGLD(1000, 0.5))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/sgld.jl#L1-L21' class='documenter-source'>source</a><br>

<a id='Turing.SMC' href='#Turing.SMC'>#</a>
**`Turing.SMC`** &mdash; *Type*.



```
SMC(n_particles::Int)
```

Sequential Monte Carlo sampler.

Usage:

```julia
SMC(1000)
```

Example:

```julia
# Define a simple Normal model with unknown mean and variance.
@model gdemo(x) = begin
  s ~ InverseGamma(2,3)
  m ~ Normal(0, sqrt(s))
  x[1] ~ Normal(m, sqrt(s))
  x[2] ~ Normal(m, sqrt(s))
  return s, m
end

sample(gdemo([1.5, 2]), SMC(1000))
```


<a target='_blank' href='https://github.com/cpfiffer/Turing.jl/blob/337864f8aee1734befb283b73d167b912e8718ad/src/samplers/smc.jl#L1-L26' class='documenter-source'>source</a><br>


<a id='Index-1'></a>

## Index

- [`Turing.Gibbs`](api.md#Turing.Gibbs)
- [`Turing.HMC`](api.md#Turing.HMC)
- [`Turing.HMCDA`](api.md#Turing.HMCDA)
- [`Turing.IPMCMC`](api.md#Turing.IPMCMC)
- [`Turing.IS`](api.md#Turing.IS)
- [`Turing.MH`](api.md#Turing.MH)
- [`Turing.NUTS`](api.md#Turing.NUTS)
- [`Turing.PG`](api.md#Turing.PG)
- [`Turing.PMMH`](api.md#Turing.PMMH)
- [`Turing.SGHMC`](api.md#Turing.SGHMC)
- [`Turing.SGLD`](api.md#Turing.SGLD)
- [`Turing.SMC`](api.md#Turing.SMC)
- [`Turing.Sampler`](api.md#Turing.Sampler)
- [`Turing.@model`](api.md#Turing.@model)
- [`Turing.@~`](api.md#Turing.@~)
