dill
====
serialize all of Python

About Dill
----------
``dill`` extends Python's ``pickle`` module for serializing and de-serializing
Python objects to the majority of the built-in Python types. Serialization
is the process of converting an object to a byte stream, and the inverse
of which is converting a byte stream back to a Python object hierarchy.

``dill`` provides the user the same interface as the ``pickle`` module, and
also includes some additional features. In addition to pickling Python
objects, ``dill`` provides the ability to save the state of an interpreter
session in a single command.  Hence, it would be feasible to save an
interpreter session, close the interpreter, ship the pickled file to
another computer, open a new interpreter, unpickle the session and
thus continue from the 'saved' state of the original interpreter
session.

``dill`` can be used to store Python objects to a file, but the primary
usage is to send Python objects across the network as a byte stream.
``dill`` is quite flexible, and allows arbitrary user defined classes
and functions to be serialized.  Thus ``dill`` is not intended to be
secure against erroneously or maliciously constructed data. It is
left to the user to decide whether the data they unpickle is from
a trustworthy source.

``dill`` is part of ``pathos``, a Python framework for heterogeneous computing.
``dill`` is in active development, so any user feedback, bug reports, comments,
or suggestions are highly appreciated.  A list of issues is located at
https://github.com/uqfoundation/dill/issues, with a legacy list maintained at
https://uqfoundation.github.io/project/pathos/query.


Major Features
--------------
``dill`` can pickle the following standard types:

* none, type, bool, int, float, complex, bytes, str,
* tuple, list, dict, file, buffer, builtin,
* Python classes, namedtuples, dataclasses, metaclasses,
* instances of classes,
* set, frozenset, array, functions, exceptions

``dill`` can also pickle more 'exotic' standard types:

* functions with yields, nested functions, lambdas,
* cell, method, unboundmethod, module, code, methodwrapper,
* methoddescriptor, getsetdescriptor, memberdescriptor, wrapperdescriptor,
* dictproxy, slice, notimplemented, ellipsis, quit

``dill`` cannot yet pickle these standard types:

* frame, generator, traceback

``dill`` also provides the capability to:

* save and load Python interpreter sessions
* save and extract the source code from functions and classes
* interactively diagnose pickling errors


Current Release
[![Downloads](https://static.pepy.tech/personalized-badge/dill?period=total&units=international_system&left_color=grey&right_color=blue&left_text=pypi%20downloads)](https://pepy.tech/project/dill)
[![Conda Downloads](https://img.shields.io/conda/dn/conda-forge/dill?color=blue&label=conda%20downloads)](https://anaconda.org/conda-forge/dill)
[![Stack Overflow](https://img.shields.io/badge/stackoverflow-get%20help-black.svg)](https://stackoverflow.com/questions/tagged/dill)
---------------
The latest released version of ``dill`` is available from:
    https://pypi.org/project/dill

``dill`` is distributed under a 3-clause BSD license.


Development Version
[![Support](https://img.shields.io/badge/support-the%20UQ%20Foundation-purple.svg?style=flat&colorA=grey&colorB=purple)](http://www.uqfoundation.org/pages/donate.html)
[![Documentation Status](https://readthedocs.org/projects/dill/badge/?version=latest)](https://dill.readthedocs.io/en/latest/?badge=latest)
[![Build Status](https://app.travis-ci.com/uqfoundation/dill.svg?label=build&logo=travis&branch=master)](https://app.travis-ci.com/github/uqfoundation/dill)
[![codecov](https://codecov.io/gh/uqfoundation/dill/branch/master/graph/badge.svg?token=rgWkSxLxPW)](https://codecov.io/gh/uqfoundation/dill)
-------------------
You can get the latest development version with all the shiny new features at:
    https://github.com/uqfoundation

If you have a new contribution, please submit a pull request.


Installation
------------
``dill`` can be installed with ``pip``::

    $ pip install dill

To optionally include the ``objgraph`` diagnostic tool in the install::

    $ pip install dill[graph]

To optionally include the ``gprof2dot`` diagnostic tool in the install::

    $ pip install dill[profile]

For windows users, to optionally install session history tools::

    $ pip install dill[readline]


Requirements
------------
``dill`` requires:

* ``python`` (or ``pypy``), **>=3.9**
* ``setuptools``, **>=42**

Optional requirements:

* ``objgraph``, **>=1.7.2**
* ``gprof2dot``, **>=2022.7.29**
* ``pyreadline``, **>=1.7.1** (on windows)


Basic Usage
-----------
``dill`` is a drop-in replacement for ``pickle``. Existing code can be
updated to allow complete pickling using::

    >>> import dill as pickle

or::

    >>> from dill import dumps, loads

``dumps`` converts the object to a unique byte string, and ``loads`` performs
the inverse operation::

    >>> squared = lambda x: x**2
    >>> loads(dumps(squared))(3)
    9

There are a number of options to control serialization which are provided
as keyword arguments to several ``dill`` functions:

* with *protocol*, the pickle protocol level can be set. This uses the
  same value as the ``pickle`` module, *DEFAULT_PROTOCOL*.
* with *byref=True*, ``dill`` to behave a lot more like pickle with
  certain objects (like modules) pickled by reference as opposed to
  attempting to pickle the object itself.
* with *recurse=True*, objects referred to in the global dictionary are
  recursively traced and pickled, instead of the default behavior of
  attempting to store the entire global dictionary.
* with *fmode*, the contents of the file can be pickled along with the file
  handle, which is useful if the object is being sent over the wire to a
  remote system which does not have the original file on disk. Options are
  *HANDLE_FMODE* for just the handle, *CONTENTS_FMODE* for the file content
  and *FILE_FMODE* for content and handle.
* with *ignore=False*, objects reconstructed with types defined in the
  top-level script environment use the existing type in the environment
  rather than a possibly different reconstructed type.

The default serialization can also be set globally in *dill.settings*.
Thus, we can modify how ``dill`` handles references to the global dictionary
locally or globally::

    >>> import dill.settings
    >>> dumps(absolute) == dumps(absolute, recurse=True)
    False
    >>> dill.settings['recurse'] = True
    >>> dumps(absolute) == dumps(absolute, recurse=True)
    True

``dill`` also includes source code inspection, as an alternate to pickling::

    >>> import dill.source
    >>> print(dill.source.getsource(squared))
    squared = lambda x:x**2

To aid in debugging pickling issues, use *dill.detect* which provides
tools like pickle tracing::

    >>> import dill.detect
    >>> with dill.detect.trace():
    >>>     dumps(squared)
    ┬ F1: <function <lambda> at 0x7fe074f8c280>
    ├┬ F2: <function _create_function at 0x7fe074c49c10>
    │└ # F2 [34 B]
    ├┬ Co: <code object <lambda> at 0x7fe07501eb30, file "<stdin>", line 1>
    │├┬ F2: <function _create_code at 0x7fe074c49ca0>
    ││└ # F2 [19 B]
    │└ # Co [87 B]
    ├┬ D1: <dict object at 0x7fe0750d4680>
    │└ # D1 [22 B]
    ├┬ D2: <dict object at 0x7fe074c5a1c0>
    │└ # D2 [2 B]
    ├┬ D2: <dict object at 0x7fe074f903c0>
    │├┬ D2: <dict object at 0x7fe074f8ebc0>
    ││└ # D2 [2 B]
    │└ # D2 [23 B]
    └ # F1 [180 B]

With trace, we see how ``dill`` stored the lambda (``F1``) by first storing
``_create_function``, the underlying code object (``Co``) and ``_create_code``
(which is used to handle code objects), then we handle the reference to
the global dict (``D2``) plus other dictionaries (``D1`` and ``D2``) that
save the lambda object's state. A ``#`` marks when the object is actually stored.


More Information
----------------
Probably the best way to get started is to look at the documentation at
http://dill.rtfd.io. Also see ``dill.tests`` for a set of scripts that
demonstrate how ``dill`` can serialize different Python objects. You can
run the test suite with ``python -m dill.tests``. The contents of any
pickle file can be examined with ``undill``.  As ``dill`` conforms to
the ``pickle`` interface, the examples and documentation found at
http://docs.python.org/library/pickle.html also apply to ``dill``
if one will ``import dill as pickle``. The source code is also generally
well documented, so further questions may be resolved by inspecting the
code itself. Please feel free to submit a ticket on github, or ask a
question on stackoverflow (**@Mike McKerns**).
If you would like to share how you use ``dill`` in your work, please send
an email (to **mmckerns at uqfoundation dot org**).


Citation
--------
If you use ``dill`` to do research that leads to publication, we ask that you
acknowledge use of ``dill`` by citing the following in your publication::

    M.M. McKerns, L. Strand, T. Sullivan, A. Fang, M.A.G. Aivazis,
    "Building a framework for predictive science", Proceedings of
    the 10th Python in Science Conference, 2011;
    http://arxiv.org/pdf/1202.1056

    Michael McKerns and Michael Aivazis,
    "pathos: a framework for heterogeneous computing", 2010- ;
    https://uqfoundation.github.io/project/pathos

Please see https://uqfoundation.github.io/project/pathos or
http://arxiv.org/pdf/1202.1056 for further information.

