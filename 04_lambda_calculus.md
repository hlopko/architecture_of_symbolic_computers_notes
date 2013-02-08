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

















