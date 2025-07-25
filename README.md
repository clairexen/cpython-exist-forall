Python Exist-Forall Extension
=============================

This repository contains an experimental implementation of the Python Exist-Forall Extension Proposal.

Syntax
------

First, we define an **Exist-Forall Expression**, with the syntax `(: Expression :)`.

Within an Exist-Forall Expression the tuple syntax is extended to allow the following:

```python
(1, 2, 3, or 5)
(10, 11, and 12)
```

I.e. the keywords `or` and `and` are allowed after a comma within a tuple. Their placement is flexible, and the following examples are all considered equivalent:

```python
(1, or 2, or 3)
(1, 2, or 3)
(1, or 2, 3)
(1, 2, 3, or)
```

However, it is a **syntax error** to mix `and` and `or` within the same tuple.

- A tuple using `or` is called an **Exist-List**.
- A tuple using `and` is called a **Forall-List**.

An expr matching `(: Expression :)`, where `Expression` does _not_ contain any Exist-Lists or Forall-Lists, is called a **Sub-Expression** and is legal code, and behaves exactly the same as `( Expression )`, with the only exception that there is an AST-Node for each Sub-Expression, enabling a wide range of other AST-Rewriting use-cases for the `(: ... :)` construct.

Semantics
---------

When an Exist-Forall Expression is parsed, it is transformed into a lambda function and a call to `__exist_forall_eval__`, as illustrated by the following example:

### Input Expression:
```python
(: (1, 2, and 5) < (3, 6, or 10) < (7, and 11) :)
```

### Transformed Expression:
```python
__exist_forall_eval__(
    lambda _ea_exist_items_, _ea_forall_items_:
                _ea_forall_items_[0] < _ea_exist_items_[0] < _ea_forall_items_[1],
    [(3, 6, 10)],         # All Exist-Lists within the EA-Expression
    [(1, 2, 5), (7, 11)]  # All Forall-Lists within the EA-Expression
)
```

The function `__exist_forall_eval__(lambda_func, exist_lists, forall_lists)` will then evaluate the lambda with various combinations of items picked from the exist and forall lists, until it can determine the overall truth value of the Exist-Forall-Expression.

The function name `__exist_forall_eval__` is resolved like any other name in Python, allowing the default implementation to be overloaded or replaced, either globally, or only in certain scopes.

If a solution to the Exist-Forall problem is found, `__exist_forall_eval__()` returns the solution (i.e. the list of values from the `exist_lists`) if `exist_lists` is non-empty, and True otherwise. And if no solution is found, None or False is returned:

```python
>>> a = "this is a test".split()
>>> b = "does this work?".split()

>>> (: (*a,or) == (*b,or) :)
('this', 'this')

>>> (: (*a,or) != (*b,and) :)
('is',)

>>> (: (*a,or) == (*b,and) :)
None

>>> (: (*a,and) != "foo" :)
True

>>> (: (*a,and) == "bar" :)
False
```

Implementation Notes
--------------------

*This is a quick hack, meant to be only a proof-of-concept.*

**I'm trying to keep this repo 3 commits ahead of upstream:**

```
c001337 HEAD    [EA-Syntax] Add EA-Expression-Syntax README.md (heads/main)
1841a6c HEAD~1  [EA-Syntax] Run `make regen-token regen-pegen regen-ast`
cd9c9c1 HEAD~2  [EA-Syntax] Implement (: ... :) Exist-Forall-Sub-Expression Syntax
```

Just cherry-picking `cd9c9c1` and then running `make regen-token regen-pegen regen-ast`
should work. The other two commits are just the re-generated files and this documentation.

**Please feel free to open an issue to leave some feedback and/or get in touch about this proposal.**
