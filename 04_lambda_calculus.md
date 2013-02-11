# Lambda Calculus #

*in code examples symbols `->` represent &lambda;*

Lambda calculus is a mathematical language for describing arbitrary
*expressions* built from the application of functions to other expressions.
It is as powerful as any other notations for describing algorithms, including
any programming language.


Main operation of &lambda; - calculus is the application of one subexpression
(function) to another (its single argument) by substitution of the argument into
the function's body. **Currying** is the normal mode of execution.

* functions have no names
* names given to things are the formal parameters of a function
* lexical scoping

## Syntax ##

	<identifier>  := a|b|c|d|e ...
	<function>    := ( -><identifier>"|"<expression>)
	<application> := (<expression><expression>)
	<expression>  := <identifier> | <function> | <application>

Examples:

	identifier:  a
	application: ((->x|(yx))a)
	function:    (->x|(yx))


## General Model Of Computation ##

In &lambda; - Calculus, what an expression means is equivalent to what it can
reduce to after all function applications have been performed.

Interpretative semantic model:

1. find all possible application subexpressions in the expression
2. pick one where the function is neither a simple identifier nor an
   application expression, but a function expression of the form `(->x|E)`
   where `E` is arbitrary expression
3. assume that the expression to the right of this function is expression `A`
4. perform the substitution `[A/x]E` - identify all free occcurences of `x` in
   `E` and replace them by `A`
5. replace the entrire application by the result of this substitution and loop
   back to step 1

Sample computation:

	((->x|(xi))((->z|((->q|q)z))h)) # applying (->x|(xi)) to ((->z|((->q|q)z))h))
	------------------------------- # ([((->z|((->q|q)z))h)/x](->x|(xi))) 
	(((->z|((->q|q)z))h)i)			# applying (->q|q) to z ([z/q]q)
	(((->z|z)h)i)					# applying (->z|z) to h ([h/z]z)
	(hi)


## Standard Simplifications ##

`(...((AB)C)...X)` is the same as `(ABC...X)`

`(AB)` is the same as `AB`, therefore `(->x|(->y|(->z|M)))` is the same as `(->x|->y|->z|M)`

`(->x|->y|->z|M)` is the same as `(->xyz|M)`

## Identifiers ##

An ***identifier*** is used in &lambda; - Calculus as a placeholder to indicate
where in a function's definition a real argument should be substituted, when
the function is applied. Each use of the identifier in the function body is
called an ***instance*** of that identifier. ***Scope*** of an identifier is
the region of the expression where instances of the identifier will always have
the same value - in &lambda; - Calculus, the rules for scoping are the same as
in conventional statically scoped languages. An instance can be ***bound*** or
***free*** depending on whether the identifier is in the argument list of the
function. belongs to the current scope.

To clarify a little bit more:
1. free variables of `x` are just `x`
2. free variables of `(->x|A)` are free variables of `A` with `x` removed
3. free variables of `(AB)` are union of free variables of `A` and `B`

Examples:

	(->x|x)		#has no free variables
	(->x|xy)	#has one free variable - y 
	(->x|xx)x	#has one free variable - x !but only last instance!


## Substitution Rules ##

An expression of the form `(->x|E)A`, where `E` and `A` are arbitrary
expressions, the evaluation of this expression involves the ***substitution***
of the expression `A` for all appropriate *free* instances of the identifier
`x` in the expression `E`. Any bound instances of `x` and any other instances
are left unchanged. 

### Renaming Rule ###

Consider an expression `(->x|(->y|B))A`. If `A` contains any free occurences of
`y`, then a blind substitution into the body of the function will end up
changing those instances of `y` in `A` from free to bound, radically changing
the value of the expression. The solution is to change the formal parameter of
the function and all free occurences of that symbol in the function's body.

We replace all `y` in B by `z` (or any not yet used name `[z/y]B`)

### Conversion Rules, Reduction, Normal Order ###

Two expressions `A` and `B` are the same if there is a formal way of converting
from to other via a series of reductions (`A >> B` means `A` reduces to `B`)

* ***alpha conversion*** corresponds to simple and safe renaming of formal
  identifiers. Two expressions are the same if they differ only in symbol
  names.
* ***beta conversion*** matches normal &lambda; - Calculus function
  applications. Two expressions are the same if one represents the result of
  performing some function application found in the other.
* ***eta conversion*** corresponds to a simple optimization of a function
  application that occurs quite often, it is basically a special case of the
  beta conversion: `(->x|Ex)A >> EA`

When we can no longer apply beta or eta conversions, the expression is in
***normal order*** 

### The Church-Rosses Theorem ###

In 1936 Alonzo Church and J. R. Rosser proved two important theorems.

***Church-Rosses  Theorem I*** states that if an expression `A` through two
different conversion sequences, can reduce to two different expressions `B` and
`C`, there is always some other expression `D` such that both `B` and `C` can
reduce to it. 

***Church-Rosses Theorem II*** states that if an expression `A` reduces to `B`,
and `B` is in normal order (therefore when `A` can be reduced to normal order),
then we can get from `A` to `B` by doing the ***leftmost reduction*** at each
step. This one is important, since it gives us a concrete algorithm for finding
normal order of the expression.


### Order of Evaluation ###

***Normal-order reduction*** follows CRT - it always locates the leftmost
function involved in the application and substitues unchanged copies of
arguments into the function body. No reductions are performed on the arguments
until after.

***Applicative-order reduction*** reduces the argument completely before
function is applied on it.

Normal-order reduction terminates with normal-order expression, but possibly by
evaluating the same expression multiple times. Applicative-order reduction is
not guaranteed to stop when doing only leftmost application.

Example of normal order reduction:

	(->x|(->w|(->y|wyw)b))((->x|xxx)(->x|xxx))((->z|z)a)
	(->w|(->y|wyw)b)((->z|z)a)
	(->y|((->z|z)a)y((->z|z)a))b
	((->z|z)a)b((->z|z)a)     #first redundant application
	(ab((->z|z)a))			  #second redundant application
	(aba)

Example of applicative order reduction:

	(->x|(->w|(->y|wyw)b))((->x|xxx)(->x|xxx))((->z|z)a)
	#evaluation of ((->x|xxx)(->x|xxx))
	([(->x|xxx)/x](xxx))
	((->x|xxx)(->x|xxx)(->x|xxx))    #and will aparently continue forewer

## Basic Arithmetic in &lambda; ##

Mathematically, we have to define how integer 0 will look like, and then the
function `s` which given an integer `k` will produce an expression for the
integer `k+1`.

Because we are using &lambda; - Calculus,  the integer 0 will be represented as a function: 

	0 = (->sz|z)

The definition of the successor function is:

	s(x) = (->xyz|y(xyz))

Some examples to hurt my brain more:

	0 = (->sz|z)
	1 = (->xyz|y(xyz))(->sz|z)
	    (->yz|y((->sz|z)yz))
		(->yz|yz)
	2 = (->xyz|y(xyz))(->sz|sz)
	    (->yz|y((->sz|sz)yz))
		(->yz|y(yz))
	3 = (->xyz|y(xyz))(->sz|s(sz))
		(->yz|y((->sz|s(sz))yz))
		(->yz|y(y(yz)))
	...

### Operations ###

***Addition***:

	(->wzyx|wy(zyx))

***Multiplication***

	(->wzy|w(zy))

Example 2 + 1:

	(->wzyx|wy(zyx))(->sz|s(sz))(->sz|sz)
	(->yx|(->sz|s(sz))y((->sz|sz)yx))
	(->yx|y(y((->sz|sz)yx)))
	(->yx|y(y(yx)))
	3

Example 1 * 2:

	(->wzy|w(zy))(->sz|sz)(->sz|s(sz))
	(->y|(->sz|sz)((->sz|s(sz))y))
	(->y|(->sz|sz)(->z|y(yz)))
	(->y|(->z|(->z|y(yz))z))
	(->y|(->z|y(yz)))
	2

Example 2 * 3

	(->wzy|w(zy))(->sz|s(sz))(->sz|s(s(sz)))
	(->y|(->sz|s(sz))((->sz|s(s(sz)))y))
	(->y|(->sz|s(sz))(->z|y(y(yz))))
	(->y|(->z|(->z|y(y(yz)))((->z|y(y(yz)))z)))
	(->y|(->z|(->z|y(y(yz)))(y(y(yz)))))
	(->y|(->z|y(y(y(y(y(yz)))))))
	6	#uff :)


## Boolean Operations in &lambda; - Calculus ##

	true = T = (->xy|x)
	false = F = (->xy|y)

On contrary to integers, where the internal body matched our concepts of
integers, these functions are used because of the way how they function when
given real arguments. Consider `Q` and `R` are arbitrary expressions:

	if P == T then PQR = T Q R = (->xy|x)QR = Q
	if P == F then PQR = F Q R = (->xy|y)QR = R

Other interesting functions:

	not = (->w|wFT)
	and = (->wz|wzF)
	or  = (->wz|wTz)
	xor = (->wz|w(zFT)(zTF))

	zero(x) = (->x|x F not F)

Examples: 

	not T = (->w|w(->xy|y)(->xy|x))(->xy|x)
			(->xy|x)(->xy|y)(->xy|x)
			(->xy|y)
			F

	and T F = (->wz|wzF)(->xy|x)(->xy|y)
			  ((->xy|x)(->xy|y)F)
			  (->xy|y)
			  F

	zero(1) = (->x|x (->xy|y) (->w|w(->xy|y)(->xy|x)) (->xy|y))(->sz|sz)
			  ((->sz|sz) (->xy|y) (->w|w(->xy|y)(->xy|x)) (->xy|y))
			  ((->xy|y) (->w|w(->xy|y)(->xy|x)) (->xy|y))
			  (->xy|y)
			  F

	zero(0) = (->x|x F not F)(->sz|z)
			  ((->sz|z) F not F)
			  (not F)
			  T

## Recursion in &lambda; - Calculus ##

Consider the application `RA`, where `R` is some recursively defined function
and `A` is some argument expression. If `A` satisfies the ***basis test*** for
`R`, then `RA` reduces to ***basis case***. If not, then `RA` reduces to some
other expression of the form `...(RB)...` where `B` is some simpler expression.
Making `R` recursive has a lot to do with making it ***repeat itself***.

How to do this self-repetition - try to evaluate following expression:

	(->x|xx)(->x|xx)

The expression has the property that it does not change regardless of how many
beta conversions are performed. Using this as a basis, for any lambda
expression `R`:

	((->x|R(xx))(->x|R(xx)))
	R((->x|R(xx))(->x|R(xx)))
	R(R((->x|R(xx))(->x|R(xx))))
	R(R(R((->x|R(xx))(->x|R(xx)))))

This allows us to compose an arbitrary function `R` on itself an infinite
number of times. So to finally tell you the secret - here comes the ***Y
combinator*** (also called fixed point combinator):

	(->y|((->x|y(xx))(->x|(xx))))

What a recursion would it be without the factorial:

	fact(n) = if zero(n) then 1 else n*fact(n-1)

&lambda; - Calculus equivalent:

	fact(n) = Y(->fn|zero n 1 (* n (f (- n 1))))

fact(4):

	R = (->rn|zero n 1 (* n (r (- n 1))))

	fact(4)
	Y R 4
	R (YR) 4
	(->n|zero n 1 (* n (YR (- n 1))))4
	zero 4 1 (* 4 (YR (- 4 1)))
	zero 4 1 (* 4 ((YR) 3))
	F 1 (* 4 ((YR) 3))
	(* 4 ((YR) 3))
	(* 4 (R (YR) 3))
	(* 4 ((->n|zero n 1 (* n (YR (- n 1))))3))
	(* 4 (zero 3 1 (* 3 (YR (- n 1)))))
	(* 4 (* 3 ((YR) 2)))
	(* 4 (* 3 (R (YR) 2)))
	(* 4 (* 3 ((->n|zero n 1 (* n (YR (- n 1))))2)))
	(* 4 (* 3 (zero 2 1 (* 2 (YR (- 2 1))))))
	(* 4 (* 3 (* 2 (R (YR) 1))))
	...
	(* 4 (* 3 (* 2 (* 1 1))))
	...
	24




















