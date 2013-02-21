# a formal basis for abstract programming #

to make &lambda; - calculus easier to use for humans, the notation called
***abstract programming*** has been developed. it is much easier to read and
still completely convertible to pure &lambda; - calculus. 

## syntax ##

	<identifier>		:= <alpha-char>{<alpha-char>|<number>}*
	<function-name> 	:= <identifier>
	<constant>      	:= <number> | <boolean> | <char-string>
	<expression>    	:= <constant> 
							| <identifier> 
							| (-><identifier> "|" <expression)
							| (<expression>+)
							| <function-name>(<expression>{,<expression>}*
							| let <definition> in <body>
							| letrec <definition> in <body>
							| <body> where <definition>
							| <body> whererec <definition>
							| if <expression> 
								then <expression> 
								else <expression>
							| ... # standard arithmetic expressions etc

	<body>				:= <expression>
	<definition>		:= <header> = <expression>
							| <definition> {and <definition}*
	<header>			:= <identifier>
							| <function-name>(<identifier>{,<identifier>}*)
	<abstract-program>	:= <expression>

example:

	letrec fact = (->n|if n == 0 then 1 else n * fact(n - 1)) in fact(4)

	fact(4) whererec fact(n) = if n == 0 then 1 else n * fact(n - 1)

	letrec fact(n) = ((if n = 0 then 1 else z)
						whererec z = n * fact(n - 1))
		in (fact(4))


## constants ##

semantically, a ***constant*** is any object whose name denotes its value
directly. in abstract programming we assume we have syntax for constants of
types ***boolean, integer*** and ***character string***. therefore any
occurences of `t` or `f` are taken as expressions `(->xy|x)` and `(->xy|y)`.
similarly numbers are translated to the form `(->sz|s^k z)`, negative numbers
can be expressed like `(->n |n - k)` where `k` is their positive equivalent.
character string will be ascii encoded.

## function applications ##

two possibilities for function application exist:

	f(a,b,c,d...) = (f a b c d ...)

where `f` is defined by enclosing `let`, `letrec` etc. expressions.

## conditionals ##

small difference between abstract conditional and the conventional conditionals
in programming languages it, that conventional languages do not define what
happens when `p` (predicate) is not `t` or `f`. in abstract programming and
&lambda; - calculus, two expressions are accepted as arguments to whatever `p`
reduces to. also, `else` is not optional in abstract programming, the result
would be curried function, that causes havoc to further processing.

## local definitions ##

abstract programming has a notation very akin to a cleaned-up combination of
macros and call-by-name. the simplest form of this notation is:

	let <identifier> = <expression> in <body>

this whole expression is equivalent in value to a copy of the `<body>` where every
free occurence of the `<identifier>` is replaced by the `<expression>`.
expressed by &lambda; - calculus:

	let x = a in e

is the same as:

	(->x|e)a


## recursive definitions ##

one limitation of the `let` and `where` expressions is that they do not permit
recursion in a function definition. a brute-force solution is to use **y
combinator** defined earlier. or one can use `letrec`. the major difference is
that any free occurence of the definition's identifier in the deinition's
expression is replaced by the expression itself. thus in:

	letrec f(n) = if zero(n) then 1 else n*f(n-1) in f(4)

the free occurences of `f` in `f(n-1)` is replaced recursively by the whole
expression `if zero(n) then 1 else n*f(n-1) in f(4)`. 

the conversion to the pure &lambda; - calculus is direct.  given `letrec f=a in
e`, we form an application where the function is an anonymous function whose
formal parameter is `f` and whose body is `e`. instead of using `a` as an
argument, we create `(->f|a)` and use that as an argument to the function `y`.
therefore pure &lambda; - calculus equivalent of:

	letrec f = a in e

is
	
	(->f|e) (y (->f|a)) = (->f|e) ((->y|(->x|y(xx))(->x|y(xx))) (->f|a))

there are some subtle differences between multiple definitions in the ***and
form*** of a `letrec` versus a `let`. in the `letrec` form, the expression in
each anded definition has complete access to every other definition. the result
is a set fo ***mutually recursive functions*** that are defined together. and
this is challenging, we will need y1 and y2 combinators.

## higher-order functions ##

are functions which accept other functions as an argument (we are ignoring
functions accepting integer, string etc. arguments, althrough they are
functions too).

***map*** takes as its arguments some arbitrary function and a list of objects.
the result is equally long list of objects returned by application of the
function to the k-th element of the input list

	map(f,x) = 	if null(x) then nil
				else cons(f(car(x)), map(f, cdr(x)))


***map2*** is identical, except that it takes 2 equally long lists and applies
function to matching elements.

	map2(f,x,y) = if null(x) then nill
					else cons(f(car(x),car(y)), map2(f, cdr(x), cdr(y)))

***reduce*** takes a function, an object, and a list. the value returned is
result of applying the function to the first element of the list and the result
of applying ***reduce*** to the rest of the list. if the list is empty, an
object (the second argument to the ***reduce***) is returned. 

	reduce(f,t,x) = if null(x) then t
					else f(car(x), reduce(f,t, cdr(x)))

***vector*** takes a list of functions and an object and returns the list of
the applications of functions in the list to the object.

	vector(o,x) = if null(x) then nil
					else cons(car(x)o,vector(o, cdr(x)))					

***while*** takes two functions and an accumulating parameter. as long as the
application of the first function to the accumulating parameter returns `T`,
***while*** recursively calls itself with the accumulating parameter modified
by applying second function parameter to it. 

	while(p,f,x) = if p(x) then while(p,f,f(x))
					else x

***compose*** takes two functions and returns a new function which is the
composition of the two.

	compose(f,g) = (->x|f(g(x)))


## Exercises ##

1. Write as an abstract program the predicate ***free***(x,e), which returns
   true if the identifier occurs free in e. 

		free(x,e) = if null(e) 
					then F
					elseif is-id(e)
					then if x = e
						 then T
						 else F
					elseif is-lambda(e) then 
							if get-lambda-arg(e) = x 
							then F
							else free(x, get-body(e))
					else or(free(x, get-function(e)),
							free(x, get-argument(e)))


2. Write an abstract definition for a function ***subs***(y,x,m) where `y` and
   `m` are any valid s-expressions and x is an atom representing variable name.
   The result is s-expression equivalent to `[y/x]m`.

		subs(y,x,m) = if null(m)
					  then nil
					  elseif is-id(m)
					  then if m = x
						   then y
						   else m
					  elseif is-lambda(m)
					  then let a = get-lambda-arg(m)
						   and b = get-body(m)
						   and f = free(a,y)
						   in if a = x
						      then m
						      elseif f
							  then let z = new-id()
							       in create-function(z, subs(y,z,subs(z,y,get-body(m))))
							  else create-function(get-arg(m), subs(y,x,get-body(m)))
					  else create-application(subs(y,x,get-function(m)),
											  subs(y,x,get-argument(m)))
					
						
							     





