Compiling nekRS
===============

This page gives a variety of information to help compile the software 
successfully how to build nekRS in a general sense, including a brief 
explanation of the 3rd party dependencies that might effect how this is 
conducted.

Requirements
------------

You will require the following to compile nekRS.

* Linux, Mac OS X (Microsoft WSL and Windows is not supported) 
* C++17/C99 compatible compiler 

    * I.E. GNU (needs to be >=9.1), IntelLLVM, Clang, ARMClang, AppleClang 
      or NVHPC are recognised
* GNU/Intel/NVHPC Fortran compiler
* MPI-3.1 or later
* CMake version 3.18 or later 

Dependencies
""""""""""""

This section will outline the main dependencies that nekRS uses and how they 
might be affected by your environment. If enabled, they are compiled as part of 
the nekRS build process from the 
`3rd_party <https://github.com/Nek5000/nekRS/tree/master/3rd_party>`__ directory.

+------------+----------+---------------------------------------------------------------------------------------------+-------------------------+--------------------------------------+
| Dependency | Optional |                                         Description                                         | Additional Requirements |            (Github) Link             |
+============+==========+=============================================================================================+=========================+======================================+
| AMGX       | ???      | GPU accelerated core solver library                                                         | CUDA                    | https://github.com/NVIDIA/AMGX       |
+------------+----------+---------------------------------------------------------------------------------------------+-------------------------+--------------------------------------+
| CVODE      | ???      | solver for stiff and nonstiff ordinary differential equation (ODE) systems form y' = f(t,y) | ???                     | https://github.com/LLNL/sundials     |
+------------+----------+---------------------------------------------------------------------------------------------+-------------------------+--------------------------------------+
| GSlib      | ???      | ???                                                                                         |                         | https://github.com/Nek5000/gslib     |
+------------+----------+---------------------------------------------------------------------------------------------+-------------------------+--------------------------------------+
| HYPRE      | ???      | ???                                                                                         |                         | https://github.com/hypre-space/hypre |
+------------+----------+---------------------------------------------------------------------------------------------+-------------------------+--------------------------------------+
| nek5000    | ???      | Fortran predecessor to nekRS                                                                |                         | https://github.com/Nek5000/Nek5000   |
+------------+----------+---------------------------------------------------------------------------------------------+-------------------------+--------------------------------------+
| OCCA       | ???      | portable, and vendor neutral framework for parallel programming on heterogeneous platforms  |                         | https://github.com/libocca/occa      |
+------------+----------+---------------------------------------------------------------------------------------------+-------------------------+--------------------------------------+

Acquiring the code
""""""""""""""""""

You will typically want to either clone the repository from `github <https://github.com/Nek5000/nekRS>`__.

.. code-block:: 

    user$ git clone https://github.com/Nek5000/nekRS.git
    user$ cd nekRS

or download a release

.. code-block::

    user$ wget https://github.com/Nek5000/nekRS/archive/refs/tags/v23.0.tar.gz
    user$ tar -xzvf v23.0.tar.gz
    user$ cd nekRS-23.0

Set NEKRS_HOME
--------------

Next, set the ``NEKRS_HOME`` environment variable to a location in your file
system where you would like to place the executables and other build files.
For example, this can be:

.. code-block::

    user$ export NEKRS_HOME=$HOME/.local/nekrs

Then, be sure to add this directory to your path:

.. code-block::

    user$ export PATH=${NEKRS_HOME}:${PATH}

To avoid repeating these steps for every new shell, you may want to add these environment
variable settings in a ``.bashrc``.

Cmake compilation
-----------------

Once within the nekRS directory, the default way to compile the code is through 
the nrsconfig helper script, appended with setting the variables for the c++ and 
fortran compilers. When run this will run CMake configure and then 
install (by confirmed with pressing enter after the summary).

.. code-block:: console

  $ CC=mpicc CXX=mpic++ FC=mpif77 ./nrsconfig
  cmake -S . -B build -Wfatal-errors
  -- Found MPI_C: /usr/local/lib/libmpi.so (found version "3.1") 
  -- Found MPI_CXX: /usr/local/bin/mpic++ (found version "3.1") 
  -- Found MPI_Fortran: /usr/local/lib/libmpi_usempif08.so (found version "3.1") 
  -- Found MPI: TRUE (found version "3.1")  
  .
  .
  .
  ----------------- Summary -----------------
  Installation directory: /home/abc/.local/nekrs
  C compiler: /usr/bin/cc
  C++ compiler: /usr/local/bin/mpic++
  Fortran compiler: /usr/bin/gfortran
  Default backend : SERIAL
  CPU backend compiler: /usr/bin/g++ (flags: -w -O3 -g -march=native -mtune=native -ffast-math)
  GPU aware MPI support: OFF
  -------------------------------------------

CMake flags
"""""""""""

Depending on your situation you may wish to customise the flags that are passed 
to CMake to compile the code.

.. code-block:: console

    CC=mpicc CXX=mpic++ FC=mpif77 ./nrsconfig -DENABLE_CVODE=ON -DENABLE_HYPRE_GPU=ON

This section details the different flags that can be provided to cmake to 
customise the build process. The features flags that are set to be on by 
default have their dependencies checked by the configure process and will be
disabled if not present (I.E. :term:`CUDA`, :term:`HIP` and :term:`DPC++` 
support will be automatically customised based on the system). Due to this, and 
that flags for the Just in Time compiler are set 
(see :ref:`just_in_time_compilation`), it is important to run the configure 
process in an environment that is representative of where you will run the final
program.

GPU support
"""""""""""

+-------------------+-----------------------------------------------------+---------+
|       Flag        |                     Description                     | Default |
+===================+=====================================================+=========+
| ``ENABLE_CUDA``   | Enables NVIDIA :term:`CUDA` :term:`GPU` support     | ON      |
+-------------------+-----------------------------------------------------+---------+
| ``ENABLE_HIP``    | Enables :term:`AMD` :term:`HIP` :term:`GPU` support | ON      |
+-------------------+-----------------------------------------------------+---------+
| ``ENABLE_DPCPP``  | Enables Intel :term:`DPC++` :term:`GPU` support     | ON      |
+-------------------+-----------------------------------------------------+---------+
| ``ENABLE_OPENCL`` | Enable Khronos :term:`OpenCL` support               | **OFF** |
+-------------------+-----------------------------------------------------+---------+
| ``ENABLE_METAL``  | Enable Apple Metal support                          | **OFF** |
+-------------------+-----------------------------------------------------+---------+
| ``NEKRS_GPU_MPI`` | Enable :term:`GPU` aware :term:`MPI`                | ON      |
+-------------------+-----------------------------------------------------+---------+

Optional features
"""""""""""""""""
+----------------------+----------------------------+---------+------------------------------------------------------------+
|         Flag         |        Description         | Default |                           Notes                            |
+======================+============================+=========+============================================================+
| ``ENABLE_HYPRE_GPU`` | Enable HYPRE GPU support   | **OFF** | Requires CUDA toolkit version >= 11 and < 12               |
+----------------------+----------------------------+---------+------------------------------------------------------------+
| ``ENABLE_AMGX``      | Enable NVIDIA AMGX support | **OFF** | Requires CUDA (I.E. ``ENABLE_CUDA`` to evaluate correctly) |
+----------------------+----------------------------+---------+------------------------------------------------------------+
| ``ENABLE_CVODE``     | Enable CVODE support       | **OFF** | Unsupported with ``OCCA_OPENCL_ENABLED``,                  |
|                      |                            |         | ``OCCA_DPCPP_ENABLED`` or ``OCCA_HIP_ENABLED``             |
+----------------------+----------------------------+---------+------------------------------------------------------------+
