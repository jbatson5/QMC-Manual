.. _input-overview:

Input file overview
===================

This chapter introduces XML as it is used in the QMCPACK input file.  The focus is on the XML file format itself and the general structure of the input file rather than an exhaustive discussion of all keywords and structure elements.

QMCPACK uses XML to represent structured data in its input file.  Instead of text blocks like

::

  begin project
    id     = vmc
    series = 0
  end project

  begin vmc
    move     = pbyp
    blocks   = 200
    steps    =  10
    timestep = 0.4
  end vmc

QMCPACK input looks like

::

  <project id="vmc" series="0">
  </project>

  <qmc method="vmc" move="pbyp">
     <parameter name="blocks"  >  200 </parameter>
     <parameter name="steps"   >   10 </parameter>
     <parameter name="timestep">  0.4 </parameter>
  </qmc>

XML elements start with ``<element\_name>``, end with ``</element\_name>}``, and can be nested within each other to denote substructure (the trial wavefunction is composed of a Slater determinant and a Jastrow factor, which are each further composed of :math:`...`).  ``id`` and ``series`` are attributes of the ``<project/>`` element.  XML attributes are generally used to represent simple values, like names, integers, or real values.  Similar functionality is also commonly provided by ``<parameter/>`` elements like those previously shown.

The overall structure of the input file reflects different aspects of the QMC simulation: the simulation cell, particles, trial wavefunction, Hamiltonian, and QMC run parameters.  A condensed version of the actual input file is shown as follows:

::

  <?xml version="1.0"?>
  <simulation>

  <project id="vmc" series="0">
    ...
  </project>

  <qmcsystem>

    <simulationcell>
      ...
    </simulationcell>

    <particleset name="e">
      ...
    </particleset>

    <particleset name="ion0">
      ...
    </particleset>

    <wavefunction name="psi0" ... >
      ...
      <determinantset>
        <slaterdeterminant>
          ..
        </slaterdeterminant>
      </determinantset>
      <jastrow type="One-Body" ... >
         ...
      </jastrow>
      <jastrow type="Two-Body" ... >
        ...
      </jastrow>
    </wavefunction>

    <hamiltonian name="h0" ... >
      <pairpot type="coulomb" name="ElecElec" ... />
      <pairpot type="coulomb" name="IonIon"   ... />
      <pairpot type="pseudo" name="PseudoPot" ... >
        ...
      </pairpot>
    </hamiltonian>

   </qmcsystem>

   <qmc method="vmc" move="pbyp">
     <parameter name="warmupSteps">   20 </parameter>
     <parameter name="blocks"     >  200 </parameter>
     <parameter name="steps"      >   10 </parameter>
     <parameter name="timestep"   >  0.4 </parameter>
   </qmc>

  </simulation>

The omitted portions ``...`` are more fine-grained inputs such as the axes of the simulation cell, the number of up and down electrons, positions of atomic species, external orbital files, starting Jastrow parameters, and external pseudopotential files.

Project
-------
