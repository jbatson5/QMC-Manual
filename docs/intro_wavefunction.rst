.. _intro_wavefunction:

Trial wavefunction specificaion
===============================

.. _trial-intro:

Introduction
------------

This section describes the input blocks associated with the specification of the trial wavefunction in a QMCPACK calculation. These sections are contained within the ``<wavefunction>`` :math:`...`  ``</wavefunction>`` xml blocks. **Users are expected to rely on converters to generate the input blocks described in this section.** The converters and the workflows are designed such that input blocks require minimum modifications from users. Unless the workflow requires modification of wavefunction blocks (e.g., setting the cutoff in a multideterminant calculation), only expert users should directly alter them.

The trial wavefunction in QMCPACK has a general product form:

.. math::
  :label: eq1

  \Psi_T(\vec{r}) = \prod_k \Theta_k(\vec{r}) ,

where each :math:`\Theta_k(\vec{r})` is a function of the electron coordinates
(and possibly ionic coordinates and variational parameters).
For problems involving electrons, the overall trial wavefunction
must be antisymmetric with respect to electron exchange,
so at least one of the functions in the product must be
antisymmetric. Notice that, although QMCPACK allows for the
construction of arbitrary trial wavefunctions based on the
functions implemented in the code
(e.g., slater determinants, jastrow functions),
the user must make sure that a correct wavefunction is
used for the problem at hand. From here on, we assume a
standard trial wavefunction for an electronic structure problem

.. math::
  :label: eq2

  Psi_T(\vec{r}) =  \textit{A}(\vec{r}) \prod_k \textit{J}_k(\vec{r}),

where :math:`\textit{A}(\vec{r})`
is one of the antisymmetric functions: (1) slater determinant, (2) multislater determinant, or (3) pfaffian and :math:`\textit{J}_k`
is any of the Jastrow functions (described in :ref:`jastrow`).  The antisymmetric functions are built from a set of single particle orbitals ``(sposet)``. QMCPACK implements four different types of ``sposet``, described in the following section. Each ``sposet`` is designed for a different type of calculation, so their definition and generation varies accordingly.

.. _singledeterminant:

Single determinant wavefunctons
-------------------------------

Placing a single determinant for each spin is the most used ansatz for the antisymmetric part of a trial wavefunction.
The input xml block for ``slaterdeterminant`` is given in :ref:`Listing 1 <Listing 1>`. A list of options is given in
:numref:`Table2`.

``slaterdeterminant`` element:


.. _Table2:
.. table::

     +-----------------+--------------------+
     | Parent elements | ``determinantset`` |
     +-----------------+--------------------+
     | Child elements  | ``determinant``    |
     +-----------------+--------------------+

Attribute:

+-----------------+----------+--------+---------+------------------------------+
| Name            | Datatype | Values | Default | Description                  |
+=================+==========+========+=========+==============================+
| ``delay_rank``  | Integer  | >=0    | 1       | Number of delayed updates.   |
+-----------------+----------+--------+---------+------------------------------+
| ``optimize``    | Text     | yes/no | yes     | Enable orbital optimization. |
+-----------------+----------+--------+---------+------------------------------+


.. centered:: Table 2 Options for the ``slaterdeterminant`` xml-block.

.. code-block::
      :caption: Slaterdeterminant set XML element.
      :name: Listing 1

      <slaterdeterminant delay_rank="32">
         <determinant id="updet" size="208">
           <occupation mode="ground" spindataset="0">
           </occupation>
         </determinant>
         <determinant id="downdet" size="208">
           <occupation mode="ground" spindataset="0">
           </occupation>
         </determinant>
       </slaterdeterminant>


Additional information:

- ``delay_rank`` This option enables delayed updates of the Slater matrix inverse when particle-by-particle move is used.
  By default or if ``delay_rank=0`` given in the input file, QMCPACK sets 1 for Slater matrices with a leading dimension :math:`<192` and 32 otherwise.
  ``delay_rank=1`` uses the Fahy's variant :cite:`Fahy1990` of the Sherman-Morrison rank-1 update, which is mostly using memory bandwidth-bound BLAS-2 calls.
  With ``delay_rank>1``, the delayed update algorithm :cite:`Luo2018delayedupdate,McDaniel2017` turns most of the computation to compute bound BLAS-3 calls.
  Tuning this parameter is highly recommended to gain the best performance on medium-to-large problem sizes (:math:`>200` electrons).
  We have seen up to an order of magnitude speedup on large problem sizes.
  When studying the performance of QMCPACK, a scan of this parameter is required and we recommend starting from 32.
  The best ``delay_rank`` giving the maximal speedup depends on the problem size.
  Usually the larger ``delay_rank`` corresponds to a larger problem size.
  On CPUs, ``delay_rank`` must be chosen as a multiple of SIMD vector length for good performance of BLAS libraries.
  The best ``delay_rank`` depends on the processor microarchitecture.
  GPU support is under development.

.. _singleparticle:

Single-particle orbitals
------------------------

.. _spo-spline:

Spline basis sets
~~~~~~~~~~~~~~~~~

In this section we describe the use of spline basis sets to expand the ``sposet``.
Spline basis sets are designed to work seamlessly with plane wave DFT code (e.g.,\ Quantum ESPRESSO as a trial wavefunction generator).

In QMC algorithms, all the SPOs :math:`\{\phi(\vec{r})\}` need to be updated
every time a single electron moves. Evaluating SPOs takes a very large portion of computation time.
In principle, PW basis set can be used to express SPOs directly in QMC, as in DFT.
But it introduces an unfavorable scaling because the basis set size increases linearly as the system size.
For this reason, it is efficient to use a localized basis with compact
support and a good transferability from the plane wave basis.

In particular, 3D tricubic B-splines provide a basis in which only
64 elements are nonzero at any given point in :cite:`blips4QMC`.
The 1D cubic B-spline is given by

.. math::
  :label: eq3

  f(x) = \sum_{i'=i-1}^{i+2} b^{i'\!,3}(x)\,\,  p_{i'},

where :math:`b^{i}(x)` is the piecewise cubic polynomial basis functions
and :math:`i = \text{floor}(\Delta^{-1} x)` is the index of the first
grid point :math:`\le x`. Constructing a tensor product in each
Cartesian direction, we can represent a 3D orbital as

.. math::
 :label: eq4

 \phi_n(x,y,z) =
     \!\!\!\!\sum_{i'=i-1}^{i+2} \!\! b_x^{i'\!,3}(x)
     \!\!\!\!\sum_{j'=j-1}^{j+2} \!\! b_y^{j'\!,3}(y)
     \!\!\!\!\sum_{k'=k-1}^{k+2} \!\! b_z^{k'\!,3}(z) \,\, p_{i', j', k',n}.

This allows the rapid evaluation of each orbital in constant time.
Furthermore, this basis is systematically improvable with a single spacing
parameter so that accuracy is not compromised compared with the plane wave basis.

The use of 3D tricubic B-splines greatly improves computational efficiency.
The gain in computation time from a plane wave basis set to an equivalent B-spline basis set
becomes increasingly large as the system size grows.
On the downside, this computational efficiency comes at
the expense of increased memory use, which is easily overcome, however, by the large
aggregate memory available per node through OpenMP/MPI hybrid QMC.

The input xml block for the spline SPOs is given in :ref:`Listing 2 <Listing 2>`. A list of options is given in
:numref:`table3`.

.. code-block::
  :caption: Determinant set XML element.
  :name: Listing 2

  <determinantset type="bspline" source="i" href="pwscf.h5"
                  tilematrix="1 1 3 1 2 -1 -2 1 0" twistnum="-1" gpu="yes" meshfactor="0.8"
                  twist="0  0  0" precision="double">
    <slaterdeterminant>
      <determinant id="updet" size="208">
        <occupation mode="ground" spindataset="0">
        </occupation>
      </determinant>
      <determinant id="downdet" size="208">
        <occupation mode="ground" spindataset="0">
        </occupation>
      </determinant>
    </slaterdeterminant>
  </determinantset>


``determinantset`` element:

.. _table3:
.. table::

  +-----------------+-----------------------+
  | Parent elements | ``wavefunction``      |
  +-----------------+-----------------------+
  | Child elements  | ``slaterdeterminant`` |
  +-----------------+-----------------------+

attribute:

+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| Name                        | Datatype   | Values                   | Default | Description                               |
+=============================+============+==========================+=========+===========================================+
| ``type``                    | Text       | Bspline                  |         | Type of ``sposet``                        |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``href``                    | Text       |                          |         | Path to hdf5 file from pw2qmcpack.x.      |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``tilematrix``              | 9 integers |                          |         | Tiling matrix used to expand supercell.   |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``twistnum``                | Integer    |                          |         | Index of the super twist.                 |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``twist``                   | 3 floats   |                          |         | Super twist.                              |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``meshfactor``              | Float      | :math:`\le 1.0`          |         | Grid spacing ratio.                       |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``precision``               | Text       | Single/double            |         | Precision of spline coefficients          |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``gpu``                     | Text       | Yes/no                   |         | GPU switch.                               |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``gpusharing``              | Text       | Yes/no                   | No      | Share B-spline table across GPUs.         |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``Spline_Size_Limit_MB``    | Integer    |                          |         | Limit B-spline table size on GPU.         |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``check_orb_norm``          | Text       | Yes/no                   | Yes     | Check norms of orbitals from h5 file.     |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``save_coefs``              | Text       | Yes/no                   | No      | Save the spline coefficients to h5 file.  |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+
| ``source``                  | Text       | Any                      | Ion0    | Particle set with atomic positions.       |
+-----------------------------+------------+--------------------------+---------+-------------------------------------------+

.. centered:: Table 3 Options for the ``determinantset`` xml-block associated with B-spline single particle orbital sets.

Additional information:

-  ``precision``. Only effective on CPU versions without mixed
   precision, “single" is always imposed with mixed precision. Using
   single precision not only saves memory use but also speeds up the
   B-spline evaluation. We recommend using single precision since we saw
   little chance of really compromising the accuracy of calculation.

-  ``meshfactor``. The ratio of actual grid spacing of B-splines used in
   QMC calculation with respect to the original one calculated from h5.
   A smaller meshfactor saves memory use but reduces accuracy. The
   effects are similar to reducing plane wave cutoff in DFT
   calculations. Use with caution!

-  ``twistnum``. If positive, it is the index. We recommend not taking
   this way since the indexing might show some uncertainty. If negative,
   the super twist is referred by ``twist``.

-  ``save_coefs``. If yes, dump the real-space B-spline coefficient
   table into an h5 file on the disk. When the orbital transformation
   from k space to B-spline requires more than the available amount of
   scratch memory on the compute nodes, users can perform this step on
   fat nodes and transfer back the h5 file for QMC calculations.

-  ``gpusharing``. If enabled, spline data is shared across multiple
   GPUs on a given computational node. For example, on a
   two-GPU-per-node system, each GPU would have half of the orbitals.
   This enables larger overall spline tables than would normally fit in
   the memory of individual GPUs to be used, potentially up to the total
   GPU memory on a node. To obtain high performance, large electron
   counts or a high-performing CPU-GPU interconnect is required. To use
   this feature, the following needs to be done:

     -  The CUDA Multi-Process Service (MPS) needs to be used (e.g., on
     Summit/SummitDev use “-alloc_flags gpumps" for bsub). If MPS is not
     detected, sharing will be disabled.

     -  CUDA_VISIBLE_DEVICES needs to be properly set to control each rank’s
     visible CUDA devices (e.g., on OLCF Summit/SummitDev one needs to
     create a resource set containing all GPUs with the respective number
     of ranks with “jsrun –task-per-rs Ngpus -g Ngpus").

- ``Spline_Size_Limit_MB``. Allows distribution of the B-spline
  coefficient table between the host and GPU memory. The compute kernels
  access host memory via zero-copy. Although the performance penalty
  introduced by it is significant, it allows large calculations to go
  through.

.. _gaussianbasis:

Gaussian basis tests
~~~~~~~~~~~~~~~~~~~~

In this section we describe the use of localized basis sets to expand the ``sposet``. The general form of a single particle orbital in this case is given by:

.. math::
  :label: eq5

  \phi_i(\vec{r}) = \sum_k C_{i,k} \ \eta_k(\vec{r}),

where :math:`\{\eta_k(\vec{r})\}` is a set of M atom-centered basis
functions and :math:`C_{i,k}` is a coefficient matrix. This should be
used in calculations of finite systems employing an atom-centered basis
set and is typically generated by the *convert4qmc* converter. Examples
include calculations of molecules using Gaussian basis sets or
Slater-type basis functions. Initial support for periodic systems is
described in :ref:`LCAO`. Even though this section is called
"Gaussian basis sets" (by far the most common atom-centered basis set),
QMCPACK works with any atom-centered basis set based on either spherical
harmonic angular functions or Cartesian angular expansions. The radial
functions in the basis set can be expanded in either Gaussian functions,
Slater-type functions, or numerical radial functions.

In this section we describe the input sections for the atom-centered basis set and the ``sposet`` for a single Slater determinant trial wavefunction. The input sections for multideterminant trial wavefunctions are described in :ref:`multideterminants`. The basic structure for the input block of a single Slater determinant is given in :ref:`Listing 3 <Listing 3>`.
A list of options for ``determinantset`` associated with this ``sposet`` is given in :numref:`table4`.

.. code-block::
   :caption: Basic input block for a single determinant trial wavefunction using a ``sposet`` expanded on an atom-centered basis set.
   :name: Listing 3

    <wavefunction id="psi0" target="e">
      <determinantset>
        <basisset>
          ...
        </basisset>
        <slaterdeterminant>
          ...
        </slaterdeterminant>
      </determinantset>
    </wavefunction>


The definition of the set of atom-centered basis functions is given by the ``basisset`` block, and the ``sposet`` is defined within ``slaterdeterminant``. The ``basisset`` input block is composed from a collection of ``atomicBasisSet`` input blocks, one for each atomic species in the simulation where basis functions are centered. The general structure for ``basisset`` and ``atomicBasisSet`` are given in :ref:`Listing 4 <Listing 4>`, and the corresponding lists of options are given in
:numref:`table5` and :numref:`table6`.

``determinantset`` element:

.. _table4:
.. table::

  +-----------------+--------------------------------------------------------------------------+
  | Parent elements | ``wavefunction``                                                         |
  +-----------------+--------------------------------------------------------------------------+
  | Child elements  | ``basisset`` , ``slaterdeterminant`` , ``sposet`` , ``multideterminant`` |
  +-----------------+--------------------------------------------------------------------------+

Attribute:

+--------------------+--------------+---------------+-------------+------------------------------------------------+
| **Name**           | **Datatype** | **Values**    | **Default** | **Description**                                |
+====================+==============+===============+=============+================================================+
| ``name/id``        | Text         | *Any*         | '' ''       | Name of determinant set                        |
+--------------------+--------------+---------------+-------------+------------------------------------------------+
| ``type``           | Text         | See below     | '' ''       | Type of ``sposet``                             |
+--------------------+--------------+---------------+-------------+------------------------------------------------+
| ``keyword``        | Text         | NMO, GTO, STO | NMO         | Type of orbital set generated                  |
+--------------------+--------------+---------------+-------------+------------------------------------------------+
| ``transform``      | Text         | Yes/no        | Yes         | Transform to numerical radial functions?       |
+--------------------+--------------+---------------+-------------+------------------------------------------------+
| ``source``         | Text         | *Any*         | Ion0        | Particle set with the position of atom centers |
+--------------------+--------------+---------------+-------------+------------------------------------------------+
| ``cuspCorrection`` | Text         | Yes/no        | No          | Apply cusp correction scheme to ``sposet``?    |
+--------------------+--------------+---------------+-------------+------------------------------------------------+

.. centered:: Table 4 Options for the ``determinantset`` xml-block associated with atom-centered single particle orbital sets.

.. code-block::
  :caption: Basic input block for ``basisset``.
  :name: Listing 4

  <basisset name="LCAOBSet">
    <atomicBasisSet name="Gaussian-G2" angular="cartesian" elementType="C" normalized="no">
      <grid type="log" ri="1.e-6" rf="1.e2" npts="1001"/>
      <basisGroup rid="C00" n="0" l="0" type="Gaussian">
        <radfunc exponent="5.134400000000e-02" contraction="1.399098787100e-02"/>
        ...
      </basisGroup>
      ...
    </atomicBasisSet>
    <atomicBasisSet name="Gaussian-G2" angular="cartesian" type="Gaussian" elementType="C" normalized="no">
      ...
    </atomicBasisSet>
    ...
  </basisset>

``basisset`` element:

.. _table5:
.. table::

  +-----------------+-------------------+
  | Parent elements | ``determinantset``|
  +-----------------+-------------------+
  | Child elements  | ``atomicBasisSet``|
  +-----------------+-------------------+

Attribute:

+-------------------+--------------+------------+-------------+----------------------------------+
| **Name**          | **Datatype** | **Values** | **Default** | **Description**                  |
+===================+==============+============+=============+==================================+
| ``name`` / ``id`` | Text         | *Any*      | " "         | Name of atom-centered basis set  |
+-------------------+--------------+------------+-------------+----------------------------------+

.. centered:: Table 5 Options for the ``basisset`` xml-block associated with atom-centered single particle orbital sets.

``AtomicBasisSet`` element:

.. _table6:
.. table::

  +-----------------+--------------------------+
  | Parent elements | ``basisset``             |
  +-----------------+--------------------------+
  | Child elements  | ``grid`` , ``basisGroup``|
  +-----------------+--------------------------+

Attribute:

+-------------------------+--------------+------------+-------------+---------------------------------------------+
| **Name**                | **Datatype** | **Values** | **Default** | **Description**                             |
+=========================+==============+============+=============+=============================================+
| ``name`` / ``id``       | Text         | *Any*      | " "         | Name of atomic basis set                    |
+-------------------------+--------------+------------+-------------+---------------------------------------------+
| ``angular``             | Text         | See below  | Default     | Type of angular functions                   |
+-------------------------+--------------+------------+-------------+---------------------------------------------+
| ``expandYlm``           | Text         | See below  | Yes         | Expand Ylm shells?                          |
+-------------------------+--------------+------------+-------------+---------------------------------------------+
| ``expM``                | Text         | See below  | Yes         | Add sign for :math:`(-1)^{m}`?              |
+-------------------------+--------------+------------+-------------+---------------------------------------------+
| ``elementType/species`` | Text         | *Any*      | e           | Atomic species where functions are centered |
+-------------------------+--------------+------------+-------------+---------------------------------------------+
| ``normalized``          | Text         | Yes/no     | Yes         | Are single particle functions normalized?   |
+-------------------------+--------------+------------+-------------+---------------------------------------------+

.. centered:: Table 6 Options for the ``atomicBasisSet`` xml-block.

``basicGroup`` element:

.. _table7
.. table::

  +-----------------+-------------------+
  | Parent elements | ``AtomicBasisSet``|
  +-----------------+-------------------+
  | Child elements  | ``radfunc``       |
  +-----------------+-------------------+

Attribute:

+-------------+--------------+------------+-------------+-------------------------------+
| **Name**    | **Datatype** | **Values** | **Default** | **Description**               |
+=============+==============+============+=============+===============================+
| ``rid/id``  | Text         | *Any*      | '' ''       | Name of the basisGroup        |
+-------------+--------------+------------+-------------+-------------------------------+
| ``type``    | Text         | *Any*      | '' ''       | Type of basisGroup            |
+-------------+--------------+------------+-------------+-------------------------------+
| ``n/l/m/s`` | Integer      | *Any*      | 0           | Quantum numbers of basisGroup |
+-------------+--------------+------------+-------------+-------------------------------+

.. centered:: Table 7 Options for the ``basisGroup`` xml-block.

.. code-block::
  :caption: Basic input block for ``slaterdeterminant`` with an atom-centered ``sposet``.
  :name: Listing 5

    <slaterdeterminant>
    </slaterdeterminant>

element:

+-----------------+-------+
| Parent elements:|       |
+-----------------+-------+
| Child elements: |       |
+-----------------+-------+

Attribute:

+-------------+--------------+------------+-------------+-------------------------+
| **Name**    | **Datatype** | **Values** | **Default** | **Description**         |
+=============+==============+============+=============+=========================+
| ``name/id`` | Text         | *Any*      | '' ''       | Name of determinant set |
+-------------+--------------+------------+-------------+-------------------------+
|             | Text         | *Any*      | '' ''       |                         |
+-------------+--------------+------------+-------------+-------------------------+

Detailed description of attributes:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the following, we give a more detailed description of all the options presented in the various xml-blocks described in this section. Only nontrivial attributes are described. Those with simple yes/no options and whose previous description is enough to explain the intended behavior are not included.

``determinantset`` attributes:

- ``type``
    Type of ``sposet``. For atom-centered based ``sposets``, use type="MolecularOrbital" or type=“MO."
    Other options described elsewhere in this manual are “spline,"
    “composite," “pw," “heg," “linearopt," etc.

- ``keyword/key``
    Type of basis set generated, which does not necessarily match the type of basis set on the input block. The three possible options are: NMO (numerical molecular orbitals), GTO (Gaussian-type orbitals), and STO (Slater-type orbitals). The default option is NMO. By default, QMCPACK will generate numerical orbitals from both GTO and STO types and use cubic or quintic spline interpolation to evaluate the radial functions. This is typically more efficient than evaluating the radial functions in the native basis (Gaussians or exponents) and allows for arbitrarily large contractions without any additional cost. To force use of the native expansion (not recommended), use GTO or STO for each type of input basis set.

- ``transform``
    Request (or avoid) a transformation of the radial functions to NMO type. The default and recommended behavior is to transform to numerical radial functions. If ``transform`` is set to *yes*, the option ``keyword`` is ignored.

- ``cuspCorrection``
    Enable (disable) use of the cusp correction algorithm (CASINO REFERENCE) for a ``basisset`` built with GTO functions. The algorithm is implemented as described in (CASINO REFERENCE) and works only with transform="yes" and an input GTO basis set. No further input is needed.

``atomicBasisSet`` attributes:

- ``name/id``
    Name of the basis set. Names should be unique.

- ``angular``
    Type of angular functions used in the expansion. In general, two angular basis functions are allowed: "spherical" (for spherical Ylm functions) and "Cartesian" (for functions of the type :math:`x^{n}y^{m}z^{l}`).

- ``expandYlm``
    Determines whether each basis group is expanded across the corresponding shell of m values (for spherical type) or consistent powers (for Cartesian functions). Options:

      - "No": Do not expand angular functions across corresponding angular shell.

      - "Gaussian": Expand according to Gaussian03 format. This function is compatible only with angular="spherical." For a given input (l,m), the resulting order of the angular functions becomes (1,-1,0) for l=1 and (0,1,-1,2,-2,...,l,-l) for general l.

      - "Natural": Expand angular functions according to (-l,-l+1,...,l-1,l).

      - "Gamess": Expand according to Gamess' format for Cartesian functions. Notice that this option is compatible only with angular="Cartesian." If angular="Cartesian" is used, this option is not necessary.

- ``expM``
    Determines whether the sign of the spherical Ylm function associated with m (:math:`-1^{m}`) is included in the coefficient matrix or not.

- ``elementType/species``
    Name of the species where basis functions are centered. Only one ``atomicBasisSet`` block is allowed per species. Additional blocks are ignored. The corresponding species must exist in the ``particleset`` given as the ``source`` option to ``determinantset``. Basis functions for all the atoms of the corresponding species are included in the basis set, based on the order of atoms in the ``particleset``.

``basisGroup`` attributes:

- ``type``
    Type of input basis radial function. Note that this refers to the type of radial function in the input xml-block, which might not match the radial function generated internally and used in the calculation (if ``transform`` is set to "yes"). Also note that different ``basisGroup`` blocks within a given ``atomicBasisSet`` can have different ``types``.

- ``n/l/m/s``
    Quantum numbers of the basis function. Note that if
    ``expandYlm`` is set to *"yes"* in ``atomicBasisSet``, a
    full shell of basis functions with the appropriate values of
    *"m"* will be defined for the corresponding value of
    *"l."* Otherwise a single basis function will be given for the
    specific combination of *"(l,m)."*

``radfunc`` attributes for ``type`` = *"Gaussian"*:

- ``TBDoc``

``slaterdeterminant`` attributes:

- ``TBDoc``

.. _spo-hybrid:

Hyrbid orbital representation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The hybrid representation of the single particle orbitals combines a localized atomic basis set around atomic cores and B-splines in the interstitial regions to reduce memory use while retaining high evaluation speed and either retaining or increasing overall accuracy. Full details are provided in :cite:`Luo2018hyb`, and **users of this feature are kindly requested to cite this paper**.
In practice, we have seen that using a meshfactor=0.5 is often possible and achieves huge memory savings.
:numref:`fig3` illustrates how the regions are assigned.

.. _fig3:
.. figure:: hybrid_new.jpg
    :width: 400
    :align: center

    Regular and hybrid orbital representation. Regular B-spline representation (left panel) contains only one region and a sufficiently fine mesh to resolve orbitals near the nucleus. The hybrid orbital representation (right panel) contains near nucleus regions (A) where spherical harmonics and radial functions are used, buffers or interpolation regions (B), and an interstitial region (C) where a coarse B-spline mesh is used.

Orbitals within region A are computed as

.. math:: \phi^A_n({\bf r})=R_{n,l,m}(r)Y_{l,m}(\hat{r})

Orbitals in region C are computed as the regular B-spline basis described in :ref:`spo-spline` above. The region B interpolates between A and C as

.. math::
  :label: eq6

  \phi^B_n(r) = S(r) \phi^A_n(r) + (1-S(r))\phi^C_n(r)

.. math::
  :label: eq7

  (S(r) = \frac{1}{2}-\frac{1}{2} tanh \left[\alpha\left(\frac{r-r_{\rm A/B}}{r_{\rm B/C}-r_{\rm A/B}}-\frac{1}{2}\right)\right]

To enable hybrid orbital representation, the input XML needs to see the tag ``hybridrep="yes"`` shown in :ref:`Listing 6 <Listing 6>`.

.. code-block::
  :caption: Hybrid orbital representation input example.
  :name: Listing 6

  <determinantset type="bspline" source="i" href="pwscf.h5"
                tilematrix="1 1 3 1 2 -1 -2 1 0" twistnum="-1" gpu="yes" meshfactor="0.8"
                twist="0  0  0" precision="single" hybridrep="yes">
    ...
  </determinantset>

Second, the information describing the atomic regions is required in the particle set, shown in :ref:`Listing 7 <Listing 7>`.

.. code-block::
  :caption: particleset elements for ions with information needed by hybrid orbital representation.
  :name: Listing 7

  <group name="Ni">
    <parameter name="charge">          18 </parameter>
    <parameter name="valence">         18 </parameter>
    <parameter name="atomicnumber" >   28 </parameter>
    <parameter name="cutoff_radius" > 1.6 </parameter>
    <parameter name="inner_cutoff" >  1.3 </parameter>
    <parameter name="lmax" >            5 </parameter>
    <parameter name="spline_radius" > 1.8 </parameter>
    <parameter name="spline_npoints">  91 </parameter>
  </group>

The parameters specific to hybrid representation are listed as

``attrib`` element

Attribute:

+---------------------+--------------+------------+-------------+---------------------------------------+
| **Name**            | **Datatype** | **Values** | **Default** | **Description**                       |
+=====================+==============+============+=============+=======================================+
| ``cutoff_radius``   | Real         | >=0.0      | *None*      | Cutoff radius for B/C boundary        |
+---------------------+--------------+------------+-------------+---------------------------------------+
| ``lmax``            | Integer      | >=0        | *None*      | Largest angular channel               |
+---------------------+--------------+------------+-------------+---------------------------------------+
| ``inner_cutoff``    | Real         | >=0.0      | Dep.        | Cutoff radius for A/B boundary        |
+---------------------+--------------+------------+-------------+---------------------------------------+
| ``spline_radius``   | Real         | >0.0       | Dep.        | Radial function radius used in spine  |
+---------------------+--------------+------------+-------------+---------------------------------------+
| ``spline_npoints``  | Integer      | >0         | Dep.        | Number of spline knots                |
+---------------------+--------------+------------+-------------+---------------------------------------+

- ``cutoff_radius``  is required for every species. If a species is intended to not be covered by atomic regions, setting the value 0.0 will put default values for all the reset parameters. A good value is usually a bit larger than the core radius listed in the pseudopotential file. After a parametric scan, pick the one from the flat energy region with the smallest variance.

- ``lmax`` is required if ``cutoff_radius`` :math:`>` 0.0. This value usually needs to be at least the highest angular momentum plus 2.

- ``inner_cutoff`` is optional and set as ``cutof\_radius`` :math:`-0.3` by default, which is fine in most cases.

- ``spline_radius} and ``spline_npoints`` are optional. By default, they are calculated based on ``cutoff_radius`` and a grid displacement 0.02 bohr.
  If users prefer inputing them, it is required that ``cutoff_radius`` <=  ``spline_radius`` :math:`-` 2 :math:`\times` ``spline_radius``/(``spline_npoints`` :math:`-` 1).

In addition, the hybrid orbital representation allows extra optimization to speed up the nonlocal pseudopotential evaluation using the batched algorithm listed in :ref:`nlpp`.

.. _pwbasis:

Plane-wave basis sets
~~~~~~~~~~~~~~~~~~~~~

.. _hegbasis:

Homogeneous electron gas
~~~~~~~~~~~~~~~~~~~~~~~~

The interacting Fermi liquid has its own special ``determinantset`` for filling up a
Fermi surface.  The shell number can be specified separately for both spin-up and spin-down.
This determines how many electrons to include of each time; only closed shells are currently
implemented.  The shells are filled according to the rules of a square box; if other lattice
vectors are used, the electrons might not fill up a complete shell.

This following example can also be used for Helium simulations by specifying the
proper pair interaction in the Hamiltonian section.

.. code-block::
  :caption: 2D Fermi liquid example: particle specification
  :name: Listing 8

  <qmcsystem>
  <simulationcell name="global">
  <parameter name="rs" pol="0" condition="74">6.5</parameter>
  <parameter name="bconds">p p p</parameter>
  <parameter name="LR_dim_cutoff">15</parameter>
  </simulationcell>
  <particleset name="e" random="yes">
  <group name="u" size="37">
  <parameter name="charge">-1</parameter>
  <parameter name="mass">1</parameter>
  </group>
  <group name="d" size="37">
  <parameter name="charge">-1</parameter>
  <parameter name="mass">1</parameter>
  </group>
  </particleset>
  </qmcsystem>

.. code-block::
  :caption: 2D Fermi liquid example (Slater Jastrow wavefunction)
  :name: Listing 9

  <qmcsystem>
    <wavefunction name="psi0" target="e">
      <determinantset type="electron-gas" shell="7" shell2="7" randomize="true">
    </determinantset>
      <jastrow name="J2" type="Two-Body" function="Bspline" print="no">
        <correlation speciesA="u" speciesB="u" size="8" cusp="0">
          <coefficients id="uu" type="Array" optimize="yes">
        </correlation>
        <correlation speciesA="u" speciesB="d" size="8" cusp="0">
          <coefficients id="ud" type="Array" optimize="yes">
        </correlation>
      </jastrow>

.. _jastrow:

Jastrow Factors
---------------

Jastrow factors are among the simplest and most effective ways of including
dynamical correlation in the trial many body wavefunction.  The resulting many body
wavefunction is expressed as the product of an antisymmetric (in the case
of Fermions) or symmetric (for Bosons) part and a correlating Jastrow factor
like so:

.. math::
  :label: eq8

  \Psi(\vec{R}) = \mathcal{A}(\vec{R}) \exp\left[J(\vec{R})\right]

In this section we will detail the types and forms of Jastrow factor used
in QMCPACK.  Note that each type of Jastrow factor needs to be specified using
its own individual ``jastrow`` XML element.  For this reason, we have repeated the
specification of the ``jastrow`` tag in each section, with specialization for the
options available for that given type of Jastrow.

.. _onebodyjastrow:

One-body Jastrow functions
~~~~~~~~~~~~~~~~~~~~~~~~~~

The one-body Jastrow factor is a form that allows for the direct inclusion
of correlations between particles that are included in the wavefunction with
particles that are not explicitly part of it.  The most common example of
this are correlations between electrons and ions.

The Jastrow function is specified within a ``wavefunction`` element
and must contain one or more ``correlation`` elements specifying
additional parameters as well as the actual coefficients.
:ref:`1bjsplineexamples` gives examples of the typical nesting of
``jastrow``, ``correlation``, and ``coefficient`` elements.


Input Specification
^^^^^^^^^^^^^^^^^^^
Jastrow element:

    +----------+--------------+------------+--------------+----------------+
    | **name** | **datatype** | **values** | **defaults** | **description**|
    |          |              |            |              |                |
    +----------+--------------+------------+--------------+----------------+
    | name     | text         |            | (required)   | Unique name    |
    |          |              |            |              | for this       |
    |          |              |            |              | Jastrow        |
    |          |              |            |              | function       |
    +----------+--------------+------------+--------------+----------------+
    | type     | text         | One-body   | (required)   | Define a       |
    |          |              |            |              | one-body       |
    |          |              |            |              | function       |
    +----------+--------------+------------+--------------+----------------+
    | function | text         | Bspline    | (required)   | BSpline        |
    |          |              |            |              | Jastrow        |
    +----------+--------------+------------+--------------+----------------+
    |          | text         | pade2      |              | Pade form      |
    +----------+--------------+------------+--------------+----------------+
    |          | text         | …          |              | …              |
    +----------+--------------+------------+--------------+----------------+
    | source   | text         | name       | (required)   | Name of        |
    |          |              |            |              | attribute of   |
    |          |              |            |              | classical      |
    |          |              |            |              | particle set   |
    +----------+--------------+------------+--------------+----------------+
    | print    | text         | yes / no   | yes          | Jastrow        |
    |          |              |            |              | factor         |
    |          |              |            |              | printed in     |
    |          |              |            |              | external       |
    |          |              |            |              | file?          |
    +----------+--------------+------------+--------------+----------------+

    +----------+--------------+------------+--------------+--------------+
    | elements |              |            |              |              |
    +----------+--------------+------------+--------------+--------------+
    |          | Correlation  |            |              |              |
    +----------+--------------+------------+--------------+--------------+
    | Contents |              |            |              |              |
    +----------+--------------+------------+--------------+--------------+
    |          | (None)       |            |              |              |
    +----------+--------------+------------+--------------+--------------+

To be more concrete, the one-body Jastrow factors used to describe correlations
between electrons and ions take the form below:

.. math::
  :label: eq9

  J1=\sum_I^{ion0}\sum_i^e u_{ab}(|r_i-R_I|)

where I runs over all of the ions in the calculation, i runs over the
electrons and :math:`u_{ab}` describes the functional form of the
correlation between them. Many different forms of :math:`u_{ab}` are
implemented in QMCPACK. We will detail two of the most common ones
below.

.. _onebodyjastrowspline:

Spline form
...........

The one-body spline Jastrow function is the most commonly used one-body
Jastrow for solids. This form was first described and used in
:cite:`EslerKimCeperleyShulenburger2012`. Here
:math:`u_{ab}` is an interpolating 1D B-spline (tricublc spline on a
linear grid) between zero distance and :math:`r_{cut}`. In 3D periodic
systems the default cutoff distance is the Wigner Seitz cell radius. For
other periodicities, including isolated molecules, the :math:`r_{cut}`
must be specified. The cusp can be set. :math:`r_i` and :math:`R_I` are
most commonly the electron and ion positions, but any particlesets that
can provide the needed centers can be used.

Correlation element:

    +-------------+-------------+-------------+-------------+----------------+
    | **Name**    | **Datatype**| **Values**  | **Defaults**| **Description**|
    |             |             |             |             |                |
    +-------------+-------------+-------------+-------------+----------------+
    | ElementType | Text        | Name        | See below   | Classical      |
    |             |             |             |             | particle       |
    |             |             |             |             | target         |
    +-------------+-------------+-------------+-------------+----------------+
    | SpeciesA    | Text        | Name        | See below   | Classical      |
    |             |             |             |             | particle       |
    |             |             |             |             | target         |
    +-------------+-------------+-------------+-------------+----------------+
    | SpeciesB    | Text        | Name        | See below   | Quantum        |
    |             |             |             |             | species        |
    |             |             |             |             | target         |
    +-------------+-------------+-------------+-------------+----------------+
    | Size        | Integer     | :math:`> 0` | (Required)  | Number of      |
    |             |             |             |             | coefficients   |
    |             |             |             |             |                |
    +-------------+-------------+-------------+-------------+----------------+
    | Rcut        | Real        | :math:`> 0` | See below   | Distance at    |
    |             |             |             |             | which the      |
    |             |             |             |             | correlation    |
    |             |             |             |             | goes to 0      |
    +-------------+-------------+-------------+-------------+----------------+
    | Cusp        | Real        |:math:`\ge 0`| 0           | Value for      |
    |             |             |             |             | use in Kato    |
    |             |             |             |             | cusp           |
    |             |             |             |             | condition      |
    +-------------+-------------+-------------+-------------+----------------+
    | Spin        | Text        | Yes or no   | No          | Spin           |
    |             |             |             |             | dependent      |
    |             |             |             |             | Jastrow        |
    |             |             |             |             | factor         |
    +-------------+-------------+-------------+-------------+----------------+

    +----------+--------------+------------+--------------+--------------+
    | Elements |              |            |              |              |
    +----------+--------------+------------+--------------+--------------+
    |          | Coefficients |            |              |              |
    +----------+--------------+------------+--------------+--------------+
    | Contents |              |            |              |              |
    +----------+--------------+------------+--------------+--------------+
    |          | (None)       |            |              |              |
    +----------+--------------+------------+--------------+--------------+

Additional information:

- ``elementType, speciesA, speciesB, spin``
    For a spin-independent Jastrow factor (spin = “no”), elementType
    should be the name of the group of ions in the classical particleset to
    which the quantum particles should be correlated. For a spin-dependent
    Jastrow factor (spin = “yes”), set speciesA to the group name in the
    classical particleset and speciesB to the group name in the quantum
    particleset.

- ``rcut``
    The cutoff distance for the function in atomic units (bohr). For 3D
    fully periodic systems, this parameter is optional, and a default of the
    Wigner Seitz cell radius is used. Otherwise this parameter is required.

- ``cusp``
    The one-body Jastrow factor can be used to make the wavefunction
    satisfy the electron-ion cusp condition :cite:``kato``. In this
    case, the derivative of the Jastrow factor as the electron approaches
    the nucleus will be given by

.. math::
  :label: eq10

  \left(\frac{\partial J}{\partial r_{iI}}\right)_{r_{iI} = 0} = -Z .

Note that if the antisymmetric part of the wavefunction satisfies the electron-ion cusp
condition (for instance by using single-particle orbitals that respect the cusp condition)
or if a nondivergent pseudopotential is used, the Jastrow should be cuspless at the
nucleus and this value should be kept at its default of 0.

Coefficients element:

    +-----------+--------------+------------+--------------+----------------+
    | **Name**  | **Datatype** | **Values** | **Defaults** | **Description**|
    |           |              |            |              |                |
    +-----------+--------------+------------+--------------+----------------+
    | Id        | Text         |            | (Required)   | Unique         |
    |           |              |            |              | identifier     |
    +-----------+--------------+------------+--------------+----------------+
    | Type      | Text         | Array      | (Required)   |                |
    +-----------+--------------+------------+--------------+----------------+
    | Optimize  | Text         | Yes or no  | Yes          | if no,         |
    |           |              |            |              | values are     |
    |           |              |            |              | fixed in       |
    |           |              |            |              | optimizations  |
    |           |              |            |              |                |
    +-----------+--------------+------------+--------------+----------------+
    +-----------+--------------+------------+--------------+----------------+
    | Elements  |              |            |              |                |
    +-----------+--------------+------------+--------------+----------------+
    | (None)    |              |            |              |                |
    +-----------+--------------+------------+--------------+----------------+
    | Contents  |              |            |              |                |
    +-----------+--------------+------------+--------------+----------------+
    | (No name) | Real array   |            | Zeros        | Jastrow        |
    |           |              |            |              | coefficients   |
    +-----------+--------------+------------+--------------+----------------+

.. _1bjsplineexamples:

Example use cases
.................

Specify a spin-independent function with four parameters. Because rcut  is not
specified, the default cutoff of the Wigner Seitz cell radius is used; this
Jastrow must be used with a 3D periodic system such as a bulk solid. The name of
the particleset holding the ionic positions is "i."

::

  <jastrow name="J1" type="One-Body" function="Bspline" print="yes" source="i">
   <correlation elementType="C" cusp="0.0" size="4">
     <coefficients id="C" type="Array"> 0  0  0  0  </coefficients>
   </correlation>
  </jastrow>

Specify a spin-dependent function with seven up-spin and seven down-spin parameters.
The cutoff distance is set to 6 atomic units.  Note here that the particleset holding
the ions is labeled as ion0 rather than "i," as in the other example.  Also in this case,
the ion is lithium with a coulomb potential, so the cusp condition is satisfied by
setting cusp="d."

::

  <jastrow name="J1" type="One-Body" function="Bspline" source="ion0" spin="yes">
    <correlation speciesA="Li" speciesB="u" size="7" rcut="6">
      <coefficients id="eLiu" cusp="3.0" type="Array">
      0.0 0.0 0.0 0.0 0.0 0.0 0.0
      </coefficients>
    </correlation>
    <correlation speciesA="C" speciesB="d" size="7" rcut="6">
      <coefficients id="eLid" cusp="3.0" type="Array">
      0.0 0.0 0.0 0.0 0.0 0.0 0.0
      </coefficients>
    </correlation>
  </jastrow>

.. _onebodyjastrowpade:

Pade form
.........

Although the spline Jastrow factor is the most flexible and most commonly used form implemented in QMCPACK,
there are times where its flexibility can make it difficult to optimize.  As an example, a spline Jastrow
with a very large cutoff can be difficult to optimize for isolated systems such as molecules because of the small
number of samples present in the tail of the function.  In such cases, a simpler functional
form might be advantageous.  The second-order Pade Jastrow factor, given in :eq:`eq11`, is a good choice
in such cases.

.. math::
  :label: eq11

  u_{ab}(r) = \frac{a*r+c*r^2}{1+b*r}

Unlike the spline Jastrow factor, which includes a cutoff, this form has an infinite range and will be applied to every particle
pair (subject to the minimum image convention).  It also is a cuspless Jastrow factor,
so it should be used either in combination with a single particle basis set that contains the proper cusp or
with a smooth pseudopotential.
