.. _R-CESM_arch:


==========================
 R-CESM Modeling Framework
==========================

R-CESM version 1.0.0 is based on the Community Earth System Model (CESM) coupling framework (CPL7, :raw-latex:`\citet{cpl7}`) distributed with CESM version 1.1 :raw-latex:`\citep{cesm1p1}`. Our modeling framework adds two regional component models to the CESM2/CIME framework: WRF for the atmosphere and ROMS for the ocean. In the R-CESM implementation, WRF, ROMS, and CLM4 (Community Land model, version 4, Lawrence et al. 2019) are coupled using the CIME implemented coupler (other CESM2 components are planned to be incorporated in the future). As is standard in the CESM2/CIME framewrok, surface fluxes over land are calculated by the land component model (CLM4) and passed to WRF through the coupler. Over the oceans, however, R-CESM can either calculate surface fluxes using the standard flux scheme in CESM2/CIME or employ WRF’s built-in surface layer schemes which are communicated to ROMS through the coupler (Fig. 1a). Thus, in R-CESM, users have the flexibility to use air-sea flux calculation schemes of their choice while keeping other model physics the same. 
The major changes made to the original ROMS and WRF source codes are the addition of a wrapper code that allows them to interface with the CESM2/CIME framework. For ROMS, these include: i) bypassing surface forcing inputs and surface air-sea flux calculations within ROMS; ii) sending sea surface temperature (SST) and surface current velocities thorough the coupler; and iii) receiving surface fluxes and surface pressure from the coupler. When using the CESM flux scheme to calculate air-sea fluxes, the changes in WRF source code include: i) bypassing surface layer scheme within WRF; ii) sending temperature, humidity, winds (on the lowest model level), surface downward radiative fluxes, and surface pressure to the coupler; and iii) receiving turbulent fluxes and stability parameters from the coupler. 
As some of the variables required by the WRF planetary boundary layer parameterization schemes were not diagnosed and readily available in the CESM flux scheme (for example, bulk Richardson number and momentum roughness length), the coupler source code is modified to enable these outputs (see Appendix B for details). As such, the CESM flux scheme can be paired with the WRF built-in planetary boundary layer scheme. Similar modifications are incorporated in the atmosphere-land flux calculations in CLM4 to get the required variables for the WRF physics parameterization schemes. 
In summary, the key aspects of the R-CESM air-sea flux schemes are:
Consistent surface flux calculations for both ocean and atmospheric components, irrespective of air-sea flux scheme choices (WRF or CESM scheme); 
Availability of the CESM air-sea flux scheme in WRF, making R-CESM an ideal testbed to validate and tune CESM atmosphere physics parameterizations at very high resolutions.



It includes active and
data regional ocean, atmosphere, land and ice components (Table
`[tab:compo] <#tab:compo>`__). The Gulf of Mexico test case utilize WRF
for atmospheric model and xROMS for ocean component. Important aspects
of the component models are discussed in this chapter.

.. table:: Model components available in CRESM (from
:raw-latex:`\citet{montuoro17}`).

   +-----------------+-----------------+-----------------+-----------------+
   | Component       | Model           | Status          | Description     |
   +=================+=================+=================+=================+
   | Atmosphere      | WRF 3.5.1       | Stable          | Adavanced       |
   |                 |                 |                 | Research        |
   |                 |                 |                 | Weather         |
   |                 |                 |                 | Reasearch and   |
   |                 |                 |                 | Forecasting     |
   |                 |                 |                 | model (WRF-ARW) |
   |                 |                 |                 | :raw-latex:`\ci |
   |                 |                 |                 | tep{wrf351}`.   |
   +-----------------+-----------------+-----------------+-----------------+
   | Ocean           | ROMS 3.5        | Stable          | Regional Ocean  |
   |                 |                 |                 | Modelling       |
   |                 |                 |                 | System, rev 568 |
   |                 |                 |                 | :raw-latex:`\ci |
   |                 |                 |                 | tep{shchepetkin |
   |                 |                 |                 | 05}`.           |
   +-----------------+-----------------+-----------------+-----------------+
   |                 | Data/Slab       | Stable          | Data/slab ocean |
   |                 |                 |                 | model, as       |
   |                 |                 |                 | provided with   |
   |                 |                 |                 | CESM 1.1        |
   +-----------------+-----------------+-----------------+-----------------+
   |                 | xROMS           | Stable          | New regional    |
   |                 |                 |                 | component,      |
   |                 |                 |                 | featuring ROMS  |
   |                 |                 |                 | nested in data  |
   |                 |                 |                 | ocean           |
   +-----------------+-----------------+-----------------+-----------------+
   | Land            | CLM 4.0         | Stable          | Community Land  |
   |                 |                 |                 | Model, version  |
   |                 |                 |                 | 4.0             |
   |                 |                 |                 | :raw-latex:`\ci |
   |                 |                 |                 | tep{clm4}`      |
   +-----------------+-----------------+-----------------+-----------------+
   |                 | WRF built-in    | Stable          | WRF-provided    |
   |                 |                 |                 | land models:    |
   |                 |                 |                 | Unified Noah,   |
   |                 |                 |                 | Noah-MP, CLM 4  |
   +-----------------+-----------------+-----------------+-----------------+
   | Sea Ice         | CICE 4.0 beta   | Experim.        | Los Alamos Sea  |
   |                 |                 |                 | Ice Model,      |
   |                 |                 |                 | version 4.0     |
   |                 |                 |                 | (beta)          |
   |                 |                 |                 | :raw-latex:`\ci |
   |                 |                 |                 | tep{cice4}`     |
   +-----------------+-----------------+-----------------+-----------------+
   | Land Ice        | stub            | Inactive        | Unavailable.    |
   |                 |                 |                 | Code for CISM   |
   |                 |                 |                 | included but    |
   |                 |                 |                 | inactive.       |
   +-----------------+-----------------+-----------------+-----------------+
   | Runoff          | stub            | Inactive        | Unavailable.    |
   |                 |                 |                 | Experimental    |
   |                 |                 |                 | code for data,  |
   |                 |                 |                 | MOSART, RTM,    |
   |                 |                 |                 | and RVIC        |
   |                 |                 |                 | included but    |
   |                 |                 |                 | inactive.       |
   +-----------------+-----------------+-----------------+-----------------+


--------------------------------
 R-CESM Configurations
--------------------------------



The current R-CESM system was developed around a coupled Gulf of Mexico configuration to try and repeat the results of the TAMU version of the model as closely as possible. During development of the model, several different component configurations and grid resolutions were tested. This document gives a brief description of these component configurations (compsets) and grids, as well as the machines that are generally supported by the CESM/CIME software used in the R-CESM.




R-CESM Components
=================

Available compsets
------------------
During the course of a CESM/R-CESM run, the model components integrate forward
in time, periodically exchanging information with the coupler.
The coupler meanwhile receives fields from the component models,
computes, maps, and merges this information, then sends the fields back
to the component models. The coupler brokers this sequence of
communication interchanges and manages the overall time progression of
the coupled system. A R-CESM component set is comprised of eight
components: one component from each model (atm, lnd, rof, ocn, ice, glc,
wav, and esp) plus the coupler. 

The active (dynamical) components available in R-CESM include the WRF atmosphere, ROMS ocean and CLM land model. Because
the active models are relatively expensive to run, data models that
cycle input data are included for testing and spin-up. Stub components
exist only to satisfy interface requirements when the component is not
needed for the model configuration (e.g., the active ocean component
forced with atmospheric data does not need land or glc components,
so lnd and glc stubs are used).


Compsets in CESM/R-CESM describe which components are active and their basic configurations for the run. R-CESM currently supports four different configurations of WRF, ROMS, CLM and various data or stub components.

 ================  ==================================
  COMPSET                     Components
 ----------------- ----------------------------------
      sf            atm      ocn    lnd     ice
 ================  ===== ========= ======== =========
 
  PKWUS2003         WRF    data     CLM4.0    data
  PRSGULF2010       data   ROMS      stub     stub
  PRDXGULF2010      data   ROMS      stub     stub
  PBSGULF2010       WRF    ROMS*    CLM4.0    data
  
 ================  ===== ========= ======== =========


* indicates that the ROMS ocean component has been extended via XROMS

- Note that the compsets describe the active components used in an experiment, and also the start date and forcing data, but not the domain or grid size. Thus, the PKWUS2003 compset can be used for the Gulf of Mexico case, if the start date is changed before runtime with the command

.. code-block:: console

    ./xmlchange RUN_STARTDATE=2010-01-01


Creating a new component set
----------------------------
To create a new configuration, you will need to create a new compset in the ``config_compsets.xml`` file in the ``cime_config`` subdirectory of your desired active component. For example, a new compset with a WRF atmosphere should go in the ``my_R-CESM_sandbox/components/wrf/cime_config/config_compsets.xml`` file. There is documentation and examples on how to specify compsets in each of the ``config_compset.xml`` files in R-CESM.

.. note::
   
   If you want to add an entirely new component model to R-CESM, please read the CIME documentation on this process at:
   https://esmci.github.io/cime/build_cpl/adding-components.html


R-CESM Grids
==============

Available resolutions/grids
---------------------------

R-CESM code has been tested to support three different grid/domain sets.

 =================  ========================
   Resolution          Description
 =================  ========================
  wus12_wus12         A 12km Western US domain. Ocean, land, and atmosphere all on the same grid. Has not been tested with ROMS.
  3x3_gulfmexico      A 3km Gulf of Mexico domain for ROMS only (not extended). Data atmosphere on the same grid.
  tx9k_g3x            A 9km atmosphere grid and 3km ocean grid (extended for XROMS) in the Gulf of Mexico (as used for the coupled simulation test case).
 =================  ========================

Once the model resolution is set, components will read in appropriate
grid files and the coupler will read in appropriate mapping weights
files. Coupler mapping weights are always generated externally in
CESM/R-CESM. The components will send the grid data to the coupler at
initialization, and the coupler will check that the component grids
are consistent with each other and with the mapping weights files.

In CESM and R-CESM, the ocean and ice must be on the same grid, but the
atmosphere, land, river runoff and land ice can each be on different grids.
Each component determines its own unique grid decomposition based upon
the total number of pes or processing elements assigned to that component.




--------------------------------
 R-CESM developmental changes
--------------------------------



Code additions for CESM flux to WRF coupling
---------------------------------------------
In order to use the CESM flux coupler with WRF, the flux coupler had to be adapted to compute and/or output several variables needed by the WRF boundary layer scheme. The following variables were needed: 
    • Bulk Richardson number BR, defined above,for ocean-atmosphere fluxes. In the land model we use the equivalent expression: 
Since the m terms etc are non-dimensional vertical gradients, multipling by  makes it dimensional. 
    • Roughness length z0, referred to in WRF as ZNT. Here it is derived using (XX). Over land it is got from routines BareGroundFluxesMod.F90, UrbanMod.F90, BiogeophysicsLakeMod.F90, CanopyFluxesMod.F90
    • Stability parameter =z/L, referred to as HOL in the CESM flux and ZOL in WRF.
    • 10m actual wind velocity, U10, V10. This is not a neutral quantity. Nor is it a relative wind. It is derived using 

where Wz, is wind speed at height z, zbot is the level of the model state variables (lowest model level) and CD is drag coefficient at that same level and stability. In the land model it is obtained from FrictionVelocityMod.F90
    • Integrated similarity functions m() h() etc, referred to as psimh, psixh in CESM air-sea flux code and PSIM, PSIH in WRF. In the land model these are got from FrictionVelocityMod.F90
    • Similarity functions m(), h() referred to in WRF code as FM, FH. For momentum, 

where roughness length z0 has been derived as above and m() has already been calculated. For heat and moisture, we use

The roughness length for heat, z uses equation (12) of Large and Yeager (2009), e.g. for unstable conditions an empirical relationship

so that  giving z~4.9e-5 m.   Similar relationships for the stable roughness length for heat, and the roughness length for moisture are given in Large and Yeager (2009).
    • Stability regime, which is dependent on BR (see above)




R-CESM Component Details
==============================




------
CIME
------

R-CESM is built upon the CESM2/CIME infrastructure. This infrastructure includes the support scripts (configure, build, run, test), a coupler component, data models, essential utility external libraries and other tools required to build a single-executable ESM. CIME uses a flexible hub-and-spoke inter-component coupling architecture with CIME’s coupler component at the hub to which other component models are connected (Figure 1 of Danabasoglu et al. 2020). 


The CIME coupler functionality includes computation of air-sea surface fluxes. CIME also includes tools for generating user-defined mapping weight files that enable regridding between various resolutions in different component models. As such, R-CESM model domain geographic coverage and resolution settings are very flexible, potentially ranging from hundreds of meters to tens of kilometers at any geographic locations depending on user’s research focuses. In addition, R-CESM adds the functionality of using active regional models within the CIME’s global data models and allows coupling between regional and global component models. Some applications include: i) the Community Atmosphere Model version 6 (CAM6) can be coupled to ROMS in a user-specified region while data ocean model provides observed SST elsewhere, ii) the Parallel Ocean Program version 2 (POP2) can be coupled to WRF in a regional domain while data atmosphere model provides observation-based atmospheric surface variables elsewhere of the globe, or iii) using a slab ocean component model coupled to WRF in a data atmosphere. This broad versatility further distinguishes R-CESM from other existing regional ESMs.

CIME developmental info
------------------------

R-CESM uses a modified version of the CIME code (tag:cime_cesm2_1_rel_06, Nov, 11, 2018) to support regional grids. The modified code is available publicly on the `repository <https://github.com/ihesp/cime>`_. The Master branch points to a fork of the main CIME repository with changes to support our regional grids. This is not kept up-to-date with the current CIME development for several reasons. The implications for updates are discussed in section 5 below. 

These data and stub components are all included in the CIME framework. All data and their components are included in this package, as well as the coupling code (Cpl 7, with MCT support). 

The CIME changes for this model are all contained within a handful of xml configuration files. There were no changes to CIME source code, the coupler source code, or any data or stub models to support the R-CESM. The changes to CIME include adding WRF and ROMS as components with relative paths to their cime-supporting scripts in cime/config/cesm/config_files.xml. WRF, ROMS, and fully coupled grids were added to the cime/config/cesm/config_grids.xml file. Finally, the clusters Ada, Terra and Stampede2 were added to cime/config/cesm/machines/config_batch.xml, config_compilers.xml, and config_machines.xml.


  The CIME infrastructure code used in this repository is kept on a fork as an external repository at https://github.com/Katetc/cime . As discussed in section 5, below, this should change in the future. But for now, all updates to CIME needed for this model are in the master branch of my CIME fork. The CIME changes for this model are all contained within a handful of xml configuration files. There were no changes to CIME source code, the coupler source code, or any data or stub models to support the R-CESM. The changes to CIME include adding WRF and ROMS as components with relative paths to their cime-supporting scripts in cime/config/cesm/config_files.xml. WRF, ROMS, and fully coupled grids were added to the cime/config/cesm/config_grids.xml file. Finally, the clusters Ada, Terra and Stampede2 were added to cime/config/cesm/machines/config_batch.xml, config_compilers.xml, and config_machines.xml.


------
ROMS
------

ROMS (Haidvogel et al. 2008; Shchepetkin and McWilliams 2005) is a primitive equation, hydrostatic, free-surface, split-explicit ocean model with horizontal curvilinear coordinates and terrain-following “z-sigma” vertical coordinates (Lemarié et al. 2012; Shchepetkin and McWilliams 2009). The ROMS component of R-CESM uses identical settings in all configurations: harmonic horizontal mixing of momentum and tracers, the K-profile Parameterization (KPP) scheme (Large et al. 1994) for vertical mixing, the 4th-order Akima horizontal and vertical tracer advection (Shchepetkin and McWilliams 2005), and 50 layers in the vertical. ROMS receives surface fluxes from CIME. Open boundary conditions are configured with radiation and nudging schemes for the three-dimensional velocity and tracers, Chapman (1985) scheme for the free-surface, and Flather (1976) scheme for the two-dimensional velocities. 

The ROMS domain in R-CESM is designed to be slightly smaller than that of WRF (Figs. 1 c-d) so that the WRF boundary artifacts (due to a dynamic inconsistency between WRF’s interior solution and prescribed lateral boundary conditions) do not affect the actively coupled ROMS region. To provide SST for the gap-region between ROMS and WRF domains, a new X-ROMS (extended-ROMS) horizontal-grid, which covers the WRF domain with matching grid points with ROMS grid over the overlapping region, has been used. The X-ROMS blends SST from ROMS with that provided from data [on X-ROMS grid, from the dataset used for ROMS initial and boundary conditions] before passing it to the coupler. 



ROMS developmental info
------------------------

 For the active ocean component, R-CESM uses a modified version of the code from the `Regional Ocean Modeling System (ROMS) <https://www.myroms.org/>`_ v3.5 (rev 568, Sep 21, 2011). The modified code is available publicly on the `roms_ihesp repository <https://github.com/ihesp/roms_ihesp>`_, and the appropriate tag is pulled from this repository during ``checkout_externals``. 


    Changes made to ROMS as it was given to NCAR from TAMU (which includes changes made by scientists at TAMU) center on four main areas of the code: the ROMS xml configuration files, the build, the namelist, and the source. All of the changes described here apply to the code on the Master branch of the TAMURegionalCESM github repository (https://github.com/NCAR/TAMURegionalCESM) as of today, January 16, 2019. The descriptions below do not include changes currently being worked on in other branches by other NCAR or TAMU software engineers and scientists.

ROMS xml configuration files


Just as in WRF, xml files needed to be added to a cime_config directory for the ROMS component to be supported correctly within CESM. These files include config_component.xml, config_compsets.xml, and config_pes.xml. None of these files were included in the TAMU version of ROMS that was distributed to NCAR, and so they were all added in this project.

ROMS build changes


    As discussed in the WRF section above, the updated version of CIME uses python to build all of the components and supports multiple instances and multiple different debug builds. To support all of these features, a few changes were made to the ROMS build files from what they were distributed to us. First, the python script “buildlib” was added to the cime_config directory. This script pulls the needed information for a ROMS build from the CESM case, and passes it to the roms.buildexe.csh script that is used to actually build the model. The roms.buildexe.csh script was changed to include an argument list to get information about the build from the python script, and then to set the appropriate environment variables based on those arguments.
    ROMS header and configuration files for building a roms-only case (not extended or XROMS), were added to the roms source directory for “Apps”, which can be seen here

    https://github.com/NCAR/TAMURegionalCESM/tree/master/components/roms/Apps . 

    A few changes were made to the roms.buildexe.csh script to get the flags correct for the CESM and Cheyenne build environment. Only one line was changed from the original makefile from TAMU and that is to add “-debug minimal” for all ROMS builds. ROMS in fully optimized model would crash on Cheyenne as some if-statements were unrolled by the compiler in a very bad way.

ROMS namelist changes


    As in the build library section, the updated CIME uses python scripts to build the component namelists in the most recent CESM. So the roms/cime_config directory had a python script added called “buildnml” that pulls the necessary information to build a namelist from the case, and passes that to the original roms.buildnml.csh script. The roms.buildnml.csh script was changed to support this information passed as arguments, to support a “gom” grid that is the internal non-extended Gulf of Mexico grid, and to copy supporting files from the CESM input data directories. Changes were made to the ROMS name list scripts by software engineers at TAMU to better support multiple instances of ROMS running at one time. And changes were made to the namelist and ROMS header scripts to better handle start dates and times so that restarting the model would work correctly.

ROMS Source changes


    Very little was changed within the actual ROMS source from when it was shared with NCAR by TAMU. The interfaces for rocn_init_mct, rocn_run_mct, and rocn_final_mct all needed to be updated in the rocn_comp_mct.F90 file to support a small change in the coupler. Indexes for river runoff fields were removed from ocn_cplindices.F90 as this model does not include a river model (and the WRF land/rivers were no longer being used). An updated version of docn_comp_mod.F90 was used for the extended ROMS ocean grid points (pulled from the same CIME release as is currently used in the R-CESM code). And the mct header files were duplicated to support both the extended XROMS ocean in rxocn_comp_mct.F90 and a simple non-extended roms grid (for use with a data atmosphere) in the rocn_comp_mct.F90 file. A simple if-statement in ocn_comp_mct.F90 supports these two modes as well, but currently the XROMS mode is hard-coded via an if-def in this file. A few changes were made to the rxocn_comp_mct.F90 module by TAMU software engineers to support multiple instances of ROMS as well. Finally, a few debug statements were added to the ROMS source, but have since all been removed.  


------
WRF
------

The Advanced Research version of the WRF model (Skamarock et al. 2008) is a fully compressible, Eulerian, nonhydrostatic atmospheric model with a terrain-following vertical coordinate. In this initial release of R-CESM, the incorporated WRF version is 3.5.1. Besides the inherited flexibility from original WRF (only three surface layer schemes are available in the first release), WRF in the R-CESM configuration is also capable of receiving air-sea fluxes from CIME and bypass the WRF built-in surface layer subroutines, which is one of the unique features in our open-source system. In this work, all the simulations are conducted with the Yonsei University (YSU) planetary boundary layer scheme (Hong et al. 2006), Purdue-Lin microphysics scheme (Lin et al. 1983), Rapid Radiative Transfer Model for GCMs (RRTMG) short wave and long wave radiation schemes (Iacono et al. 2008). No cumulus parameterization is utilized in any set of experiments.


WRF developmental info
------------------------

The active atmospheric model used by R-CESM is a modified version of NCAR's `Weather Research and Forecasting Model (WRF) <https://www.mmm.ucar.edu/weather-research-and-forecasting-model>`_ v3.5.1 (rev 6868, Sep 23, 2013). The iHESP team modified parts of the WRF code (See next section) to work alongside the R-CESM driver. The modified code is available publicly on the `wrf_ihesp repository <https://github.com/ihesp/wrf_ihesp>`_, and the appropriate tag is pulled from this repository during ``checkout_externals``.

    While the WRF atmospheric model has been integrated into the CESM at least one time before this project, it is not the typical atmosphere used in the fully coupled CESM. All usages of WRF within CESM should be considered extremely experimental.


    Changes made to WRF as it was given to NCAR from TAMU (which includes changes made by scientists at TAMU) center on four main areas of the code: the WRF xml configuration files, the build, the namelist, and the source. All of the changes described here apply to the code on the Master branch of the TAMURegionalCESM github repository (https://github.com/NCAR/TAMURegionalCESM) as of today, January 16, 2019. The descriptions below do not include changes currently being worked on in other branches by other NCAR or TAMU software engineers and scientists.

WRF xml configuration files 


    In order to work with CIME, components in the newest versions of CESM require a sub-directory called “cime_config” in the main component directory, with files such as config_component.xml, config_compsets.xml, and config_pes.xml. These files tell CIME what area the component belongs to (atm, ocn, etc) and the name of the component, the compsets the component supports, and pe layouts that work with this component and compsets. None of these files existed in the older version of the coupling framework from TAMU, and so they were added to our repository. The files and their contents can be viewed here: https://github.com/NCAR/TAMURegionalCESM/tree/master/components/wrf/cime_config

WRF build changes


    The updated version of CIME uses python to build all of the components and supports multiple instances and multiple different debug builds. To support all of these features, a few changes were made to the WRF build files from what they were distributed to us. First, the python script “buildlib” was added to the cime_config directory. This script pulls the needed information for a WRF build from the CESM case, and passes it to the wrf.buildexe.csh script that is used to actually build the model. The wrf.buildexe.csh script was changed to include an argument list to get information about the build from the python script, and then to set the appropriate environment variables based on those arguments.
    WRF uses a configuration script to support the makefile build on each machine. The configure.wrf.cheyenne-intel file is used when our model is built on Cheyenne. This script is very similar to the configure.wrf.intel script that was distributed with the TAMU code. There is some added logic to build the necessary paths for CIME to support multiple instances and debug builds. Some flags needed to be changed with the C Pre Processor (cpp) which is different on Cheyenne than Yellowstone. These CPP flag changes were propagated throughout the WRF build in Makefiles in most subdirectories.  Flags to include the CESM coupler MCT software and shared files were added as well. To keep the WRF build a little more simple, we changed the directory architecture to a more flat structure like that of ROMS as well. The files themselves did not change, in most cases.

WRF namelist changes


    As in the build library section, the updated CIME uses python scripts to build the component namelists in the most recent CESM. So the wrf/cime_config directory had a python script added called “buildnml” that pulls the necessary information to build a namelist from the case, and passes that to the old wrf.buildnml.csh script. The wrf.buildnml.csh script was changed to support this information passed as arguments, to remove nesting from some example namelists, to update and improve the start date/time and run length logic (necessary for restarts), and to make the final wrf namelist as close to that of the TAMU Gulf of Mexico example case as possible, when that grid is used.
    The needed boundary data and forcing files have all been moved into the CESM inputdata directory. The wrf.buildnml.csh script copies or links these files into the case run directory, and we updated this script to pull the files from the inputdata location.
    Changes were made by TAMU software engineers to the buildnml script in this repository to support multiple drivers, and better support running multiple WRF instances as well. 

WRF Source changes


    All of the source changes made in WRF were done to the MCT cap files. These files in wrf/drivers/ needed first to have their extension changed from “.F90” to “.F” to work with the new build system. Updates to the atm_init_mct function arguments were required to match the newest interface in CIME (changing some arguments from intend(in) to intent(inout)). The logz0 surface roughness length was no longer passed through the coupler, so the code supporting this is commented out in atm_comp_mct.F and atm_cpl_indices.F. The length of two message strings in wrf/share/mediation_wrfmain.F needed to change from 80 characters to 250 characters to support longer CESM restart file names.


------
CLM4
------

CLM4 is the standard land component in the R-CESM system. There are several key differences between native CLM4 and WRF built-in CLM4 package. As of WRF3.5.1, lake and urban canopy models, including CLM4 urban parameterization (CLMU; Oleson et al. 2010), are not available in WRF built-in CLM4, whereas they are actively working in R-CESM. This is of critical importance to assess the regional climate, as high population density makes urban climate significantly different from other land surface area and may have discernible impacts on the regional climate (Zhao et al. 2021). Secondly, the number of vertical soil levels in WRF CLM4 is 10 but it is 15 in R-CESM CLM4. Thirdly, WRF CLM4 is “cold” started by direct interpolation of soil data from atmospheric reanalysis products (typically 4 vertical layers) using the WRF preprocessing system, while R-CESM CLM4 is first integrated for 10 years using the surface atmospheric forcing from reanalysis (CESM data atmosphere) products as spin-up to “warm” start.


CLM4 developmental info
------------------------

  The active model used is the `Community Land Model (CLM) <http://www.cesm.ucar.edu/models/clm/>`_, v4.0 (tag:clm4_5_14_r213). CLM is the typical land model in the coupled CESM system. This is a global land surface model that works in a regional mode when given a regional grid.  In the original, TAMU-built R-CESM 1.0.0, the internal WRF land surface model was used. In our system, the CESM land surface from CLM is passed through the coupler to WRF. This has several implications discussed further in sections 4 and 5. 


The land model in the R-CESM is only used when the model is fully coupled or in WRF-atmosphere mode (data ocean). We used CLM version 4.0 because that version was supported in the earlier WRF integration into CESM 1.2 and surface data set files for a western US case were available for testing. Most of the changes to CLM were in namelist_defaults_clm4_0.xml, because as new grids for WRF were added, the CLM namelist generating scripts needed to be updated with those grids. The I-compset (CLM only) was updated in this model to work with stub ice and stub ocean. Some debug output was added to two or three files to help track down an error with extremely high LWUP in CLM. These, apparently, did not get removed, but will only trigger when LWUP > 6000 W/m2. 

    Other changes include the tolerance for surface data mis-match increased to 0.5 degrees lat/lon in surfrdMod.F90 to help get some early tests running. Function names for mct_gsMap calls had to be changed when a newer version of CIME was added to the repo in March 2018. And the automatically generated files for buildcppc and buildnmlc were accidently added to the repository a few times. These could be removed.


  


