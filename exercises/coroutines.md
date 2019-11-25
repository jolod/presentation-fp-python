# Coroutines and "IO interpreters"

## Credit

This code is originally an adaptation of Magnus Therning's code at https://magnus.therning.org/posts/2017-01-31-000-on-mocks-and-stubs-in-python--free-monad-or-interpreter-pattern-.html. In particular, I added the `Actions` class to make it almost effortless to change code from "direct" to "interpreted" style.

Magnus' code was:

```python
Op = collections.namedtuple('Op', ['op', 'args'])

def gen_open(fn):
    return Op('open', [fn])

def gen_read(fd):
    return Op('read', [fd])

def gen_close(fd):
    return Op('close', [fd])

def count_chars_of_file(fn):
    fd = yield gen_open(fn)
    text = yield gen_read(fd)
    n = len(text)
    yield gen_close(fd)
    return n
```

and my original adaptation can be found at https://gist.github.com/jolod/982f66beb423bd9f998d05c5c5f2fbc0.

## Prelude

Start with

```python
import collections
import types

Action = collections.namedtuple('Action', ['qualifier', 'name', 'args', 'kwargs'])

class Actions:
    def __init__(self, namespace):
        self.__namespace = namespace

    def __getattribute__(self, attr):
        cls = type(self)
        if attr.startswith('_' + cls.__name__ + '__'):
            return super(cls, self).__getattribute__(attr)
        else:
            return lambda *args, **kwargs: Action(self.__namespace, attr, args, kwargs)

def run_action(action, handlers):
    base = handlers.get(action.qualifier, None)
    if not base:
        raise Exception("No handlers for qualifier: " + action.qualifier)
    action_name = action.name
    try:
        f = base.__getattribute__(action_name)
    except:
        raise Exception("No handler for {!s}.{!s}".format(action.qualifier, action_name))
    return f(*action.args, **action.kwargs)

def run_program(prog, **actions):
    try:
        action = next(prog)
        while True:
            assert isinstance(action, Action)
            action = prog.send(run_action(action, actions))
    except StopIteration as e:
        return e.value
```

(You don't need to understand how `Actions` and `run_action` works to continue with the exercises.)

Define a "program" with

```python
os = Actions('os')

def count_chars_of_file(fn):
    fd = yield os.open(fn)
    text = yield os.read(fd)
    n = len(text)
    yield os.close(fd)
    return n

filename = ...
program = lambda: count_chars_of_file(filename)
```

Run the program with

```python
class MyOs:
    @staticmethod
    def open(filename):
        return open(filename)

    @staticmethod
    def read(f):
        return f.read()

    @staticmethod
    def close(f):
        f.close()

output = run_program(program(), os = MyOs())
print(output)
```

and check that it works.

## Optional: refactor `count_chars_of_file`

Refactor `count_chars_of_file` to make use of

```python
def read_and_count(fd):
    text = yield os.read(fd)
    n = len(text)
    return n
```

(Hint: use `yield from`.)

## Create a `FakeOs` class/interpreter

Create a `FakeOs` class so that files always contain their filenames. Test it with:

```python
output = run_program(program(), os = FakeOs())
print(output == len(filename)) # True
```

## Create a `run_programs_async` function

```python
console = Actions('console')

def hello(name):
    yield console.println("Hello " + name)
    yield console.println("Goodbye " + name)

programs = [
    hello("World"),
    hello("GothPy"),
]

class Console:
    @staticmethod
    def println(s):
        print(s)

run_programs_async(programs, console = Console())
```

```text
Hello World
Hello GothPy
Goodbye World
Goodbye GothPy
```

## Use a generator interpreter to inspect the performed actions

Instead of running the program, let's rewrite `run_program` as `interpret_program` and make it into a generator which yields the actions and the action's return values.

```python
for (action, result) in interpret_program(hello("GothPy"), console = Console()):
    print("Action: " + str(action))
    print("Result: " + str(result))
    print()
```

```text
Hello GothPy
Action: Action(qualifier='console', name='println', args=('Hello GothPy',), kwargs={})
Result: None

Goodbye GothPy
Action: Action(qualifier='console', name='println', args=('Goodbye GothPy',), kwargs={})
Result: None
```

Now, use `interleave_generators`,

```python
import itertools
def interleave_generators(*generators):
    return itertools.zip_longest(*generators, fillvalue = None)
```

to run two or more programs concurrently:

```python
programs = [
    hello("World"),
    hello("GothPy"),
]
interpreters = (interpret_program(program, console = Console()) for program in programs)
for step in list(interleave_interpreters(interpreters)):
    print(step)
```

```text
Hello World
Hello GothPy
Goodbye World
Goodbye GothPy
((Action(qualifier='console', name='println', args=('Hello World',), kwargs={}), None),
 (Action(qualifier='console', name='println', args=('Hello GothPy',), kwargs={}), None))
((Action(qualifier='console', name='println', args=('Goodbye World',), kwargs={}), None),
 (Action(qualifier='console', name='println', args=('Goodbye GothPy',), kwargs={}), None))
```
