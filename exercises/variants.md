# Exercises on variants

These exercises will hopefully make you comfortable using lambdas and thinking in terms of functions.

The last exercise, error handling, shows how you can express control flow using functions.

## Case expressions
The simplest variant data type are booleans. We can encode them in Python as

```python
TRUE  = lambda then, otherwise: then()
FALSE = lambda then, otherwise: otherwise()
```

We already have booleans, so there is little point in reinventing them. However, we want to use `if` as an expression in lambdas. We have `... if ... else ...`, but we can create an alternative syntax:

```python
IF(..., lambda: ..., lambda : ...)
```

e.g.

```python
abs = lambda x: IF(
    x >= 0,
    then = lambda: x,
    otherwise = lambda: -x,
)
```

Define `IF` and use it to create a `case` function:

```python
def sign(x):
    return case(
        lambda: x < 0,
        lambda: "-",

        lambda: x > 0,
        lambda: "+",

        lambda: "0",
    )

print(sign(-10)) # -
print(sign(10))  # +
print(sign(0))   # 0
```

## `None` handling

We can create a function `let`, which encodes "assignment as expression":

```python
def let(x, f):
    return f(x)

print(
    let(3, lambda x:
    let(5, lambda y:
        x + y))
) # 8
```

Write a new version of `let` which automatically stop a computation if any "assignment" contains `None`.

```python
print(
    let(None, lambda x:
    let(5, lambda y:
        x + y))
) # None
```

## Error handling

Like `None` handling, write a `let` which short-cirsuits on errors:

```python
def TODO():
    raise Exception('TODO')

success = lambda value: lambda success, failure: success(value)
failure = lambda error: lambda success, failure: failure(error)

def to_string(result):
    return TODO()

def divide(x, y):
    if y == 0:
        return failure('Division by zero: {}/{}'.format(x, y))
    else:
        return success(x / y)

def let_result(result, f):
    return TODO()

def f(a):
    return let_result(divide(1, a), lambda a_inv:
           let_result(success(a_inv - a), lambda diff:
               divide(diff + 3, diff)))

print(to_string(f(0))) # Error: Division by zero: 1/0
print(to_string(f(1))) # Error: Division by zero: 2/0.0
print(to_string(f(2))) # Success: -1
```

`f(0)` fails in the first `divide` and `f(1)` fails in the second `divide`.

In Haskell `f` would be written as

```haskell
f a = do
    aInv <- divide 1 a
    let diff = aInv - a -- or: diff <- pure (aInv - a)
    divide (diff + 3) diff
```

where `<-` is the "special let" and `let` is normal assignment. Haskell automatically determines which special let to use, but it is restricted to use the same special let in the same `do` block.
