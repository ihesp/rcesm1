.. _configurations:

======================
 RCESM Configurations
======================

The current RCESM system was developed around a coupled Gulf of Mexico configuration to try and repeat the results of the TAMU version of the model as closely as possible. Durring development of the model, several different component configurations and grid resolutions were tested. This document gives a brief description of these component configurations (compsets) and grids, as well as the machines that are generally supported by the CESM/CIME software used in the RCESM.


RCESM Components
----------------

Compsets in CESM/RCESM describe which components are active and their basic configurations for the run. In the RCESM, some pertinant compsets are:

 ================  ========================
  COMPSET Name         Components Used
 ================  ========================
  PKWUS2003         WRF atmosphere, CLM 4.0 land, data ice and data ocean
  PRSGULF2010       Data atmosphere, stub land, stub ice and ROMS ocean
  PRDXGULF2010      Data atmosphere, stub land, stub ice and ROMS ocean extended via XROMS
  PBSGULF2010       WRF atmosphere, CLM 4.0 land, stub ice and ROMS ocean extended via XROMS
 ================  ========================

- Note that the compsets describe the active components used in an experiment, and also the start date and forcing data, but not the domain or grid size. Thus, the PKWUS2003 compset can be used for the Gulf of Mexico case, if the start date is changed before runtime with the command ::

    ./xmlchange RUN_STARTDATE=2010-01-01

During the course of a CESM/RCESM run, the model components integrate forward
in time, periodically exchanging information with the coupler.
The coupler meanwhile receives fields from the component models,
computes, maps, and merges this information, then sends the fields back
to the component models. The coupler brokers this sequence of
communication interchanges and manages the overall time progression of
the coupled system. A RCESM component set is comprised of eight
components: one component from each model (atm, lnd, rof, ocn, ice, glc,
wav, and esp) plus the coupler. 

The active (dynamical) components available in RCESM include the WRF atmosphere, ROMS ocean and CLM land model. Because
the active models are relatively expensive to run, data models that
cycle input data are included for testing and spin-up. Stub components
exist only to satisfy interface requirements when the component is not
needed for the model configuration (e.g., the active ocean component
forced with atmospheric data does not need land or glc components,
so lnd and glc stubs are used).

RCESM Grids
-----------

In the RCESM, the currently available resoultions are:

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
CESM/RCESM. The components will send the grid data to the coupler at
initialization, and the coupler will check that the component grids
are consistent with each other and with the mapping weights files.

In CESM and RCESM, the ocean and ice must be on the same grid, but the
atmosphere, land, river runoff and land ice can each be on different grids.
Each component determines its own unique grid decomposition based upon
the total number of pes or processing elements assigned to that component.

Creating a New Domain or Regional Set Up
----------------------------------------

Changing the domain, time, or "application" for the CRESM requires answering a series of questions in order to
determine which files need to be changed or updated to support your new application. Eventually, your goal is
to have a *compset*, a *grid*, and the needed *namelist file* support to create a new experiment case from
which to run your simulation. Consider these questions to direct the changes needed:

1. Do you want to use a set of components that has already been defined and tested or a new set?

   The most basic part of configuring a CESM or RCESM case is deciding which components you want to run with. The RCESM code supports four different configurations of WRF, ROMS, CLM and various data or stub components (See the section above: `RCESM Components`_ ). If you want to use any of these configurations, then you can move on to the next question. If you want a new configuration, you will need to create a new compset in the ``config_compsets.xml`` file in the ``cime_config`` subdirectory of your desired active component. For example, a new compset with a WRF atmosphere should go in the ``my_rcesm_sandbox/components/wrf/cime_config/config_compsets.xml`` file. There is documentation and examples on how to specify compsets in each of the ``config_compset.xml`` files in RCESM.

.. note::
   
   If you want to add an entirely new component model to RCESM, please read the CIME documentation on this process at:
   https://esmci.github.io/cime/build_cpl/adding-components.html

2. Do you want to use a grid resolution and domain that has already been tested or create a new one?

   The RCESM code has been tested and includes support for three different grid/domain sets. These are described in the section above: `RCESM Grids`_ . If one of these supported grids works for your new application, then you can move on to the next question. If you need to add a new domain or grid, there are several files that will need to be updated with grid information, and you will need several coupler domain and mapping files to support it.

   - ``cime/config/cesm/config_grids.xml``  This file needs to be edited in two places. Early in the file, grid names and the
     components to which they apply are specified. Later in the file, the grid domain size and links to the coupler domain file
     are specified. To see what edits need to be made, search the file for the grid "gom3" and consider how this grid was
     added to the ``config_grids.xml`` file. Note that the CIME code base is maintained in a seperate repository from the RCESM,
     so changes to this file will need a seperate commit and pull request from any other changes to the code base.

   - ``components/wrf/bld/wrf.buildnml.csh`` and ``components/wrf/cime_config/config_pes.xml``  If your grid is applicable to the
     WRF atmospheric component, then these files need to be updated to include your grid and domain. The WRF namelist is built
     based on the name of the grid WRF is running with, so new grids will need to specify a new section in the wrf.buildnml.csh
     script. The config_pes.xml file specifies the PE layout that is best for a given grid. There is a default value that should
     work for most components, but adding a section for your new grid will give you flexibility to update this in the future.

   - ``components/clm/bld/namelist_files/namelist_definition_clm4_0.xml`` and ``components/clm/bld/namelist_files/namelist_defaults_clm4_0.xml``  If your grid is applicable to the CLM (land) component, then these files need to be updated. You may need to
     specify a land surface start up file (``fsurdat``) for the grid, or decide to run with a "cold start". For
     more information on creating domain and land surface files see:
     http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x11573.html

     And for more information about CLM 4.0 namelists and start types see:
     http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x1230.html

   - ``components/roms/bld/roms.buildnml.csh`` and ``components/roms/cime_config/config_pes.xml``  Much like in WRF,  if your
     grid is applicable to the
     ROMS ocean component, then these files need to be updated to include your grid and domain. The ROMS namelist is built
     based on the name of the grid ROMS is running with, so new grids will need to specify a new section in the roms.buildnml.csh
     script. The config_pes.xml file specifies the PE layout that is best for a given grid. There is a default value that should
     work for most components, but adding a section for your new grid will give you flexibility to update this in the future.
     Note that it is not common for active atmospheric models to run on the same grid as an active ocean model, so it is very
     likely that your atmospheric grid will be named differently and specified differently in the ``config_grids.xml`` file
     from your ocean grid.

   - ``components/roms/Apps`` The ROMS header files that specify ROMS settings are based on the domain or grid name. You will
     need to create a new sub directory of this directory by the name of the new grid and create the necessary namelist
     and header files within it. Look at the directory ``components/roms/Apps/gom3x/`` as an example.

3. Do you want to change the time, date, or physical/parameterization options of an application?

   Both ROMS and WRF use CIME xml variables and namelist options to specify various parameters for their runs. To change the
   date, length of a run or change parameterizations, you will first need to add support for these changes to the appropriate
   namelist via the namelist generating scripts (``components/wrf/bld/wrf.buildnml.csh`` and/or
   ``components/roms/bld/roms.buildnml.csh``). You may also need to make changes to the ROMS header or namelist files in the
   ``components/roms/Apps`` directory. Once this is all done, you can create an experiment case for your application from which
   you can set up your runs. To learn more about creating a case, changing xml variables, and running the model, see the
   `CRESM Quick Start guide <https://ncar.github.io/TAMURegionalCESM/quickstart.html>`.


Creating New Domain and Land Surface Data Files for CLM
-------------------------------------------------------

In order to build and run a new domain in the RCESM model, currently the CLM component and coupler requires domain files and surface data files for the regional domain. For more information on these files (and a description of how to create them) consider browsing through these websites:
 - `Creating mappping files that mksurfdata_map will use <http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x11775.html>`_
 - `Creating a domain file for CLM and DATM <http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x11812.html>`_
 - `Using mksurfdata_map to create surface datasets from grid datasets <http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x11868.html>`_

These websites are older, a bit out of date, and not exactly what we need for our model. Below are more specific instructions for generating the domain and surface dataset files needed for a CESM run that includes CLM.

Step 1: Generate SCRIP grid mapping files for your domain. This can be done with a python script available at TAMU. Contact Jaison Kurian if needed (jaisonk@tamu.edu).

Step 2: Create the domain files using the gen_domain tool. This tool is available from many places, and it is not entirely necessary to use the one from an older version of CLM. The version available through CIME in a full checkout of RCESM should work. The tool is located in: ``cime/tools/mapping/gen_domain_files`` .

Step 2-a: Build the gen_domain tool using the directions in the INSTALL file that is located in the same directory.

Step 2-b: Run the gen_domain tool using the command:
``$ gen_domain -m <input mapping file name> -o <ocean grid name> -l <land grid name>``
This will produce three domain files, one for the land and atmosphere, one for the ocean and sea ice, and one for the ocean on the atmospheric grid.

Step 3: Create mapping files from the input data sets to the land grid. In order to map the surface data into a file on the land grid, the mapping weights must be generated with the ``mkmapdata.sh`` script. This is the tool that needs to use a version of CLM that matches the one in the repository. 

One way to get this is to checkout an older tag of clm (such as ``svn co https://svn-ccsm-models.cgd.ucar.edu/clm2/trunk_tags/clm4_5_13_r211``) and build the tool on your machine from its location in that repository (``components/clm/tools/shared/mkmapdata``). However, this could require some detailed porting to a new machine, so the easiest way is to use some older versions of this tool that were previously ported from Yellowstone to Cheyenne by the Paleo working group. These are located on Cheyenne at: ``/gpfs/fs1/p/cesm/palwg/cesm1_2_0/models/lnd/clm/tools/shared/mkmapdata/`` . The script ``mkmapdata-tamu-gom3x-02.sh`` was successfully used to create the surface data to 3km wrf grid mapping files on April 8, 2019. 

Step 3-a: Copy the ``mkmapdata-tamu-gom3x-02.sh`` script to a new file and edit it as needed for your new grid and application.

Step 3-b: Make a pbs submission script to submit this job to a compute node on cheyenne. This job takes a while and will need the memory from compute nodes, which is larger than the login nodes. The script should call the mkmapdata.sh script as:
``/gpfs/fs1/p/cesm/palwg/cesm1_2_0/models/lnd/clm/tools/shared/mkmapdata/mkmapdata-tamu-gom3x-02.sh -p clm4_0 -t regional -v -r [your grid name] -f [full path to atm scrip grid file] &> output.txt`` . It will need 1 CPU and approximately 4 hours to complete.

Step 3-c: Submit the pbs script. You can run `tail -f output.txt` to track the script as it runs. When it is finished, it should have created a mapping file from each input data file grid to your new grid.

Step 4: Create the land surface data file using the mksurfdata_map tool. This is another tool that is useful to have the older version of. Again, you can checkout an older version of clm (such as ``svn co https://svn-ccsm-models.cgd.ucar.edu/clm2/trunk_tags/clm4_5_13_r211``). The mksurfdata_map tool is located in ``components/clm/tools/clm4_0/mksurfdata_map`` .

Step 4-a: Follow the instructions in the README file to build the mksurfdata_map tool. This worked on Cheyenne without porting the entire clm tag. 

Step 4-b: Call the mksurfdata.pl script with user generated grid specifiers as:
``$ ./mksurfdata.pl -res usrspec -usr_gname [your grid name] -usr_gdate [date on mapping files] -usr_mapdir [directory containing map files]``
This script should run quickly on a login node, and produce four files. Two log files and two netcdf surface data files for CLM.

RCESM and CESM2 Machines
------------------------

Scripts for `supported machines
<http://www.cesm.ucar.edu/models/cesm2/cesm/machines.html>`_ and
userdefined machines are provided with the RCESM and CESM release. Machines are supported in RCESM/CESM on an individual basis
and are usually listed by their common site-specific name. To get a
machine ported and functionally supported in RCESM, local batch, run,
environment, and compiler information must be configured in the CIME
scripts. The machine name "userdefined" machines refer to any machine
that the user defines and requires that a user edit the resulting xml
files to fill in information required for the target platform. This
functionality is handy in accelerating the porting process and quickly
getting a case running on a new platform. For more information on
porting, see the `CIME porting guide
<http://esmci.github.io/cime/users_guide/porting-cime.html>`_.  The
list of available machines are documented in `CESM supported machines
<http://www.cesm.ucar.edu/models/cesm2/cesm/machines.html>`_.
Running **query_config** with the ``--machines`` option will also show
the list of all machines for the current local version of
CESM. Supported machines have undergone the full CESM porting
process. The machines available in each of these categories changes as
access to machines change over time.

For the RCESM, three new machines are supported in our version of CIME that may or may not be supported by CESM. These include Ada, Terra and Stampede2.





