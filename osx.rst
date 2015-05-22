OSX
================

install name 
----------------------------------------

When creating a library `libdep.dylib`, the **install name** of the library is
embedded in binary file `libdep.dylib`.

When linking an executable `main` (or a library `libmain.dylib`) with a
library `libdep.dylib` it depends on, the **install name** of `libdep.dylib`
is copied in the executable `main` (or library `libmain.dylib`).

When loading an executable `main` (or a library `libmain.dylib`), the
dependent library `libdep.dylib` is searched accoring to the **install name**
embbedded in `main` (or `libmain.dylib`).

**install name** are used **only** at **link time**, while **install name** of
dependant libraries are used **only** at **load time**.

Install name with absolute paths
----------------------------------------

The simple case in when install name of a dependant library is an absolute
path, for example:

.. code-block:: bash

    # install name          : /somewhere/lib/libfoo.dylib
    /somewhere/lib/libfoo.dylib

    # Success:
    # dependant install name: /somewhere/lib/libfoo.dylib
    /somewhere/bin/xfoo

If the library in moved to another path, load the executable fails.

.. code-block:: bash

    # install name          : /somewhere/lib/libfoo.dylib
    /anotherplace/lib/libfoo.dylib

    # Failure:
    # dependant install name: /somewhere/lib/libfoo.dylib
    /somewhere/bin/xfoo

The solution is to update the dependant name in `libfoo.dylib` (for futur
link), and the install name of the dependant library in `xfoo`.

Install name with relative paths
----------------------------------------

Consider the following directories and files:

.. code-block:: bash

    # install name          : ../lib/libfoo.dylib
    /somewhere/lib/libfoo.dylib

    # dependant install name: ../lib/libfoo.dylib
    /somewhere/bin/xfoo

When invoking `xfoo`, `libfoo.dylib` is searched in `../lib/libfoo.dylib`, but
relatively to where `xfoo` is invoked from:

.. code-block:: bash

    # Success: libfoo.dylib searched in /somewhere/bin/../lib/libfoo.dylib
    cd /somewhere/bin
    ./xfoo

    # Failure: libfoo.dylib searched in /anotherplace/../lib/libfoo.dylib
    cd /anotherplace
    ./xfoo


Install name with @executable_path
----------------------------------------

If **install name** starts with `@executable_path`,  `@executable_path` is
replace at **load time** with absolute path of the executable, independently
of where it is invoked from.

.. code-block:: bash

    # install name              : @executable_path/../lib/libfoo.dylib
    /somewhere/foo/lib/libfoo.dylib

    # dependant install name    : @executable_path/../lib/libfoo.dylib
    /somewhere/foo/bin/xfoo

    # Success.
    cd /anotherplace
    ./xfoo

The `foo` directory can be moved to `elsewhere`, loading `xfoo` will still
find `libfoo.dylib`.


@loader_path
^^^^^^^^^^^^^^^^

`@loader_path` is the same thing, but works also in the case when shared
library `MAIN` is loaded and searches for a dependant shared library `DEP`.

@rpath
^^^^^^^^^^^^^^^^

If in `main`, **install name** of the dependent library is
**@rpath/path/to/libdep.dylib**, when loading `main`, `path/to/libdep.dylib`
is searched in a list of `rpath` embedded `main`.

A library is installed somewhere, user want to create `yfoo`, which links with
`libfoo.dylib`.  Using **@loader** path in **install name** and **dependant
intall name** `yfoo` fails:

.. code-block:: bash

    # install name          : @rpath/libfoo.dylib
    /somewhere/lib/libfoo.dylib

    # Success:
    # dependant install name: @rpath/libfoo.dylib
    # rpath: ['@loader_path/../lib']
    /somewhere/bin/xfoo

    # Success:
    # dependant install name: @rpath/libfoo.dylib
    # rpath: ['/somewhere/lib/']
    /home/user/yfoo

    # Failure, rpath is ignore at @rpath is not in dependant install name
    # dependant install name: libfoo.dylib
    # rpath: ['/somewhere/lib/']
    /home/user/yfoo


Usefull commands
------------------------

Linking with a library:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`-L` is used to search a library at **link time only**.

.. code-block:: bash

    clang++ -o <library or executable> -L/path/to/lib/dir -l<name> <sources>

install name
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 
Setting the **install name** of a libary at its creation (by default, it is
`lib<name>.dylib`:

.. code-block:: bash
 
     clang++ -shared -install_name <install name> -o lib<name>.dylib <sources>

Print **install name** of a shared library:
 
.. code-block:: bash

     otool -D <library>

Change the **install name** of a library:
 
.. code-block:: bash

     install_name_tool -id /new/install/name /path/to/lib<name>.dylib

dependant install name
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 
Print dependent dynamics libraries and its **dependant install name**:

.. code-block:: bash
 
     otool -L <excutable or library>
 
 
Change **dependant install name** of a dependent library:
 
.. code-block:: bash

     install_name_tool -change old/path/libdep.so new/path/libdep.so libmain.dylib

rpath
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Set the rpath list a executable or library creation:

.. code-block:: bash

    clang++ -o <executable or library> -Wl,-rpath,<path0>, -Wl,-rpath<path1> ... <sources

Print the **LC_RPATH**:
 
.. code-block:: bash

     otool -l <executable or library> # look at the section LC_RPATH

Add **LC_RPATH** to executable or library:

.. code-block:: bash

    install_name_tool -add_rpath /some/path <executable or library>

Delete **LC_RPATH** of executable or library:

.. code-block:: bash

    install_name_tool -delete_rpath /some/path <executable or library>

Modifiy **LC_RPATH** of executable or library:

.. code-block:: bash

    install_name_tool -rpath /some/path <executable or library>
 
Notes
------------------------

Note that environment variable can shortcuts the use of the **install name** of
the dependent shared library, typically using `DYLD_LIBRARY_PATH`.

When linking a executable with a shared library, there seems to be no
`clang++`/`dyld` options to specify a dependent shared library **install name**
different that the one in the shared library linked.
