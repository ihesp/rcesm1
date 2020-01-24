.. _rcesm_arch:

===============================
RCESM developmental information
===============================


Software Versions and Sourcing
==============================

CESM
----

The current version of RCESM is based CESM v2.1 (2019). 


Common Infrastructure for Modeling the Earth (CIME) 
---------------------------------------------------

	The Common Infrastructure for Modeling the Earth (CIME) provides the top level driver . RCESM uses a modified version of the CIME code (tag:cime_cesm2_1_rel_06, Nov, 11, 2018) to support regional grids. The modified code is available publicly on the `repository <https://github.com/ihesp/cime>`_. The Master branch points to a fork of the main CIME repository with changes to support our regional grids. This is not kept up-to-date with the current CIME development for several reasons. The implications for updates are discussed in section 5 below. 

	These data and stub components are all included in the CIME framework. All data and their components are included in this package, as well as the coupling code (Cpl 7, with MCT support). 

Atmospheric Component
---------------------

	The active atmospheric model used by RCESM is a modified version of NCAR's `Weather Research and Forecasting Model (WRF) <https://www.mmm.ucar.edu/weather-research-and-forecasting-model>`_ v3.5.1 (rev 6868, Sep 23, 2013). The iHESP team modified parts of the WRF code (See next section) to work alongside the RCESM driver. The modified code is available publicly on the `wrf_ihesp repository <https://github.com/ihesp/wrf_ihesp>`_, and the appropriate tag is pulled from this repository during ``checkout_externals``.

	While the WRF atmospheric model has been integrated into the CESM at least one time before this project, it is not the typical atmosphere used in the fully coupled CESM. All usages of WRF within CESM should be considered extremely experimental.
    
Ocean Component
---------------

    For the active ocean component, RCESM uses a modified version of the code from the `Regional Ocean Modeling System (ROMS) <https://www.myroms.org/>`_ v3.5 (rev 568, Sep 21, 2011). The modified code is available publicly on the `roms_ihesp repository <https://github.com/ihesp/roms_ihesp>`_, and the appropriate tag is pulled from this repository during ``checkout_externals``. 

Land Component
--------------

    The active model used is the `Community Land Model (CLM) <http://www.cesm.ucar.edu/models/clm/>`_, v4.0 (tag:clm4_5_14_r213). CLM is the typical land model in the coupled CESM system. This is a global land surface model that works in a regional mode when given a regional grid.  In the original, TAMU-built RCESM 1.0.0, the internal WRF land surface model was used. In our system, the CESM land surface from CLM is passed through the coupler to WRF. This has several implications discussed further in sections 4 and 5. 





Development Changes Made by NCAR
================================

WRF
---

    Changes made to WRF as it was given to NCAR from TAMU (which includes changes made by scientists at TAMU) center on four main areas of the code: the WRF xml configuration files, the build, the namelist, and the source. All of the changes described here apply to the code on the Master branch of the TAMURegionalCESM github repository (https://github.com/NCAR/TAMURegionalCESM) as of today, January 16, 2019. The descriptions below do not include changes currently being worked on in other branches by other NCAR or TAMU software engineers and scientists.

WRF xml configuration files 
===========================

    In order to work with CIME, components in the newest versions of CESM require a sub-directory called “cime_config” in the main component directory, with files such as config_component.xml, config_compsets.xml, and config_pes.xml. These files tell CIME what area the component belongs to (atm, ocn, etc) and the name of the component, the compsets the component supports, and pe layouts that work with this component and compsets. None of these files existed in the older version of the coupling framework from TAMU, and so they were added to our repository. The files and their contents can be viewed here: https://github.com/NCAR/TAMURegionalCESM/tree/master/components/wrf/cime_config

WRF build changes
=================

    The updated version of CIME uses python to build all of the components and supports multiple instances and multiple different debug builds. To support all of these features, a few changes were made to the WRF build files from what they were distributed to us. First, the python script “buildlib” was added to the cime_config directory. This script pulls the needed information for a WRF build from the CESM case, and passes it to the wrf.buildexe.csh script that is used to actually build the model. The wrf.buildexe.csh script was changed to include an argument list to get information about the build from the python script, and then to set the appropriate environment variables based on those arguments.
    WRF uses a configuration script to support the makefile build on each machine. The configure.wrf.cheyenne-intel file is used when our model is built on Cheyenne. This script is very similar to the configure.wrf.intel script that was distributed with the TAMU code. There is some added logic to build the necessary paths for CIME to support multiple instances and debug builds. Some flags needed to be changed with the C Pre Processor (cpp) which is different on Cheyenne than Yellowstone. These CPP flag changes were propagated throughout the WRF build in Makefiles in most subdirectories.  Flags to include the CESM coupler MCT software and shared files were added as well. To keep the WRF build a little more simple, we changed the directory architecture to a more flat structure like that of ROMS as well. The files themselves did not change, in most cases.

WRF namelist changes
====================

    As in the build library section, the updated CIME uses python scripts to build the component namelists in the most recent CESM. So the wrf/cime_config directory had a python script added called “buildnml” that pulls the necessary information to build a namelist from the case, and passes that to the old wrf.buildnml.csh script. The wrf.buildnml.csh script was changed to support this information passed as arguments, to remove nesting from some example namelists, to update and improve the start date/time and run length logic (necessary for restarts), and to make the final wrf namelist as close to that of the TAMU Gulf of Mexico example case as possible, when that grid is used.
    The needed boundary data and forcing files have all been moved into the CESM inputdata directory. The wrf.buildnml.csh script copies or links these files into the case run directory, and we updated this script to pull the files from the inputdata location.
    Changes were made by TAMU software engineers to the buildnml script in this repository to support multiple drivers, and better support running multiple WRF instances as well. 

WRF Source changes
==================

    All of the source changes made in WRF were done to the MCT cap files. These files in wrf/drivers/ needed first to have their extension changed from “.F90” to “.F” to work with the new build system. Updates to the atm_init_mct function arguments were required to match the newest interface in CIME (changing some arguments from intend(in) to intent(inout)). The logz0 surface roughness length was no longer passed through the coupler, so the code supporting this is commented out in atm_comp_mct.F and atm_cpl_indices.F. The length of two message strings in wrf/share/mediation_wrfmain.F needed to change from 80 characters to 250 characters to support longer CESM restart file names.

ROMS
----

    Changes made to ROMS as it was given to NCAR from TAMU (which includes changes made by scientists at TAMU) center on four main areas of the code: the ROMS xml configuration files, the build, the namelist, and the source. All of the changes described here apply to the code on the Master branch of the TAMURegionalCESM github repository (https://github.com/NCAR/TAMURegionalCESM) as of today, January 16, 2019. The descriptions below do not include changes currently being worked on in other branches by other NCAR or TAMU software engineers and scientists.

ROMS xml configuration files
============================

Just as in WRF, xml files needed to be added to a cime_config directory for the ROMS component to be supported correctly within CESM. These files include config_component.xml, config_compsets.xml, and config_pes.xml. None of these files were included in the TAMU version of ROMS that was distributed to NCAR, and so they were all added in this project.

ROMS build changes
==================

    As discussed in the WRF section above, the updated version of CIME uses python to build all of the components and supports multiple instances and multiple different debug builds. To support all of these features, a few changes were made to the ROMS build files from what they were distributed to us. First, the python script “buildlib” was added to the cime_config directory. This script pulls the needed information for a ROMS build from the CESM case, and passes it to the roms.buildexe.csh script that is used to actually build the model. The roms.buildexe.csh script was changed to include an argument list to get information about the build from the python script, and then to set the appropriate environment variables based on those arguments.
    ROMS header and configuration files for building a roms-only case (not extended or XROMS), were added to the roms source directory for “Apps”, which can be seen here

    https://github.com/NCAR/TAMURegionalCESM/tree/master/components/roms/Apps . 

    A few changes were made to the roms.buildexe.csh script to get the flags correct for the CESM and Cheyenne build environment. Only one line was changed from the original makefile from TAMU and that is to add “-debug minimal” for all ROMS builds. ROMS in fully optimized model would crash on Cheyenne as some if-statements were unrolled by the compiler in a very bad way.

ROMS namelist changes
=====================

    As in the build library section, the updated CIME uses python scripts to build the component namelists in the most recent CESM. So the roms/cime_config directory had a python script added called “buildnml” that pulls the necessary information to build a namelist from the case, and passes that to the original roms.buildnml.csh script. The roms.buildnml.csh script was changed to support this information passed as arguments, to support a “gom” grid that is the internal non-extended Gulf of Mexico grid, and to copy supporting files from the CESM input data directories. Changes were made to the ROMS name list scripts by software engineers at TAMU to better support multiple instances of ROMS running at one time. And changes were made to the namelist and ROMS header scripts to better handle start dates and times so that restarting the model would work correctly.

ROMS Source changes
===================

    Very little was changed within the actual ROMS source from when it was shared with NCAR by TAMU. The interfaces for rocn_init_mct, rocn_run_mct, and rocn_final_mct all needed to be updated in the rocn_comp_mct.F90 file to support a small change in the coupler. Indexes for river runoff fields were removed from ocn_cplindices.F90 as this model does not include a river model (and the WRF land/rivers were no longer being used). An updated version of docn_comp_mod.F90 was used for the extended ROMS ocean grid points (pulled from the same CIME release as is currently used in the RCESM code). And the mct header files were duplicated to support both the extended XROMS ocean in rxocn_comp_mct.F90 and a simple non-extended roms grid (for use with a data atmosphere) in the rocn_comp_mct.F90 file. A simple if-statement in ocn_comp_mct.F90 supports these two modes as well, but currently the XROMS mode is hard-coded via an if-def in this file. A few changes were made to the rxocn_comp_mct.F90 module by TAMU software engineers to support multiple instances of ROMS as well. Finally, a few debug statements were added to the ROMS source, but have since all been removed.  

CLM
---

    The land model in the RCESM is only used when the model is fully coupled or in WRF-atmosphere mode (data ocean). We used CLM version 4.0 because that version was supported in the earlier WRF integration into CESM 1.2 and surface data set files for a western US case were available for testing. Most of the changes to CLM were in namelist_defaults_clm4_0.xml, because as new grids for WRF were added, the CLM namelist generating scripts needed to be updated with those grids. The I-compset (CLM only) was updated in this model to work with stub ice and stub ocean. Some debug output was added to two or three files to help track down an error with extremely high LWUP in CLM. These, apparently, did not get removed, but will only trigger when LWUP > 6000 W/m2. 

    Other changes include the tolerance for surface data mis-match increased to 0.5 degrees lat/lon in surfrdMod.F90 to help get some early tests running. Function names for mct_gsMap calls had to be changed when a newer version of CIME was added to the repo in March 2018. And the automatically generated files for buildcppc and buildnmlc were accidently added to the repository a few times. These could be removed.

CIME/coupler
------------

    The CIME infrastructure code used in this repository is kept on a fork as an external repository at https://github.com/Katetc/cime . As discussed in section 5, below, this should change in the future. But for now, all updates to CIME needed for this model are in the master branch of my CIME fork. The CIME changes for this model are all contained within a handful of xml configuration files. There were no changes to CIME source code, the coupler source code, or any data or stub models to support the RCESM. The changes to CIME include adding WRF and ROMS as components with relative paths to their cime-supporting scripts in cime/config/cesm/config_files.xml. WRF, ROMS, and fully coupled grids were added to the cime/config/cesm/config_grids.xml file. Finally, the clusters Ada, Terra and Stampede2 were added to cime/config/cesm/machines/config_batch.xml, config_compilers.xml, and config_machines.xml.

Note on Git Histories
---------------------

    All of the notes here about changes to the various parts of the RCESM code came from histories generated in GitHub. To see a history of every change made to the RCESM code since the creation of repository, simply go to the main repository (https://github.com/NCAR/TAMURegionalCESM) and click on the “XX Commits” (Some number of commits) link, just above the branch pull-down menu. You can also look at certain directories or files in GitHub, and click the “History” button (top left) to see every commit made that impacts those files. When looking at a specific file, you can click the “Blame” button (top left) to see which commits resulted in each line of code within the file.