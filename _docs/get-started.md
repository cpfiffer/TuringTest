---
permalink: /docs/get-started
excerpt: "How to quickly install and setup Minimal Mistakes for use with GitHub Pages."
last_modified_at: 2018-07-19T11:31:38-04:00
redirect_from:
  - /theme-setup/
toc: true
---

<a id='Installation-1'></a>

## Installation


To use Turing, you need install Julia first and then install Turing.


<a id='Install-Julia-1'></a>

### Install Julia


You will need to install Julia 1.0, which you can get from [the official Julia website](http://julialang.org/downloads/).


<a id='Install-Turing.jl-1'></a>

### Install Turing.jl


Turing is an officially registered Julia package, so the following will install a stable version of Turing while inside Julia's package manager (press `]` from the REPL):


```julia
add Turing
```


If you want to use the latest version of Turing with some experimental features, you can try the following instead:


```julia
add Turing#master
test Turing
```


If all tests pass, you're ready to start using Turing.
