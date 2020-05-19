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
