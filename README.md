# ArnoldiMethodTransformations

A package for easily interfacing with [ArnoldiMethod](https://github.com/haampie/ArnoldiMethod.jl), using the suggested [transformations](https://haampie.github.io/ArnoldiMethod.jl/stable/usage/02_spectral_transformations.html) suggested in the [documentation](https://haampie.github.io/ArnoldiMethod.jl/stable/index.html).


## Installation

In REPL, type either `] add ArnoldiMethodTransformations` or
````JULIA
using Pkg
Pkg.add("ArnoldiMethodTransformations")
````

This package mainly extends some methods of [ArnoldiMethod](https://github.com/haampie/ArnoldiMethod.jl), which needs to be separately installed.
It exports three constants: `USOLVER`, `PSOLVER`, and `MSOLVER`, used to indicate whether to use UMFPACK, Pardiso, or MUMPS.

## Example
Ordinary eigenvalue problem `Ax=λx`
````JULIA
using LinearAlgebra
using ArnoldiMethod
using ArnoldiMethodTransformations

# construct fixed eval matrix in random basis
D = diagm(0=>[0,1,2,3,4,5,6,7,8,9])
S = randn(10,10)
A = S\D*S

# find eigenpairs closest to 5.001 (cannot be 5 as algorithm is unstable if σ is exactly an eval)
σ = 5.001
decomp, hist = partialschur(A,σ)

# get evecs
λ, v = partialeigen(decomp,σ)

display(λ)
norm(A*v-v*diagm(0=>λ))
# should be ~1e-11 or smaller
````

Generalized eigenvalue problem `Ax=λBx`
````JULIA
using LinearAlgebra
using ArnoldiMethod
using ArnoldiMethodTransformations

# construct fixed eval matrix in random basis
A = rand(ComplexF64,10,10)
B = rand(ComplexF64,10,10)

# find eigenpairs closest to .5
σ = .5
decomp, hist = partialschur(A,B,σ)

# get evecs
λ, v = partialeigen(decomp,σ)

display(λ)
norm(A*v-B*v*diagm(0=>λ))
# should be ~1e-14 or smaller
````

Note that in both cases, `ArnoldiMethod` needed to be explicitly brought into scope with `using`.

## Methods
This package exports none of its own methods, but extends `partialschur`  and `partialeigen` from [ArnoldiMethod](https://github.com/haampie/ArnoldiMethod.jl).

It does export three constants: `USOLVER`, `PSOLVER`, `MSOLVER`.

---------------
    `partialschur(A, [B], σ; [diag_inv_B, lupack=USOLVER, kwargs...]) -> decomp, history`

Partial Schur decomposition of `A`, with shift `σ` and mass matrix `B`, solving `A*v=σ*B*v`

Keyword `diag_inv_B` defaults to `true` if `B` is both diagonal and invertible. This enables
a simplified shift-and-invert scheme.

Keyword `lupack` determines what linear algebra library to use. Options are `USOLVER` (UMFPACK, the default), `PSOLVER` (Pardiso), and the default `MSOLVER` (MUMPS).

The relevant solver must be explicitly loaded at the top level to use it (e.g., `using Pardiso` must be called before `lupack=PSOLVER` can be used).

For other keywords, see `ArnoldiMethod.partialschur`

---------------
    partialeigen(decomp, σ)

Transforms a partial Schur decomposition into an eigendecomposition, but undoes the shift-and-invert of the eigenvalues by `σ`.

------------
Note that the shifting to an exact eigenvalue poses a problem, see note on [purification](https://haampie.github.io/ArnoldiMethod.jl/stable/theory.html#Purification-1).


## Linear Solvers
There are two solvers currently available for use in this package: UMFPACK (via `Base.LinAlg`), and [Pardiso](https://pardiso-project.org) (via [`Pardiso`](https://github.com/JuliaSparse/Pardiso.jl)).

Pardiso is often faster, and uses significantly less memory, but require separate installation, which not all users will want to do. This optional dependency is implemented with [Requires.jl](https://github.com/MikeInnes/Requires.jl).

The default solver is UMFPACK. To use another solver, such as Pardiso (assuming it is installed), use the keyword `:lupack=PSOLVER` in `partialschur`.

To do: add [MUMPS](http://mumps.enseeiht.fr) to the available solvers.
