# AIGER format for Quantified Boolean Formulas

## Preface

This document defines the QAIGER format to represent quantified Boolean formulas (QBF) and their certificates based on the popular AIGER format. [AIGER](http://fmv.jku.at/aiger/) is a simple circuit format that comes with substantial tool support ([AIGER tools](http://fmv.jku.at/aiger/aiger-1.9.9.tar.gz), [ABC](https://people.eecs.berkeley.edu/~alanmi/abc/)). It is already used to represent systems and problems in the [Hardware Model Checking Competition (HWMCC)](http://fmv.jku.at/hwmcc17/) and the [Reactive Synthesis Competition (SyntComp)](http://www.syntcomp.org/). 

The QAIGER format consists of two parts. First we define how QBFs are represented and then we define how certificates (i.e. proofs of correctness for results of QBF solvers) are represented. 

## Input Format

Throughout this section we use the following example QBF: ∀ x ∃ y. x ↔ y 

The format is based on the [AIGER 1.9](http://fmv.jku.at/papers/BiereHeljankoWieringa-FMV-TR-11-2.pdf) format.


### Variables

QBF variables are encoded as inputs.
The AIGER symbol table stores the necessary information to reconstruct the quantifier prefix.
We propose the following format: `[level] [name]`, e.g., `1 x` and `2 y`. In this case, odd numbers would encode universal quantifier levels and even numbers existential ones.


Alternatives:

* Use positive/negative numbers for quantifier levels
* Build the quantifier levels explicitly `[quantifier-type] [name]`, then the order of variables would be relevant, e.g., `a x` and `e y`.


### Properties

We propose to use the standard AIGER semantics for invariant and bad state constraints, that is $c \rightarrow g$ where $g = \overline{b}$, $c \equiv \bigwedge_{k=1}^C c_k$, and $b \equiv \bigvee_{k=1}^B b_k$.

Alternative: Use the single output as constraint as in AIGER pre 1.9

### Example

```aiger
aag 5 2 0 0 3 1
2                        universal x
4                        existential y
11                       bad !(AND 10)
6 2 5                    x & !y
8 3 4                    !x & y
10 7 9                   !(x & !y) & !(!x & y) = (!x | y) & (x | !y)
i0 1 x
i1 2 y
c
forall x, exists y, x <-> y
```


## Not Contained/Possible Extensions

* currently prenex-only
* no support for free variables


## Certification

The format for certification is also based on AIGER. 
The QBF solvers [DepQBF](http://lonsing.github.io/depqbf/), [CAQE](https://www.react.uni-saarland.de/tools/caqe/), and [CADET](https://www.react.uni-saarland.de/tools/caqe/) already support the AIGER format for certificates, but show subtle differences in their encodings. This document is also an effort to unify the tool support for certificates of QBF solvers. 

The inputs and outputs of the AIGER file are based on the result of the satisfiability check:

* If the instance is true (satisfiable), the inputs and outputs are universal and existential variables, respectively.
* If the instance is false (unsatisfiable), the inputs and outputs are existential and universal variables, respectively.

The matching of variables to the instance is done by the variable names as specified in the symbol table.

```aiger
aag 2 1 0 1 0
2                        input x
2                        output y
i0 x
o0 y
```

### Certification Workflow

Given instance `instance.aag` and certificate `certificate.aag`.

1. Check whether certificate is Skolem or Herbrand (i.e. if it represents a proof of a true QBF or if it represents a refutation of a false QBF).
2. Create copy of `instance.aag` where variables are replaced by their functions from `certificate.aag`.
3. Check whether the resulting AIGER file satisfies/violates the given properties.





