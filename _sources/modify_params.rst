.. _modify_params:

================================
Modifying R-CESM parameters
================================

All files required during R-CESM running phase are called input files and
files created by R-CESM during a run are called output files. Please note
that for models like ROMS configurable options are chosen during both
compilation and running phases. Since the source code always contain
files required during compilation stage, their list is not presented
here. But the important files for the compilation phase are discussed in
detail. Since the files needed during model running phase (for a new
experiment) are not available as a package like the R-CESM source code,
their list is presented for each component model. Again, most important
among these are discussed in detail. Please use the input files for the
Gulf of Mexico test case (Chapter `[cha:qkstart] <#cha:qkstart>`__) as a
reference for setting up a new experiment. Only a basic description is
provided in this chapter and for information on tools and procedure for
creating some of these input files, please see Chapter
`[cha:tools] <#cha:tools>`__.



.. note:: 

   If the model run involves many restart runs, keep the input files (for the running phase) which are invariant across the runs in a separate directory and make symbolic links in the run directory as required.


.. _sec:inpR:

ROMS Input Files
================

.. _sec:inpRcomp:

Compilation Phase
-----------------

For the ROMS model, some of the scheme/parametrization choices are made
pre-compilation using header (file with ‘.h’ extension) files. File
txgla_3dnudg.h in src/R-CESM-1.0.0/ directory is the ROMS header file for
the Gulf of Mexico test case. A copy of the header file used during
compilation will be available in src ‘exec/include’ directory (Section
`[sec:comp] <#sec:comp>`__).

.. _sec:inpRrun:

Running Phase
-------------

For ROMS, the 3D nudging file is required only if the 3D nudging is
turned on (see Section `[sec:3Dnudg] <#sec:3Dnudg>`__). All other files
are required for realistic configurations (ROMS do have the options to
provide grid, boundary and initial files through analytical functions
for idealized configurations). A list of ROMS input files are given in
Table `[tab:inpR] <#tab:inpR>`__.

.. table:: List of ROMS input files. Optional file is marked with #.



   +----+-------------------+-------------+---------------------------------+
   | #  |      Filename     |  File Type  | Description                     |
   +----+-------------------+-------------+---------------------------------+
   | 1  |  ∗_roms_grd.nc    |   netCDF    | Grid file                       |
   +----+-------------------+-------------+---------------------------------+
   | 2  | ∗_roms_ini/rst.nc |   netCDF    | Initial condition or restart file |
   +----+-------------------+-------------+---------------------------------+
   | 3  |  ∗_roms_bry.nc    |   netCDF    | Boundary condition file         |
   +----+-------------------+-------------+---------------------------------+
   | 4  | #∗_roms_nudg.nc   |   netCDF    | 3D nudging file (temp/salt/u/v) |
   +----+-------------------+-------------+---------------------------------+
   | 5  | ocean.in          |   txt       |   ROMS namelist/options file    |
   +----+-------------------+-------------+---------------------------------+
   | 5  | varinfo.dat       |   txt       | Input & Output variable details |
   +----+-------------------+-------------+---------------------------------+

  



ocean.in
~~~~~~~~

ROMS model options, including domain decomposition are specified in the
input file ocean.in. The total PE in ocean.in, computed as the product
of NtileI and NtileJ (eg. 16*24=384) should be same as ocn_tasks in
drv_in (&ccsm_pes namelist) (see Section `3.2.3 <#sec:drvin>`__).

.. _sec:inpW:

WRF Input Files
===============


WRF input netCDF and namelist files are listed in Table
`[tab:inpW] <#tab:inpW>`__, which are mandatory for all cases.

.. table:: List of WRF input netCDF and namelist files. Item 4 is
required only for restart runs.

   +----+-------------------+-------------+---------------------------------+
   | #  |      Filename     |  File Type  | Description                     |
   +----+-------------------+-------------+---------------------------------+
   | 1  |  wrfinput_d01.nc  |   netCDF    | Grid file                       |
   +----+-------------------+-------------+---------------------------------+
   | 2  |  wrflowinp_d01    |   netCDF    | Lower boundary conditions       |
   +----+-------------------+-------------+---------------------------------+
   | 3  |  wrfbdy_d01       |   netCDF    | Lateral boundary conditions     |
   +----+-------------------+-------------+---------------------------------+
   | 4  | #atm.r.2010-01-04.∗.nc |   netCDF    |  WRF restart files         |
   +----+-------------------+-------------+---------------------------------+
   | 5  | namelist.input    |   txt       |   WRF namelist/options file     |
   +----+-------------------+-------------+---------------------------------+
   


.. _sec:nlistW:

namelist.input
~~~~~~~~~~~~~~

Please adapt the namelist.input file provided with Gulf of Mexico test
case for a new application rather than using namelist.input file
available with independant WRF distribution. Please note that the domain
decomposition (processor tiling) is automatically determined by R-CESM
and there is no field in namelist.input to control this aspect for the
WRF component. Please see Section `3.2.3 <#sec:drvin>`__ for details
about processor specifying options for R-CESM.

.. _sec:inpC:

R-CESM/Coupler Input Files
=========================




All of the required configuration options for an experiment with the RCESM are encapsulated in XML variables within various files in the case directory. While it is possible to edit these files directly, it is recommended that users use the ``xmlquery`` and ``xmlchange`` scripts to view and modify the value of these variables. 

As an example, the model is set to run for 5 days by default. This is controlled by the ``$STOP_N`` and
``$STOP_OPTION`` variables. To view the current values of these variables, use the ``xlmquery`` command

.. code-block:: console

   ./xmlquery STOP_OPTION,STOP_N

The **Gulf of Mexico example** from above has mostly been tested for this 30 day period. If you wanted to run it for a period of three months, you can use ``xmlchange``

.. code-block:: console

      ./xmlchange STOP_OPTION=nmonths,STOP_N=3
.. code-block:: console

The advantage of using ``xmlchange`` compared to manually editing the XML config files are 1. The input values are automatically checked for errors before the variable is modified. 2. Each call to ``xmlchange`` is logged in the ``CaseStatus`` file so that we can keep track of all our changes from the default in any given experiment.

    
CESM xml variables are fully documented in the CESM2.1 release documents.  Here is a short compilation of variables that may be useful in testing or running RCESM experiments.

 ===================  ========================
  XML Variable           Description
 ===================  ========================
  PROJECT                Account project number to charge compute time to
  JOB_QUEUE              Which queue to submit a job, if different than default
  JOB_WALLCLOCK_TIME     Wall time to request for a job
  STOP_OPTION            What units to use for the specified run length. Valid values: nsteps, ndays, nmonths, nyears
  STOP_N                 The number of STOP_OPTION units that the experiment will complete
  RUN_STARTDATE          The date on which the experimental run begins
  DEBUG                  Whether to compile the model with debug flags on
  DOUT_S                 Turns archiving of history and restart files on (TRUE) or off (FALSE)
  DIN_LOC_ROOT           Location of the input data directory structure
 ===================  ========================


.. _sec:rpointer:

rpointer Files
~~~~~~~~~~~~~~

The rpointer here means "restart pointer" which informs R-CESM about
restart date and time. There are 5 rpointer files, one for each
component as shown below:

+-----------------+-----------------------------------------------+
| atmosphere/WRF: | rpointer.atm                                  |
+-----------------+-----------------------------------------------+
| data ocean:     | rpointer.docn                                 |
+-----------------+-----------------------------------------------+
| driver/coupler: | rpointer.drv                                  |
+-----------------+-----------------------------------------------+
| ROMS:           | rpointer.roms                                 |
+-----------------+-----------------------------------------------+
| ocean:          | rpointer.ocn (symbolic link to rpointer.roms) |
+-----------------+-----------------------------------------------+

Please note that the time format for these files are different (like
2010-01-04_00_00_00 for atm and 2010-01-04-00000 for drv). The
rpointer.drv file use acronym cpl in it (like
TXGLO.cpl.r.2010-01-04-00000.nc) instead of drv. Also there are two
entries for the rpointer.docn and other have just one entry. The usage
details of rpointer files are listed below.

For the very first run from initial condition files, set the entries in
all rpointer files to expected restart date.

For all successfull runs, the rpointer files are automatically updated
with the most recet restart date and time.

If you intend to continue a run from most recent restart files, just
these updated rpointer files to the restart run directory.

If you are making restart run from a different restart file, please
update the rpointer files accordingly.

For restart runs, the first entry in rpointer files should correspond to
the restart date and time.

::

         TXGLO.atm.r.2010-04-20_00_00_00.nc   
         TXGLO.atm.r.2010-04-30_00_00_00.nc

Please note that the restart file writing frequency is not determined by
rpointer files but by the value of "restart_n" in drv_in (see Section
`[sec:rstfr] <#sec:rstfr>`__ for details).

.. _sec:docnyr:

docn_ocn_in
~~~~~~~~~~~

The syntax for years in the streams entry of "docn_ocn_in" is as
follows:

::

          streams = "docn.streams.txt.prescribed YrAlign yrFirst yrLast"

It appears that the YrAlign should be same as YrFirst always!!!!!

.. _sec:drvin:

drv_in
~~~~~~

The number of processors/cores (PEs) for running R-CESM and its component
models should be clearly mentioned in drv_in (&ccsm_pes namelist). If
drv_in is edited to update PE count or layout, pleae edit the ocean.in
(Section `1.2.5 <#sec:nlistR>`__) and run_R-CESM.job file (Section
`4.1 <#sec:jobfl>`__) accordingly. Please note that the total number of
PEs are devided between atm_ntasks and ocn_ntasks. Also, atm_rootpe is 0
and ocn_rootpe is same as atm_ntasks. All other component model mirrors
the settings for the atm. Two examples for total PE counts of 552 and
120 are provided in Table `[tab:pe] <#tab:pe>`__.

.. table:: PE layout in drv_in. Total number of PE is determined by the
sum of number of atm model PE (atm_ntasks) and ocn model PE
(ocn_ntasks). Please note that the root PE for atm is 0 and that for ocn
in atm_ntasks. Other component models mirror atm model settings.

   +--------------+--------------+--------------+
   | ccsm_pe      | Total        | Total        |
   +--------------+--------------+--------------+
   | field        | PE=552       | PE=120       |
   +--------------+--------------+--------------+
   | atm_ntasks   | 168          | 40           |
   +--------------+--------------+--------------+
   | atm_nthreads | 1            | 1            |
   +--------------+--------------+--------------+
   | atm_rootpe   | 0            | 0            |
   +--------------+--------------+--------------+
   | atm_pestride | 1            | 1            |
   +--------------+--------------+--------------+
   | atm_layout   | ‘concurrent’ | ‘concurrent’ |
   +--------------+--------------+--------------+
   | lnd_ntasks   | 168          | 40           |
   +--------------+--------------+--------------+
   | lnd_nthreads | 1            | 1            |
   +--------------+--------------+--------------+
   | lnd_rootpe   | 0            | 0            |
   +--------------+--------------+--------------+
   | lnd_pestride | 1            | 1            |
   +--------------+--------------+--------------+
   | lnd_layout   | ‘concurrent’ | ‘concurrent’ |
   +--------------+--------------+--------------+
   | ocn_ntasks   | 384          | 80           |
   +--------------+--------------+--------------+
   | ocn_nthreads | 1            | 1            |
   +--------------+--------------+--------------+
   | ocn_rootpe   | 168          | 40           |
   +--------------+--------------+--------------+
   | ocn_pestride | 1            | 1            |
   +--------------+--------------+--------------+
   | ocn_layout   | ‘concurrent’ | ‘concurrent’ |
   +--------------+--------------+--------------+
   | ice_ntasks   | 168          | 40           |
   +--------------+--------------+--------------+
   | ice_nthreads | 1            | 1            |
   +--------------+--------------+--------------+
   | ice_rootpe   | 0            | 0            |
   +--------------+--------------+--------------+
   | ice_pestride | 1            | 1            |
   +--------------+--------------+--------------+
   | ice_layout   | ‘concurrent’ | ‘concurrent’ |
   +--------------+--------------+--------------+
   | glc_ntasks   | 168          | 40           |
   +--------------+--------------+--------------+
   | glc_nthreads | 1            | 1            |
   +--------------+--------------+--------------+
   | glc_rootpe   | 0            | 0            |
   +--------------+--------------+--------------+
   | glc_pestride | 1            | 1            |
   +--------------+--------------+--------------+
   | glc_layout   | ‘concurrent’ | ‘concurrent’ |
   +--------------+--------------+--------------+
   | cpl_ntasks   | 168          | 40           |
   +--------------+--------------+--------------+
   | cpl_nthreads | 1            | 1            |
   +--------------+--------------+--------------+
   | cpl_rootpe   | 0            | 0            |
   +--------------+--------------+--------------+
   | cpl_pestride | 1            | 1            |
   +--------------+--------------+--------------+

.. _sec:docnstr:

docn.streams.txt.prescribed
~~~~~~~~~~~~~~~~~~~~~~~~~~~

This file provides path and filenames for the domain info file (eg.
domain.txglo.nc) and the xROMS sea surface temperature (SST) and ice
fields (eg. :math:`*`\ \_xroms_sstice.nc). Please update the value for
"<filePath>" and "<fileNames>" for both "<domainInfo>" and "<fieldInfo>"
entries as appropriate.

.. _sec:map:


