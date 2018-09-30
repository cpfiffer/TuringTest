
---


---


<a id='Probabalistic-Programming-in-Thirty-Seconds-1'></a>

# Probabalistic Programming in Thirty Seconds


If you are already well-versed in probabalistic programming and just want to take a quick look at how Turing's syntax works or otherwise just want a model to start with, we have provided a Bayesian coin-flipping model to play with.


This example can be run on however you have Julia installed (see [Getting Started](get-started.md)), but you will need to install the packages `Turing`, `Distributions`, `MCMCChain`, and `StatPlots` if you have not done so already.


This is an excerpt from a more formal example introducing probabalistic programming which can be found in Jupyter notebook form [here](https://github.com/TuringLang/TuringTutorials/blob/master/0_Introduction.ipynb) or as part of the documentation website [here](0_Introduction.md).


```julia
# Import libraries.
using Turing, Distributions, MCMCChain, StatPlots, Random

# Set the true probability of heads in a coin.
p_true = 0.5

# Iterate from having seen 0 observations to 100 observations.
Ns = 0:100;

# Draw data from a Bernoulli distribution, i.e. draw heads or tails.
Random.seed!(12)
data = rand(Bernoulli(p_true), last(Ns))

# Here's what the first five coin flips look like:
data[1:5]

# Declare our Turing model.
@model coinflip(y) = begin
    # Our prior belief about the probability of heads in a coin.
    p ~ Beta(1, 1)

    # The number of observations.
    N = length(y)
    for n in 1:N
        # Heads or tails of a coin are drawn from a Bernoulli distribution.
        y[n] ~ Bernoulli(p)
    end
end;

# Settings of the Hamiltonian Monte Carlo (HMC) sampler.
iterations = 1000
ϵ = 0.05
τ = 10

# Start sampling.
chain = sample(coinflip(data), HMC(iterations, ϵ, τ));

# Construct summary of the sampling process for the parameter p, i.e. the probability of heads in a coin.
p_summary = Chains(chain[:p])
histogramplot(p_summary)
```

