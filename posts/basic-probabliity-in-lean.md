---
author: 'Rémy Degenne'
category: 'Tutorial'
date: 2024-10-02 16:41:48 UTC+02:00
description: ''
has_math: true
link: ''
slug: basic-probability-in-lean
tags: ''
title: Basic probability in Lean
type: text
---

How do I define a probability space and two independent random variables in Lean? Should I use `IsProbabilityMeasure` or `ProbabilityMeasure`?
How do I condition on an event?

This post answers these and other simple questions about how to express probability concepts using Mathlib.
We will mostly not discuss theorems, but focus on definitions. The goal is to have enough knowledge about the definitions in Mathlib related to probability to state probability theory lemmas.

<!-- TEASER_END -->

The code examples will mostly not mention imports: use `import Mathlib` in a project that depends on Mathlib (and then prune the imports with `#min_imports` if you like).

# Probability spaces and probability measures

First, in order to work on probability we need a measurable space.
We can define a probability measure on such a space as follows.
```lean
variable {Ω : Type*} [MeasurableSpace Ω] {P : Measure Ω} [IsProbabilityMeasure P]
```
The class `MeasurableSpace Ω` defines a sigma-algebra on `Ω`. We then introduced a measure `P` and specified that it should be a probability measure.
If we want to work on `ℝ` or another well known type the typeclass inference system will find `[MeasurableSpace ℝ]` on its own. We can write simply
```lean
variable {P : Measure ℝ} [IsProbabilityMeasure P]
```

With the code above, we can introduce several probability measures on the same space. When using lemmas and definitions about those measures, we will need to specify which measure we are talking about.
```lean
TODO example
```
But perhaps we just want a space with a canonical probability measure, which would ideally be the one used without us having to tell Lean explicitly.
That can be done with the `MeasureSpace` class. A `MeasureSpace` is a `MeasurableSpace` with a canonical measure, called `volume`.
The probability library of Mathlib defines a notation `ℙ` for that measure. We still need to tell that we want it to be a probability measure though.
```lean
variable {Ω : Type*} [MeasureSpace Ω] [IsProbabilityMeasure (ℙ : Measure Ω)]
```
Note: in the code above we can't write only `[IsProbabilityMeasure ℙ]` because Lean would then not know to which space the default measure `ℙ` refers to.
That will not be necessary when we use `ℙ` in proofs because the context will be enough to infer `Ω`.

## `IsProbabilityMeasure` vs `ProbabilityMeasure`

The examples above used `{P : Measure Ω} [IsProbabilityMeasure P]` to define a probability measure. That's the standard way to do it.
Mathlib also contains a type `ProbabilityMeasure Ω`. The goal of that type is to work on the set of probability measures on `Ω`.
In particular, that type comes with a topology, the topology of convergence in distribution (weak convergence of measures).
If we don't need to work with that topology, `{P : Measure Ω} [IsProbabilityMeasure P]` should be preferred.

## Probability of events

A `Measure` can be applied to a set like a function, and returns a value in `ENNReal` (denoted by the notation `ℝ≥0∞`, available after `open scoped ENNReal`).
```lean
import Mathlib
open scoped ENNReal

example (P : Measure ℝ) (s : Set ℝ) : ℝ≥0∞ := P s
```
The type `ℝ≥0∞` represents the nonnegative reals and infinity: the measure of a set is a nonnegative real number which in general may be infinite.
Measures can in general take infinite values, but since our `ℙ` is a probabilty measure, it actually takes only values up to 1.
`simp` knows that a probability measure is finite and will use the lemmas `measure_ne_top` or `measure_lt_top` to prove that `ℙ s ≠ ∞` or `ℙ s < ∞`.
The operations on `ℝ≥0∞` are not as nicely behaved as on `ℝ`: `ℝ≥0∞` is not a ring and subtraction truncates to zero for example. If one finds that lemma `lemma_name` used to transform an equation does not apply to `ℝ≥0∞`, a good thing to try is to find a lemma named like `ENNReal.lemma_name_of_something` and use that instead (it will typically require that one variable is not infinite).

For many lemmas to apply, the set will need to be a measurable set. The way to expressed that a set `s` is measurable is `MeasurableSet s`.

## Random variables

A random variable is a measurable function from a measurable space to another.
```lean
variable {Ω : Type*} [MeasurableSpace Ω] {X : Ω → ℝ} (hX : Measurable X)
```
In that code we defined a random variable `X` from the measurable space `Ω` to `ℝ` (for which the typeclass inference system finds a measurable space instance). `hX` states that `X` is measurable, which is necessary for most manipulations.

TODO

## Expectation of a random variable

TODO

TODO: Lebesgue and Bochner integrals

## Discrete probability

TODO: `.of_discrete`, `[DiscreteMeasurableSpace]`

TODO: what about PMF? (I don't know anything about those)

## Additional typeclasses on measurable spaces

Some results in probability theory require the sigma-algebra to be the Borel sigma-algebra, generated by the open sets.
For that we first need `Ω` to be a topological space and we then need to add a `[BorelSpace Ω]` variable, which links topology and measurability.
```lean
variable {Ω : Type*} [MeasurableSpace Ω] [TopologicalSpace Ω] [BorelSpace Ω]
```

For properties related to conditional distributions and the existence of posterior probability distributions, it is often convenient or necessary to work in a standard Borel space (a measurable space arising as the Borel sets of some Polish topology). See the `StandardBorelSpace` typeclass.

# Other probability topics

The goal of this section is to give pointers to the Mathlib definitions for various probability notions.
That list might be out of date when you read this! Look around in the documentation.

## Known probability distributions

See the Probability/Distributions folder.

- Exponential
- Gamma
- Gaussian (only in 1D)
- Geometric
- Pareto
- Poisson
- Uniform

## CDF, pdf, Variance, moments

TODO:

- Probability density function: `MeasureTheory.pdf X P Q`
- Cumulative distribution function: `cdf P`
- Expectation: `P[X]`
- Variance: `variance X P`
- Moment of order `p`: `moment X p P`
- Central moment of order `p`: `centralMoment X p P`
- Moment generating function: `mgf X P t`
- Cumulant generating function: `cgf X P t`

## Conditioning

TODO: two meanings of conditioning. `cond` vs `condexp` and friends.

## Identically distributed

`IdentDistrib X Y P Q` (or `IdentDistrib X Y` in `MeasureSpace`).

## Independence

TODO: independence of sigma-algebras, sets, functions (random variables).

### Unconditional independence

Two independent random variables:
```lean
variable {Ω : Type*} [MeasurableSpace Ω] {P : Measure Ω}
  {X Y : Ω → ℝ} {Y : Ω → ℕ} (hX : Measurable X) (hY : Measurable Y)
  (hXY : IndepFun X Y P)
```
Or on a measure space (note the missing measure argument for `IndepFun`):
```lean
variable {Ω : Type*} [MeasureSpace Ω]
  {X Y : Ω → ℝ} {Y : Ω → ℕ} (hX : Measurable X) (hY : Measurable Y)
  (hXY : IndepFun X Y)
```

A family of independent random variables:
```lean
variable {Ω ι : Type*} [MeasureSpace Ω]
  {X : ι → Ω → ℝ} (hX : iIndepFun (fun _ ↦ inferInstance) X)
```
TODO: that's ugly. Do we need the explicit measurable spaces in iIndepFun?

### Conditional independence

TODO

## Martingales, filtrations

`ℱ : MeasureTheory.Filtration ι m0`

`MeasureTheory.Martingale X ℱ P`

adapted

stopping time

## Transition kernels

# Additional resources


