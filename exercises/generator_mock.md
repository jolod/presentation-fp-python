# Generators instead of mocks

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

def runAction(action, actions):
    base = actions.get(action.qualifier, None)
    if not base:
        raise Exception("No handlers for qualifier: " + action.qualifier)
    action_name = action.name
    try:
        f = base.__getattribute__(action_name)
    except:
        raise Exception("No handler for {!s}.{!s}".format(action.qualifier, action_name))
    return f(*action.args, **action.kwargs)

def runProgram(prog, **actions):
    try:
        action = next(prog)
        while True:
            if isinstance(action, Action):
                action = prog.send(runAction(action, actions))
            else:
                raise Exception(action)
    except StopIteration as e:
        return e.value
```

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

output = runProgram(program(), os = MyOs())
print(output)
```

and check that it works.

## Refactor `count_chars_of_file`

Refactor `count_chars_of_file` to make use of

```python
def read_and_count(fd):
    text = yield os.read(fd)
    n = len(text)
    return n
```

## Create a `FakeOs` class/interpreter

Create a `FakeOs` class so that files always contain their filenames. Test it with:

```python
output = runProgram(program(), os = FakeOs())
print(output == len(filename)) # True
```
