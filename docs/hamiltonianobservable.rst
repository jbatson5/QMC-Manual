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

MPC Interaction/correction
~~~~~~~~~~~~~~~~~~~~~~~~~~
