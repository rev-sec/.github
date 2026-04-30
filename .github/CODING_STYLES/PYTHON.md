# Python Style Guide

---
This is still under construction.

All source code should be formatted using [Black](https://github.com/psf/black) with a
line length of 88. This line length is already configured in [pyproject.toml](pyproject.toml). The
following guide is adapted from the [Google](https://google.github.io/styleguide/pyguide.html) style guide.

## 1 Language Rules

---

### 1.1 Importing

Use `import` statements for packages and modules only, not for individual types,
classes, or functions. This provides a namespace for the module contents and avoids
naming conflicts.

+ Use `import` for importing packages and modules.
+ Use `from package-name import module-name` when the package name is inconveniently
  long and the module name provides a distinct namespace.
+ Use `from package-name import module-name as alias` in any of the following
  circumstances:
    + Two modules with the same name are to be imported.
    + The module name conflicts with another top-level name in the current module.
    + The module name conflicts with a common parameter name that is part of the public API.
    + The module name is inconveniently long.
    + The module name is too generic in the context of the code
      (e.g. `from storage.file_system import options as fs_options`).
+ Use `import module-name as alias` only when the alias is a standard abbreviation
  (e.g. `import pandas as pd` or `import numpy as np`).
+ Do not use relative names in imports even if the module is in the same package. Always
  use the full package name.
+ Symbols from the following modules may be imported individually:
    + `typing` module.
    + `collections.abc` module.

### 1.2 Exceptions

Exceptions are allowed but must be used carefully as they can cause control flow become
confusing and hard to debug.

+ Make use of built-in exception classes when it makes sense. For example, raise a
  `ValueError` to indicate a programming mistake like a violated precondition, such as may
  happen when validating function arguments.
+ Do not use assert statements in place of conditionals or validating preconditions.
  They must not be critical to the application logic. A litmus test would be that the
  `assert` could be removed without breaking the code. `assert` conditionals are not
  guaranteed to be evaluated. For example:

```python
def connect_to_next_port(self, minimum: int) -> int:
    """Connects to the next available port.

    Args:
      minimum: A port value greater or equal to 1024.

    Returns:
      The new minimum port.

    Raises:
      ConnectionError: If no available port is found.
    """
    if minimum < 1024:
      # Note that this raising of ValueError is not mentioned in the doc
      # string's "Raises:" section because it is an unchecked exception 
      # related to misuse of the API (logic error) as opposed to an external 
      # problem (runtime error).
      raise ValueError(f'Min. port must be at least 1024, not {minimum}.')
    port = self._find_next_open_port(minimum)
    if port is None:
      raise ConnectionError(
          f'Could not connect to service on port {minimum} or higher.')
    # The code does not depend on the result of this assert.
    assert port >= minimum, (
        f'Unexpected port {port} when minimum was {minimum}.')
    return port
```

Not acceptable

```python
def connect_to_next_port(self, minimum: int) -> int:
    """Connects to the next available port.

    Args:
      minimum: A port value greater or equal to 1024.

    Returns:
      The new minimum port.
    """
    assert minimum >= 1024, 'Minimum port must be at least 1024.'
    # The following code depends on the previous assert.
    port = self._find_next_open_port(minimum)
    assert port is not None
    # The type checking of the return statement relies on the assert.
    return port
```

+ Libraries or packages may define their own exceptions. When doing so they must inherit
  from an existing exception class. Exception names should end in Error and should not
  introduce repetition (`foo.FooError`).
+ Never use catch-all except: statements, or catch Exception or StandardError, unless
  you are
    + re-raising the exception, or
    + creating an isolation point in the program where exceptions are not propagated but
    + are recorded and suppressed instead, such as protecting a thread from crashing by
    + guarding its outermost block.

  Python is very tolerant in this regard and `except:` will really catch everything
  including misspelled names, sys.exit() calls, Ctrl+C interrupts, unittest failures and
  all kinds of other exceptions that you simply don’t want to catch.
+ Minimise the amount of code in a `try`/`except` block. The larger the body of the
  `try`, the more likely that an exception will be raised by a line of code that you
  didn’t expect to raise an exception. In those cases, the `try`/`except` block hides a
  real error.
+ Use the `finally` clause to execute code whether an exception is raised in the
  `try` block. This is often useful for clean-up, i.e., closing a file.

### 1.3 Mutable Global State

Avoid mutable global state as it breaks encapsulation and make unit testing difficult.
In those rare cases where using global state is warranted, mutable global entities
should be declared at the module level or as a class attribute and made internal by
prepending an `_` to the name. If necessary, external access to mutable global state
must be done through public functions or class methods. See [Naming]() below. Please explain
the design reasons why mutable global state is being used in a comment or a doc linked
to from a comment.

Module-level constants are permitted and encouraged. Constants must be named using all
caps with underscores. See [Naming]() below.

### 1.4 Nested/Local/Inner Classes and Functions

A class can be defined inside a method, function, or class. A function can be defined
inside a method or function. Nested functions have read-only access to variables defined
in enclosing scopes. Commonly used for implementing decorators, however they cannot be
directly tested and make the outer function longer and less readable.

They are fine with some caveats. Avoid nested functions or classes except when closing
over a local value other than `self` or `cls`. Do not nest a function just to hide it
from users of a module. Instead, prefix its name with an `_` at the module level so that
it can still be accessed by tests.

### 1.5 Comprehensions & Generator Expressions

List, Dict, and Set comprehensions as well as generator expressions provide a concise
and efficient way to create container types and iterators without resorting to the use
of traditional loops, `map()`, `filter()`, or `lambda`.

Comprehensions are allowed, however multiple `for` clauses or filter expressions are
not permitted. Optimise for readability, not conciseness. The following expressions are
acceptable:

```python
result = [mapping_expr for value in iterable if filter_expr]

result = [
  is_valid(metric={'key': value})
  for value in interesting_iterable
  if a_longer_filter_expression(value)
]

descriptive_name = [
  transform({'key': key, 'value': value}, color='black')
  for key, value in generate_iterable(some_input)
  if complicated_condition_is_met(key, value)
]

result = []
for x in range(10):
for y in range(5):
  if x * y > 10:
    result.append((x, y))

return {
  x: complicated_transform(x)
  for x in long_generator_function(parameter)
  if x is not None
}

return (x**2 for x in range(10))

unique_names = {user.name for user in users if user is not None}
```

The following expressions are not acceptable:

```python
result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]

return (
  (x, y, z)
  for x in range(5)
  for y in range(5)
  if x != y
  for z in range(5)
  if y != z
)
```

### 1.6 Default Iterators and Operators

Container types, like dictionaries and lists, define default iterators and membership
test operators (“in” and “not in”). The default iterators and operators are simple and
efficient. They express the operation directly, without extra method calls.

Use default iterators and operators for types that support them, like lists,
dictionaries, and files. The built-in types define iterator methods, too. Prefer these
methods to methods that return lists, except that you should not mutate a container
while iterating over it. The following are preferred:

```python
for key in adict: ...
if obj in alist: ...
for line in afile: ...
for k, v in adict.items(): ...
```

While these are not:

```python
for key in adict.keys(): ...
for line in afile.readlines(): ...
```

### 1.7 Generators

A generator function returns an iterator that yields a value each time it executes a
yield statement. After it yields a value, the runtime state of the generator function
is suspended until the next value is needed. A generator uses less memory than a
function that creates an entire list of values at once. However, local variables in the
generator will not be garbage collected until the generator is either consumed to
exhaustion or itself garbage collected.

Use as needed. Use “Yields:” rather than “Returns:” in the docstring for generator
functions. If the generator manages an expensive resource, make sure to force the
clean-up. A good way to do the clean-up is by wrapping the generator with a context
manager [PEP-0533](https://peps.python.org/pep-0533/).

### 1.8 Lambda Functions

Lambdas define anonymous functions in an expression, as opposed to a statement. They
are harder to read and debug than local functions. The lack of names means stack traces
are more difficult to understand. Expressiveness is limited because the function may
only contain an expression.

Lambdas are allowed. If the code inside the lambda function spans multiple lines or is
longer than 60-80 chars, it might be better to define it as a regular nested function.

For common operations like multiplication, use the functions from the `operator` module
instead of lambda functions. For example, prefer `operator.mul` to
`lambda x, y: x * y`. Prefer generator expressions over `map()` or `filter()` with a
`lambda`.

### 1.9 Conditional Expressions

Conditional expressions (sometimes called a “ternary operator”) are mechanisms that
provide a shorter syntax for if statements. For example: `x = 1 if cond else 2`.

Okay to use for simple cases. Each portion must fit on one line: true-expression,
if-expression, else-expression. Use a complete if statement when things get more
complicated. The following are acceptable:

```python
one_line = 'yes' if predicate(value) else 'no'
slightly_split = ('yes' if predicate(value)
                  else 'no, nein, nyet')
the_longest_ternary_style_that_can_be_done = (
    'yes, true, affirmative, confirmed, correct'
    if predicate(value)
    else 'no, false, negative, nay')
```

While these are not.

```python
bad_line_breaking = ('yes' if predicate(value) else
                     'no')
portion_too_long = ('yes'
                    if some_long_module.some_long_predicate_function(
    really_long_variable_name)
                    else 'no, false, negative, nay')
```

### 1.10 Default Values

You can specify values for variables at the end of a function’s parameter list,
e.g., `def foo(a, b=0):`. If `foo` is called with only one argument, `b` is set to 0.
If it is called with two arguments, `b` has the value of the second argument. Default
arguments provide a way to approximate function overloading, without defining multiple
functions. However, default arguments are evaluated once at module load time. This may
cause problems if the argument is a mutable object such as a list or a dictionary. If
the function modifies the object (e.g., by appending an item to a list), the default
value is modified.

Default values are okay, however, mutable objects are not permitted as default values.

### 1.11 Properties

A way to wrap method calls for getting and setting an attribute as a standard attribute
access. Properties may be used to control getting or setting attributes that require
trivial computations or logic. Property implementations must match the general
expectations of regular attribute access: that they are cheap, straightforward, and
unsurprising. They:

+ Can be used to make read-only attributes.
+ Allow calculations to be lazy.
+ Provide a way to maintain the public interface of a class when the internals evolve.

However, they can also:

+ Hide side effects much like operator overloading
+ Make subclassing confusing

Properties are allowed, but, like operator overloading, should only be used when
necessary and match the expectations of typical attribute access; follow the [getters
and setters rules]() otherwise.

For example, using a property to simply both get and set an internal attribute isn’t
allowed: there is no computation occurring, so the property is unnecessary (make the
attribute public instead). In comparison, using a property to control attribute access
or to calculate a trivially derived value is allowed: the logic is simple and
unsurprising.

Properties should be created with the `@property` decorator. Manually implementing a
property descriptor is considered a power feature.

Inheritance with properties can be non-obvious. Do not use properties to implement
computations a subclass may ever want to override and extend.

### 1.12 True/False Evaluations

Python evaluates certain values as `False` when in a boolean context. A quick “rule of
thumb” is that all “empty” values are considered false, so `0`, `None`, `[]`, `{}`,
`''` all evaluate as false in a boolean context.

Use the “implicit” false if possible, e.g., `if foo:` rather than `if foo != []:`.
There are a few caveats:

+ Always use `if foo is None:` (or `is not None`) to check for a `None` value. E.g.,
  when testing whether a variable or argument that defaults to `None` was set to some
  other value. The other value might be a value that’s false in a boolean context!
+ Never compare a boolean variable to `False` using `==`. Use `if not x:` instead. If
  you need to distinguish `False` from `None` then chain the expressions, such as
  `if not x and x is not None:`.
+ For sequences (strings, lists, tuples), use the fact that empty sequences are false,
  so `if seq:` and `if not seq:` are preferable to `if len(seq):` and `if not len(seq):`
  respectively.
+ When handling integers, implicit false may involve more risk than benefit (i.e.,
  accidentally handling `None` as 0). You may compare a value which is known to be an
  integer (and is not the result of `len()`) against the integer 0.

```python
# These are acceptable
if not users:
     print('no users')

 if i % 10 == 0:
     self.handle_multiple_of_ten()

 def f(x=None):
     if x is None:
         x = []

# These are not acceptable
if len(users) == 0:
     print('no users')

 if not i % 10:
     self.handle_multiple_of_ten()

 def f(x=None):
     x = x or []
```

+ Note that `'0'` (i.e., `0` as string) evaluates to true.
+ Note that Numpy arrays may raise an exception in an implicit boolean context. Prefer
  the `.size` attribute when testing emptiness of a `np.array` (e.g. `if not users.size`).

### 1.13 Lexical Scoping

A nested Python function can refer to variables defined in enclosing functions, but
cannot assign to them. Variable bindings are resolved using lexical scoping, that is,
based on the static program text. Any assignment to a name in a block will cause
Python to treat all references to that name as a local variable, even if the use
precedes the assignment. If a global declaration occurs, the name is treated as a
global variable.

An example of the use of this feature is:

```python
def get_adder(summand1: float) -> Callable[[float], float]:
    """Returns a function that adds numbers to a given number."""
    def adder(summand2: float) -> float:
        return summand1 + summand2

    return adder
```

Often results in clearer, more elegant code. Especially comforting to experienced Lisp
and Scheme (and Haskell and ML and …) programmers but can cause confusing bugs such as
this example based on [PEP-0227](https://peps.python.org/pep-0227/):

```python
i = 4
def foo(x: Iterable[int]):
    def bar():
        print(i, end='')
    # ...
    # A bunch of code here
    # ...
    for i in x:  # Ah, i *is* local to foo, so this is what bar sees
        print(i, end='')
    bar()
```

So `foo([1, 2, 3])` will print `1 2 3 3`, not `1 2 3 4`. They are okay to use.

### 1.14 Decorators

Decorators are syntactic sugar. For some function `my_decorator`, this:

```python
class C:
    @my_decorator
    def method(self):
        # method body ...
```

Is equivalent to:

```python
class C:
    def method(self):
        # method body ...
    method = my_decorator(method)
```

Decorators elegantly specifies some transformation on a method; the transformation
might eliminate some repetitive code, enforce invariants, etc.

However, decorators can perform arbitrary operations on a function’s arguments or
return values, resulting in surprising implicit behaviour. Additionally, decorators
execute at object definition time. For module-level objects (classes, module
functions, …) this happens at import time. Failures in decorator code are pretty much
impossible to recover from.

Use decorators judiciously when there is a clear advantage. Decorators should follow
the same import and naming guidelines as functions. A decorator docstring should
clearly state that the function is a decorator. Write unit tests for decorators.

Avoid external dependencies in the decorator itself (e.g. don’t rely on files,
sockets, database connections, etc.), since they might not be available when the
decorator runs (at import time, perhaps from pydoc or other tools). A decorator that
is called with valid parameters should (as much as possible) be guaranteed to succeed
in all cases.

Decorators are a special case of “top-level code” - see [main]() for more discussion.

Never use `staticmethod` unless forced to in order to integrate with an API defined in
an existing library. Write a module-level function instead.

Use `classmethod` only when writing a named constructor, or a class-specific routine
that modifies necessary global state such as a process-wide cache.

### 1.15 Threading

Do not rely on the atomicity of built-in types.

While Python’s built-in data types such as dictionaries appear to have atomic
operations, there are corner cases where they aren’t atomic (e.g. if `__hash__` or
`__eq__` are implemented as Python methods) and their atomicity should not be relied
upon. Neither should you rely on atomic variable assignment (since this in turn
depends on dictionaries).

Use the `queue` module’s `Queue` data type as the preferred way to communicate data
between threads. Otherwise, use the threading module and its locking primitives.
Prefer condition variables and threading.Condition instead of using lower-level
locks.

In general, threading is something we generally avoid.

### 1.16 Power Features

Python is an extremely flexible language and gives you many fancy features such as
custom metaclasses, access to bytecode, on-the-fly compilation, dynamic inheritance,
object reparenting, import hacks, reflection (e.g. some uses of `getattr()`),
modification of system internals, `__del__` methods implementing customised clean-up,
etc.

It’s very tempting to use these features when they’re not absolutely necessary. It’s
harder to read, understand, and debug code that’s using unusual features underneath.
It doesn’t seem that way at first (to the original author), but when revisiting the
code, it tends to be more difficult than code that is longer but is straightforward.

Avoid these features in your code. Standard library modules and classes that
internally use these features are okay to use (for example, `abc.ABCMeta`,
`dataclasses`, and `enum`).

### 1.17 Modern Python: from __future__ imports

Being able to turn on some of the more modern features via `from __future__ import`
statements allows early use of features from expected future Python versions.

These are not permitted.

### 1.18 Type Annotated Code

You can annotate Python code with type hints that can be checked at build time with a
type checking tool. In most cases, when feasible, type annotations are in source files.
For third-party or extension modules, annotations can be in stub `.pyi` files.

Type annotations improve the readability and maintainability of your code. The type
checker will convert many runtime errors to build-time errors.

You are strongly encouraged to enable Python type analysis when updating code. When
adding or modifying public APIs, include type annotations and enable checking in the
build system.

## 2 Python Style Rules

---

### 2.1 Semicolons

Do not terminate your lines with semicolons, and do not use semicolons to put two
statements on the same line.

### 2.2 Line Length

Maximum line length is 80 characters.

Explicit exceptions to the 80-character limit:

+ Long import statements.
+ URLs, path names, or long flags in comments.
+ Long string module-level constants not containing whitespace that would be
  inconvenient to split across lines such as URLs or path names.

Do not use a backslash for
[explicit line continuation](https://docs.python.org/3/reference/lexical_analysis.html#explicit-line-joining).

Instead, make use of Python’s
[implicit line joining inside parentheses, brackets and braces](http://docs.python.org/reference/lexical_analysis.html#implicit-line-joining).
If necessary, you can add an extra pair of parentheses around an expression.

Note that this rule doesn’t prohibit backslash-escaped newlines within strings.

```python
# These are acceptable
foo_bar(self, width, height, color='black', design=None, x='foo',
         emphasis=None, highlight=0)
if (width == 0 and height == 0 and
     color == 'red' and emphasis == 'strong'):

 (bridge_questions.clarification_on
  .average_airspeed_of.unladen_swallow) = 'African or European?'

 with (
     very_long_first_expression_function() as spam,
     very_long_second_expression_function() as beans,
     third_thing() as eggs,
 ):
   place_order(eggs, beans, spam, beans)

# These are not
if width == 0 and height == 0 and \
     color == 'red' and emphasis == 'strong':

 bridge_questions.clarification_on \
     .average_airspeed_of.unladen_swallow = 'African or European?'

 with very_long_first_expression_function() as spam, \
       very_long_second_expression_function() as beans, \
       third_thing() as eggs:
   place_order(eggs, beans, spam, beans)
```

When a literal string won’t fit on a single line, use parentheses for implicit line
joining.

```python
x = ('This will build a very long long '
     'long long long long long long string')
```

Prefer to break lines at the highest possible syntactic level. If you must break a
line twice, break it at the same syntactic level both times.

```python
# These are acceptable
bridgekeeper.answer(
     name="Arthur", quest=questlib.find(owner="Arthur", perilous=True))

 answer = (a_long_line().of_chained_methods()
           .that_eventually_provides().an_answer())

 if (
     config is None
     or 'editor.language' not in config
     or config['editor.language'].use_spaces is False
 ):
   use_tabs()

# These are not
bridgekeeper.answer(name="Arthur", quest=questlib.find(
    owner="Arthur", perilous=True))

answer = a_long_line().of_chained_methods().that_eventually_provides(
    ).an_answer()

if (config is None or 'editor.language' not in config or config[
    'editor.language'].use_spaces is False):
  use_tabs()
```

Within comments, put long URLs on their own line if necessary.

```python
# Acceptable

# See details at
# https://www.example.com/us/developer/documentation/api/content/v2.0/csv_file_name_extension_full_specification.html

# Not acceptable

# See details at
# https://www.example.com/us/developer/documentation/api/content/
# v2.0/csv_file_name_extension_full_specification.html
```

Docstring summary lines must remain within the 80-character limit.

### 2.3 Parentheses

Use parentheses sparingly.

It is fine, though not required, to use parentheses around tuples. Do not use them in
return statements or conditional statements unless using parentheses for implied line
continuation or to indicate a tuple.

```python
# Acceptable
if foo:
    bar()
while x:
    x = bar()
if x and y:
    bar()
if not x:
    bar()
# For a 1 item tuple the ()s are more visually obvious than the comma.
onesie = (foo,)
return foo
return spam, beans
return (spam, beans)
for (x, y) in dict.items(): ...

# Unacceptable
if (x):
    bar()
if not(x):
    bar()
return (foo)
```

### 2.4 Indentation

Indent your code blocks with 4 spaces.

Never use tabs. Implied line continuation should align wrapped elements vertically
(see [line length examples](#22-line-length)), or use a hanging 4-space indent. Closing
(round, square or curly) brackets can be placed at the end of the expression, or on
separate lines, but then should be indented the same as the line with the
corresponding opening bracket.

```python
# The following are acceptable

# Aligned with opening delimiter.
foo = long_function_name(var_one, var_two,
                         var_three, var_four)
meal = (spam,
        beans)

# Aligned with opening delimiter in a dictionary.
foo = {
   'long_dictionary_key': value1 +
                          value2,
   ...
}

# 4-space hanging indent; nothing on first line.
foo = long_function_name(
   var_one, var_two, var_three,
   var_four)
meal = (
   spam,
   beans)

# 4-space hanging indent; nothing on first line,
# closing parenthesis on a new line.
foo = long_function_name(
   var_one, var_two, var_three,
   var_four
)
meal = (
   spam,
   beans,
)

# 4-space hanging indent in a dictionary.
foo = {
   'long_dictionary_key':
       long_dictionary_value,
   ...
}

# The following are not acceptable
# Stuff on first line forbidden.
foo = long_function_name(var_one, var_two,
   var_three, var_four)
meal = (spam,
   beans)

# 2-space hanging indent forbidden.
foo = long_function_name(
 var_one, var_two, var_three,
 var_four)

# No hanging indent in a dictionary.
foo = {
   'long_dictionary_key':
   long_dictionary_value,
   ...
}
```

#### 2.4.1 Trailing Commas

Trailing commas in sequences of items are recommended only when the closing container
token `]`, `)`, or `}` does not appear on the same line as the final element, as well
as for tuples with a single element.

```python
# Acceptable

golomb3 = [0, 1, 3]
golomb4 = [
   0,
   1,
   4,
   6,
]

# Not acceptable
golomb4 = [
   0,
   1,
   4,
   6,]
```

### 2.5 Blank Lines

Two blank lines between top-level definitions, be they function or class definitions.
One blank line between method definitions and between the docstring of a `class` and
the first method. No blank line following a `def` line. Use single blank lines as you
judge appropriate within functions or methods.

Blank lines need not be anchored to the definition. For example, related comments
immediately preceding function, class, and method definitions can make sense. Consider
if your comment might be more useful as part of the docstring.

### 2.6 Whitespace

Follow standard typographic rules for the use of spaces around punctuation.

No whitespace inside parentheses, brackets or braces.

```python
# Acceptable
spam(ham[1], {'eggs': 2}, [])

# Not Acceptable
spam(ham[1], {'eggs': 2}, [])
```

No whitespace before a comma, semicolon, or colon. Do use whitespace after a comma,
semicolon, or colon, except at the end of the line.

```python
# Acceptable
if x == 4:
    print(x, y)
x, y = y, x

# Not acceptable
if x == 4 :
    print(x , y)
x , y = y , x
```

No whitespace before the open paren/bracket that starts an argument list, indexing or
slicing.

```python
# Acceptable
spam(1)
dict['key'] = list[index]

# Not acceptable
spam(1)
dict['key'] = list[index]
```

No trailing whitespace.

Surround binary operators with a single space on either side for assignment (`=`),
comparisons (`==`, `<`, `>`, `!=`, `<>`, `<=`, `>=`, `in`, `not in`, `is`, `is not`),
and Booleans (`and`, `or`, `not`). Use your better judgment for the insertion of
spaces around arithmetic operators (`+`, `-`, `*`, `/`, `//`, `%`, `**`, `@`).

```python
# Yes
x == 1

# No  
x < 1
```

Never use spaces around `=` when passing keyword arguments or defining a default
parameter value, with one exception: when a type annotation is present, do use spaces
around the `=` for the default parameter value.

```python
# Yes 
def complex(real, imag=0.0): return Magic(r=real, i=imag)
def complex(real, imag: float = 0.0): return Magic(r=real, i=imag)

# No
def complex(real, imag = 0.0): return Magic(r = real, i = imag)
def complex(real, imag: float=0.0): return Magic(r = real, i = imag)
```

Don’t use spaces to vertically align tokens on consecutive lines, since it becomes a
maintenance burden (applies to `:`, `#`, `=`, etc.):

```python
# Yes
foo = 1000  # comment
long_name = 2  # comment that should not be aligned

dictionary = {
    'foo': 1,
    'long_name': 2,
}

# No
foo       = 1000  # comment
long_name = 2     # comment that should not be aligned

dictionary = {
    'foo'      : 1,
    'long_name': 2,
}
```

### 2.7 Comments and Docstrings

Be sure to use the right style for module, function, method docstrings and inline
comments.

#### 2.7.1 Docstrings

Python uses docstrings to document code. A docstring is a string that is the first
statement in a package, module, class or function. These strings can be extracted
automatically through the `__doc__` member of the object and are used by `pydoc`.
(Try running pydoc on your module to see how it looks.) Always use the
three-double-quote `"""` format for docstrings (per
[PEP 257](https://peps.python.org/pep-0257/)). A docstring should be organised as a
summary line (one physical line not exceeding 80 characters) terminated by a period,
question mark, or exclamation point. When writing more (encouraged), this must be
followed by a blank line, followed by the rest of the docstring starting at the same
cursor position as the first quote of the first line. There are more formatting
guidelines for docstrings below.

#### 2.7.2 Modules

Every file should contain licence boilerplate. Choose the appropriate boilerplate for
the licence used by the project (for example, Apache 2.0, BSD, LGPL, GPL).

Files should start with a docstring describing the contents and usage of the module.

```python
"""A one-line summary of the module or program, terminated by a period.

Leave one blank line.  The rest of this docstring should contain an
overall description of the module or program.  Optionally, it may also
contain a brief description of exported classes and functions and/or usage
examples.

Typical usage example:

  foo = ClassFoo()
  bar = foo.function_bar()
"""
```

##### 2.7.2.1 Test Modules

Module-level docstrings for test files are not required. They should be included only
when there is additional information that can be provided.

Examples include some specifics on how the test should be run, an explanation of an
unusual setup pattern, dependency on the external environment, and so on.

```python
"""This blaze test uses golden files.

You can update those files by running
`blaze run //foo/bar:foo_test -- --update_golden_files` from the `google3`
directory.
"""
```

Docstrings that do not provide any new information should not be used.

```python
"""Tests for foo.bar."""
```

#### 2.7.3 Functions and Methods

In this section, “function” means a method, function, generator, or property.

A docstring is mandatory for every function that has one or more of the following
properties:

+ being part of the public API
+ nontrivial size
+ non-obvious logic

A docstring should give enough information to write a call to the function without
reading the function’s code. The docstring should describe the function’s calling
syntax and its semantics, but generally not its implementation details, unless those
details are relevant to how the function is to be used. For example, a function that
mutates one of its arguments as a side effect should note that in its docstring.
Otherwise, subtle but important details of a function’s implementation that are not
relevant to the caller are better expressed as comments alongside the code than within
the function’s docstring.

The docstring may be descriptive-style (`"""Fetches rows from a Bigtable."""`) or
imperative-style (`"""Fetch rows from a Bigtable."""`), but the style should be
consistent within a file. The docstring for a `@property` data descriptor should use
the same style as the docstring for an attribute or a function argument
(`"""The Bigtable path."""`, rather than `"""Returns the Bigtable path."""`).

Certain aspects of a function should be documented in special sections, listed below.
Each section begins with a heading line, which ends with a colon. All sections other
than the heading should maintain a hanging indent of two or four spaces (be consistent
within a file). These sections can be omitted in cases where the function’s name and
signature are informative enough that it can be aptly described using a one-line
docstring.

##### Args

List each parameter by name. A description should follow the name, and be separated by
a colon followed by either a space or newline. If the description is too long to fit
on a single 80-character line, use a hanging indent of 2 or 4 spaces more than the
parameter name (be consistent with the rest of the docstrings in the file). The
description should include required type(s) if the code does not contain a
corresponding type annotation. If a function accepts *foo (variable length argument
lists) and/or `**bar` (arbitrary keyword arguments), they should be listed as `*foo`
and `**bar`.

##### Returns (or Yields for generators)

Describe the semantics of the return value, including any type information that the
type annotation does not provide. If the function only returns None, this section is
not required. It may also be omitted if the docstring starts with “Return”, “Returns”,
“Yield”, or “Yields” (e.g. `"""Returns row from Bigtable as a tuple of strings."""`)
and the opening sentence is sufficient to describe the return value. Do not imitate
older ‘NumPy style’ (example), which frequently documented a tuple return value as if
it were multiple return values with individual names (never mentioning the tuple).
Instead, describe such a return value as: “Returns: A tuple (mat_a, mat_b), where
mat_a is …, and …”. The auxiliary names in the docstring need not necessarily
correspond to any internal names used in the function body (as those are not part of
the API). If the function uses `yield` (is a generator), the Yields: section should
document the object returned by `next()`, instead of the generator object itself that
the call evaluates to.

##### Raises

List all exceptions that are relevant to the interface followed by a description. Use
a similar exception name + colon + space or newline and hanging indent style as
described in Args:. You should not document exceptions that get raised if the API
specified in the docstring is violated (because this would paradoxically make behaviour
under violation of the API part of the API).

```python
def fetch_smalltable_rows(
    table_handle: smalltable.Table,
    keys: Sequence[bytes | str],
    require_all_keys: bool = False,
) -> Mapping[bytes, tuple[str, ...]]:
    """Fetches rows from a Smalltable.
    
    Retrieves rows pertaining to the given keys from the Table instance
    represented by table_handle.  String keys will be UTF-8 encoded.
    
    Args:
        table_handle: An open smalltable.Table instance.
        keys: A sequence of strings representing the key of each table
          row to fetch.  String keys will be UTF-8 encoded.
        require_all_keys: If True only rows with values set for all keys will be
          returned.
    
    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:
    
        {b'Serak': ('Rigel VII', 'Preparer'),
         b'Zim': ('Irk', 'Invader'),
         b'Lrrr': ('Omicron Persei 8', 'Emperor')}
    
        Returned keys are always bytes.  If a key from the keys argument is
        missing from the dictionary, then that row was not found in the
        table (and require_all_keys must have been False).
    
    Raises:
        IOError: An error occurred accessing the smalltable.
    """
```

Similarly, this variation on Args: with a line break is also allowed:

```python
def fetch_smalltable_rows(
    table_handle: smalltable.Table,
    keys: Sequence[bytes | str],
    require_all_keys: bool = False,
) -> Mapping[bytes, tuple[str, ...]]:
    """Fetches rows from a Smalltable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by table_handle.  String keys will be UTF-8 encoded.

    Args:
      table_handle:
        An open smalltable.Table instance.
      keys:
        A sequence of strings representing the key of each table row to
        fetch.  String keys will be UTF-8 encoded.
      require_all_keys:
        If True only rows with values set for all keys will be returned.

    Returns:
      A dict mapping keys to the corresponding table row data
      fetched. Each row is represented as a tuple of strings. For
      example:

      {b'Serak': ('Rigel VII', 'Preparer'),
       b'Zim': ('Irk', 'Invader'),
       b'Lrrr': ('Omicron Persei 8', 'Emperor')}

      Returned keys are always bytes.  If a key from the keys argument is
      missing from the dictionary, then that row was not found in the
      table (and require_all_keys must have been False).

    Raises:
      IOError: An error occurred accessing the smalltable.
    """
```

##### 2.7.3.1 Overridden Methods

A method that overrides a method from a base class does not need a docstring if it is
explicitly decorated with `@override` (from `typing_extensions` or `typing` modules),
unless the overriding method’s behaviour materially refines the base method’s contract,
or details need to be provided (e.g., documenting additional side effects), in which
case a docstring with at least those differences is required on the overriding method.

```python
from typing_extensions import override

class Parent:
  def do_something(self):
    """Parent method, includes docstring."""

# Child class, method annotated with override.
class Child(Parent):
  @override
  def do_something(self):
    pass

# Child class, but without @override decorator, a docstring is required.
class Child(Parent):
  def do_something(self):
    pass

# Docstring is trivial, @override is sufficient to indicate that docs can be
# found in the base class.
class Child(Parent):
  @override
  def do_something(self):
    """See base class."""
```

#### 2.7.4 Classes

TBD
