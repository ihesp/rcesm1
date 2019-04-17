.. _software_vers:

==============================
Software Versions and Sourcing
==============================

Atmospheric Component
---------------------

    The regional atmospheric model used in our system is the Weather Research and Forecasting Model (WRF) version 3.5.1 as distributed from Texas A&M university as part of the RCESM 1.0.0 from October 2017. The WRF code is very similar to that initially distributed, with all changes described in the next section. While the WRF atmospheric model has been integrated into the CESM at least one time before this project, it is not the typical atmosphere used in the fully coupled CESM. All usages of WRF within CESM should be considered extremely experimental.
    
Ocean Component
---------------

    The regional ocean model used is the Regional Ocean Modeling System (ROMS) with version 3.5 (release version 568, dated 2011-09-21). This code can be found on the ROMS website at: https://www.myroms.org/ . The ROMS code is almost entirely the same as that used in RCESM 1.0.0 and distributed by Texas A&M university in October 2017. Any changes are described in the next section. 

Land Component
--------------

    The land model used is the Community Land Model (CLM), version 4.0. The version of CLM in our repository is from the CLM trunk tag clm4_5_14_r213. This is a global land surface model that works in a regional mode when given a regional grid. CLM is the typical land model in the coupled CESM system. In the original, TAMU-built RCESM 1.0.0, the internal WRF land surface model was used. In our system, the CESM land surface from CLM is passed through the coupler to WRF. This has several implications discussed further in sections 4 and 5. 

Other Components
----------------

    The NCAR RCESM provides support for several different configurations. Many of these use stub or data models that are inactive and simply report data to the coupler from pre-written files. For instance, in testing configurations with a WRF atmosphere but no ROMS ocean, data ice and data ocean models are used. In testing configurations with a ROMS ocean but no active atmosphere, a data atmosphere, stub land and stub ice are used. For more information on the supported component configurations (RCESM “compsets”), please see the model usage documentation on the GitHub page: https://github.com/NCAR/TAMURegionalCESM . 
    These data and stub components are all included in the Common Infrastructure for Modeling the Earth (CIME) software framework that supports CESM. All data and their components are included in this package, as well as the coupling code (Cpl 7, with MCT support). While the NCAR RCESM uses a more up-to-date version of the infrastructure than the original TAMU model, two things should be noted. First, this is almost exactly the same code in the coupler itself as was used in the TAMU model. Changes made to support the updated coupler are described in section 4.  Secondly, the code for CIME is kept in an external repository so that it can be publicly distributed and managed. In our repository, the Master branch points to a fork of the main CIME repository with changes to support our regional grids. This is not kept up-to-date with the current CIME development for several reasons. The implications for updates are discussed in section 5 below. 
