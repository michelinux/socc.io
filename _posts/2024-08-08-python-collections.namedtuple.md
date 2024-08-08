---
layout: post
title: Python collections.namedtuple
date: 2024-08-08 02:45 +0200
published: true
---

While reading the Fluent Python 2nd Edition, the [very first example](https://learning.oreilly.com/library/view/fluent-python-2nd/9781492056348/ch01.html#ex_pythonic_deck) uses the `collections.namedtuple`. As a C++ developer, I was fascinated from this.

```python
import collections.namedtuple
Card = collections.namedtuple('Card', ['rank', 'suit'])
```

The first argument `Card` is the name of the class. We assign it to a variable called also `Card` so that we can refer to it.

```python
>>> import collections
>>> Carta = collections.namedtuple('Card', ['rank', 'suit'])
>>> my_card = Carta(rank=10, suit="danari")
>>> print(type(my_card))
<class '__main__.Card'>
```

So the variable `Carta` in this case is only a handle and the name of the class remain `Card`, as you can see when you print the type of `my_card`, or even just with:

```python
>>> Carta
<class '__main__.Card'>
```

My next curiousity: what is that `__main__`? Well, it's the name of the current module. If you were to use the namedtuple factory to define a tuple in a module, it will take the name of that. Quick example:

```bash
cd /tmp
mkdir my_module
touch my_module/__init__.py
```

```python
# The _init_.py file
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
```

Then from the REPL:

```python
>>> from my_module import Point
>>> Point
<class 'my_module.Point'>
```

Alternatively, you can also force a different module directly at the _factory_:

```python
>>> Point = namedtuple('Point', ['x', 'y'], module='my_module')
>>> Point
<class 'my_module.Point'>
```

So far, so good. Now, let's say that you want to create a new tuple and one of its field is a reserved word:

```python
>>> SalaryEvolution = namedtuple('SalaryEvolution', ['employee_id', 'year', 'current_salary', 'raise'])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/lib/python3.12/collections/__init__.py", line 403, in namedtuple
    raise ValueError('Type names and field names cannot be a '
ValueError: Type names and field names cannot be a keyword: 'raise'
```

If you were to write this code manually, you could just change the `raise` field name. But what if you are importing data automatically and have no control over the field name? Either you handle the exception, or you use `rename=True`

```python
>>> SalaryEvolution = namedtuple('SalaryEvolution', ['employee_id', 'year', 'current_salary', 'raise'], rename=True)
>>> se = SalaryEvolution(19, 2024, 10000, 0.04)
>>> se
SalaryEvolution(employee_id=19, year=2024, current_salary=10000, _3=0.04)
```

The `_3` is less than optimal IMHO, but make sure you are not throwing away all the other fields at least.

The followup question would be: in which case would you have no control over the field names? For example when reading a csv file. And to simplify this case you can just pass the name of the fields a string of words separated by commas:

```python
>>> first_row = 'employee_id, year, current_salary, raise' 
>>> SalaryEvolution = namedtuple('SalaryEvolution', first_row, rename=True)
>>> se = SalaryEvolution(19, 2024, 10000, 0.04)
>>> se
SalaryEvolution(employee_id=19, year=2024, current_salary=10000, raise=0.04)
```

Unfortunately commas and whitespaces are the only accepted separators. Directly from the [namedtuple implementation in the standard factory](https://github.com/python/cpython/blob/8f4892ac529b8f7f32d9707eb713300188247b7b/Lib/collections/__init__.py#L379C1-L385C1):

```python
    # Validate the field names.  At the user's option, either generate an error
    # message or automatically replace the field name with a valid name.
    if isinstance(field_names, str):
        field_names = field_names.replace(',', ' ').split()
    field_names = list(map(str, field_names))
    typename = _sys.intern(str(typename))
```

As in Europe csv files mostly use the a "semicolon" as fields separator, I wouldn't mind to propose a change, something like:

```diff
diff --git a/Lib/collections/__init__.py b/Lib/collections/__init__.py
index b47e728484..471d822240 100644
--- a/Lib/collections/__init__.py
+++ b/Lib/collections/__init__.py
@@ -380,7 +380,7 @@ def namedtuple(typename, field_names, *, rename=False, defaults=None, module=Non
     # Validate the field names.  At the user's option, either generate an error
     # message or automatically replace the field name with a valid name.
     if isinstance(field_names, str):
-        field_names = field_names.replace(',', ' ').split()
+        field_names = field_names.replace(',', ' ').replace(';', ' ').split()
     field_names = list(map(str, field_names))
     typename = _sys.intern(str(typename))
 
```

And no, I do not think adding a parameter would be better, it would just add extra complexity to the interface.

And yes, I do think this:

```python
SalaryEvolution = namedtuple('SalaryEvolution', first_row, rename=True)
```

is clearer than:

```python
SalaryEvolution = namedtuple('SalaryEvolution', first_row.replace(';', ' '), rename=True)
```

There is more to `collections.namedtuple`, and there are other ways to build _data classes_. I just didn't get to [Chapter 5 yet](https://learning.oreilly.com/library/view/fluent-python-2nd/9781492056348/ch05.html#classic_named_tuples_sec).
