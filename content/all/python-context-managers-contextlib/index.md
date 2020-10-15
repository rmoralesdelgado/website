+++
categories = ["tutorial", ]
author = "Raul Morales Delgado"
title = "Context Managers with Python's contextlib"
description = "Because managing resources in Python is better with context managers — and context managers are easier with `contextlib`. This article is a small tutorial on why context managers are easier with `contextmanager` from `contextlib`, where its similarities to a full implementation of with-statement context managers start to become blurrier, and how to effectively handle exceptions."
date = "2019-08-25"
toc = true
tags = ["python", "contextlib", ]
+++

In Python, context managers allow to efficiently manage resources by letting users set rules on what to do with those resources when working with them. Creating context managers from scratch, though, might get a little [tedious](https://docs.python.org/3.8/library/stdtypes.html#context-manager-types) since `__enter__()` and `__exit__()` methods have to be defined, as well as boilerplate code to deal with exceptions. 

To simplify this, out-of-the-box context managers can be created using [`contextlib`](https://docs.python.org/3/library/contextlib.html)'s `contextmanager` module — it allows to write context managers in just a few lines.

>**Note:** This is not a tutorial on context managers in general, but about their usage with Python's built-in `contextlib`. Please see the [last section](#6-acknowledgements) for additional resources.


## 1. Creating a Context Manager with "contextlib"
To create a context manager, the first step is to import the `contextmanager` module from `contextlib`. This module is a decorator used to define factory functions for with-statement context managers.[^1]

[^1]: https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager

```python
from contextlib import contextmanager
```

The syntax to create a context manager is fairly simple. See the following example right from the Python [docs](https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager):

```python
@contextmanager
def managed_resource(*args, **kwds):
    # Code to acquire resource, e.g.:
    resource = acquire_resource(*args, **kwds)
    try:
        yield resource
    finally:
        # Code to release resource, e.g.:
        release_resource(resource)
```

After declaring the `@contextmanager` decorator, a [generator](https://docs.python.org/3/glossary.html#term-generator) — a regular function with a `yield` statement (instead of `return`) — is defined. This context manager can now be called in a `with` statement, whose syntax is as follows:[^2]

[^2]: https://docs.python.org/3/reference/compound_stmts.html#the-with-statement

```python
with_stmt ::=  "with" with_item ("," with_item)* ":" suite
with_item ::=  expression ["as" target]
```

And applied to the previous example of a context manager:
```python
with managed_resource(*args, **kwds) as resource:
    do_thing_with_resource(resource)
```

The block below the `with` clause is called the *suite*, or (colloquially) *with-statement body*. When a `with` statement is executed, the following takes place:

1. The context expression — both decorator and generator — are evaluated and a context manager is obtained and loaded
2. Everything up to (and including) the `yield` statement is executed 
3. If a `target` was included, the yielded object will be assigned to it
4. The with-statement body is executed
5. Everything *after* the `yield` statement is executed

Note that the object generated at `yield` — a single-value [generator-iterator](https://docs.python.org/3/glossary.html#term-generator-iterator) — will be assigned to `target` (if defined) and will be accessible from the scope from which the `with` statement was executed. Likewise, anything declared inside the with-statement body will also be accessible in the same scope. More on this, later on.


## 2. A Very Simple Context Manager
First, let's create a very simple context manager — one that only appends a tag *before* and a tag *after* the yielded object using `print()`.

```python
@contextmanager
def tagger(text):
    print(f"Top tagger: {text}")
    yield
    print(f"Bottom tagger: {text}")
```

Once created, our context manager `tagger` is used in a `with` statement. In this case, the with-statement body will only consist of a `print` function:

```python
with tagger("hello"):
    print("foo")
```
```text
Top tagger: hello
foo
Bottom tagger: hello
```

When executed altogether, the tags are printed before and after the `print` function, which is what was expected.


## 3. The "as" Clause
What happens, though, when an `as` clause and a `target` are added to the `with` clause? The `as` keyword, in this context, is meant to assign the object — or objects — "yielded" by the context manager to a variable named after `target`.

In the context manager, `yield` indicates *when* the generated object will be passed to the caller — the `with` statement — and assigned to the target. After this, the with-statement body is executed. Note that if no object is stated in the `yield` statement, `None` will be yielded. This is shown below:

```python
with tagger('hello again!') as mytag:
    print(mytag)
```

```text
Top tagger: hello again!
None
Bottom tagger: hello again!
```

Let's explore a little further what `yield` does. Below is a second example of `tagger` — `tagger_2`, but now with a string object being yielded:

```python
@contextmanager
def tagger_2(text):
    print(f"Top tagger_2: {text}")
    yield text + ' — I am being yielded!'
    print(f"Bottom tagger_2: {text}")
```

The `as` clause will be used to assign the yielded object to `tagged`, which will then be called in the with-statement body by a `print` function:

```python
with tagger_2("hey!") as tagged:
    print("foo")
    print(tagged)
```

```text
Top tagger_2: hey!
foo
hey! — I am being yielded!
Bottom tagger_2: hey!
```

Note that if the `as` clause is not used to assign the yielded object to `target`, then it will not be possible to use such object within the with-statement body or later on. Also, note how the tags are placed at the top and bottom of the output; inside them, the with-statement body is executed as is.


## 4. When are Objects Yielded?
When working with generators — and, consequently, with context managers — it is important to know when yielded objects are *actually* yielded. As previously stated, when a `with` statement is executed, the context manager is loaded and everything until (and including) the `yield` statement is executed. At this point, control is given back to the caller, such that the with-statement body can be executed. Look at the following example:

```python
@contextmanager
def tagger(text):
    print(f"Top tagger: {text}")
    yield 123, "I am a string", print("I have been generated")
    print(f"Bottom tagger: {text}")

with tagger("hello") as tagged:
    print("foo")
    print(tagged)
```

```text
Top tagger: hello
I have been generated
foo
(123, 'I am a string', None)
Bottom tagger: hello
```

When the `with` statement was run, the following can be observed:
1. Top tag appears first (block before `yield` statement)
2. Objects in `yield` statement are executed and generated (i.e. "yielded")
3. The with-statement body is executed — first `print("foo")`, then `print(tagged)`
4. Bottom tag appears last.

This means that whatever computation is executed up to (and including) the `yield` statement will be done before the with-statement body has even started — this is equivalent to successfully running the `__enter__()` method in the full implementation of a context manager. 

Finally, note that `tagged` holds a tuple with all the elements that were generated by the context manager. Particularly, the third item is `None`, which is what `print` returns after being successfully executed.


## 5. Variable Scope
One characteristic of `with` statements is that objects assigned in the `with` statement are placed in the same scope where the statement was executed. In other words, objects generated by the context manager and assigned to `target`, and objects assigned or declared in the with-statement body will be accessible from the same scope — e.g. for all previous examples, the global scope.

Let's try this out. If a `with` statement is run from the global scope, then its variables will be accessible when called from the same scope:

```python
with tagger("hey!") as tagged:
    var = "I am var"
    
print(tagged)
print(var)
```

```text
Top tagger: hey!
I have been generated
Bottom tagger: hey!
(123, 'I am a string', None)
I am var
```

If the same `with` statement is now run from a function, then its variables will only be accessible from the scope of that function:

```python
def f(text):
    with tagger(text) as tagged_local:
        var_local = "I am local var"
    
    print(tagged_local)
    print(var_local)
    
f('hello')
```

```text
Top tagger: hello
I have been generated
Bottom tagger: hello
(123, 'I am a string', None)
I am local var
```

If those same variables are called from the global scope, a `NameError` exception will be thrown stating that those variables are not defined — they do not exist in this scope.

```python
print(tagged_local)
print(var_local)
```

```text
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-7-48f00078c951> in <module>
----> 1 print(tagged_local)
      2 print(var_local)

NameError: name 'tagged_local' is not defined
```

>**Note:** There are resources that, once opened, require closing — e.g. files, database connections, etc. Handling these resources with context managers is extremely useful because closing procedures can be guaranteed whether the with-statement body was successfully executed or not. As you might expect, these resources are opened when yielded and closed when the with-statement body is exited. However, after exiting the context manager, while the resource will still be assigned to the `target`, it will no longer be accessible — e.g. you will not be able to read or write that file anymore, or run a query against that database. See [this example](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files) about opening files with a context manager.


## 6. Equivalence to a Full Implementation
If you are familiar with the context manager [implementation](https://docs.python.org/3/reference/datamodel.html#with-statement-context-managers) for `with` statements (the *full* implementation), you know that a class with `__enter__()` and `__exit__()` methods needs to be defined[^3]. In this implementation, the `__enter__()` method provides the context for entering the resource and returns it after its successful execution — such that it becomes available for the with-statement body, if needed —, while the `__exit__()` method provides the context for exiting the resource and is in charge of handling exceptions that occur in the with-statement body.

[^3]: https://docs.python.org/3/library/stdtypes.html#typecontextmanager

However, these equivalences go a little further and become more blurry as you dig deeper, specially around exception handling. The table below shows these equivalences, from a high-level perspective, and where the equivalences become less clear:

Full implementation | Implementation using `contextmanager`
--------|------
Has a class with `__enter__()` and `__exit__()` methods defined | Has a generator function under the `contextmanager` decorator
`__init__()` method needs to be defined if arguments are to be passed into the context manager |Arguments can be simply defined in generator function at definition
`__enter__()` method defines context to enter resource |Everything up to (and including) `yield` statement defines context to enter resource
`__exit__()` method defines context to exit resource |Everything after `yield` statement defines context to exit resource
Exceptions in with-statement body are handled by `__exit__()` method. If it returns `True`, exception is handled, if `False`, exception is re-raised |Exceptions in with-statement body are handled by `try..except..finally` block inside generator function (i.e. inside the context manager)
Exceptions in `__enter__()` method are **not** handled by `__exit__()` method | Exceptions in everything up to (and including) `yield` statement are handled by `try..except..finally` block inside generator function (i.e. inside the context manager)
`__exit__()` method is only called if `__enter__()` method is successfully executed, regardless of what happens with the with-statement body | Depends on `try..except..finally` block implementation

As you might see, it is quite clear where the line draws if no exceptions were ever to happen. But this is not realistic — exceptions happen —, and knowing how and when to handle those exception is of vital importance. In the following section, a set of cases of exception handling are presented to try to understand the role of a `try..except..finally` block when handling exceptions.


## 7. Handling Exceptions
As explained in the previous section, handling exceptions works differently on context managers implemented with `contextlib`. In the following subsections, implementations of `try..except..finally` aim to demonstrate the way exceptions are handled with this implementation.

### 7.1 Handling exceptions in with-statament body
In the (full) implementation of a context manager for `with` statements, if an exception occurs in the with-statement body, the `__exit__()` method will be called immediately. If the method returns `True`, the exception will be handled and the context manager exited. If it returns `False`, the exception will be re-raised.[^4]

[^4]: https://docs.python.org/3/reference/datamodel.html#with-statement-context-managers

While the code *after* `yield` is equivalent to that of the `__exit__()` method, it will not only be executed if a `try..finally` block has not been implemented for any exceptions thrown by the with-statement body.

Let's look at this with an example. Suppose the next context manager is meant to do divisions of a numerator and denominator — both provided by the user —, and if successful, add the division result to a log.

```python
from contextlib import contextmanager

log = []

@contextmanager
def division(num, denom):
    try:
        print(f"Starting: num={num}, denom={denom}")
        yield num / denom
    except Exception:
        print("Exception caught!")
    finally:
        print("Completed")

with division(20, 5) as div:
    print('Starting with-statement body')
    log.append(div)
    raise Exception("Exception in with-statement body.")
    print("Log updated!")
```

```text
Starting: num=20, denom=5
Starting with-statement body
Exception caught!
Completed
```

Above, the `try` statement is fully executed (this is the equivalent of `__enter__()`), the with-statement body is executed and interrupted when the exception is raised, and finally the `except` and `finally` statements are called. If we inspect `log`,  we will find its value is `[4.0]`— the appending occured before the exception was raised. This example is the equivalent of a fully implemented context manager with an `__exit__()` that returns `True`.

If an `except` statement is not implemented, the exception would be raised. This, in contrast, would be the equivalent of a fully implemented context manager with an `__exit__()` that returns `False`.

### 7.2 Handling exceptions in the context manager
What happens if an exception is thrown in the context manager itself — that is, in the block of code in the `try` statement? 

Let's reuse the previous example and use `0` as the denominator to get a `ZeroDivisionError`:

```python
with division(20, 0) as div:
    print('Starting with-statement body')
    log.append(div)
    raise Exception("Exception in with-statement body.")
    print("Log updated!")
```

```text
Starting: num=20, denom=0
Exception caught!
Completed
---------------------------------------------------------------------------
RuntimeError                              Traceback (most recent call last)
<ipython-input-16-cf7a8f2b5b3e> in <module>
----> 1 with division(20, 0) as div:
      2     print('Starting with-statement body')
      3     log.append(div)
      4     raise Exception("Exception in with-statement body.")
      5     print("Log updated!")

~/anaconda3/lib/python3.7/contextlib.py in __enter__(self)
    112             return next(self.gen)
    113         except StopIteration:
--> 114             raise RuntimeError("generator didn't yield") from None
    115 
    116     def __exit__(self, type, value, traceback):

RuntimeError: generator didn't yield
```

As shown above, the first thing to notice is that, because the exception happened in the context manager (i.e. generator), the with-statement block is not executed. The second one, while weird, is that the exception thrown is a `RuntimeError`. So where is the `ZeroDivisionError`? Well, what happens here has to do with the fact that a `try..except..finally` block was implemented in the context manager — because `try` was interrupted by the `ZeroDivisionError`, the `finally` statement was called, thus never really yielding anything. This is what this exception is reporting, a generator that could not `return next(self.gen)`.

So, are all exceptions in this section going to raise a `RuntimeError`? And, what to do to actually retrieve the correct exception to later be able to debug? Well, everything that keeps the generator from yielding will lead to the same `RuntimeError`. 

To get the "actual" exception instead of a `RuntimeError`, it is necessary to catch the latter. However, this will only work as long as the "actual" exception is not an actual `RuntimeError`. Let's take a look at this.
```python
log = []

@contextmanager
def division(num, denom):
    try:
        print(f"Starting: num={num}, denom={denom}")
        yield num / denom
    except RuntimeError:
        print("RuntimeError caught")
    finally:
        print("Completed")

with division(20, 0) as div:
    print('Starting with-statement body')
    log.append(div)
    raise Exception("Exception in with-statement body.")
    print("Log updated!")
```

```text
Starting: num=20, denom=0
Completed
---------------------------------------------------------------------------
ZeroDivisionError                         Traceback (most recent call last)
<ipython-input-22-f522d955669c> in <module>
     15         print("Completed")
     16 
---> 17 with division(20, 0) as div:
     18     print('Starting with-statement body')
     19     log.append(div)

~/anaconda3/lib/python3.7/contextlib.py in __enter__(self)
    110         del self.args, self.kwds, self.func
    111         try:
--> 112             return next(self.gen)
    113         except StopIteration:
    114             raise RuntimeError("generator didn't yield") from None

<ipython-input-22-f522d955669c> in division(num, denom)
      9 #         if denom < 1:
     10 #             raise ZeroDivisionError("'denom' must be >=1")
---> 11         yield num / denom
     12     except RuntimeError:
     13         print("Runtime Error caught")

ZeroDivisionError: division by zero
```

Just to make a case, if a `RuntimeError` exception really happens, then the generator we will be back at the previous case:
```python
@contextmanager
def division(num, denom):
    try:
        print(f"Starting: num={num}, denom={denom}")
        raise RuntimeError('Just to make a case')
        yield num / denom
    except RuntimeError:
        print("RuntimeError caught")
    finally:
        print("Completed")

with division(20, 0) as div:
    print('Starting with-statement body')
    log.append(div)
    raise Exception("Exception in with-statement body.")
    print("Log updated!")
```

```text
Starting: num=20, denom=0
RuntimeError caught
Completed
---------------------------------------------------------------------------
RuntimeError                              Traceback (most recent call last)
<ipython-input-25-8f8a9d2ed9e6> in <module>
     14         print("Completed")
     15 
---> 16 with division(20, 0) as div:
     17     print('Starting with-statement body')
     18     log.append(div)

~/anaconda3/lib/python3.7/contextlib.py in __enter__(self)
    112             return next(self.gen)
    113         except StopIteration:
--> 114             raise RuntimeError("generator didn't yield") from None
    115 
    116     def __exit__(self, type, value, traceback):

RuntimeError: generator didn't yield
```

### 7.3 Summary
In this subsection, a table with the summary of the previous cases is presented.

If a context manager with a `try..except..finally` block is implemented, the following can be said about how exceptions are being handled:

`try` block implementation | Where exception happens | Is Exception raised? | Equivalent to...
----|----|----|----
`try..finally` | With-statement body | Yes | Exception in with-statement body and `__exit__()` that returns `False`
`try..except..finally` catching `Exception` (base class) | With-statement body | No | Exception in with-statement body and `__exit__()` that returns `True`
`try..except..finally` catching `Exception` (base class) | Context manager | Yes, but `RuntimeError` for any case | Exception in `__enter__()`
`try..except..finally` catching `RuntimeError` | Context manager | Yes, "actual" exception raised unless it's a `RuntimeError` | Exception in `__enter__()`

## 8. Using the Context Decorator
The [`ContextDecorator`](https://docs.python.org/3/library/contextlib.html#contextlib.ContextDecorator) module provides an out-of-the-box base class to develop your own context managers — it is even used by `contextmanager` for its own implementation. One of the "advantages" of using it is that you still need to go through the steps of a full implementation; and I quote that because what's the point of going through the full implementation if the overall idea is to simplify things? Well, in my experience this depends on how you want to handle those exceptions — if you are well acquainted with the code, then go for `contextmanager`, if you are not sure if exceptions can happen in either in the with-statement body or in the context manager itself, then use `ContextDecorator`. 

To show how this can be useful, I will reimplement the previous example and show how I can get the `ZeroDivisionError` without working around exceptions:
```python
from contextlib import ContextDecorator

log = []

class division(ContextDecorator):
    def __init__(self, num, denom):
        self.num = num
        self.denom = denom
        
    def __enter__(self):
        print(f"Starting: num={self.num}, denom={self.denom}")
        return self.num / self.denom

    def __exit__(self, *exc):
        print('__exit__ block called!')
        if exc[0] is not None:
            print(f"Exception of type {exc[0]} caught: {exc[1]}")
        print("Completed")
        return True

with division(20, 0) as div:
    print('Starting with-statement body')
    log.append(div)
    raise Exception("Exception in with-statement body.")
    print("Log updated!")
```

```text
Starting: num=20, denom=0
---------------------------------------------------------------------------
ZeroDivisionError                         Traceback (most recent call last)
[<-- Clipped -->]

ZeroDivisionError: division by zero
```

As seen above, the `ZeroDivisionError` not only raises but also the `__exit__()` block is not called — just as one would expect from a full implementation. To finish rounding this idea, let's raise an exception from the with-statement body:
```python
with division(20, 5) as div:
    print('Starting with-statement body')
    log.append(div)
    raise Exception("Exception in with-statement body.")
    print("Log updated!")
```

```text
Starting: num=20, denom=5
Starting with-statement body
__exit__ block called!
Exception of type <class 'Exception'> caught: Exception in with-statement body.
Completed
```

In this example, the `__exit__()` method is called and the exception is successfully handled (because of the `return True`). As shown in this section, a full implementation using `ContextDecorator` allows a more granular control over exception handling than what `contextmanager` does. 

Two final notes on `ContextDecorator`:
* The context manager `division` can now be used as a decorator to manage a resource for the function it decorates:
```python
@division(20,5)
def log(): print("logging")
log()
```
```text
Starting: num=20, denom=5
logging
__exit__ block called!
Completed
```
* However, whatever value is returned (i.e. yielded) by the context manager cannot be assigned to a target, since this syntax does not allow the use of an `as` clause, and therefore cannot be retrieved in the decorated function.

>**Note:** `ContextDecorator` is not needed to fully implement a context manager. It only gives you the ability of using that context manager as a decorator for functions you may want to define later on. The example above would have worked perfectly without inheriting from the `ContextDecorator` class.


## 9. Suppressing Specific Exceptions
There are times where a specific set of exceptions that you know beforehand could happen need to be suppressed. For such situations, a great tool to use is [`contextlib.suppress(*exceptions)`](https://docs.python.org/3/library/contextlib.html#contextlib.suppress) — it returns a context manager that will suppress any exceptions passed in as arguments that happen in the with-statement body.

Look at this example:

```python
from contextlib import suppress

with suppress(FileNotFoundError):
    os.remove("non_existent_file.tmp")
    print("hello")
```

Two things to be aware of when using this module are:
* If an exception that was passed in as an argument is thrown at any point in the with-statement body, then the exception will not be raised but execution will resume *after* the with-statement body. In the example above, if `non_existent_file.tmp` is not found, then `print("hello")` will not be executed — execution will resume after the `with` statement
* This module should only be used to catch very specific exceptions, since they will be passed on silently. In other words, if you expect a given type of exception to happen and you deem it as non important to the execution of the code, then you can use this module. Never use it to catch something like a base class `Exception`.

## 10. Conclusion
As seen in this small tutorial, using `contextlib`'s `contextmanager` decorator for implementing context managers is fast and easy. However, dealing with exceptions can get a little tricky when it comes to exception handling — this type of implementation should only be used when you are very well aware of the type of exceptions that both the context to enter the resource and the with-statement body can throw during runtime, such that you can create an adequate `try..except..finally` block to catch exceptions properly.

In case you are not sure of those exceptions, implementing a context manager from scratch is not a bad idea, and using `ContextDecorator` is good since it even gives you the ability of using it as a decorator for several functions you may want to define.

Finally, supressing a set of known exceptions can be performed with the `suppress` module, as long as making those exceptions silent do not affect your application or debugging.

## 11. Acknowledgements

* [The Python Standard Library > Python Runtime Services > `contextlib` — Utilities for `with`-statement contexts](https://docs.python.org/3/library/contextlib.html)
* ["Introduction to Python Generators"](https://realpython.com/introduction-to-python-generators/) by Real Python
* ["A Gentle Introduction to Context Managers: The Pythonic Way of Managing Resources"](https://alysivji.github.io/managing-resources-with-context-managers-pythonic.html) by Aly Sivji
* ["Python with Context Managers"](https://jeffknupp.com/blog/2016/03/07/python-with-context-managers/) by Jeff Knupp
