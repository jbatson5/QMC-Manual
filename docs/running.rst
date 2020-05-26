.. _running:

Running QMCPACK
===============

QMCPACK requires at least one xml input file, and is invoked via:

``qmcpack [command line options] <XML input file(s)>``

.. _commandline

Command line options
--------------------

QMCPACK offers several command line options that affect how calculations
are performed. If the flag is absent, then the corresponding
option is disabled:

- ``--dryrun`` Validate the input file without performing the simulation. This is a good way to ensure that QMCPACK will do what you think it will.

- ``--enable-timers=none|coarse|medium|fine`` Control the timer granularity when the build option ``ENABLE_TIMERS`` is enabled.

- ``help``Print version information as well as a list of optional
  command-line arguments.

- ``noprint`` Do not print extra information on Jastrow or pseudopotential.
  If this flag is not present, QMCPACK will create several ``.dat`` files
  that contain information about pseudopotentials (one file per PP) and Jastrow
  factors (one per Jastrow factor). These file might be useful for visual inspection
  of the Jastrow, for example.

- ``--verbosity=low|high|debug`` Control the output verbosity. The default low verbosity is concise and, for example, does not include all electron or atomic positions for large systems to reduce output size. Use "high" to see this information and more details of initialization, allocations, QMC method settings, etc.

- ``version`` Print version information and optional arguments. Same as ``help``.

.. _inputs:

Input files
-----------

The input is one or more XML file(s), documented in :ref:`input_overview`.

Output files
------------

QMCPACK generates multiple files documented in :ref:`output_overview`.

.. _parallelrunning:

Running in parallel with MPI
----------------------------

QMCPACK is fully parallelized with MPI. When performing an ensemble job, all
the MPI ranks are first equally divided into groups that perform individual
QMC calculations. Within one calculation, all the walkers are fully distributed
across all the MPI ranks in the group. Since MPI requires distributed memory,
there must be at least one MPI per node. To maximize the efficiency, more facts
should be taken into account. When using MPI+threads on compute nodes with more
than one NUMA domain (e.g., AMD Interlagos CPU on Titan or a node with multiple
CPU sockets), it is recommended to place as many MPI ranks as the number of
NUMA domains if the memory is sufficient (e.g., one MPI task per socket). On clusters with more than one
GPU per node (NVIDIA Tesla K80), it is necessary to use the same number of MPI
ranks as the number of GPUs per node to let each MPI rank take one GPU.

.. _openmprunning:

Using OpenMP threads
--------------------

Modern processors integrate multiple identical cores even with
hardware threads on a single die to increase the total performance and
maintain a reasonable power draw. QMCPACK takes advantage of this
compute capability by using threads and the OpenMP programming model
as well as threaded linear algebra libraries. By default, QMCPACK is
always built with OpenMP enabled. When launching calculations, users
should instruct QMCPACK to create the right number of threads per MPI
rank by specifying environment variable OMP\_NUM\_THREADS. Assuming
one MPI rank per socket, the number of threads should typically be the
number of cores on that socket. Even in the GPU-accelerated version,
using threads significantly reduces the time spent on the calculations
performed by the CPU.

Nested OpenMP threads
~~~~~~~~~~~~~~~~~~~~~

Nested threading is an advanced feature requiring experienced users to finely tune runtime parameters to reach the best performance.

For small-to-medium problem sizes, using one thread per walker or for multiple walkers is most efficient. This is the default in QMCPACK and achieves the shortest time to solution.

For large problems of at least 1,000 electrons, use of nested OpenMP threading can be enabled to reduce the time to solution further, although at some loss of efficiency. In this scheme multiple threads are used in the computations of each walker. This capability is implemented for some of the key computational kernels: the 3D spline orbital evaluation, certain portions of the distance tables, and implicitly the BLAS calls in the determinant update. Use of the batched nonlocal pseudopotential evaluation is also recommended.
