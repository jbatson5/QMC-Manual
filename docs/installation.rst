.. _obtaininginstalling:

Obtaining, installing, and validating QMCPACK
=============================================

This chapter describes how to obtain, build, and validate QMCPACK. This
process is designed to be as simple as possible and should be no harder
than building a modern plane-wave density functional theory code such as
Quantum ESPRESSO, QBox, or VASP. Parallel builds enable a complete
compilation in under 2 minutes on a fast multicore system. If you are
unfamiliar with building codes we suggest working with your system
administrator to install QMCPACK.

Installation steps
------------------

To install QMCPACK, follow the steps below. Full details of each step
are given in the referenced sections.

#. Download the source code from :ref:`obrelease`
   or :ref:`obdevelopment`.

#. Verify that you have the required compilers, libraries, and tools
   installed (:ref:`prerequisites`).

#. If you will use Quantum ESPRESSO, download and patch it. The patch
   adds the pw2qmcpack utility
   (:ref:`buildqe`).

#. Run the cmake configure step and build with make
   (:ref:`cmake` and :ref:`cmakequick`). Examples for
   common systems are given in
   :ref:`installexamples`.

#. Run the tests to verify QMCPACK
   (:ref:`testing`).

#. Build the ppconvert utility in QMCPACK
   (:ref:`buildppconvert`).

Hints for high performance are in
:ref:`buildperformance`.
Troubleshooting suggestions are in
:ref:`troubleshoot`.

Note that there are two different QMCPACK executables that can be
produced: the general one, which is the default, and the “complex”
version, which supports periodic calculations at arbitrary twist angles
and k-points. This second version is enabled via a cmake configuration
parameter (see :ref:`cmakeoptions`).
The general version supports only wavefunctions that can be made real.
If you run a calculation that needs the complex version, QMCPACK will
stop and inform you.

.. _obrelease:

Obtaining the latest release version
------------------------------------

Major releases of QMCPACK are distributed from http://www.qmcpack.org.
Because these versions undergo the most testing, we encourage using them
for all production calculations unless there are specific reasons not to
do so.

Releases are usually compressed tar files that indicate the version
number, date, and often the source code revision control number
corresponding to the release. To obtain the latest release:

-  Download the latest QMCPACK distribution from http://www.qmcpack.org.

-  Untar the archive (e.g., ``tar xvf qmcpack_v1.3.tar.gz``).

Releases can also be obtained from the ‘master’ branch of the QMCPACK
git repository, similar to obtaining the development version
(:ref:`obdevelopment`).

.. _obdevelopment:

Obtaining the latest development version
----------------------------------------

The most recent development version of QMCPACK can be obtained
anonymously via
::

  git clone https://github.com/QMCPACK/qmcpack.git

Once checked out, updates can be made via the standard ``git pull``.

The ‘develop’ branch of the git repository contains the day-to-day
development source with the latest updates, bug fixes, etc. This version
might be useful for updates to the build system to support new machines,
for support of the latest versions of Quantum ESPRESSO, or for updates
to the documentation. Note that the development version might not be
fully consistent with the online documentation. We attempt to keep the
development version fully working. However, please be sure to run tests
and compare with previous release versions before using for any serious
calculations. We try to keep bugs out, but occasionally they crawl in!
Reports of any breakages are appreciated.

.. _prerequisites:

Prerequisites
-------------

The following items are required to build QMCPACK. For workstations,
these are available via the standard package manager. On shared
supercomputers this software is usually installed by default and is
often accessed via a modules environment—check your system
documentation.

**Use of the latest versions of all compilers and libraries is strongly
encouraged** but not absolutely essential. Generally, newer versions are
faster; see :ref:`buildperformance`
for performance suggestions.

-  C/C++ compilers such as GNU, Clang, Intel, and IBM XL. C++ compilers
   are required to support the C++ 14 standard. Use of recent (“current
   year version”) compilers is strongly encouraged.

-  An MPI library such as OpenMPI (http://open-mpi.org) or a
   vendor-optimized MPI.

-  BLAS/LAPACK, numerical, and linear algebra libraries. Use
   platform-optimized libraries where available, such as Intel MKL.
   ATLAS or other optimized open source libraries can also be used
   (http://math-atlas.sourceforge.net).

-  CMake, build utility (http://www.cmake.org).

-  Libxml2, XML parser (http://xmlsoft.org).

-  HDF5, portable I/O library (http://www.hdfgroup.org/HDF5/). Good
   performance at large scale requires parallel version :math:`>=` 1.10.

-  BOOST, peer-reviewed portable C++ source libraries
   (http://www.boost.org). Minimum version is 1.61.0.

-  FFTW, FFT library (http://www.fftw.org/).

To build the GPU accelerated version of QMCPACK, an installation of
NVIDIA CUDA development tools is required. Ensure that this is
compatible with the C and C++ compiler versions you plan to use.
Supported versions are included in the NVIDIA release notes.

Many of the utilities provided with QMCPACK use Python (v2). The numpy
and matplotlib libraries are required for full functionality.

Note that the standalone einspline library used by previous versions of
QMCPACK is no longer required. A more optimized version is included
inside. The standalone version should *not* be on any standard search
paths because conflicts between the old and new include files can
result.

C++ 14 standard library
-----------------------

The C++ standard consists of language features—which are implemented in
the compiler—and library features—which are implemented in the standard
library. GCC includes its own standard library and headers, but many
compilers do not and instead reuse those from an existing GCC install.
Depending on setup and installation, some of these compilers might not
default to using a GCC with C++ 14 headers (e.g., GCC 4.8 is common as a
base system compiler, but its standard library only supports C++ 11).

The symptom of having header files that do not support the C++ 14
standard is usually compile errors involving standard include header
files. Look for the GCC library version, which should be present in the
path to the include file in the error message, and ensure that it is 5.0
or greater. To avoid these errors occurring at compile time, QMCPACK
tests for a C++ 14 standard library during configuration and will halt
with an error if one is not found.

At sites that use modules, running is often sufficient to load a newer
GCC and resolve the issue.

Intel compiler
~~~~~~~~~~~~~~

The Intel compiler version must be 18 or newer. The version 17 compiler
cannot compile some of the C++ 14 constructs in the code.

If a newer GCC is needed, the ``-cxxlib`` option can be used to point to a different
GCC installation. (Alternately, the ``-gcc-name`` or ``-gxx-name`` options can be used.) Be sure to
pass this flag to the C compiler in addition to the C++ compiler. This
is necessary because CMake extracts some library paths from the C
compiler, and those paths usually also contain to the C++ library. The
symptom of this problem is C++ 14 standard library functions not found
at link time.

.. _cmake:

Building with CMake
-------------------

The build system for QMCPACK is based on CMake. It will autoconfigure
based on the detected compilers and libraries. The most recent version
of CMake has the best detection for the greatest variety of systems. The
minimum required version of CMake is 3.6, which is the oldest version to
support correct application of C++ 14 flags for the Intel compiler. Most
computer installations have a sufficiently recent CMake, though it might
not be the default.

If no appropriate version CMake is available, building it from source is
straightforward. Download a version from https://cmake.org/download/ and
unpack the files. Run ``./bootstrap`` from the CMake directory, and then run ``make`` when that
finishes. The resulting CMake executable will be in the directory. The
executable can be run directly from that location.

Previously, QMCPACK made extensive use of toolchains, but the build
system has since been updated to eliminate the use of toolchain files
for most cases. The build system is verified to work with GNU, Intel,
and IBM XLC compilers. Specific compile options can be specified either
through specific environment or CMake variables. When the libraries are
installed in standard locations (e.g., /usr, /usr/local), there is no
need to set environment or CMake variables for the packages.

.. _cmakequick:

Quick build instructions (try first)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are feeling lucky and are on a standard UNIX-like system such as
a Linux workstation, the following might quickly give a working QMCPACK:

The safest quick build option is to specify the C and C++ compilers
through their MPI wrappers. Here we use Intel MPI and Intel compilers.
Move to the build directory, run CMake, and make

::

  cd build
  cmake -DCMAKE_C_COMPILER=mpiicc -DCMAKE_CXX_COMPILER=mpiicpc ..
  make -j 8

You can increase the "8" to the number of cores on your system for
faster builds. Substitute mpicc and mpicxx or other wrapped compiler names to suit
your system. For example, with OpenMPI use

::

  cd build
  cmake -DCMAKE_C_COMPILER=mpicc -DCMAKE_CXX_COMPILER=mpicxx ..
  make -j 8

If you are feeling particularly lucky, you can skip the compiler specification:

::

  cd build
  cmake ..
  make -j 8

The complexities of modern computer hardware and software systems are
such that you should check that the autoconfiguration system has made
good choices and picked optimized libraries and compiler settings
before doing significant production. That is, check the following details. We
give examples for a number of common systems in :ref:`installexamples`.

.. _envvar:

Environment variables
~~~~~~~~~~~~~~~~~~~~~

A number of environment variables affect the build.  In particular
they can control the default paths for libraries, the default
compilers, etc.  The list of environment variables is given below:

::

  CXX              C++ compiler
  CC               C Compiler
  MKL_ROOT         Path for MKL
  HDF5_ROOT        Path for HDF5
  BOOST_ROOT       Path for Boost
  FFTW_HOME        Path for FFTW

.. _cmakeoptions:

Configuration Options
~~~~~~~~~~~~~~~~~~~~~

In addition to reading the environment variables, CMake provides a
number of optional variables that can be set to control the build and
configure steps.  When passed to CMake, these variables will take
precedent over the environment and default variables.  To set them,
add -D FLAG=VALUE to the configure line between the CMake command and
the path to the source directory.

- Key QMCPACK build options

  ::

    QMC_CUDA              Enable CUDA and GPU acceleration (1:yes, 0:no)
    QMC_COMPLEX           Build the complex (general twist/k-point) version (1:yes, 0:no)
    QMC_MIXED_PRECISION   Build the mixed precision (mixing double/float) version
                          (1:yes (GPU default), 0:no (CPU default)).
                          The CPU support is experimental.
                          Use float and double for base and full precision.
                          The GPU support is quite mature.
                          Use always double for host side base and full precision
                          and use float and double for CUDA base and full precision.
    ENABLE_SOA            Enable data layout and algorithm optimizations using
                          Structure-of-Array (SoA) datatypes (1:yes (default), 0:no).
    ENABLE_TIMERS         Enable fine-grained timers (1:yes, 0:no (default)).
                          Timers are off by default to avoid potential slowdown in small
                          systems. For large systems (100+ electrons) there is no risk.

- General build options

  ::

    CMAKE_BUILD_TYPE     A variable which controls the type of build
                         (defaults to Release). Possible values are:
                         None (Do not set debug/optmize flags, use
                         CMAKE_C_FLAGS or CMAKE_CXX_FLAGS)
                         Debug (create a debug build)
                         Release (create a release/optimized build)
                         RelWithDebInfo (create a release/optimized build with debug info)
                         MinSizeRel (create an executable optimized for size)
    CMAKE_C_COMPILER     Set the C compiler
    CMAKE_CXX_COMPILER   Set the C++ compiler
    CMAKE_C_FLAGS        Set the C flags.  Note: to prevent default
                         debug/release flags from being used, set the CMAKE_BUILD_TYPE=None
                         Also supported: CMAKE_C_FLAGS_DEBUG,
                         CMAKE_C_FLAGS_RELEASE, and CMAKE_C_FLAGS_RELWITHDEBINFO
    CMAKE_CXX_FLAGS      Set the C++ flags.  Note: to prevent default
                         debug/release flags from being used, set the CMAKE_BUILD_TYPE=None
                         Also supported: CMAKE_CXX_FLAGS_DEBUG,
                         CMAKE_CXX_FLAGS_RELEASE, and CMAKE_CXX_FLAGS_RELWITHDEBINFO
    CMAKE_INSTALL_PREFIX Set the install location (if using the optional install step)
    INSTALL_NEXUS        Install Nexus alongside QMCPACK (if using the optional install step)

- Additional QMCPACK build options

  ::

    QE_BIN                    Location of Quantum Espresso binaries including pw2qmcpack.x
    QMC_DATA                  Specify data directory for QMCPACK performance and integration tests
    QMC_INCLUDE               Add extra include paths
    QMC_EXTRA_LIBS            Add extra link libraries
    QMC_BUILD_STATIC          Add -static flags to build
    QMC_SYMLINK_TEST_FILES    Set to zero to require test files to be copied. Avoids space
                              saving default use of symbolic links for test files. Useful
                              if the build is on a separate filesystem from the source, as
                              required on some HPC systems.
    QMC_VERBOSE_CONFIGURATION Print additional information during cmake configuration
                              including details of which tests are enabled.

- Intel MKL related

  ::

    ENABLE_MKL          Enable Intel MKL libraries (1:yes (default for intel compiler),
                                                  0:no (default otherwise)).
    MKL_ROOT            Path to MKL libraries (only necessary for non intel compilers
                        or intel without standard environment variables.)
                        One of the above environment variables can be used.

- libxml2 related

  ::

    LIBXML2_INCLUDE_DIR   Include directory for libxml2

    LIBXML2_LIBRARY       Libxml2 library

- HDF5 related

  ::

    HDF5_PREFER_PARALLEL 1(default for MPI build)/0, enables/disable parallel HDF5 library searching.
    ENABLE_PHDF5         1(default for parallel HDF5 library)/0, enables/disable parallel collective I/O.

- FFTW related

  ::

    FFTW_INCLUDE_DIRS   Specify include directories for FFTW
    FFTW_LIBRARY_DIRS   Specify library directories for FFTW

- CTest related

  ::

    MPIEXEC_EXECUTABLE     Specify the mpi wrapper, e.g. srun, aprun, mpirun, etc.
    MPIEXEC_NUMPROC_FLAG   Specify the number of mpi processes flag,
                           e.g. "-n", "-np", etc.
    MPIEXEC_PREFLAGS       Flags to pass to MPIEXEC_EXECUTABLE directly before the executable to run.

- LLVM/Clang Developer Options

  ::

    LLVM_SANITIZE_ADDRES  link with the Clang address sanitizer library
    LLVM_SANITIZE_MEMORY  link with the Clang memory sanitizer library

`Clang address sanitizer library <https://clang.llvm.org/docs/AddressSanitizer.html>`_

`Clang memory sanitizer library <https://clang.llvm.org/docs/MemorySanitizer.html>`_

See :ref:`LLVM-Sanitizer-Libraries` for more information.

Installation from CMake
~~~~~~~~~~~~~~~~~~~~~~~

Installation is optional. The QMCPACK executable can be run from the ``bin`` directory in the build location.
If the install step is desired, run the ``make install`` command to install the QMCPACK executable, the converter,
and some additional executables.
Also installed is the ``qmcpack.settings`` file that records options used to compile QMCPACK.
Specify the ``CMAKE_INSTALL_PREFIX`` CMake variable during configuration to set the install location.

Role of QMC\_DATA
~~~~~~~~~~~~~~~~~

QMCPACK includes a variety of optional performance and integration
tests that use research quality wavefunctions to obtain meaningful
performance and to more thoroughly test the code. The necessarily
large input files are stored in the location pointed to by QMC\_DATA (e.g., scratch or long-lived project space on a supercomputer). These
files are not included in the source code distribution to minimize
size. The tests are activated if CMake detects the files when
configured. See tests/performance/NiO/README,
tests/solids/NiO\_afqmc/README, tests/performance/C-graphite/README, and tests/performance/C-molecule/README
for details of the current tests and input files and to download them.

Currently the files must be downloaded via https://anl.box.com/s/yxz1ic4kxtdtgpva5hcmlom9ixfl3v3c.

The layout of current complete set of files is given below. If a file is missing, the appropriate performance test is skipped.

::

  QMC_DATA/C-graphite/lda.pwscf.h5
  QMC_DATA/C-molecule/C12-e48-pp.h5
  QMC_DATA/C-molecule/C12-e72-ae.h5
  QMC_DATA/C-molecule/C18-e108-ae.h5
  QMC_DATA/C-molecule/C18-e72-pp.h5
  QMC_DATA/C-molecule/C24-e144-ae.h5
  QMC_DATA/C-molecule/C24-e96-pp.h5
  QMC_DATA/C-molecule/C30-e120-pp.h5
  QMC_DATA/C-molecule/C30-e180-ae.h5
  QMC_DATA/C-molecule/C60-e240-pp.h5
  QMC_DATA/NiO/NiO-fcc-supertwist111-supershift000-S1.h5
  QMC_DATA/NiO/NiO-fcc-supertwist111-supershift000-S2.h5
  QMC_DATA/NiO/NiO-fcc-supertwist111-supershift000-S4.h5
  QMC_DATA/NiO/NiO-fcc-supertwist111-supershift000-S8.h5
  QMC_DATA/NiO/NiO-fcc-supertwist111-supershift000-S16.h5
  QMC_DATA/NiO/NiO-fcc-supertwist111-supershift000-S32.h5
  QMC_DATA/NiO/NiO-fcc-supertwist111-supershift000-S64.h5
  QMC_DATA/NiO/NiO-fcc-supertwist111-supershift000-S128.h5
  QMC_DATA/NiO/NiO-fcc-supertwist111-supershift000-S256.h5
  QMC_DATA/NiO/NiO_afm_fcidump.h5
  QMC_DATA/NiO/NiO_afm_wfn.dat
  QMC_DATA/NiO/NiO_nm_choldump.h5

Configure and build using CMake and make
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
To configure and build QMCPACK, move to build directory, run CMake, and make

::

  cd build
  cmake ..
  make -j 8

As you will have gathered, CMake encourages "out of source" builds,
where all the files for a specific build configuration reside in their
own directory separate from the source files. This allows multiple
builds to be created from the same source files, which is very useful
when the file system is shared between different systems. You can also
build versions with different settings (e.g., QMC\_COMPLEX) and
different compiler settings. The build directory does not have to be
called build---use something descriptive such as build\_machinename or
build\_complex. The ".." in the CMake line refers to the directory
containing CMakeLists.txt. Update the ".." for other build
directory locations.

Example configure and build
~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Set the environments (the examples below assume bash, Intel compilers, and MKL library)

  ::

    export CXX=icpc
    export CC=icc
    export MKL_ROOT=/usr/local/intel/mkl/10.0.3.020
    export HDF5_ROOT=/usr/local
    export BOOST_ROOT=/usr/local/boost
    export FFTW_HOME=/usr/local/fftw

- Move to build directory, run CMake, and make

  ::

    cd build
    cmake -D CMAKE_BUILD_TYPE=Release ..
    make -j 8

Build scripts
~~~~~~~~~~~~~

We recommended creating a helper script that contains the
configure line for CMake.  This is particularly useful when avoiding
environment variables, packages are installed in custom locations,
or the configure line is long or complex.  In this case it is also
recommended to add "rm -rf CMake*" before the configure line to remove
existing CMake configure files to ensure a fresh configure each time
the script is called. Deleting all the files in the build
directory is also acceptable. If you do so we recommend adding some sanity
checks in case the script is run from the wrong directory (e.g.,
checking for the existence of some QMCPACK files).

Some build script examples for different systems are given in the
config directory. For example, on Cray systems these scripts might
load the appropriate modules to set the appropriate programming
environment, specific library versions, etc.

An example script build.sh is given below. It is much more complex
than usually needed for comprehensiveness:

::

  export CXX=mpic++
  export CC=mpicc
  export ACML_HOME=/opt/acml-5.3.1/gfortran64
  export HDF5_ROOT=/opt/hdf5
  export BOOST_ROOT=/opt/boost

  rm -rf CMake*

  cmake                                                \
    -D CMAKE_BUILD_TYPE=Debug                         \
    -D LIBXML2_INCLUDE_DIR=/usr/include/libxml2      \
    -D LIBXML2_LIBRARY=/usr/lib/x86_64-linux-gnu/libxml2.so \
    -D FFTW_INCLUDE_DIRS=/usr/include                 \
    -D FFTW_LIBRARY_DIRS=/usr/lib/x86_64-linux-gnu    \
    -D QMC_EXTRA_LIBS="-ldl ${ACML_HOME}/lib/libacml.a -lgfortran" \
    -D QMC_DATA=/projects/QMCPACK/qmc-data            \
    ..

Using vendor-optimized numerical libraries (e.g., Intel MKL)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Although QMC does not make extensive use of linear algebra, use of
vendor-optimized libraries is strongly recommended for highest
performance. BLAS routines are used in the Slater determinant update, the VMC wavefunction optimizer,
and to apply orbital coefficients in local basis calculations. Vectorized
math functions are also beneficial (e.g., for the phase factor
computation in solid-state calculations). CMake is generally successful
in finding these libraries, but specific combinations can require
additional hints, as described in the following:

Using Intel MKL with non-Intel compilers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To use Intel MKL with, e.g. an MPICH wrapped gcc:

::

  cmake \
    -DCMAKE_C_COMPILER=mpicc -DCMAKE_CXX_COMPILER=mpicxx \
    -DENABLE_MKL=1 -DMKL_ROOT=$MKLROOT/lib \
    ..
