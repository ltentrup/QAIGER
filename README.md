# AIGER format for Quantified Boolean Formulas

## Preface

This document defines the QAIGER format to represent quantified Boolean formulas (QBF) and their certificates based on the popular AIGER format. [AIGER](http://fmv.jku.at/aiger/) is a simple circuit format that comes with substantial tool support ([AIGER tools](http://fmv.jku.at/aiger/aiger-1.9.9.tar.gz), [ABC](https://people.eecs.berkeley.edu/~alanmi/abc/)). It is already used to represent systems and problems in the [Hardware Model Checking Competition (HWMCC)](http://fmv.jku.at/hwmcc17/) and the [Reactive Synthesis Competition (SyntComp)](http://www.syntcomp.org/). 

The QAIGER format consists of two parts. First we define how QBFs are represented and then we define how certificates (i.e. proofs of correctness for results of QBF solvers) are represented. 

## Input Format

QAIGER input format defines QBFs in prenex form, i.e. it represents formulas that start with a quantifier prefix followed by a propositional (i.e. quantifier-free) part. For example consider this QBF in prenex form: ∀ x ∃ y. x ↔ y 

Throughout this section we use this example formua to discuss the QAIGER input format. 

### Variables

Variables of the QBF are represented as the inputs of the AIGER circuit. 
All inputs must be given a quantifier level in the symbol table, optionally followed by space character and an arbitrary name. 
Quantifier levels are numbers from 0 to INT_MAX (i.e. 2147483647), where even numbers represent existential quantifiers and odd numbers represent universal quantifiers. 
Quantifiers with higher numbers are in the scope of quantifiers with lower numbers. 
In particular, variables with quantifier level 0 are constants. 

### Semantics

QAIGER circuits have a single output that indicates if the formula is true or false for the given variable assignment (i.e. the inputs). 

That is, for now we restrict QAIGER to the [original AIGER format](http://fmv.jku.at/papers/Biere-FMV-TR-07-1.pdf), without the extensions discussed for [AIGER 1.9](http://fmv.jku.at/papers/BiereHeljankoWieringa-FMV-TR-11-2.pdf). 
This is because an important tool for minimizing circuits, the ABC model checker, still has only limited support for the new AIGER format. 


### Example

```aiger
aag 5 2 0 1 3
2                        first input, universal variable x
4                        second input, existential variable y
10                       defines output as signal 10
6 2 5                    defines signal 6 as x & !y
8 3 4                    defines signal 8 as !x & y
10 7 9                   defines signal 10 as !6 & !8
i0 1 x                   defines variable x to be on quantifier level 1
i1 2 y                   defines variable y to be on quantifier level 2
c
forall x, exists y, x <-> y
```


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





