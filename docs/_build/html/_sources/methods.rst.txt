.. _qmcmethods:

Quantum Monte Carlo Methods
===========================

``qmc`` factory element:

  +-----------------+----------------------+
  | Parent elements | ``simulation, loop`` |
  +-----------------+----------------------+
  | type selector   | ``method`` attribute |
  +-----------------+----------------------+

  type options:

  +--------+-----------------------------------------------+
  | vmc    | Variational Monte Carlo                       |
  +--------+-----------------------------------------------+
  | linear | Wavefunction optimization with linear method  |
  +--------+-----------------------------------------------+
  | dmc    | Diffusion Monte Carlo                         |
  +--------+-----------------------------------------------+
  | rmc    | Reptation Monte Carlo                         |
  +--------+-----------------------------------------------+

  shared attributes:

  +----------------+--------------+--------------+-------------+---------------------------------+
  | **Name**       | **Datatype** | **Values**   | **Default** | **Description**                 |
  +================+==============+==============+=============+=================================+
  | ``method``     | text         | listed above | invalid     | QMC driver                      |
  +----------------+--------------+--------------+-------------+---------------------------------+
  | ``move``       | text         | pbyp, alle   | pbyp        | Method used to move electrons   |
  +----------------+--------------+--------------+-------------+---------------------------------+
  | ``gpu``        | text         | yes/no       | dep.        | Use the GPU                     |
  +----------------+--------------+--------------+-------------+---------------------------------+
  | ``trace``      | text         |              | no          | ???                             |
  +----------------+--------------+--------------+-------------+---------------------------------+
  | ``checkpoint`` | integer      | -1, 0, n     | -1          | Checkpoint frequency            |
  +----------------+--------------+--------------+-------------+---------------------------------+
  | ``record``     | integer      | n            | 0           | Save configuration ever n steps |
  +----------------+--------------+--------------+-------------+---------------------------------+
  | ``target``     | text         |              |             | ???                             |
  +----------------+--------------+--------------+-------------+---------------------------------+
  | ``completed``  | text         |              |             | ???                             |
  +----------------+--------------+--------------+-------------+---------------------------------+
  | ``append``     | text         | yes/no       | no          | ???                             |
  +----------------+--------------+--------------+-------------+---------------------------------+

Additional information:

-  ``move``: There are two ways to move electrons. The more used method
   is the particle-by-particle move. In this method, only one electron
   is moved for acceptance or rejection. The other method is the
   all-electron move; namely, all the electrons are moved once for
   testing acceptance or rejection.

-  ``gpu``: When the executable is compiled with CUDA, the target
   computing device can be chosen by this switch. With a regular
   CPU-only compilation, this option is not effective.

-  ``checkpoint``: This enables and disables checkpointing and
   specifying the frequency of output. Possible values are:

   - **[-1]** No checkpoint (default setting).

   - **[0]** Dump after the completion of a QMC section.

   - **[n]** Dump after every :math:`n` blocks.  Also dump at the end of the run.

The particle configurations are written to a ``.config.h5`` file.

.. code-block::
  :caption: The following is an example of running a simulation that can be restarted.
  :name: Listing 42

  <qmc method="dmc" move="pbyp"  checkpoint="0">
    <parameter name="timestep">         0.004  </parameter>
    <parameter name="blocks">           100   </parameter>
    <parameter name="steps">            400    </parameter>
  </qmc>

The checkpoint flag instructs QMCPACK to output walker configurations.  This also
works in VMC.  This outputs an h5 file with the name ``projectid.run-number.config.h5``.
Check that this file exists before attempting a restart.

To continue a run, specify the ``mcwalkerset`` element before your VMC/DMC block:

.. code-block::
  :caption: Restart (read walkers from previous run).
  :name: Listing 43

  <mcwalkerset fileroot="BH.s002" version="0 6" collected="yes"/>
   <qmc method="dmc" move="pbyp"  checkpoint="0">
     <parameter name="timestep">         0.004  </parameter>
     <parameter name="blocks">           100   </parameter>
     <parameter name="steps">            400    </parameter>
   </qmc>

``BH`` is the project id, and ``s002`` is the calculation number to read in the walkers from the previous run.

In the project id section, make sure that the series number is different from any existing ones to avoid overwriting them.

.. _vmc:

Variational Monte Carlo
-----------------------

``vmc`` method:

  parameters:

  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | **Name**                       | **Datatype** | **Values**              | **Default** | **Description**                               |
  +================================+==============+=========================+=============+===============================================+
  | ``walkers``                    | integer      | :math:`> 0`             | dep.        | Number of walkers per MPI task                |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | ``blocks``                     | integer      | :math:`\geq 0`          | 1           | Number of blocks                              |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | ``steps``                      | integer      | :math:`\geq 0`          | 1           | Number of steps per block                     |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | ``warmupsteps``                | integer      | :math:`\geq 0`          | 0           | Number of steps for warming up                |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | ``substeps``                   | integer      | :math:`\geq 0`          | 1           | Number of substeps per step                   |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | ``usedrift``                   | text         | yes,no                  | yes         | Use the algorithm with drift                  |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | ``timestep``                   | real         | :math:`> 0`             | 0.1         | Time step for each electron move              |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | ``samples``                    | integer      | :math:`\geq 0`          | 0           | Number of walker samples for DMC/optimization |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | ``stepsbetweensamples``        | integer      | :math:`> 0`             | 1           | Period of sample accumulation                 |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | ``samplesperthread``           | integer      | :math:`\geq 0`          | 0           | Number of samples per thread                  |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | ``storeconfigs``               | integer      | all values              | 0           | Show configurations o                         |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+
  | ``blocks_between_recompute``   | integer      | :math:`\geq 0`          | dep.        | Wavefunction recompute frequency              |
  +--------------------------------+--------------+-------------------------+-------------+-----------------------------------------------+

Additional information:

- ``walkers`` The number of walkers per MPI task. The initial default number of \ixml{walkers} is one per OpenMP thread or per MPI task if threading is disabled. The number is rounded down to a multiple of the number of threads with a minimum of one per thread to ensure perfect load balancing. One walker per thread is created in the event fewer ``walkers`` than threads are requested.

- ``blocks`` This parameter is universal for all the QMC
  methods. The MC processes are divided into a number of
  ``blocks``, each containing a number of steps. At the end of each block,
  the statistics accumulated in the block are dumped into files,
  e.g., ``scalar.dat``. Typically, each block should have a sufficient number of steps that the I/O at the end of each block is negligible
  compared with the computational cost. Each block should not take so
  long that monitoring its progress is difficult. There should be a
  sufficient number of ``blocks`` to perform statistical analysis.

- ``warmupsteps`` - ``warmupsteps`` are used only for
  equilibration. Property measurements are not performed during
  warm-up steps.

- ``steps`` - ``steps`` are the number of energy and other property measurements to perform per block.

- ``substeps``  For each substep, an attempt is made to move each of the electrons once only by either particle-by-particle or an
  all-electron move.  Because the local energy is evaluated only at
  each full step and not each substep, ``substeps`` are computationally cheaper
  and can be used to reduce the correlation between property measurements
  at a lower cost.

- ``usedrift`` The VMC is implemented in two algorithms with
  or without drift. In the no-drift algorithm, the move of each
  electron is proposed with a Gaussian distribution. The standard
  deviation is chosen as the time step input. In the drift algorithm,
  electrons are moved by Langevin dynamics.

- ``timestep`` The meaning of time step depends on whether or not
  the drift is used. In general, larger time steps reduce the
  time correlation but might also reduce the acceptance ratio,
  reducing overall statistical efficiency. For VMC, typically the
  acceptance ratio should be close to 50% for an efficient
  simulation.

- ``samples`` Seperate from conventional energy and other
  property measurements, samples refers to storing whole electron
  configurations in memory ("walker samples") as would be needed by subsequent
  wavefunction optimization or DMC steps. *A standard VMC run to
  measure the energy does not need samples to be set.*

  .. math::

     \texttt{samples}=
     \frac{\texttt{blocks}\cdot\texttt{steps}\cdot\texttt{walkers}}{\texttt{stepsbetweensamples}}\cdot\texttt{number of MPI tasks}

- ``samplesperthread`` This is an alternative way to set the target amount of samples and can be useful when preparing a stored population for a subsequent DMC calculation.

  .. math::

     \texttt{samplesperthread}=
     \frac{\texttt{blocks}\cdot\texttt{steps}}{\texttt{stepsbetweensamples}}

- ``stepsbetweensamples`` Because samples generated by consecutive steps are correlated, having ``stepsbetweensamples`` larger than 1 can be used to reduces that correlation. In practice, using larger substeps is cheaper than using ``stepsbetweensamples`` to decorrelate samples.

- ``storeconfigs`` If ``storeconfigs`` is set to a nonzero value, then electron configurations during the VMC run are saved to files.

- ``blocks_between_recompute`` Recompute the accuracy critical determinant part of the wavefunction
  from scratch: =1 by default when using mixed precision. =0 (no
  recompute) by default when not using mixed precision. Recomputing
  introduces a performance penalty dependent on system size.

An example VMC section for a simple VMC run:

::

  <qmc method="vmc" move="pbyp">
    <estimator name="LocalEnergy" hdf5="no"/>
    <parameter name="walkers">    256 </parameter>
    <parameter name="warmupSteps">  100 </parameter>
    <parameter name="substeps">  5 </parameter>
    <parameter name="blocks">  20 </parameter>
    <parameter name="steps">  100 </parameter>
    <parameter name="timestep">  1.0 </parameter>
    <parameter name="usedrift">   yes </parameter>
  </qmc>

Here we set 256 ``walkers`` per MPI, have a brief initial equilibration of 100 ``steps``, and then have 20 ``blocks`` of 100 ``steps`` with 5 ``substeps`` each.

The following is an example of VMC section storing configurations (walker samples) for optimization.

::

  <qmc method="vmc" move="pbyp" gpu="yes">
     <estimator name="LocalEnergy" hdf5="no"/>
     <parameter name="walkers">    256 </parameter>
     <parameter name="samples">    2867200 </parameter>
     <parameter name="stepsbetweensamples">    1 </parameter>
     <parameter name="substeps">  5 </parameter>
     <parameter name="warmupSteps">  5 </parameter>
     <parameter name="blocks">  70 </parameter>
     <parameter name="timestep">  1.0 </parameter>
     <parameter name="usedrift">   no </parameter>
   </qmc>

.. _optimization:

Wavefunction optimization
-------------------------

Optimizing wavefunction is critical in all kinds of real-space QMC calculations
because it significantly improves both the accuracy and efficiency of computation.
However, it is very difficult to directly adopt deterministic minimization approaches because of the stochastic nature of evaluating quantities with MC.
Thanks to the algorithmic breakthrough during the first decade of this century and the tremendous computer power available,
it is now feasible to optimize tens of thousands of parameters in a wavefunction for a solid or molecule.
QMCPACK has multiple optimizers implemented based on the state-of-the-art linear method.
We are continually improving our optimizers for robustness and friendliness and are trying to provide a single solution.
Because of the large variation of wavefunction types carrying distinct characteristics, using several optimizers might be needed in some cases.
We strongly suggested reading recommendations from the experts who maintain these optimizers.

A typical optimization block looks like the following. It starts with method="linear" and contains three blocks of parameters.

::

  <loop max="10">
   <qmc method="linear" move="pbyp" gpu="yes">
     <!-- Specify the VMC options -->
     <parameter name="walkers">              256 </parameter>
     <parameter name="samples">          2867200 </parameter>
     <parameter name="stepsbetweensamples">    1 </parameter>
     <parameter name="substeps">               5 </parameter>
     <parameter name="warmupSteps">            5 </parameter>
     <parameter name="blocks">                70 </parameter>
     <parameter name="timestep">             1.0 </parameter>
     <parameter name="usedrift">              no </parameter>
     <estimator name="LocalEnergy" hdf5="no"/>
     ...
     <!-- Specify the correlated sampling options and define the cost function -->
     <parameter name="minwalkers">            0.3 </parameter>
          <cost name="energy">               0.95 </cost>
          <cost name="unreweightedvariance"> 0.00 </cost>
          <cost name="reweightedvariance">   0.05 </cost>
     ...
     <!-- Specify the optimizer options -->
     <parameter name="MinMethod">    OneShiftOnly </parameter>
     ...
   </qmc>
  </loop>

  -  Loop is helpful to repeatedly execute identical optimization blocks.

  -  The first part is highly identical to a regular VMC block.

  -  The second part is to specify the correlated sampling options and
     define the cost function.

  -  The last part is used to specify the options of different optimizers,
     which can be very distinct from one to another.

VMC run for the optimization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The VMC calculation for the wavefunction optimization has a strict requirement
that ``samples`` or ``samplesperthread`` must be specified because of the optimizer needs for the stored ``samples``.
The input parameters of this part are identical to the VMC method.

Recommendations:

-  Run the inclusive VMC calculation correctly and efficiently because
   this takes a significant amount of time during optimization. For
   example, make sure the derived ``steps`` per block is 1 and use larger ``substeps`` to
   control the correlation between ``samples``.

-  A reasonable starting wavefunction is necessary. A lot of
   optimization fails because of a bad wavefunction starting point. The
   sign of a bad initial wavefunction includes but is not limited to a
   very long equilibration time, low acceptance ratio, and huge
   variance. The first thing to do after a failed optimization is to
   check the information provided by the VMC calculation via
   ``*.scalar.dat files``.

Correlated sampling and cost function
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After generating the samples with VMC, the derivatives of the wavefunction with respect to the parameters are computed for proposing a new set of parameters by optimizers.
And later, a correlated sampling calculation is performed to quickly evaluate values of the cost function on the old set of parameters and the new set for further decisions.
The input parameters are listed in the following table.

``linear`` method:

  parameters:

  +----------------+--------------+-------------+-------------+--------------------------------------------------+
  | **Name**       | **Datatype** | **Values**  | **Default** | **Description**                                  |
  +================+==============+=============+=============+==================================================+
  | ``nonlocalpp`` | text         | yes, no     | no          | include non-local PP energy in the cost function |
  +----------------+--------------+-------------+-------------+--------------------------------------------------+
  | ``minwalkers`` | real         | 0--1        | 0.3         | Lower bound of the effective weight              |
  +----------------+--------------+-------------+-------------+--------------------------------------------------+
  | ``maxWeight``  | real         | :math:`> 1` | 1e6         | Maximum weight allowed in reweighting            |
  +----------------+--------------+-------------+-------------+--------------------------------------------------+

Additional information:

- ``maxWeight`` The default should be good.

- ``nonlocalpp`` The ``nonlocalpp`` contribution to the local energy depends on the
  wavefunction. When a new set of parameters is proposed, this
  contribution needs to be updated if the cost function consists of local
  energy. Fortunately, nonlocal contribution is chosen small when making a
  PP for small locality error. We can ignore its change and avoid the
  expensive computational cost. An implementation issue with GPU code is
  that a large amount of memory is consumed with this option.

- ``minwalkers`` This is a ``critical`` parameter. When the ratio of effective samples to actual number of samples in a reweighting step goes lower than ``minwalkers``,
  the proposed set of parameters is invalid.

The cost function consists of three components: energy, unreweighted variance, and reweighted variance.

::

     <cost name="energy">                   0.95 </cost>
     <cost name="unreweightedvariance">     0.00 </cost>
     <cost name="reweightedvariance">       0.05 </cost>

Optimizers
~~~~~~~~~~

QMCPACK implements a number of different optimizers each with different
priorities for accuracy, convergence, memory usage, and stability. The
optimizers can be switched among “OneShiftOnly” (default), “adaptive,”
“descent,” “hybrid,” and “quartic” (old) using the following line in the
optimization block:

::

<parameter name="MinMethod"> THE METHOD YOU LIKE </parameter>

OneShiftOnly Optimizer
~~~~~~~~~~~~~~~~~~~~~~

The OneShiftOnly optimizer targets a fast optimization by moving parameters more aggressively. It works with OpenMP and GPU and can be considered for large systems.
This method relies on the effective weight of correlated sampling rather than the cost function value to justify a new set of parameters.
If the effective weight is larger than ``minwalkers``, the new set is taken whether or not the cost function value decreases.
If a proposed set is rejected, the standard output prints the measured ratio of effective samples to the total number of samples
and adjustment on ``minwalkers`` can be made if needed.

``linear`` method:

  parameters:

  +--------------+--------------+-------------+-------------+---------------------------------------------------+
  | **Name**     | **Datatype** | **Values**  | **Default** | **Description**                                   |
  +==============+==============+=============+=============+===================================================+
  | ``shift_i``  | real         | :math:`> 0` | 0.01        | Direct stabilizer added to the Hamiltonian matrix |
  +--------------+--------------+-------------+-------------+---------------------------------------------------+
  | ``shift_s``  | real         | :math:`> 0` | 1.00        | Initial stabilizer based on the overlap matrix    |
  +--------------+--------------+-------------+-------------+---------------------------------------------------+

Additional information:

-  ``shift_i`` This is the direct term added to the diagonal of the Hamiltonian
   matrix. It provides more stable but slower optimization with a large
   value.

-  ``shift_s`` This is the initial value of the stabilizer based on the overlap
   matrix added to the Hamiltonian matrix. It provides more stable but
   slower optimization with a large value. The used value is
   auto-adjusted by the optimizer.

Recommendations:

- Default ``shift_i``, ``shift_s`` should be fine.

- For hard cases, increasing ``shift_i`` (by a factor of 5 or 10) can significantly stabilize the optimization by reducing the pace towards the optimal parameter set.

- If the VMC energy of the last optimization iterations grows significantly, increase ``minwalkers`` closer to 1 and make the optimization stable.

- If the first iterations of optimization are rejected on a reasonable initial wavefunction,
  lower the ``minwalkers`` value based on the measured value printed in the standard output to accept the move.

We recommended using this optimizer in two sections with a very small ``minwalkers`` in the first and a large value in the second, such as the following.
In the very beginning, parameters are far away from optimal values and large changes are proposed by the optimizer.
Having a small ``minwalkers`` makes it much easier to accept these changes.
When the energy gradually converges, we can have a large ``minwalkers`` to avoid risky parameter sets.

::

  <loop max="6">
   <qmc method="linear" move="pbyp" gpu="yes">
     <!-- Specify the VMC options -->
     <parameter name="walkers">                1 </parameter>
     <parameter name="samples">            10000 </parameter>
     <parameter name="stepsbetweensamples">    1 </parameter>
     <parameter name="substeps">               5 </parameter>
     <parameter name="warmupSteps">            5 </parameter>
     <parameter name="blocks">                25 </parameter>
     <parameter name="timestep">             1.0 </parameter>
     <parameter name="usedrift">              no </parameter>
     <estimator name="LocalEnergy" hdf5="no"/>
     <!-- Specify the optimizer options -->
     <parameter name="MinMethod">    OneShiftOnly </parameter>
     <parameter name="minwalkers">           1e-4 </parameter>
   </qmc>
  </loop>
  <loop max="12">
   <qmc method="linear" move="pbyp" gpu="yes">
     <!-- Specify the VMC options -->
     <parameter name="walkers">                1 </parameter>
     <parameter name="samples">            20000 </parameter>
     <parameter name="stepsbetweensamples">    1 </parameter>
     <parameter name="substeps">               5 </parameter>
     <parameter name="warmupSteps">            2 </parameter>
     <parameter name="blocks">                50 </parameter>
     <parameter name="timestep">             1.0 </parameter>
     <parameter name="usedrift">              no </parameter>
     <estimator name="LocalEnergy" hdf5="no"/>
     <!-- Specify the optimizer options -->
     <parameter name="MinMethod">    OneShiftOnly </parameter>
     <parameter name="minwalkers">            0.5 </parameter>
   </qmc>
  </loop>

For each optimization step, you will see

::

  The new set of parameters is valid. Updating the trial wave function!

or

::

  The new set of parameters is not valid. Revert to the old set!

Occasional rejection is fine. Frequent rejection indicates potential
problems, and users should inspect the VMC calculation or change
optimization strategy. To track the progress of optimization, use the
command ``qmca -q ev *.scalar.dat`` to look at the VMC energy and
variance for each optimization step.

Adaptive Organizer
~~~~~~~~~~~~~~~~~~

The default setting of the adaptive optimizer is to construct the linear
method Hamiltonian and overlap matrices explicitly and add different
shifts to the Hamiltonian matrix as “stabilizers.” The generalized
eigenvalue problem is solved for each shift to obtain updates to the
wavefunction parameters. Then a correlated sampling is performed for
each shift’s updated wavefunction and the initial trial wavefunction
using the middle shift’s updated wavefunction as the guiding function.
The cost function for these wavefunctions is compared, and the update
corresponding to the best cost function is selected. In the next
iteration, the median magnitude of the stabilizers is set to the
magnitude that generated the best update in the current iteration, thus
adapting the magnitude of the stabilizers automatically.

When the trial wavefunction contains more than 10,000 parameters,
constructing and storing the linear method matrices could become a
memory bottleneck. To avoid explicit construction of these matrices, the
adaptive optimizer implements the block linear method (BLM) approach.
:cite:`Zhao:2017:blocked_lm` The BLM tries to find an
approximate solution :math:`\vec{c}_{opt}` to the standard LM
generalized eigenvalue problem by dividing the variable space into a
number of blocks and making intelligent estimates for which directions
within those blocks will be most important for constructing
:math:`\vec{c}_{opt}`, which is then obtained by solving a smaller, more
memory-efficient eigenproblem in the basis of these supposedly important
block-wise directions.
