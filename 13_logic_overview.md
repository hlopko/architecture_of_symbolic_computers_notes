# A Logic Overview #

The primary kind of statement with which logic computing deals is a ***fact***.
A ***fact*** is an expression that some object or set of objects satisfies some
specific relationship.

![Logic computing](figures/13_01_logic_computing.jpg)

## Formal Logic Systems ##

![A logic system](fingures/13_5_logic_system.jpg)

The ***sytax*** of a logic model revolves around ***logical expressions***
built up from a set of more basic ***symbols***. A subset of symbol strings
representing valid expressions is called ***well-formed formulas*** (or
***wffs***). Individually, they are called ***statement, proposition,*** or
***sentence***.

Typisal syntaxes include allowances for:

* ***constants*** (specific objects)
* ***functions*** applied to such objects to yield other objects
* ***predicates*** to test membership of tuples of objects in relations
* ***identifiers*** or ***logic variables*** stand for *unknown* objects
* other operations which combine these or place constraints on values that
  various variables may take on.

The ultimate purpose of a system of logic is to divide the universe of wffs
into pieces; those to be considered true, those that are false, and those about
which no absolute statement can be made.

In most real logic programming languages the user provides ***axioms***, rules
that are considered true. 

## Rules of Inference ##

Axiom specification provides only part of the information needed. In a properly
written program, other parts of the language's semantics extend these
statements to cover other statemenets which fully define the true and false
sides of the wff set. This extension is done via ***inferencing***, which uses
specific patterns of known sets of wffs to predict or deduce the veracity of
other wffs. Each such pattern is called an ***interence rule***. 

The actual process of inferencing involves finding some inference rule `R`,
some subset `X` of wffs already in the true set, and some other wff `a`
such that `X + a = R`. Given our definition of `R` as a set of wffs, this means
that the wff `a` should also be considered true. We say that `a` is an
***inference, logical consequence, conclusion*** or ***direct consequence***
from the set `X` using the inference rule `R`. 

Common inference rule is ***modus ponens***. This rule takes some wff `A` and
some other wff of the form `if A then B`, and infers that the wff `B` is also
true:

	modus-ponens={(A, 'if A then B', B)|A and B any valid wffs}

A ***proof*** is sequence of wffs `a1...an` such taht eack `ak` is either an
axiom or direct consequence of some subset of the prior `aj`.

A ***theorem*** is a wff `a` which is a member of some proof sequence, usually
as the last wff. The notation Γ 
