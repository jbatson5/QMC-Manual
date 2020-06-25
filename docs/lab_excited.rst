.. _excited:

Lab 5: Excited state calculations
=================================

.. Lab author: Kayahan Saritas, Oak Ridge National Laboratory. Creation date: November 29, 2018

Topics covered in this lab
--------------------------

-  Tiling DFT primitive cells into optimal QMC supercells

-  Fundamentals of between neutral and charged calculations

-  Calculating quasiparticle excitation energies of condensed matter
   systems

-  Calculating optical excitation energies of condensed matter systems

Lab directories and files
-------------------------

::

  labs/lab5_excited_properties/
  ├── band.py           - Band structure calculation for Carbon Diamond
  ├── optical.py        - VMC optical gap calculation using the tiling matrix from band.py
  ├── quasiparticle.py  - VMC quasiparticle gap calculation using the tiling matrix from band.py
  └── pseudopotentials      - pseudopotential directory
      ├── C.BFD.upf         - C PP for Quantum ESPRESSO
      └── C.BFD.xml         - C PP for QMCPACK

The goal of this lab is to perform neutral and charged excitation
calculations in condensed matter systems using QMCPACK. Throughout this
lab, a working knowledge of *Lab4 Condensed Matter Calculations* is
assumed. First, we will introduce the concepts of neutral and charged
excitations. We will briefly discuss these in relation to the specific
experimental studies that must be used to benchmark DMC results.
Secondly, we will perform charged (quasiparticle) and neutral (optical)
excitations calculations on C-diamond.

Basics and excited state experiments
------------------------------------

Although VMC and DMC methods are better suited for studying ground state
properties of materials, they can still provide useful information
regarding the excited states. Unlike the applications of band structure
theory such as DFT and GW, it is more challenging to obtain the complete
excitation spectra using DMC. However, it is relatively straightforward
to calculate the band gap minimum of a condensed matter system using
DMC.

We will briefly discuss the two main ways of obtaining the band gap
minimum through experiments: photoemission and absorption studies. The
energy required to remove an electron from a neutral system is called
the IP (ionization potential), which is available from direct
photoemission experiments. In contrast, the emission energy of a
negatively charged system (or the energy required to convert a
negatively charged system to a neutral system), known as electron
affinity (EA), is available from inverse photoemission experiments.
Outlines of these experiments are shown in :numref:`fig22`.

.. _fig22:
.. figure:: /figs/lab_excited_experiments.png
  :width: 500
  :align: center

  Direct and inverse photoemission experiments involve charged excitations, whereas optical absorption experiments involve excitations that are just enough to be excited to the conduction band. From :cite:`Onida2002a`

Following the explanation in the previous paragraph and :numref:`fig22`, the *quasiparticle* band gap of a material can be defined as:

.. math::
  :label: eq76

  E_g=EA-IP=(E_{N+1}^{CBM}-E_{N}^{K'})-(E_{N}^{K'}-E_{N-1}^{VBM})=E_{N+1}^{CBM}+E_{N-1}^{VBM}-2*E_{N}^{K'}\label{eq:qp}\:,

where :math:`N` is the number of electrons in the neutral system and
:math:`E_{N}` is the ground state energy of the neutral system. CBM and
VBM stand for the conduction band minimum and valence band maximum,
respectively. K’ can formally be arbitrary at the infinite limit.
However, in practical calculations, a supertwist that accommodates both
CBM and VBM can be more efficient in terms of computational time and
systematic finite-size error cancellation. In the literature, the
quasiparticle gap is also called the electronic gap. The term electronic
comes from the fact that in both photoemission experiments, it is
assumed that the perturbed electron is not interacting with the sample.

Additionally, absorption experiments can be performed in which electrons
are perturbed at relatively lower energies, just enough to be excited
into the conduction band. In absorption experiments, electrons are
perturbed at lower energies. Therefore, they are not completely free and
the system is still considered neutral. Since a *quasihole* and
*quasielectron* are formed simultaneously, a bound state is created,
unlike the free electron in the quasiparticle gap as described
previously. This process is also known as *optical* excitation, which is
schematically shown in :numref:`fig22`, under “Absorption." The
optical gap can be formulated as follows:

.. math::
  :label: eq77

  E_g^{K_1 {\rightarrow} K_2}=E^{K_1 {\rightarrow} K_2}- E_{0}\label{eq:optical}\:,

where :math:`E^{K_1 {\rightarrow} K_2}` is the energy of the system when
a valence electron at wavevector :math:`K_1` is promoted to the
conduction band at wavevector :math:`K_2`. Therefore, the
:math:`E_g^{K_1 {\rightarrow} K_2}` is called the optical gap for
promoting an electron at :math:`K_1` to :math:`K_2`. If both CBM and VBM
are on the same k-vector then the material is called direct band gap
since it can directly emit photons without any external perturbation
(phonons). However, if CBM and VBM share different k-vectors, then the
photon-emitting electron has to transfer some of its momenta to the
crystal lattice and then decay to the ground state. As this process
involves an intermediate step, this property is called the indirect band
gap. The difference between the optical and electronic band gaps is
called the exciton binding energy. Exciton binding energy is very
important for optoelectronic applications such as lasers. Since the
recombination usually occurs between free holes and free electrons, a
bound electron and hole state means that the spectrum of emission
energies will be narrower. In the examples that follow, we will
investigate the optical excitations of C-diamond.

.. _lab-ex-prep:

Preparation for the excited state calculations
----------------------------------------------

In this section, we will study the preparation steps to perform excited
state calculations with QMC. Here, the most basic steps are listed in
the implementation order:

#. Identify the high-symmetry k-points of the standardized primitive
   cell.

#. Perform DFT band structure calculation along high-symmetry paths.

#. Find a supertwist that includes all the k-points of interest.

#. Identify the indexing of k-points in the supertwist to be used in
   QMCPACK.

.. _lab-ex-highk:

Identifying high-symmetry k-points
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Primitive cell is the most basic, nonunique repeat unit of a crystal in
real space. However, the translations of the repeat unit, the Bravais
lattice is unique for each crystal and can be represented using discrete
translation operations, :math:`R_n`:

.. math::
  :label: eq78

  {\bf R_n} = n_1{\bf a_1} + n_2{\bf a_2} + n_3{\bf a_3}\:,

:math:`a_n` are the real-space lattice vectors in three dimensions.
Thanks to the periodicity of the Bravais lattice, a crystal can also be
represented using periodic functions in the reciprocal space:

.. math::
  :label: eq79

  f({\bf R_n + r})= \sum_{m}f_me^{iG_m({\bf R_n+r})}\label{eqn:lab_ex_rec_real}\:,

where :math:`G_m` are called as the reciprocal lattice vectors.
:eq:`eq79` also satisfies the equality
:math:`G_m\cdot{R_n}=2{\pi}N`. High-symmetry structures can be
represented using a subspace of the BZ, which is called as the
irreducible Brillouin Zone (iBZ). If we choose a series of paths of
high-symmetry k-points that encapsulates the iBZ, we can determine the
band gap and electronic structure of the material. For more discussion,
please refer to any solid-state physics textbook.

There are multiple practical ways to find the high-symmetry k-point path.
For example, pymatgen, :cite:`Ong2013` XCRYSDEN :cite:`Kokalj1999` or SeeK-path :cite:`Hinuma2017` can be used. 
