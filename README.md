# AIGER format for Quantified Boolean Formulas

## Preface

AIGER is a popular format among the verification community, it provides the base for the Hardware Model Checking Competition (HWMCC) and Reactive Synthesis Competition (SyntComp).
In the context of quantified Boolean Formulas (QBF), AIGER is often used as a format to encode certificates.

## Format

Throughout this section we use the following example QBF: $\forall x \exists y \colon x \leftrightarrow y$.

The format is based on the AIGER 1.9 format.


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

1. Check whether certificate is Skolem or Herbrand.
2. Create copy of `instance.aag` where variables are replaced by their functions from `certificate.aag`.
3. Check whether the resulting AIGER file satisfies/violates the given properties.





