Frequently asked questions
##########################

(under construction)

Limitations involving reference arguments
=========================================

In C++, it's fairly common to pass arguments using mutable references or
mutable pointers, which allows both read and write access to the value
supplied by the caller. This is sometimes done for efficiency reasons, or to
realize functions that have multiple return values. Here are two very basic
examples:

.. code-block:: cpp

    void increment(int &i) { i++; }
    void increment_ptr(int *i) { (*i)++; }

In Python, all arguments are passed by reference, so there is no general
issue in binding such code from Python.

However, certain basic Python types (like ``str``, ``int``, ``bool``,
``float``, etc.) are **immutable**. This means that the following attempt
to port the function to Python doesn't have the same effect on the value
provided by the caller -- in fact, it does nothing at all.

.. code-block:: python

    def increment(i):
        i += 1 # nope..

pybind11 is also affected by such language-level conventions, which means that
binding ``increment`` or ``increment_ptr`` will also create Python functions
that don't modify their arguments.

Although inconvenient, one workaround is to encapsulate the immutable types in
a custom type that does allow modifications. 

An other alternative involves binding a small wrapper lambda function that
returns a tuple with all output arguments (see the remainder of the
documentation for examples on binding lambda functions). An example:

.. code-block:: cpp

    int foo(int &i) { i++; return 123; }

and the binding code

.. code-block:: cpp

   m.def("foo", [](int i) { int rv = foo(i); return std::make_tuple(rv, i); });

Working with ancient Visual Studio 2009 builds on Windows
=========================================================

The official Windows distributions of Python are compiled using truly
ancient versions of Visual Studio that lack good C++11 support. Some users
implicitly assume that it would be impossible to load a plugin built with
Visual Studio 2015 into a Python distribution that was compiled using Visual
Studio 2009. However, no such issue exists: it's perfectly legitimate to
interface DLLs that are built with different compilers and/or C libraries.
Common gotchas to watch out for involve not ``free()``-ing memory region
that that were ``malloc()``-ed in another shared library, using data
structures with incompatible ABIs, and so on. pybind11 is very careful not
to make these types of mistakes.
