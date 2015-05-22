GNU/Linux
===============================

Note: everything mentionned for loading an executable `main` is also valid for
loading a library `libmain.so`.

soname
----------------------------------------

When creating a library `libdep.so`, the **soname** of the library is embedded
in binary file `libdep.so`.

When linking an executable `main` with a library `libdep.so` it depends on,
the **soname** of `libdep.dylib` is copied in the executable `main`.

When loading an executable `main`, the dependent library `libdep.so` is
searched accoring to the **soname** embbedded in `main`.

**soname** are used **only** at **link time**, while **soname** of dependant
libraries are used **only** at **load time**.

relative paths
----------------------------------------

Is **soname** is a relative path, it it search in the **rpath** list embedded
in the executable.

$ORIGIN
----------------------------------------

If **soname** starts with **$ORIGIN**, **ORIGIN** is replaced at load time
with the absolute path of the executable.

Usefull commands
------------------------

soname
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Set **soname** of a library at its creation:

.. code-block:: bash

     g++ -shared -o lib<name>.so -Wl,-soname,<soname> <sources>

Print **soname** of a shared library:

.. code-block:: bash

    objdump -p <library> | grep SONAME

dependant soname
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Evaluate where dependent dynamics libraries are found:

.. code-block:: bash

    ldd <excutable or library>

rpath
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Set the **rpath list** at link time:

.. code-block:: bash

   g++ -o <executable of library> -l... -L... -Wl,-rpath,/path1 -Wl,-rpath,/path2

Print the **rpath list**:

.. code-block:: bash

    readelf -d <executable or library> | grep rpath
