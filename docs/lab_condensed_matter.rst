.. _lab-condensed-matter:

Lab 4: Condensed Matter Calculations
====================================



.. **Lab author: Luke Shulenburger**
   Sandia National Laboratories is a multiprogram laboratory managed and
   operated by Sandia Corporation, a wholly owned subsidiary of Lockheed
   Martin Corporation, for the U.S. Department of Energy’s National
   Nuclear Security Administration under Contract No. DE-AC04-94AL85000.

Topics covered in this lab
--------------------------

-  Tiling DFT primitive cells into QMC supercells

-  Reducing finite-size errors via extrapolation

-  Reducing finite-size errors via averaging over twisted boundary
   conditions

-  Using the B-spline mesh factor to reduce memory requirements

-  Using a coarsely resolved vacuum buffer region to reduce memory
   requirements

-  Calculating the DMC total energies of representative 2D and 3D
   extended systems

Lab directories and files
-------------------------

::

  labs/lab4_condensed_matter/
  ├── Be-2at-setup.py           - DFT only for prim to conv cell
  ├── Be-2at-qmc.py             - QMC only for prim to conv cell
  ├── Be-16at-qmc.py            - DFT and QMC for prim to 16 atom cell
  ├── graphene-setup.py         - DFT and OPT for graphene
  ├── graphene-loop-mesh.py     - VMC scan over orbital bspline mesh factors
  ├── graphene-final.py         - DMC for final meshfactor
  └── pseudopotentials          - pseudopotential directory
      ├── Be.ncpp                 - Be PP for Quantum ESPRESSO
      ├── Be.xml                  - Be PP for QMCPACK
      ├── C.BFD.upf               - C  PP for Quantum ESPRESSO
      └── C.BFD.xml               - C  PP for QMCPACK

The goal of this lab is to introduce you to the somewhat specialized problems involved in performing DMC calculations on condensed matter as opposed to the atoms and molecules that were the focus of the preceding labs.   Calculations will be performed on two different systems.  Firstly, we will perform a series of calculations on BCC beryllium, focusing on the necessary methodology to limit finite-size effects.  Secondly, we will perform calculations on graphene as an example of a system where QMCPACK’s capability to handle cases with mixed periodic and open boundary conditions is useful.  This example will also focus on strategies to limit memory usage for such systems.
All of the calculations performed in this lab will use the Nexus workflow management system, which vastly simplifies the process by automating the steps of generating trial wavefunctions and performing DMC calculations.

Preliminaries
-------------

For any DMC calculation, we must start with a trial wavefunction. As is typical for our calculations of condensed matter, we will produce this wavefunction using DFT.  Specifically, we will use QE to generate a Slater determinant of SPOs.  This is done as a three-step process.  First, we calculate the converged charge density by performing a DFT calculation with a fine grid of k-points to fully sample the Brillouin zone.  Next, a non-self- consistent calculation is performed at the specific k-points needed for the supercell and twists needed in the DMC calculation (more on this later).  Finally, a wavefunction is converted from the binary representation used by QE to the portable hdf5 representation used by QMCPACK.

The choice of k-points necessary to generate the wavefunctions depends on both the supercell chosen for the DMC calculation and by the supercell twist vectors needed.  Recall that the wavefunction in a plane-wave DFT calculation is written using Bloch's theorem as:

.. math::
  :label: eq70

  \Psi(\vec{r}) = e^{i\vec{k}\cdot\vec{r}}u(\vec{r})\:,

where :math:`\vec{k}` is confined to the first Brillouin zone of the
cell chosen and :math:`u(\vec{r})` is periodic in this simulation cell.
A plane-wave DFT calculation stores the periodic part of the
wavefunction as a linear combination of plane waves for each SPO at all
k-points selected. The symmetry of the system allows us to generate an
arbitrary supercell of the primitive cell as follows: Consider the set
of primitive lattice vectors, :math:`\{ \mathbf{a}^p_1, \mathbf{a}^p_2,
\mathbf{a}^p_3\}`. We may write these vectors in a matrix,
:math:`\mathbf{L}_p`, the rows of which are the primitive lattice
vectors. Consider a nonsingular matrix of integers, :math:`\mathbf{S}`.
A corresponding set of supercell lattice vectors,
:math:`\{\mathbf{a}^s_1, \mathbf{a}^s_2, \mathbf{a}^s_3\}`, can be
constructed by the matrix product

.. math::
  :label: eq71

  \mathbf{a}^s_i = S_{ij} \mathbf{a}^p_j]\:.

If the primitive cell contains :math:`N_p` atoms, the supercell will
then contain :math:`N_s = |\det(\mathbf{S})| N_p` atoms.

Now, the wavefunciton at any point in this new supercell can be related
to the wavefunction in the primitive cell by finding the linear
combination of primitive lattice vectors that maps this point back to
the primitive cell:

.. math::
  :label: eq72

  \vec{r}' = \vec{r} + x \mathbf{a}^p_1 + y \mathbf{a}^p_2 + z\mathbf{a}^p_3 = \vec{r} + \vec{T}\:,

where :math:`x, y, z` are integers. Now the wavefunction in the
supercell at point :math:`\vec{r}'` can be written in terms of the
wavefunction in the primitive cell at :math:`\vec{r}'` as:

.. math::
  :label: eq73

  \Psi(\vec{r}) = \Psi(\vec{r}') e^{i \vec{T} \cdot \vec{k}}\:,


where :math:`\vec{k}` is confined to the first Brillouin zone of the
primitive cell. We have also chosen the supercell twist vector, which
places a constraint on the form of the wavefunction in the supercell.
The combination of these two constraints allows us to identify family of
N k-points in the primitive cell that satisfy the constraints. Thus, for
a given supercell tiling matrix and twist angle, we can write the
wavefunction everywhere in the supercell by knowing the wavefunction a N
k-points in the primitive cell. This means that the memory necessary to
store the wavefunction in a supercell is only linear in the size of the
supercell rather than the quadratic cost if symmetry were neglected.

Total energy of BCC beryllium
-----------------------------
