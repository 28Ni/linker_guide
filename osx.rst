OSX
================

install name and dependant install name
----------------------------------------

During the loading of shared libraries, two important concepts are involved:

   - install name,
   - dependant install name.

**install name** are used at **link time**, and **dependant install name** are
used at **load time**.

Simillary, **install name** are not used at **load time**, and **dependant
install name** are not used at **link time**.

- When a shared library is created, the **install name** of the library is
  written in the header of the library.

- When linking an executable `MAIN` (or a shared library `MAIN`) with a shared
  library `DEP`, the **install name** of `DEP` is copied into the header of the
  executable `MAIN`, and become the **dependant install name** in `MAIN` of the dependant
  shared library `DEP` (`-L/path/to/lib/dir` to find the shared library `DEP` at
  link time)

- When the executable `MAIN` is executed, the dependent shared library `DEP` is
  searched accoring to the **dependent install name** in `MAIN`

The **install name** in both library `DEP` or executable `MAIN` can be changed
after their creation, see the section **usefull commands** bellow.

The problem with absolute paths
----------------------------------------

Why no just set the **install name** of a library to its absolute path when creating it?
Consider the typical `./configure ; make ; make install` process:


- During the `make`:
    - library would be created with **install name**
      `/tmp/build/lib/libfoo.dylib`
    - executable is linked with library, and **install name** in executable would
      be `/tmp/build/lib/libfoo.dylib`

- During the `make install`:
    - library and executable are move to `/usr/local/lib` and `/usr/local/bin`.

Now, when executable is executed, it would search library in
`/tmp/build/lib/libfoo.dylib`, and  would not find it.

Moreover, creating new libraries that links to `/tmp/build/lib/libfoo.dylib`
would inherit the wrong **install name** `/tmp/build/lib/libfoo.dylib`, and
fails to be executed too.

A solution is to update all **dependant install name** and **install name** (using
`install_name_tool` after the `make install`), which works.

However, if libraries and executable are moved again (for example, installing it in
a conda environment), all absolute paths must be updated again.

Using relative path is a more general approch which avoid having to update
**install name** and **dependant install name**.

Using relative paths
----------------------------------------

Using relative paths for **dependant install name** and **install name** allows **relocation**.

Consider the following directories and files:

.. code-block:: bash

    # install name          : ../lib/libfoo.dylib
    /somewhere/lib/libfoo.dylib

    # dependant install name: ../lib/libfoo.dylib
    /somewhere/bin/xfoo

Where invoking `xfoo`, `libfoo.dylib` would be searched in
`../lib/libfoo.dylib`, but relatively to where `xfoo` is invoked from:

.. code-block:: bash

    # Success: libfoo.dylib searched in /somewhere/bin/../lib/libfoo.dylib
    cd /somewhere/bin
    ./xfoo

    # Failure: libfoo.dylib searched in /anotherplace/../lib/libfoo.dylib
    cd /anotherplace

Special variables
----------------------------------------

OSX provides several options to fix this:

@executable_path
^^^^^^^^^^^^^^^^

Setting the **install name** to `@executable_path/../lib` will always replace
`@executable_path` with absolute path of the executable, independently of where
it is invoked from.

@loader_path
^^^^^^^^^^^^^^^^

`@loader_path` is the same thing, but works also in the case when shared
library `MAIN` is loaded and searches for a dependant shared library `DEP`.

@rpath
^^^^^^^^^^^^^^^^

`@rpath` allow to give to the executable whatever path we want to search
library in.

First @rpath example
""""""""""""""""""""""

A library is installed somewhere, user want to create `yfoo`, which links with
`libfoo.dylib`.  Using **@loader** path in **install name** and **dependant
intall name** `yfoo` fails:

.. code-block:: bash

    # install name          : @loader_path/../lib/libfoo.dylib
    /somewhere/lib/libfoo.dylib

    # Success:
    # dependant install name: @loader_path/../lib/libfoo.dylib
    /somewhere/bin/xfoo

    # Failure:
    # dependant install name: @loader_path/../lib/libfoo.dylib
    # library is search in /home/user/bin/../lib/libfoo.dylib
    /home/user/bin/yfoo

Using **@rpath** in **install name** allows to define at link time whaever
value we want for **@rpath** in the **dependant install name**. This can be
`@loadpath/../lib` if the executable is relative to the library, or an absolute
path otherwise:

.. code-block:: bash

    # install name          : @rpath/libfoo.dylib
    /somewhere/lib/libfoo.dylib

    # dependant install name: @loader_path/../libfoo.dylib
    # Linker flag used: -Wl,-rpath=@loader_path/..
    /somewhere/bin/xfoo

    # dependant install name: /somewhere/lib//libfoo.dylib
    # Linker flag used: -Wl,-rpath=/somewhere/lib/
    /home/user/bin/yfoo


Second @rpath example
""""""""""""""""""""""

Two different version of `libfoo.dylib` is instelled in different directories,
and they use @rpath in their **install name**:

.. code-block:: bash
 
     # install name is @rpath/libfoo.dylib
     /somewhere/lib/libfoo.dylib

     # install name is @rpath/libfoo.dylib
     /anotherplace/lib/libfoo.dylib

     # Success:
     # dependant install name is /somewhere/lib/libfoo.dylib
     # Linker flag used: -Wl,-rpath=/somewhere/lib/
     /home/user/bin/xfoo

     # or

     # Success:
     # dependant install name is /antherplace/lib/libfoo.dylib
     # Linker flag used: -Wl,-rpath=/anotherplace/lib/
     /home/user/bin/xfoo


Usefull commands
------------------------
 
Setting the **install name** of a libary at its creation:

.. code-block:: bash
 
     clang <sources> -dynamiclib -install_name <install name> -o lib<name>.dylib
 

Print **install name** of a shared library:
 
.. code-block:: bash

     otool -D <library>
 

Lists dependent dynamics libraries and its **dependant install name**:

.. code-block:: bash
 
     otool -L <excutable or library>
 

Show the **rpath**:
 
.. code-block:: bash

     otool -l <executable or library> # look at the section LC_RPATH
 

Change the **install name** of a library:
 
.. code-block:: bash

     install_name_tool -id /new/install/name /path/to/lib<name>.dylib
 
Change **dependant install name** of a dependent library:
 
.. code-block:: bash

     install_name_tool -change old/path/libdep.so new/path/libdep.so libmain.dylib
 
`install_name_tool` also has options to modify or add or delete **rpath** in a
executable or shared library.

Note that environment variable can shortcuts the use of the **install name** of the dependent
shared library, typically using `DYLD_LIBRARY_PATH`.

When linking a executable the a shared library, there seems to be no
`clang`/`dyld` options to specify a dependent shared library **install name**
different that the one in the shared library linked.
