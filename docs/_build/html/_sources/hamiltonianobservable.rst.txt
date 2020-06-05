.. _hamiltobs:

Hamiltonian and Observables
===========================

QMCPACK is capable of the simultaneous measurement of the Hamiltonian and many other quantum operators.  The Hamiltonian attains a special status among the available operators (also referred to as observables) because it ultimately generates all available information regarding the quantum system.  This is evident from an algorithmic standpoint as well since the Hamiltonian (embodied in the projector) generates the imaginary time dynamics of the walkers in DMC and reptation Monte Carlo (RMC).

This section covers how the Hamiltonian can be specified, component by component, by the user in the XML format native to \qmcpack. It also covers the input structure of statistical estimators corresponding to quantum observables such as the density, static structure factor, and forces.

The Hamiltonian
---------------

The many-body Hamiltonian in Hartree units is given by

.. math::
  :label: eq28

  \hat{H} = -\sum_i\frac{1}{2m_i}\nabla_i^2 + \sum_iv^{ext}(r_i) + \sum_{i<j}v^{qq}(r_i,r_j)   + \sum_{i\ell}v^{qc}(r_i,r_\ell)   + \sum_{\ell<m}v^{cc}(r_\ell,r_m)\:.

Here, the sums indexed by :math:`i/j` are over quantum particles, while
:math:`\ell/m` are reserved for classical particles. Often the quantum
particles are electrons, and the classical particles are ions, though is
not limited in this way. The mass of each quantum particle is denoted
:math:`m_i`, :math:`v^{qq}/v^{qc}/v^{cc}` are pair potentials between
quantum-quantum/quantum-classical/classical-classical particles, and
:math:`v^{ext}` denotes a purely external potential.

QMCPACK is designed modularly so that any potential can be supported with
minimal additions to the code base. Potentials currently supported
include Coulomb interactions in open and periodic boundary conditions,
the MPC potential, nonlocal pseudopotentials, helium pair potentials,
and various model potentials such as hard sphere, Gaussian, and modified
Poschl-Teller.

Reference information and examples for the ``<hamiltonian/>`` XML
element are provided subsequently. Detailed descriptions of the input
for individual potentials is given in the sections that follow.

``hamiltonian`` element:

  +------------------+----------------------------------------------------+
  | parent elements: | ``simulation, qmcsystem``                          |
  +------------------+----------------------------------------------------+
  | child elements:  | ``pairpot extpot estimator constant`` (deprecated) |
  +------------------+----------------------------------------------------+

attributes:

  +------------------------+--------------+----------------------+-------------+------------------------------------------+
  | **Name**               | **Datatype** | **Values**           | **Default** | **Description**                          |
  +========================+==============+======================+=============+==========================================+
  | ``name/id``:math:`^o`  | text         | *anything*           | h0          | Unique id for this Hamiltonian instance  |
  +------------------------+--------------+----------------------+-------------+------------------------------------------+
  | ``type``:math:`^o`     | text         |                      | generic     | *No current function*                    |
  +------------------------+--------------+----------------------+-------------+------------------------------------------+
  | ``role``:math:`^o`     | text         | primary/extra        | extra       | Designate as Hamiltonian or not          |
  +------------------------+--------------+----------------------+-------------+------------------------------------------+
  | ``source``:math:`^o`   | text         | ``particleset.name`` | i           | Identify classical ``particleset``       |
  +------------------------+--------------+----------------------+-------------+------------------------------------------+
  | ``target``:math:`^o`   | text         | ``particleset.name`` | e           | Identify quantum ``particleset``         |
  +------------------------+--------------+----------------------+-------------+------------------------------------------+
  | ``default``:math:`^o`  | boolean      | yes/no               | yes         | Include kinetic energy term implicitly   |
  +------------------------+--------------+----------------------+-------------+------------------------------------------+

Additional information:

-  **target:** Must be set to the name of the quantum ``particleset``.
   The default value is typically sufficient. In normal usage, no other
   attributes are provided.

.. code-block::
  :caption: All electron Hamiltonian XML element.
  :name: Listing 14

  <hamiltonian target="e">
    <pairpot name="ElecElec" type="coulomb" source="e" target="e"/>
    <pairpot name="ElecIon"  type="coulomb" source="i" target="e"/>
    <pairpot name="IonIon"   type="coulomb" source="i" target="i"/>
  </hamiltonian>

.. code-block::
  :caption: Pseudopotential Hamiltonian XML element.
  :name: Listing 15

  <hamiltonian target="e">
    <pairpot name="ElecElec"  type="coulomb" source="e" target="e"/>
    <pairpot name="PseudoPot" type="pseudo"  source="i" wavefunction="psi0" format="xml">
      <pseudo elementType="Li" href="Li.xml"/>
      <pseudo elementType="H" href="H.xml"/>
    </pairpot>
    <pairpot name="IonIon"    type="coulomb" source="i" target="i"/>
  </hamiltonian>

Pair potentials
---------------

Many pair potentials are supported.  Though only the most commonly used pair potentials are covered in detail in this section, all currently available potentials are listed subsequently.  If a potential you desire is not listed, or is not present at all, feel free to contact the developers.

``pairpot`` factory element:

  +------------------+--------------------+
  | parent elements: | ``hamiltonian``    |
  +------------------+--------------------+
  | child elements:  | ``type`` attribute |
  +------------------+--------------------+

  +------------------+---------+-----------------------------------------------+
  | **type options** | coulomb | Coulomb/Ewald potential                       |
  +------------------+---------+-----------------------------------------------+
  |                  | pseudo  | Semilocal pseudopotential                     |
  +------------------+---------+-----------------------------------------------+
  |                  | mpc     | Model periodic Coulomb interaction/correction |
  +------------------+---------+-----------------------------------------------+
  |                  | cpp     | Core polarization potential                   |
  +------------------+---------+-----------------------------------------------+
  |                  | skpot   | *Unknown*                                     |
  +------------------+---------+-----------------------------------------------+

shared attributes:

  +-----------------------+--------------+----------------------+------------------------+---------------------------------+
  | **Name**              | **Datatype** | **Values**           | **Default**            | **Description**                 |
  +=======================+==============+======================+========================+=================================+
  | ``type``:math:`^r`    | text         | *See above*          | 0                      | Select pairpot type             |
  +-----------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``name``:math:`^r`    | text         | *Anything*           | any                    | Unique name for this pairpot    |
  +-----------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``source``:math:`^r`  | text         | ``particleset.name`` | ``hamiltonian.target`` | Identify interacting particles  |
  +-----------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``target``:math:`^r`  | text         | ``particleset.name`` | ``hamiltonian.target`` | Identify interacting particles  |
  +-----------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``units``:math:`^o`   | text         |                      | hartree                | *No current function*           |
  +-----------------------+--------------+----------------------+------------------------+---------------------------------+

Additional information:

-  **type:** Used to select the desired pair potential. Must be selected
   from the list of type options.

-  **name:** A unique name used to identify this pair potential. Block
   averaged output data will appear under this name in ``scalar.dat``
   and/or ``stat.h5`` files.

-  **source/target:** These specify the particles involved in a pair
   interaction. If an interaction is between classical (e.g., ions) and
   quantum (e.g., electrons), ``source``/``target`` should be the name
   of the classical/quantum ``particleset``.

-  Only ``Coulomb, pseudo``, and ``mpc`` are described in detail in the
   following subsections. The older or less-used types (``cpp, skpot``)
   are not covered.

-  Available only if ``QMC_CUDA`` is not defined: ``skpot``.

-  Available only if ``OHMMS_DIM==3``: ``mpc, vhxc, pseudo``.

-  Available only if ``OHMMS_DIM==3`` and ``QMC_CUDA`` is not defined:
   ``cpp``.

Coulomb potentials
~~~~~~~~~~~~~~~~~~

The bare Coulomb potential is used in open boundary conditions:

.. math::
  :label: eq29

  V_c^{open} = \sum_{i<j}\frac{q_iq_j}{\left|{r_i-r_j}\right|}\:.

When periodic boundary conditions are selected, Ewald summation is used automatically:

.. math::
  :label: eq30

  V_c^{pbc} = \sum_{i<j}\frac{q_iq_j}{\left|{r_i-r_j}\right|} + \frac{1}{2}\sum_{L\ne0}\sum_{i,j}\frac{q_iq_j}{\left|{r_i-r_j+L}\right|}\:.

The sum indexed by $L$ is over all nonzero simulation cell lattice vectors.  In practice, the Ewald sum is broken into short- and long-range parts in a manner optimized for efficiency (see :cite:`Natoli1995`) for details.

For information on how to set the boundary conditions, consult :ref:`simulationcell`.

``pairpot type=coulomb`` element:

  +------------------+-----------------+
  | parent elements: | ``hamiltonian`` |
  +------------------+-----------------+
  | child elements:  | *None*          |
  +------------------+-----------------+

attributes:

  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | **Name**                | **Datatype** | **Values**           | **Default**            | **Description**                 |
  +=========================+==============+======================+========================+=================================+
  | ``type``:math:`^r`      | text         | **coulomb**          |                        | Must be coulomb                 |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``name/id``:math:`^r`   | text         | *anything*           | ElecElec               | Unique name for interaction     |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``source``:math:`^r`    | text         | ``particleset.name`` | ``hamiltonian.target`` | Identify interacting particles  |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``target``:math:`^r`    | text         | ``particleset.name`` | ``hamiltonian.target`` | Identify interacting particles  |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``pbc``:math:`^o`       | boolean      | yes/no               | yes                    | Use Ewald summation             |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``physical``:math:`^o`  | boolean      | yes/no               | yes                    | Hamiltonian(yes)/Observable(no) |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``forces``              | boolean      | yes/no               | no                     | *Deprecated*                    |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+

Additional information:

-  **type/source/target:** See description for the previous generic
   ``pairpot`` factory element.

-  **name:** Traditional user-specified names for electron-electron,
   electron-ion, and ion-ion terms are ``ElecElec``, ``ElecIon``, and
   ``IonIon``, respectively. Although any choice can be used, the data
   analysis tools expect to find columns in ``*.scalar.dat`` with these
   names.

-  **pbc**: Ewald summation will not be performed if
   ``simulationcell.bconds== n n n``, regardless of the value of
   ``pbc``. Similarly, the ``pbc`` attribute can only be used to turn
   off Ewald summation if ``simulationcell.bconds!= n n n``. The default
   value is recommended.

-  **physical**: If ``physical==yes``, this pair potential is included
   in the Hamiltonian and will factor into the ``LocalEnergy`` reported
   by QMCPACK and also in the DMC branching weight. If ``physical==no``,
   then the pair potential is treated as a passive observable but not as
   part of the Hamiltonian itself. As such it does not contribute to the
   outputted ``LocalEnergy``. Regardless of the value of ``physical``
   output data will appear in ``scalar.dat`` in a column headed by
   ``name``.

.. code-block::
  :caption: QMCPXML element for Coulomb interaction between electrons.
  :name: Listing 16

  <pairpot name="ElecElec" type="coulomb" source="e" target="e"/>

.. code-block::
  :caption: QMCPXML element for Coulomb interaction between electrons and ions (all-electron only).
  :name: Listing 17

  <pairpot name="ElecIon"  type="coulomb" source="i" target="e"/>

.. code-block::
  :caption: QMCPXML element for Coulomb interaction between ions.
  :name: Listing 18

  <pairpot name="IonIon"   type="coulomb" source="i" target="i"/>

.. _nlpp:

Pseudopotentials
~~~~~~~~~~~~~~~~

QMCPACK supports pseudopotentials in semilocal form, which is local in the
radial coordinate and nonlocal in angular coordinates. When all angular
momentum channels above a certain threshold (:math:`\ell_{max}`) are
well approximated by the same potential
(:math:`V_{\bar{\ell}}\equiv V_{loc}`), the pseudopotential separates
into a fully local channel and an angularly nonlocal component:

.. math::
  :label: eq31

  V^{PP} = \sum_{ij}\Big(V_{\bar{\ell}}(\left|{r_i-\tilde{r}_j}\right|) + \sum_{\ell\ne\bar{\ell}}^{\ell_{max}}\sum_{m=-\ell}^\ell |{Y_{\ell m}}\rangle{\big[V_\ell(\left|{r_i-\tilde{r}_j}\right|) - V_{\bar{\ell}}(\left|{r_i-\tilde{r}_j}\right|) \big]}\langle{Y_{\ell m}}| \Big)\:.

Here the electron/ion index is :math:`i/j`, and only one type of ion is
shown for simplicity.

Evaluation of the localized pseudopotential energy
:math:`\Psi_T^{-1}V^{PP}\Psi_T` requires additional angular integrals.
These integrals are evaluated on a randomly shifted angular grid. The
size of this grid is determined by :math:`\ell_{max}`. See
:cite:`Mitas1991` for further detail.

uses the FSAtom pseudopotential file format associated with the “Free
Software Project for Atomic-scale Simulations” initiated in 2002. See
http://www.tddft.org/fsatom/manifest.php for more information. The
FSAtom format uses XML for structured data. Files in this format do not
use a specific identifying file extension; instead they are simply
suffixed with “``.xml``.” The tabular data format of CASINO is also
supported.

``pairpot type=pseudo`` element:

  +------------------+-----------------+
  | parent elements: | ``hamiltonian`` |
  +------------------+-----------------+
  | child elements:  | ``pseudo``      |
  +------------------+-----------------+

attributes:

  +-----------------------------+--------------+-----------------------+------------------------+---------------------------------------------+
  | **Name**                    | **Datatype** | **Values**            | **Default**            | **Description**                             |
  +=============================+==============+=======================+========================+=============================================+
  | ``type``:math:`^r`          | text         | **pseudo**            |                        | Must be pseudo                              |
  +-----------------------------+--------------+-----------------------+------------------------+---------------------------------------------+
  | ``name/id``:math:`^r`       | text         | *anything*            | PseudoPot              | *No current function*                       |
  +-----------------------------+--------------+-----------------------+------------------------+---------------------------------------------+
  | ``source``:math:`^r`        | text         | ``particleset.name``  | i                      | Ion ``particleset`` name                    |
  +-----------------------------+--------------+-----------------------+------------------------+---------------------------------------------+
  | ``target``:math:`^r`        | text         | ``particleset.name``  | ``hamiltonian.target`` | Electron ``particleset`` name               |
  +-----------------------------+--------------+-----------------------+------------------------+---------------------------------------------+
  | ``pbc``:math:`^o`           | boolean      | yes/no                | yes*                   | Use Ewald summation                         |
  +-----------------------------+--------------+-----------------------+------------------------+---------------------------------------------+
  | ``forces``                  | boolean      | yes/no                | no                     | *Deprecated*                                |
  +-----------------------------+--------------+-----------------------+------------------------+---------------------------------------------+
  | ``wavefunction``:math:`^r`  | text         | ``wavefunction.name`` | invalid                | Identify wavefunction                       |
  +-----------------------------+--------------+-----------------------+------------------------+---------------------------------------------+
  | ``format``:math:`^r`        | text         | xml/table             | table                  | Select file format                          |
  +-----------------------------+--------------+-----------------------+------------------------+---------------------------------------------+
  | ``algorithm``:math:`^o`     | text         | batched/default       | default                | Choose NLPP algorithm                       |
  +-----------------------------+--------------+-----------------------+------------------------+---------------------------------------------+
  | ``DLA``:math:`^o`           | text         | yes/no                | no                     | Use determinant localization approximation  |
  +-----------------------------+--------------+-----------------------+------------------------+---------------------------------------------+

Additional information:

-  **type/source/target** See description for the generic ``pairpot``
   factory element.

-  **name:** Ignored. Instead, default names will be present in
   ``*scalar.dat`` output files when pseudopotentials are used. The
   field ``LocalECP`` refers to the local part of the pseudopotential.
   If nonlocal channels are present, a ``NonLocalECP`` field will be
   added that contains the nonlocal energy summed over all angular
   momentum channels.

-  **pbc:** Ewald summation will not be performed if
   ``simulationcell.bconds== n n n``, regardless of the value of
   ``pbc``. Similarly, the ``pbc`` attribute can only be used to turn
   off Ewald summation if ``simulationcell.bconds!= n n n``.

-  **format:** If ``format``\ ==table, QMCPACK looks for ``*.psf`` files
   containing pseudopotential data in a tabular format. The files must
   be named after the ionic species provided in ``particleset`` (e.g.,
   ``Li.psf`` and ``H.psf``). If ``format``\ ==xml, additional
   ``pseudo`` child XML elements must be provided (see the following).
   These elements specify individual file names and formats (both the
   FSAtom XML and CASINO tabular data formats are supported).

-  **algorithm** The default algorithm evaluates the ratios of
   wavefunction components together for each quadrature point and then
   one point after another. The batched algorithm evaluates the ratios
   of quadrature points together for each wavefunction component and
   then one component after another. Internally, it uses
   ``VirtualParticleSet`` for quadrature points. Hybrid orbital
   representation has an extra optimization enabled when using the
   batched algorithm.

-  **DLA** Determinant localization approximation
   (DLA) :cite:`Zen2019DLA` uses only the fermionic part of
   the wavefunction when calculating NLPP.

.. code-block::
  :caption: QMCPXML element for pseudopotential electron-ion interaction (psf files).
  :name: Listing 19

    <pairpot name="PseudoPot" type="pseudo"  source="i" wavefunction="psi0" format="psf"/>

.. code-block::
  :caption: QMCPXML element for pseudopotential electron-ion interaction (xml files).
  :name: Listing 20

    <pairpot name="PseudoPot" type="pseudo"  source="i" wavefunction="psi0" format="xml">
      <pseudo elementType="Li" href="Li.xml"/>
      <pseudo elementType="H" href="H.xml"/>
    </pairpot>

Details of ``<pseudo/>`` input elements are shown in the following. It
is possible to include (or construct) a full pseudopotential directly in
the input file without providing an external file via ``href``. The full
XML format for pseudopotentials is not yet covered.

``pseudo`` element:

  +------------------+-----------------------------+
  | parent elements: | ``pairpot type=pseudo``     |
  +------------------+-----------------------------+
  | child elements:  | ``header local grid``       |
  +------------------+-----------------------------+

attributes:

  +-----------------------------------+--------------+-----------------+-------------+---------------------------+
  | **Name**                          | **Datatype** | **Values**      | **Default** | **Description**           |
  +===================================+==============+=================+=============+===========================+
  | ``elementType/symbol``:math:`^r`  | text         | ``groupe.name`` | none        | Identify ionic species    |
  +-----------------------------------+--------------+-----------------+-------------+---------------------------+
  | ``href``:math:`^r`                | text         | *filepath*      | none        | Pseudopotential file path |
  +-----------------------------------+--------------+-----------------+-------------+---------------------------+
  | ``format``:math:`^r`              | text         | xml/casino      | xml         | Specify file format       |
  +-----------------------------------+--------------+-----------------+-------------+---------------------------+
  | ``cutoff``:math:`^o`              | real         |                 |             | Nonlocal cutoff radius    |
  +-----------------------------------+--------------+-----------------+-------------+---------------------------+
  | ``lmax``:math:`^o`                | integer      |                 |             | Largest angular momentum  |
  +-----------------------------------+--------------+-----------------+-------------+---------------------------+
  | ``nrule``:math:`^o`               | integer      |                 |             | Integration grid order    |
  +-----------------------------------+--------------+-----------------+-------------+---------------------------+

.. code-block::
  :caption: QMCPXML element for pseudopotential of single ionic species.
  :name: Listing 21

    <pseudo elementType="Li" href="Li.xml"/>

MPC Interaction/correction
~~~~~~~~~~~~~~~~~~~~~~~~~~

The MPC interaction is an alternative to direct Ewald summation. The MPC
corrects the exchange correlation hole to more closely match its
thermodynamic limit. Because of this, the MPC exhibits smaller
finite-size errors than the bare Ewald interaction, though a few
alternative and competitive finite-size correction schemes now exist.
The MPC is itself often used just as a finite-size correction in
post-processing (set ``physical=false`` in the input).

``pairpot type=mpc`` element:

  +------------------+-----------------+
  | parent elements: | ``hamiltonian`` |
  +------------------+-----------------+
  | child elements:  | *None*          |
  +------------------+-----------------+

attributes:

  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | **Name**                | **Datatype** | **Values**           | **Default**            | **Description**                 |
  +=========================+==============+======================+========================+=================================+
  | ``type``:math:`^r`      | text         | **mpc**              |                        | Must be MPC                     |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``name/id``:math:`^r`   | text         | *anything*           | MPC                    | Unique name for interaction     |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``source``:math:`^r`    | text         | ``particleset.name`` | ``hamiltonian.target`` | Identify interacting particles  |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``target``:math:`^r`    | text         | ``particleset.name`` | ``hamiltonian.target`` | Identify interacting particles  |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``physical``:math:`^o`  | boolean      | yes/no               | no                     | Hamiltonian(yes)/observable(no) |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+
  | ``cutoff``              | real         | :math:`>0`           | 30.0                   | Kinetic energy cutoff           |
  +-------------------------+--------------+----------------------+------------------------+---------------------------------+

Remarks:

-  ``physical``: Typically set to ``no``, meaning the standard Ewald
   interaction will be used during sampling and MPC will be measured as
   an observable for finite-size post-correction. If ``physical`` is
   ``yes``, the MPC interaction will be used during sampling. In this
   case an electron-electron Coulomb ``pairpot`` element should not be
   supplied.

-  **Developer note:** Currently the ``name`` attribute for the MPC
   interaction is ignored. The name is always reset to ``MPC``.

.. code-block::
  :caption: MPC for finite-size postcorrection.
  :name: Listing 22

    <pairpot type="MPC" name="MPC" source="e" target="e" ecut="60.0" physical="no"/>

General estimators
------------------

A broad range of estimators for physical observables are available in QMCPACK.
The following sections contain input details for the total number
density (``density``), number density resolved by particle spin
(``spindensity``), spherically averaged pair correlation function
(``gofr``), static structure factor (``sk``), static structure factor
(``skall``), energy density (``energydensity``), one body reduced
density matrix (``dm1b``), :math:`S(k)` based kinetic energy correction
(``chiesa``), forward walking (``ForwardWalking``), and force
(``Force``) estimators. Other estimators are not yet covered.

When an ``<estimator/>`` element appears in ``<hamiltonian/>``, it is
evaluated for all applicable chained QMC runs (e.g.,
VMC\ :math:`\rightarrow`\ DMC\ :math:`\rightarrow`\ DMC). Estimators are
generally not accumulated during wavefunction optimization sections. If
an ``<estimator/>`` element is instead provided in a particular
``<qmc/>`` element, that estimator is only evaluated for that specific
section (e.g., during VMC only).

``estimator`` factory element:

  +------------------+----------------------+
  | parent elements: | ``hamiltonian, qmc`` |
  +------------------+----------------------+
  | type selector:   | ``type`` attribute   |
  +------------------+----------------------+

  +------------------+------------------+-----------------------------------------------------------+
  | **type options** | density          | Density on a grid                                         |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | spindensity      | Spin density on a grid                                    |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | gofr             | Pair correlation function (quantum species)               |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | sk               | Static structure factor                                   |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | SkAll            | Static structure factor needed for finite size correction |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | structurefactor  | Species resolved structure factor                         |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | species kinetic  | Species resolved kinetic energy                           |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | latticedeviation | Spatial deviation between two particlesets                |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | momentum         | Momentum distribution                                     |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | energydensity    | Energy density on uniform or Voronoi grid                 |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | dm1b             | One body density matrix in arbitrary basis                |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | chiesa           | Chiesa-Ceperley-Martin-Holzmann kinetic energy correction |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | Force            | Family of "force" estimators (see :ref:`force-est`        |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | ForwardWalking   | Forward walking values for existing estimators            |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | orbitalimages    | Create image files for orbitals, then exit                |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | flux             | Checks sampling of kinetic energy                         |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | localmoment      | Atomic spin polarization within cutoff radius             |
  +------------------+------------------+-----------------------------------------------------------+
  |                  | Pressure         | *No current function*                                     |
  +------------------+------------------+-----------------------------------------------------------+

shared attributes:

  +---------------------+--------------+-------------+-------------+--------------------------------+
  | **Name**            | **Datatype** | **Values**  | **Default** | **Description**                |
  +=====================+==============+=============+=============+================================+
  | ``type``:math:`^r`  | text         | *See above* | 0           | Select estimator type          |
  +---------------------+--------------+-------------+-------------+--------------------------------+
  | ``name``:math:`^r`  | text         | *anything*  | any         | Unique name for this estimator |
  +---------------------+--------------+-------------+-------------+--------------------------------+

Chiesa-Ceperley-Martin-Holzmann kinetic energy correction
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This estimator calculates a finite-size correction to the kinetic energy following the formalism laid out in :cite:`Chiesa2006`.  The total energy can be corrected for finite-size effects by using this estimator in conjunction with the MPC correction.

``estimator type=chiesa`` element:

  +------------------+----------------------+
  | parent elements: | ``hamiltonian, qmc`` |
  +------------------+----------------------+
  | child elements:  | *None*               |
  +------------------+----------------------+

attributes:

  +-----------------------+--------------+------------------------+-------------+----------------------------+
  | **Name**              | **Datatype** | **Values**             | **Default** | **Description**            |
  +=======================+==============+========================+=============+============================+
  | ``type``:math:`^r`    | text         | **chiesa**             |             | Must be chiesa             |
  +-----------------------+--------------+------------------------+-------------+----------------------------+
  | ``name``:math:`^o`    | text         | *anything*             | KEcorr      | Always reset to KEcorr     |
  +-----------------------+--------------+------------------------+-------------+----------------------------+
  | ``source``:math:`^o`  | text         | ``particleset.name``   | e           | Identify quantum particles |
  +-----------------------+--------------+------------------------+-------------+----------------------------+
  | ``psi``:math:`^o`     | text         | ``wavefunction.name``  | psi0        | Identify wavefunction      |
  +-----------------------+--------------+------------------------+-------------+----------------------------+

.. code-block::
  :caption: "Chiesa" kinetic energy finite-size postcorrection.
  :name: Listing 23

     <estimator name="KEcorr" type="chiesa" source="e" psi="psi0"/>

Density estimator
~~~~~~~~~~~~~~~~~

The particle number density operator is given by

.. math::
  :label: eq32

  \hat{n}_r = \sum_i\delta(r-r_i)\:.

The ``density`` estimator accumulates the number density on a uniform
histogram grid over the simulation cell. The value obtained for a grid
cell :math:`c` with volume :math:`\Omega_c` is then the average number
of particles in that cell:

.. math::
  :label: eq33

  n_c = \int dR \left|{\Psi}\right|^2 \int_{\Omega_c}dr \sum_i\delta(r-r_i)\:.

``estimator type=density`` element:

  +------------------+----------------------+
  | parent elements: | ``hamiltonian, qmc`` |
  +------------------+----------------------+
  | child elements:  | *None*               |
  +------------------+----------------------+

attributes:

  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+
  | **Name**                 | **Datatype**  | **Values**                                | **Default**                              | **Description**                          |
  +==========================+===============+===========================================+==========================================+==========================================+
  | ``type``:math:`^r`       | text          | **density**                               |                                          | Must be density                          |
  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+
  | ``name``:math:`^r`       | text          | *anything*                                | any                                      | Unique name for estimator                |
  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+
  | ``delta``:math:`^o`      | real array(3) | :math:`0\le v_i \le 1`                    | 0.1 0.1 0.1                              | Grid cell spacing, unit coords           |
  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+
  | ``x_min``:math:`^o`      | real          | :math:`>0`                                | 0                                        | Grid starting point in x (Bohr)          |
  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+
  | ``x_max``:math:`^o`      | real          | :math:`>0`                                | :math:`|` ``lattice[0]`` :math:`|`       | Grid ending point in x (Bohr)            |
  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+
  | ``y_min``:math:`^o`      | real          | :math:`>0`                                | 0                                        | Grid starting point in y (Bohr)          |
  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+
  | ``y_max``:math:`^o`      | real          | :math:`>0`                                | :math:`|` ``lattice[1]`` :math:`|`       | Grid ending point in y (Bohr)            |
  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+
  | ``z_min``:math:`^o`      | real          | :math:`>0`                                | 0                                        | Grid starting point in z (Bohr)          |
  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+
  | ``z_max``:math:`^o`      | real          | :math:`>0`                                | :math:`|` ``lattice[2]`` :math:`|`       | Grid ending point in z (Bohr)            |
  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+
  | ``potential``:math:`^o`  | boolean       | yes/no                                    | no                                       | Accumulate local potential, *deprecated* |
  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+
  | ``debug``:math:`^o`      | boolean       | yes/no                                    | no                                       | *No current function*                    |
  +--------------------------+---------------+-------------------------------------------+------------------------------------------+------------------------------------------+

Additional information:

-  ``name``: The name provided will be used as a label in the
   ``stat.h5`` file for the blocked output data. Postprocessing tools
   expect ``name="Density."``

-  ``delta``: This sets the histogram grid size used to accumulate the
   density:
   ``delta="0.1 0.1 0.05"``\ :math:`\rightarrow 10\times 10\times 20`
   grid,
   ``delta="0.01 0.01 0.01"``\ :math:`\rightarrow 100\times 100\times 100`
   grid. The density grid is written to a ``stat.h5`` file at the end of
   each MC block. If you request many :math:`blocks` in a ``<qmc/>``
   element, or select a large grid, the resulting ``stat.h5`` file could
   be many gigabytes in size.

-  ``*_min/*_max``: Can be used to select a subset of the simulation
   cell for the density histogram grid. For example if a (cubic)
   simulation cell is 20 Bohr on a side, setting ``*_min=5.0`` and
   ``*_max=15.0`` will result in a density histogram grid spanning a
   :math:`10\times 10\times 10` Bohr cube about the center of the box.
   Use of ``x_min, x_max, y_min, y_max, z_min, z_max`` is only
   appropriate for orthorhombic simulation cells with open boundary
   conditions.

-  When open boundary conditions are used, a ``<simulationcell/>``
   element must be explicitly provided as the first subelement of
   ``<qmcsystem/>`` for the density estimator to work. In this case the
   molecule should be centered around the middle of the simulation cell
   (:math:`L/2`) and not the origin (:math:`0` since the space within
   the cell, and hence the density grid, is defined from :math:`0` to
   :math:`L`).

.. code-block::
  :caption: QMCPXML,caption=Density estimator (uniform grid).
  :name: Listing 24

     <estimator name="Density" type="density" delta="0.05 0.05 0.05"/>

Spin density estimator
~~~~~~~~~~~~~~~~~~~~~~

The spin density is similar to the total density described previously.  In this case, the sum over particles is performed independently for each spin component.

``estimator type=spindensity`` element:

  +------------------+----------------------+
  | parent elements: | ``hamiltonian, qmc`` |
  +------------------+----------------------+
  | child elements:  | *None*               |
  +------------------+----------------------+

attributes:

  +-----------------------+--------------+-----------------+-------------+-------------------------------+
  | **Name**              | **Datatype** | **Values**      | **Default** | **Description**               |
  +=======================+==============+=================+=============+===============================+
  | ``type``:math:`^r`    | text         | **spindensity** |             | Must be spindensity           |
  +-----------------------+--------------+-----------------+-------------+-------------------------------+
  | ``name`:math:`^r`     | text         | *anything*      | any         | Unique name for estimator     |
  +-----------------------+--------------+-----------------+-------------+-------------------------------+
  | ``report``:math:`^o`  | boolean      | yes/no          | no          | Write setup details to stdout |
  +-----------------------+--------------+-----------------+-------------+-------------------------------+

parameters:

  +----------------------------+------------------+----------------------+-------------+----------------------------------+
  | **Name**                   | **Datatype**     | **Values**           | **Default** | **Description**                  |
  +============================+==================+======================+=============+==================================+
  | ``grid``:math:`^o`         | integer array(3) | :math:`v_i>`         |             | Grid cell count                  |
  +----------------------------+------------------+----------------------+-------------+----------------------------------+
  | ``dr``:math:`^o`           | real array(3)    | :math:`v_i>`         |             | Grid cell spacing (Bohr)         |
  +----------------------------+------------------+----------------------+-------------+----------------------------------+
  | ``cell``:math:`^o`         | real array(3,3)  | *anything*           |             | Volume grid exists in            |
  +----------------------------+------------------+----------------------+-------------+----------------------------------+
  | ``corner``:math:`^o`       | real array(3)    | *anything*           |             | Volume corner location           |
  +----------------------------+------------------+----------------------+-------------+----------------------------------+
  | ``center``:math:`^o`       | real array (3)   | *anything*           |             | Volume center/origin location    |
  +----------------------------+------------------+----------------------+-------------+----------------------------------+
  | ``voronoi``:math:`^o`      | text             | ``particleset.name`` |             | *Under development*              |
  +----------------------------+------------------+----------------------+-------------+----------------------------------+
  | ``test_moves``:math:`^o`   | integer          | :math:`>=0`          | 0           | Test estimator with random moves |
  +----------------------------+------------------+----------------------+-------------+----------------------------------+

Additional information:

-  ``name``: The name provided will be used as a label in the
   ``stat.h5`` file for the blocked output data. Postprocessing tools
   expect ``name="SpinDensity."``

-  ``grid``: The grid sets the dimension of the histogram grid. Input
   like ``<parameter name="grid"> 40 40 40 </parameter>`` requests a
   :math:`40 \times 40\times 40` grid. The shape of individual grid
   cells is commensurate with the supercell shape.

-  ``dr``: The ``dr`` sets the real-space dimensions of grid cell edges
   (Bohr units). Input like
   ``<parameter name="dr"> 0.5 0.5 0.5 </parameter>`` in a supercell
   with axes of length 10 Bohr each (but of arbitrary shape) will
   produce a :math:`20\times 20\times 20` grid. The inputted ``dr``
   values are rounded to produce an integer number of grid cells along
   each supercell axis. Either ``grid`` or ``dr`` must be provided, but
   not both.

-  ``cell``: When ``cell`` is provided, a user-defined grid volume is
   used instead of the global supercell. This must be provided if open
   boundary conditions are used. Additionally, if ``cell`` is provided,
   the user must specify where the volume is located in space in
   addition to its size/shape (``cell``) using either the ``corner`` or
   ``center`` parameters.

-  ``corner``: The grid volume is defined as
   :math:`corner+\sum_{d=1}^3u_dcell_d` with :math:`0<u_d<1` (“cell”
   refers to either the supercell or user-provided cell).

-  ``center``: The grid volume is defined as
   :math:`center+\sum_{d=1}^3u_dcell_d` with :math:`-1/2<u_d<1/2`
   (“cell” refers to either the supercell or user-provided cell).
   ``corner/center`` can be used to shift the grid even if ``cell`` is
   not specified. Simultaneous use of ``corner`` and ``center`` will
   cause QMCPACK to abort.

.. code-block::
  :caption: Spin density estimator (uniform grid).
  :name: Listing 25

  <estimator type="spindensity" name="SpinDensity" report="yes">
    <parameter name="grid"> 40 40 40 </parameter>
  </estimator>

.. code-block::
  :caption: Spin density estimator (uniform grid centered about origin).
  :name: Listing 26

  <estimator type="spindensity" name="SpinDensity" report="yes">
    <parameter name="grid">
      20 20 20
    </parameter>
    <parameter name="center">
      0.0 0.0 0.0
    </parameter>
    <parameter name="cell">
      10.0  0.0  0.0
       0.0 10.0  0.0
       0.0  0.0 10.0
    </parameter>
  </estimator>

Pair correlation function, :math:`g(r)`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The functional form of the species-resolved radial pair correlation function operator is

.. math::
  :label: eq34

  g_{ss'}(r) = \frac{V}{4\pi r^2N_sN_{s'}}\sum_{i_s=1}^{N_s}\sum_{j_{s'}=1}^{N_{s'}}\delta(r-|r_{i_s}-r_{j_{s'}}|)\:,

where :math:`N_s` is the number of particles of species :math:`s` and
:math:`V` is the supercell volume. If :math:`s=s'`, then the sum is
restricted so that :math:`i_s\ne j_s`.

In QMCPACK, an estimate of :math:`g_{ss'}(r)` is obtained as a radial
histogram with a set of :math:`N_b` uniform bins of width
:math:`\delta r`. This can be expressed analytically as

.. math::
  :label: eq35

  \tilde{g}_{ss'}(r) = \frac{V}{4\pi r^2N_sN_{s'}}\sum_{i=1}^{N_s}\sum_{j=1}^{N_{s'}}\frac{1}{\delta r}\int_{r-\delta r/2}^{r+\delta r/2}dr'\delta(r'-|r_{si}-r_{s'j}|)\:,

where the radial coordinate :math:`r` is restricted to reside at the bin
centers, :math:`\delta r/2, 3 \delta r/2, 5 \delta r/2, \ldots`.

``estimator type=gofr`` element:

  +------------------+----------------------+
  | parent elements: | ``hamiltonian, qmc`` |
  +------------------+----------------------+
  | child elements:  | *None*               |
  +------------------+----------------------+

attributes:

  +-------------------------------+--------------+----------------------+------------------------+-------------------------+
  | **Name**                      | **Datatype** | **Values**           | **Default**            | **Description**         |
  +===============================+==============+======================+========================+=========================+
  | ``type``:math:`^r`            | text         | **gofr**             |                        | Must be gofr            |
  +-------------------------------+--------------+----------------------+------------------------+-------------------------+
  | ``name``:math:`^o`            | text         | *anything*           | any                    | *No current function*   |
  +-------------------------------+--------------+----------------------+------------------------+-------------------------+
  | ``num_bin``:math:`^r`         | integer      | :math:`>1`           | 20                     | # of histogram bins     |
  +-------------------------------+--------------+----------------------+------------------------+-------------------------+
  | ``rmax``:math:`^o`            | real         | :math:`>0`           | 10                     | Histogram extent (Bohr) |
  +-------------------------------+--------------+----------------------+------------------------+-------------------------+
  | ``dr``:math:`^o`              | real         | :math:`0`            | 0.5                    | *No current function*   |
  +-------------------------------+--------------+----------------------+------------------------+-------------------------+
  | ``debug``:math:`^o`           | boolean      | yes/no               | no                     | *No current function*   |
  +-------------------------------+--------------+----------------------+------------------------+-------------------------+
  | ``target``:math:`^o`          | text         | ``particleset.name`` | ``hamiltonian.target`` | Quantum particles       |
  +-------------------------------+--------------+----------------------+------------------------+-------------------------+
  | ``source/sources``:math:`^o`  | text array   | ``particleset.name`` | ``hamiltonian.target`` | Classical particles     |
  +-------------------------------+--------------+----------------------+------------------------+-------------------------+

Additional information:

-  ``num_bin:`` This is the number of bins in each species pair radial
   histogram.

-  ``rmax:`` This is the maximum pair distance included in the
   histogram. The uniform bin width is
   :math:`\delta r=\texttt{rmax/num\_bin}`. If periodic boundary
   conditions are used for any dimension of the simulation cell, then
   the default value of ``rmax`` is the simulation cell radius instead
   of 10 Bohr. For open boundary conditions, the volume (:math:`V`) used
   is 1.0 Bohr\ :math:`^3`.

-  ``source/sources:`` If unspecified, only pair correlations between
   each species of quantum particle will be measured. For each classical
   particleset specified by ``source/sources``, additional pair
   correlations between each quantum and classical species will be
   measured. Typically there is only one classical particleset (e.g.,
   ``source="ion0"``), but there can be several in principle (e.g.,
   ``sources="ion0 ion1 ion2"``).

-  ``target:`` The default value is the preferred usage (i.e.,
   ``target`` does not need to be provided).

-  Data is output to the ``stat.h5`` for each QMC subrun. Individual
   histograms are named according to the quantum particleset and index
   of the pair. For example, if the quantum particleset is named “e" and
   there are two species (up and down electrons, say), then there will
   be three sets of histogram data in each ``stat.h5`` file named
   ``gofr_e_0_0``, ``gofr_e_0_1``, and ``gofr_e_1_1`` for up-up,
   up-down, and down-down correlations, respectively.

.. code-block::
  :caption: Pair correlation function estimator element.
  :name: Listing 27

  <estimator type="gofr" name="gofr" num_bin="200" rmax="3.0" />

.. code-block::
  :caption: Pair correlation function estimator element with additional electron-ion correlations.
  :name: Listing 28

  <estimator type="gofr" name="gofr" num_bin="200" rmax="3.0" source="ion0" />

Static structure factor, :math:`S(k)`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let
:math:`\rho^e_{\mathbf{k}}=\sum_j e^{i \mathbf{k}\cdot\mathbf{r}_j^e}`
be the Fourier space electron density, with :math:`\mathbf{r}^e_j` being
the coordinate of the j-th electron. :math:`\mathbf{k}` is a wavevector
commensurate with the simulation cell. QMCPACK allows the user to
accumulate the static electron structure factor :math:`S(\mathbf{k})` at
all commensurate :math:`\mathbf{k}` such that
:math:`|\mathbf{k}| \leq (LR\_DIM\_CUTOFF) r_c`. :math:`N^e` is the
number of electrons, ``LR_DIM_CUTOFF`` is the optimized breakup
parameter, and :math:`r_c` is the Wigner-Seitz radius. It is defined as
follows:

.. math::
  :label: eq36

  S(\mathbf{k}) = \frac{1}{N^e}\langle \rho^e_{-\mathbf{k}} \rho^e_{\mathbf{k}} \rangle\:.

``estimator type=sk`` element:

  +------------------+----------------------+
  | parent elements: | ``hamiltonian, qmc`` |
  +------------------+----------------------+
  | child elements:  | *None*               |
  +------------------+----------------------+

attributes:

  +---------------------+--------------+------------+-------------+-----------------------------------------------------+
  | **Name**            | **Datatype** | **Values** | **Default** | **Description**                                     |
  +=====================+==============+============+=============+=====================================================+
  | ``type``:math:`^r`  | text         | sk         |             | Must sk                                             |
  +---------------------+--------------+------------+-------------+-----------------------------------------------------+
  | ``name``:math:`^r`  | text         | *anything* | any         | Unique name for estimator                           |
  +---------------------+--------------+------------+-------------+-----------------------------------------------------+
  | ``hdf5``:math:`^o`  | boolean      | yes/no     | no          |  Output to ``stat.h5`` (yes) or ``scalar.dat`` (no) |
  +---------------------+--------------+------------+-------------+-----------------------------------------------------+

Additional information:

-  ``name:`` This is the unique name for estimator instance. A data
   structure of the same name will appear in ``stat.h5`` output files.

-  ``hdf5:`` If ``hdf5==yes``, output data for :math:`S(k)` is directed
   to the ``stat.h5`` file (recommended usage). If ``hdf5==no``, the
   data is instead routed to the ``scalar.dat`` file, resulting in many
   columns of data with headings prefixed by ``name`` and postfixed by
   the k-point index (e.g., ``sk_0 sk_1 …sk_1037 …``).

-  This estimator only works in periodic boundary conditions. Its
   presence in the input file is ignored otherwise.

-  This is not a species-resolved structure factor. Additionally, for
   :math:`\mathbf{k}` vectors commensurate with the unit cell,
   :math:`S(\mathbf{k})` will include contributions from the static
   electronic density, thus meaning it will not accurately measure the
   electron-electron density response.

.. code-block::
  :caption: Static structure factor estimator element.
  :name: Listing 29

    <estimator type="sk" name="sk" hdf5="yes"/>

Static structure factor, ``SkAll``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
