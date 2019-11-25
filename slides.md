# FP & Python

[GothPy](https://www.meetup.com/GothPy/) - Gothenburg's Python Meetup group

Johan Lodin, 2019-11-21

github.com/jolod

## Outline

* Motivating examples
* Challenges applying FP
* What *is* FP?
* State in functional programs
* Principles
  * Abstraction
  * Interpretation
  * Composition
* Tools for FP in Python
* Summary

--

* Slides are written in Markdown.
* Slides are wordy and simple, so that they can be read directly on GitHub.

`github.com/jolod/presentation-fp-python`

# Motivating examples

## Reasoning

```python
x = 3
y = f(x)
z = g(y)
print(x) # 3
```

--

```perl
my $x = 3;
my $y = f($x);
my $z = g($y);
say $x; # ???
```

--

```python
x = [3]
y = f(x)
z = g(y)
print(x[0]) # 3 ???
```

--

```python
x = [3]
y = f(x)
y = g(y)
y = h(y)
y = i(y)
y = j(y)
y = k(y)
y = l(y)
y = m(y)
y = n(y)
y = o(y)
y = p(y)
y = q(y)
print(x[0]) # 3 ???
```

## Compositional APIs

```python
import matplotlib.pyplot as plt

x = [1, 2, 3]
y = [2, 4, 8]

plt.plot(x, y, '-')
plt.show()

plt.plot(x, y, '.-')
plt.show()

plt.subplot(1,2,1)
plt.plot(x, y, '-')
plt.subplot(1,2,2)
plt.plot(x, y, '.-')
plt.show()
```

## Compositional APIs

```R
library("ggplot2")
x <- c(1, 2, 3)
y <- c(2, 4, 8)
p <- ggplot() + aes(x, y)

p

p1 <- p + geom_line()
p1

p2 <- p1 + geom_point()
p2

require(gridExtra)
grid.arrange(p1, p2, ncol=2)
```

## Honorable mention: generators

```python
def fibonacci(n):
    """Compute the nth Fibonacci number."""
    (a, b) = (0, 1)
    for k in range(n):
        (a, b) = (b, a + b)
    return a
```

Consider updating to take the `n`th number divisible by `k` (or some other criterion).

* Write a new function?
* Pass in filter criterion?

--

```python
def fibonacci(n, pred):
    """Compute the nth Fibonacci number that fulfills the criterion pred."""
    if n <= 0:
        return
    (a, b) = (0, 1)
    k = 1 if pred(a) else 0
    while k < n:
        (a, b) = (b, a + b)
        if pred(a):
            k += 1
    return a
```

## Honorable mention: generators

Generators are mutating objects, so not very functional, but serve to *invert the control*.

```python
def fibonacci_gen():
    """Generate the Fibonacci sequence."""
    (a, b) = (0, 1)
    while True:
        yield a
        (a, b) = (b, a + b)
```

```python
def fibonacci(n):
    g = fibonacci_gen()
    skip(n, g)
    return next(g)
```

```python
n = 3
k = 7
g = (a for a in fibonacci_gen() if a % k == 0)
skip(n - 1, g)
print(next(g))
```

## Common selling points

* Easy to reason about / Reduces complexity
* Inherently testable
* Easy to parallelize

--

All this is true for Matlab too!

* Matlab is imperative but with pure functions.
* Matlab has (clever) copy-on-write semantics.

# Challenges applying FP

## Abstraction inversion

https://softwareengineering.stackexchange.com/questions/9006/whats-your-strongest-opinion-against-functional-programming

--

Answer by Mason Wheeler, 2010-10-02: (bold emphasis added by me)

> I think that the reason functional programming isn't used very widely is because it gets in your way too much. It's hard to take a serious look at, for example, Lisp or Haskell, without saying "this whole language is **one big abstraction inversion**."

--

> When you establish baseline abstractions that the coder can't get beneath when necessary, you establish things that **the language simply can't do**, and the more functional the language is, the more of these it tends to have.

--

> Take Haskell, for example. In the name of functional purity, you're required to use **brain-breaking abstraction inversions that nobody understands** in order to manage state and I/O, *the two most fundamental parts of any and every computer program that interacts with anything!* That gets old fast.

## "Brain breaking"?

* Often brain breaking because you've *learned a different way already*.
* 20 years ago some companies did not allow `map` in the code base because "the next programmer wouldn't understand it".

What's brain breaking today, might very well be obvious tomorrow.

## The learning resources gap

A large portion of articles write about functional programming in the small.

* Map, filter, reduce.
* Partial functions and currying.

--

A small portion of articles write about functional programming in the large.

* IO
* Modularity
* Extensibility
* Testing

--

![fail-vault]

## The learning resources gap

A large portion of articles write about functional programming in the small.

* Map, filter, reduce.
* Partial functions and currying.

A small portion of articles write about functional programming in the large.

* IO
* Modularity
* Extensibility
* Testing

![awesome-vault]

# What do I mean by FP?

Not just immutable data and higher-order functions

Structure vs techniques

## The IP-FP spectrum

Extreme imperative programming (IP):

> A *Turing machine* is a mathematical model of computation that defines an abstract machine, which manipulates symbols on a strip of tape according to a table of rules. Despite the model's simplicity, given any computer algorithm, a Turing machine capable of simulating that algorithm's logic can be constructed. (1936)

Easy to imagine how a physical Turing machine affects the world.

http://turingmachine.io/

--

You don't program Turing machines:

* Register machines (assembly; gotos)
* Structured programming (statements, while, if)
* Procedural programming (blocks, scopes, return)

## The IP-FP spectrum

Extreme functional programming (FP):

> *Lambda calculus* is a formal system in mathematical logic for expressing computation based on **function abstraction** and **application** using variable binding and substitution. It is a universal model of computation that can be used to simulate any Turing machine. (1930s)

--

[Visualization](https://www.cl.cam.ac.uk/~rmk35/lambda_calculus/lambda_calculus.html) of 2 + 3.

```text
((fn n m => fn f x => n f (m f x)) (fn f => fn x => f (f x)) (fn f => fn x => f (f (f x))))
```

--

Not realized to be a "programming language" until around 1960.

## Church encoding examples

```python
TRUE = lambda then: lambda _: then
FALSE = lambda _: lambda otherwise: otherwise
IF = lambda cond: lambda then: lambda otherwise: cond(then)(otherwise) # Redundant
AND = lambda p: lambda q: p(q)(p)

TWO = lambda f: lambda x: f(f(x))
FIVE = lambda f: lambda x: f(f(f(f(f(x)))))
INC = lambda n: lambda f: lambda x: f(n(f)(x))
THREE = INC(TWO)
PLUS = lambda n: n(INC)
DEC = lambda n: lambda f: lambda x: n(lambda g: lambda h: h(g(f)))(lambda _: x)(lambda y: y)
MINUS = lambda m: lambda n: n(DEC)(m)
ISZERO = lambda n: n(lambda x: FALSE)(TRUE)
EQ = lambda m: lambda n: AND(ISZERO(MINUS(m)(n)))(ISZERO(MINUS(n)(m)))

PAIR = lambda a: lambda b: lambda selector: selector(a)(b)
FIRST = lambda pair: pair(lambda a: lambda _: a)
SECOND = lambda pair: pair(lambda _: lambda b: b)

RESULT = EQ(PLUS(TWO)(THREE))(FIVE)
```

```python
outputs = PAIR("It's true!")("I think there is a bug somewhere.")
print(
    IF(RESULT) \
        (FIRST(outputs)) \
        (SECOND(outputs))
)
```

```text
It's true!
```

## The point: functions are "all you need"

Data:

* Numerals => chars
* Variants (pairs) => lists
* Chars and lists => strings
* Lists and pairs => dictionaries
* Dictionaries and lambdas => prototypal OO
* **Etc!**

Logic:

* Variants for branching
* Recursion for looping
* **Etc!**

Pure λ-calculus isn't exactly "ergonomic" though.

## Characteristics of λ-calculus

`Data == functions               (because only functions)`

--

`Control flow == data            (because only functions)`

--

`Higher order functions          (because only functions)`

--

`Only one-argument functions     (reductionist, but practical!)`

--

`Non-strict evaluation           (for technical reasons)`

(You don't actually *evaluate* λ-calculus, you *reduce* terms.)

--

`No mutable data                 (makes no sense)`

--

`No statements, only expressions (makes no sense)`

--

`No side effects of any kind     (makes no sense)`

--

Not here:

* Pattern matching
  * Variants very important though!
* Types
  * Can carry information!

# State

"There is no *shared mutable* state in FP"

## "Imperative" vs functional state

--

Raise your hand when you are *certain* of what this does:

--

```python
def foo(n, k=0):
  result = 0
  while True:
    if k >= n:
      break
    result += k
    k += 1
  return result
```

--

What is `foo(5, 3)`?

--

```python
def foo(n, k=0):
  result = 0
  for m in range(k, n):
    result += m
  return result
```

* Which is easier to understand?
* Why?

## "Imperative" vs functional state

```python
def range_sum(n, k=0):
  result = 0
  while True:
    if k >= n:
      break
    result += k
    k += 1
  return result
```

* What happens if we do `k += 1` before `result += k`?

--

```python
def range_sum(n, k=0, result=0):
  if k >= n:
    return result
  else:
    return range_sum(n, k + 1, result + k)
```

State moved from loop variables to function arguments.

--

* `k + 1` and `result + k` are unordered.
* Still a "manual" loop though.
* Doesn't sell FP on anyone.

## "Imperative" vs functional state

```python
def range_sum(n, k=0):
  result = 0
  for m in range(k, n):
    result = result + m
  return result
```

--

Another "abstraction inversion":

```python
def range_sum(n, k=0):
  return reduce(
    lambda result, m: result + m,
    range(k, n),
    0
  )
```

* 1-to-1 with the imperative version.
* But even more structure.

## "Imperative" vs functional state

FP has "local" state just as IP does:

* General recursion is the "least structured" way.
* Often the most structured way is favored.
  * Just like `for` is favored over `while`.

Key difference: *all* state at *every level* is local state. (Except the top level, where IO is allowed.)

--

Yet, at this scale, almost no benefit regarding

* reasonability,
* testability, or
* parallelism.

# Some core FP principles

Abstraction, interpretation & composition

# Abstraction

## Higher-order functions

* `reduce` abstracts over *structure*.

--

* `reduce` is a function and can be *abstracted over*.
  * Cannot abstract over `for` directly.

--

```python
def range_sum(reducer, n, k=0):
    return reducer(operator.add, range(k, n), 0)
```

* `range_sum` is now a *third*-order function (takes a second-order function).

## Inversion of Control (IoC)

* Higher-order functions (HOF) are a form of dependency inversion (DI).
* Too high order might hint at mixing high and low level abstractions.

```python
def range_sum(summer, n, k=0):
    return summer(range(k, n))
```

* Reduced to second-order again.

## Building computations

Functions as values.

```python
def range_sum(n, k=0):
    return lambda summer: summer(range(k, n))
```

--

Can talk about range sums without talking about how to sum it (yet).

```python
def plus(a, b):
    return lambda summer: summer(n(summer) for n in [a, b])
```

Very light-weight "pattern".

(This is why it is practical to have one-argument functions only (or "curried" functions); you don't have to be explicit about whether `summer` is the third argument of `plus`, or an argument of the return value.)

--

```python
total = plus(range_sum(5, 1), range_sum(3, 1))

mysum = lambda xs: reduce(operator.add, xs, 0)
print(total(mysum)) # 1+2+3+4 + 1+2 = 13

myprod = lambda xs: reduce(operator.mul, xs, 1)
print(total(myprod)) # 1*2*3*4 * 1*2 = 48

print(total(lambda xs: xs)) # [range(1, 5), range(1, 3)]
```

The lack of (visible) argument names for the closures can make this style harder to read.

# Interpretation

Inversion of control *flow*

## Interpretation vs dependency injection

* No side effects.
* Turn a value into another value.
* Turn a structure into another structure.

--

Point of view: all values are structures.

* Lists (etc) are structures (interpreted using `for`/`reduce`).
* Records/objects are structures (interpreted using attribute lookup).
* Functions are structures (interpreted by calling them).
* Booleans are structures (interpreted using `if`).

Booleans are the simplest example of a *variant data type*.

## Variant data types

Cannot rely on only passing in behavior.

* Want side effects close to the top level.
* Must communicate actions through return values.

--

* Variants model branching, in contrast to aggregations.
* Python (and most OO languages) do not support branching on variants syntactically (except booleans).
* Variants ~~ visitor pattern.
  * Visitor implementations is rare in OOP, but variants are *everywhere* in FP.

Remember: booleans are variants.

## Booleans and beyond

```haskell
-- Haskell, sorry. :-(

data Boolean = True | False

case result of
  True -> ...
  False -> ...
```

--

```haskell
-- Explicitly encoding possibility of None.

data Maybe a = Just a | Nothing

case result of
  Just x -> ... -- Make good use of x.
  Nothing -> ... -- Do thing else.
```

* `Just` is like `True` but carries a value.
* `Nothing` is like `False`. (Or vice versa.)

--

```haskell
-- Handle exceptions etc.
data Result error value = Failure error | Success value
```

Variants can have any number of cases, and store any number of values.

## Variants in Python

```python
def f(result):
    if result.tag == "success":
        value = result.value
        return ...
    elif result.tag == "failure":
        error = result.error
        return ...
    else:
        raise Exception("Unknown tag: %s" % (result.tag,))
```

This might look upsetting to some.

Remember: booleans are variants!

## How to *not* use variants

```haskell
data Shape a = Circle a
             | Ellipse a a
             | Square a
```

This will lead to pain down the line.

--

Use variants when

* The cases aren't meaningful *by themselves*.
* **And** the cases are fundamental/primitives.
  * Fundamental means that you don't want to add new cases.
  * If you *do* add cases you *want* to update all the code using them.


--

Booleans are good:

* True is only meaningful as a contrast to false (and vice versa).

--

`Maybe` is good:

* `Just` is only meaningful in the presence of `Nothing`.

--

`Shape` above is bad:

* Circles are useful by themselves.
* Might want to add more shapes (e.g. `Rectangle`)

## Variants in Python redux

Booleans:

```python
TRUE = lambda then, otherwise: then()
FALSE = lambda then, otherwise: otherwise()

print(TRUE(lambda: 42, lambda: "Dunno"))  # 42
print(FALSE(lambda: 42, lambda: "Dunno")) # Dunno
```

Very light-weight compared to the visitor pattern.

See one of the exercises.

# Composition

## Abstraction + interpretation

Abstraction and interpretation work in tandem to
* focus libraries on the core problem, and
* push side effects to the edge.

Libraries > frameworks w.r.t. composability

Libraries are built using other libraries.
Frameworks are built using ...?

## Acting on results

"Naïve" variants only take you so far.

What if you want to act on the result of an action that requires interpretation?

For instance:

```python
# Ignore exceptions for now.
import os

def count_chars_of_file(filename):
    f = os.open(filename)
    text = os.read(f)
    n = len(text)
    os.close(f)
    return n
```

--

(The answer is *not* monads in Python.)

## Acting on results

Made-up node.js library:

```javascript
function countCharsOfFile(filename) {
    return action("open", filename).then((f) => {
          return action("read", f).then((text) => {
              let n = text.length();
              return action("close", f).then(() => {
                  return result(n);
              })
          })
    })
}
```

Builds computations stored in variants (`action` and `result`).

## Acting on results

In Python:

```javascript
def count_chars_of_file(filename):
    def after_open(f):
        def after_read(text):
            n = len(text)
            def after_close():
                return Result(n)
            return Action("close", f).then(after_close)
        return Action("read", f).then(after_read)
    return Action("open", filename).then(after_open)
```

Not pretty. :-(

But see one of the exercises nonetheless.

## Coroutines

(Credit: Magnus Therning, https://magnus.therning.org/posts/2017-01-31-000-on-mocks-and-stubs-in-python--free-monad-or-interpreter-pattern-.html)

```python
os = Actions('os')

def count_chars_of_file(filename):
    f = yield os.open(filename)
    text = yield os.read(f)
    n = len(text)
    yield os.close(fd)
    return n
```

* `os` uses dynamic lookup to return the method name and arguments as data instead of performing the action.
* The consumer of `count_chars_of_file` uses the `send` method to communicate the results back.
* `... = yield from ...` allows refactoring.

--

* The caller makes the interpretation, or passes on the description.
* Depend on globally available *descriptions* (i.e. data).

See one of the exercises.

# Tools for FP in Python

## Libraries

* [functools]
* [operator]
* [pyrsistent]

Honorable mention:

* [itertools]

## Syntax

* lambda expressions: `lambda ...: ...`
* if expressions: `... if ... else ...`
* decorators

Honorable metion:

* `yield` (generators and coroutines)

## Issue: scope of `for` variables

```python
fs = []
for n in range(5):
    fs.append(lambda: n)

for f in fs:
  print(f())
```

```text
4
4
4
4
4
```

--

Sneaky bug:

```python
n = 2
k = 13
g = (a for a in fibonacci_gen() if (a % k) == 0 and a > 0)
for k in range(1, n):
    next(g)
print(next(g))
```

## Issue: no statements in lambdas

* `if` is doable as expression.
* `for` is doable as expression.
* `=` is the biggest problem.

```python
def LET(x, f):
    return f(x)

print(
    LET(3, lambda x:
    LET(x + 1, lambda y:
        x + y))
)
```

```text
7
```

Don't do this please. :-)

But see one of the exercises.

## Issue: no tuple unpacking in lambdas

PEP 3113 removed tuple destructuring. The "solution":

```python
def fxn(a, b_c, d):
    b, c = b_c
    return a + b + c + d
```

Doesn't work for lambdas.

```python
fnx = lambda a, b_c, d: LET(b_c[0]: lambda b: LET(b_c[1], lambda c: a + b + c + d))
```

:-(

# Summary

**Modularity via interpretation**

## Description & interpretation

* Plotting
* Generators
* Data in λ-calculus
* Build computations
* Variant data types
* Mix data and computations

All came together to push side effects to the very edge.

# Exercises

github.com/jolod/presentation-fp-python

[fail-vault]: images/fail-vault.gif
[awesome-vault]: images/awesome-vault.gif
[pyrsistent]: https://pypi.org/project/pyrsistent/
[itertools]: https://docs.python.org/3.8/library/itertools.html
[functools]: https://docs.python.org/3.8/library/functools.html
[operator]: https://docs.python.org/3.8/library/operator.html
