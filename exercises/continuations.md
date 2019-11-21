# Interpreted continuations instead of coroutines

Start with

```python
class Continuation:
    def __init__(self, computation, cont):
        self.computation = computation
        self.cont = cont

class Action:
    def __init__(self, tag, *args):
        self.tag = tag
        self.args = args

class Result:
    def __init__(self, value):
        self.value = value

for cls in [Action, Result, Continuation]:
    cls.then = lambda self, then: Continuation(self, then)

def run(c):
    while isinstance(c, Continuation):
        v = c.computation
        if isinstance(v, Action):
            if v.tag == 'open':
                c = c.cont(open(*v.args))
            elif v.tag == 'read':
                (f, *args) = v.args
                c = c.cont(f.read(*args))
            elif v.tag == 'close':
                (f, *args) = v.args
                f.close(*args)
                c = c.cont()
            else:
                raise Exception(v.tag)
        elif isinstance(v, Result):
            c = c.cont(v.value)
        elif isinstance(v, Continuation):
            c = run(v)
        else:
            raise Exception(v)
    return c
```

and write the function

```python
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

Test `count_chars_of_file` on some file using

```python
filename = ...
print(run(count_chars_of_file(filename)).value)
```

## Implement `fake_run`

Implement `fake_run` so that all files seem to contain their file names. Test it with

```python
run_fake(count_chars_of_file(filename)).value == len(filename)
```

## Refactor `count_chars_of_file` to use `with_file` and `read_and_count`

Refactor `count_chars_of_file` so that you have a function `read_and_count`,

```python
def read_and_count(f):
    def after_read(text):
        n = len(text)
        return Result(n)
    return Action("read", f).then(after_read)
```

and create `with_file`,

```python
def with_file(filename, block):
    ...
```

so that the following code is true:

```python
run_fake(with_file(filename, read_and_count)).value == len(filename)
```
