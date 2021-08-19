.. _quickstart:

.. include:: special.rst


====================================
 Quick Start (R-CESM Model Workflow)
====================================

This quick start guide walks through the steps necessary to set up an R-CESM model run. We will the use steps involved in the fully-coupled, low-resolution (27km ocean/atm/land), Gulf of Mexico (GoM) case to illustrate the workflow. Before you can proceed, you will need to download the R-CESM source code (see `Downloading R-CESM <downloading_cesm.html>`_).

.. note:: 

   This tutorial assumes that the R-CESM model code has been ported to your machine or computational cluster. Please refer to the `R-CESM Porting Documentation <porting.html>`_ for details on porting the model to the target machine.

.. note:: 

    If you are unfamiliar with the CIME Case Control system, we strongly recommend reading the `CIME basic usage guide <https://esmci.github.io/cime/index.html>`_ first.




Relevant directories
======================

R-CESM follows the CESM/CIME directory structure. The following directories are most relevant for model users:

- ``$SRCROOT`` This is the location where the R-CESM source code is cloned to. This is typically placed in a location that is backed up (Ex: user's home directory). In the `previous section <downloading_cesm.html>`_ , we cloned the source code to `$SRCROOT=~/my_rcesm_sandbox`
- ``$CIMEROOT`` This is the subdirectory of `$SRCROOT` containing the CIME source code. In our example, `CIMEROOT=~/my_rcesm_sandbox/cime`
- ``$CASEROOT`` refers to the fully-qualified path where the R-CESM case (`$CASE`) has been created. Note that this is different from `$SRCROOT`.
- ``$CIME_OUTPUT_ROOT`` is the base directory for the CESM build and model output. Contains the **bld** and **run** directories. **$CIME_OUTPUT_ROOT/run** is where R-CESM generates .nc and log files.
- ``$DIN_LOC_ROOT`` The CESM inputdata directory.
- ``$DOUT_S_ROOT``  The CESM short-term archive directory. If variable `$DOUT_S` was set to TRUE, then the log, history, and restart files will be copied into this directory by the short-term archiver. 



.. note:: 

   It's strongly recommend to point the ``$CIME_OUTPUT_ROOT`` to a location using a parallel Lustre or GPFS filesystems. On most HPC clusters, this is usually the scratch directory.


Assuming the R-CESM model has already been ported to your cluster, you may need to edit the paths in ``CIME_OUTPUT_ROOT``, ``DIN_LOC_ROOT`` and ``DOUT_S_ROOT`` for your machine/cluster in ``$CIMEROOT/config/cesm/machines/config_machines.xml``. For a full list of modifications, please consult the `R-CESM porting guide <porting.html>`_.



Note: To edit any of
the env xml files, use the *xmlchange* command.
invoke **xmlchange --help** for help.)



1. Create the new case
=======================

The first step in setting up an R-CESM model run is to create the case directory, which contains all of the configuration xml files, case control scripts, and namelists to start the run. It also contains the README document which contains information about the case as it was created, and the CaseStatus document that keeps track of changes as you go.


The **create_newcase** command creates a case directory containing the scripts and XML files to configure a case for the requested resolution, component set, and machine. The simplified syntax of the the **create_newcase** is the following:

.. code-block:: bash

    $ ./create_newcase --case CASENAME --compset COMPSET --res GRID --mach MACH 


where:

- ``CASENAME`` defines the name of your case (stored in the ``$CASE`` XML variable). This is a very important piece of metadata that will be used in filenames, internal metadata and directory paths. **create_newcase** will create the *case directory* with the same name as the ``CASENAME``. If ``CASENAME`` is simply a name (not a path), the case directory is created in the directory where you executed create_newcase. If ``CASENAME`` is a relative or absolute path, the case directory is created there, and the name of the case will be the last component of the path. The full path to the case directory will be stored in the ``$CASEROOT`` XML variable. R-CESM case names should follow the `CESM2 experiment naming conventions <http://www.cesm.ucar.edu/models/cesm2/naming_conventions.html#casenames>`_.

- ``COMPSET`` Component sets (compsets) define both the specific model components that will be used in a given R-CESM configuration, and any component-specific namelist or configuration settings that are specific to this configuration. This value can be the compset longname, shortname or the alias. See `component set <cesm_configurations.html>`_ for a list of compsets available in R-CESM. 

- ``GRID`` is the long name or alias of the model grid set used, which describes the grids and domains used by each component in the experiment. See `grid resolution <cesm_configurations.html>`_. for the list of currently-supported grid resolutions.

- ``MACH`` is the name of the machine. This argument allows CIME to load the correct environment and libraries, set up applicable node and task configurations, and configure submission scripts for the correct queues. On many NCAR-supported machines (such as Cheyenne) this flag is optional, as CIME can automatically determine the name of the machine. The specified machine name must be previously defined in ``$CIMEROOT/config/cesm/machines/config_machines.xml``. If your machine hasn't been ported yet, check out the `R-CESM porting guide <porting.html>`_. 

- ``run-unsupported`` This option is required for all R-CESM compsets, as these are still considered experimental configurations.

- On machines/clusters where a project account needed is for submitting jobs, you must either specify the ``--project`` argument to **create_newcase** or set the ``$PROJECT`` variable using `xmlchange`. 


- To see the full list of options, invoke **create_newcase --help** for help. 



In the following step, we are going to create a new case on the TAMU cluster Grace, using the **PBSGULF2010** compset and the 27km Gulf of Mexico grid **txlw27k_g27x**.


.. code-block:: console

   my_rcesm_sandbox/cime/scripts/create_newcase --case ./my_case_dirs/PBSGULF2010 --compset PBSGULF2010 --res txlw27k_gom27k --mach grace --run-unsupported


The result of the above command is the newly-created case directory `./my_case_dirs/PBSGULF2010`. 


.. note::

    To display the entire list of compsets, grids availabe in the model, you can use the *query_config* tool located
    in ``$CIMEROOT/cime/scripts``. 

    ``$CIMEROOT/cime/scripts/query_config --help``


2. Set up the case
=====================

Once the case has been created, navigate to the $CASEROOT and issue the ``case.setup`` command to create the scripts needed to run the model, along with namelist files for each component (``user_nl_xxx``). Before invoking **case.setup**, modify
the ``env_mach_pes.xml`` file in the case directory using the ``xmlchange`` command as needed for the experiment.

Continuing the low-res GoM workflow:

.. code-block:: console

    cd my_case_dirs/PBSGULF2010
    ./case.setup  



.. note::

    1. Prior to running ``case.setup``, the variable ``DIN_LOC_ROOT`` must point to an existing directory. Create the directory if required. 

    2. If you have modified any variables since the original ``case.setup``, you can run the command again using the ``--reset`` argument

    ``./case.setup --reset``



3. View/edit case parameters
=============================

All of the required configuration options for an experiment with the RCESM are encapsulated in XML variables within various files in the case directory. While it is possible to edit these files directly, it is recommended that users use the "xmlquery" and "xmlchange" scripts to access and manipulate the xml variables. These scripts give more information about each variable, do error checking on changes, and keep track of changes in the CaseStatus file so it is easy to see exactly what has been changed from the default in any given experiment.


As an example, the model is set to run for 5 days by default. This is controlled by the ``$STOP_N`` and
``$STOP_OPTION`` variables. To view the current values of these variables, use the ``xlmquery`` command

.. code-block:: console

   ./xmlquery STOP_OPTION,STOP_N

The **Gulf of Mexico example** from above has mostly been tested for this 5 day period. If you wanted to run it for a period of three months, you can use ``xmlchange``

.. code-block:: console

      ./xmlchange STOP_OPTION=nmonths,STOP_N=3
.. code-block:: console

    
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



4. Build the executable for the case
=====================================

After the case is successfully setup, we can proceed to building the executables using ``case.build`` command from inside the ``$CASEROOT``.

.. code-block:: console

    ./case.build 


This step can take 20-30 mins to complete, depending on the active models, with the WRF build taking the longest to finish. The build files and executables are created in ``$EXEROOT``, but we will not need to directly access these files.


.. note::

    If you have made changes to the case configuration since the original build, you will need to clean the build using the ``--clean`` or ``--clean-all`` arguments.

    Use ``./case.build --clean`` to clean the build files all model components, except supporting libraries (Ex: MCT, PIO).
    Use ``./case.build --clean-all`` to clean everything associated with the build.
    Use ``./case.build --clean compname`` to clean the build files for just the component ``compname``.



5. Obtaining the Input Files
==============================

The standard test case for R-CESM is a Gulf of Mexico configuration with 27 km WRF and 27 km ROMS. 

These files are for a run initialized on 2010-01-01_00:00:00. Time
varying data is available till 2010-01-11_00:00:0.

There are several different input files for the CRESM, which are
discssed in detail in Chapter `[cha:io] <#cha:io>`__. See this section
before trying to adapt the test case discussed below for a new
application.

Please note that this test case have 3D-nuding option for ROMS (nudging
3D temperature and salinity to HYCOM data). This is not normally used
for ROMS runs. See Section `[sec:3Dnudg] <#sec:3Dnudg>`__ for tuning off
the 3D-nudging.



6. Running an RCESM case and Looking at model output
=====================================================

After the model builds successfully, you can submit a model run to the compute queue with the command

.. code-block:: console

      ./case.submit

from the case directory. This will rebuild all of the model namelists and recheck to make sure that all of the correct input data has been linked and moved to the correct places within the run directory. It will then put together a submit script for the machine batch system and submit it. You can check on the status of your run either through the job status commands on your system (``qstat`` on Cheyenne) or by investigating the log output in the run directory.

When the job is complete, the simulation outputs are placed in different  located as follows

- *Log files*: If the simulation encounters an error, all log and output files will remain in the run directory. If the model successfully completes the simulation, log files will be zipped and copied to the ``logs/`` subdirectory of the case directory. 

- *WRF per process output*: If the WRF component is running as the atmosphere, it produces two output files for each process, an rsl.out.XXXX file and an rsl.error.XXXX file (where XXXX is the process rank, ie. 0036). The standard output and standard error streams can be found in these files, which will remain in the run directory regardless of the success or failure of the model run.

- *History files*: In the model's default configuration and after a successful run, all history files are moved to an archive directory on the user's larger scratch space. On Cheyenne, this is located at ``\glade\scratch\{$user}\case_name\{$component_name}\hist``

This behavior can be turned off (and history files remain in the run directory) by setting the xml variable ``DOUT_S`` to False in the case directory before submition. For more information on XML variables and how to query or change them, see `View/edit case parameters`_.

- Restart files: Currently, restarts have not been tested and are not supported in the RCESM. This is an important "to do" item on the list of `Bugs, Issues and Future work`_. Restart files are written and copied into the archive directory at ``\glade\scratch\{$user}\case_name\{$component_name}\rest``

But there is no guarentee they will work currently.


 most of the model output will be written to the experiment's run directory. For example, on the TAMU Ada machine, the run directories
are usually placed in ``/scratch/user/$USER`` ). Review the following directories and files, whose
locations can be found with **xmlquery** (note: **xmlquery** can be run with a list of
comma separated names and no spaces):

.. code-block:: console

   ./xmlquery RUNDIR,CASE,CASEROOT

- ``$RUNDIR``

  This directory is set in the ``env_run.xml`` file. This is the
  location where RCESM was run. There should be log files there for every
  component (i.e. of the form cpl.log.yymmdd-hhmmss). 
  Each component writes its own log file. Also see whether any restart or history files were
  written. To check that a run completed successfully, check the last
  several lines of the cpl.log file for the string "SUCCESSFUL
  TERMINATION OF CPL7-cesm".

- ``$CASEROOT``

  There could be standard out and/or standard error files output from the batch system.

- ``$CASEROOT/CaseDocs``

  The case namelist files are copied into this directory from the ``$RUNDIR``.

- ``$CASEROOT/timing``

  There should be two timing files there that summarize the model performance.



To post-process the results, please follow the steps outlined in the section `Post-processing utilities <prepost_tools.html>`_

Restart the case
================

**Not presently supported**

RCESM supports the ability to restart from any point that restart files are written. To
set the frequency that restart files are written, first check the ``$REST_N`` and
``$REST_OPTION`` variables:

.. code-block:: console

   $./xmlquery REST_OPTION,REST_N

The default values for these variables are set to the end of the simulation (so they should
be the same as ``$STOP_N`` and ``$STOP_OPTION`` variables initially). To set the
frequency so that restart files are written more often, change the variables to reflect the
values you want. For example, to write daily restart files, set:

.. code-block:: console

      $ ./xmlchange REST_OPTION=ndays,REST_N=1

Once your simulation is finished and you want to restart the run, you will need to change the
``$CONTINUE_RUN`` xml variable to true, and then submit the run again.  

.. code-block:: console

      ./xmlchange CONTINUE_RUN=TRUE
      ./case.submit

This change on its own will restart the model from the point of the last written restart. This
is because the model uses the ``rpointer.*`` files in the case run directory to determine
the restart date and time. These files are automatically set to point to the most recently written
set of restart files. If you were interested in restarting from an earlier write, you would need to
manually edit the filenames in each of the ``rpointer.*`` files in the case run directory.




Checking Output
===============

Please obtain E01_run1_output.tar from Jaison Kurian (jaisonk@tamu.edu)
for verifying the output from above test run and E01_run2_output.tar for
verifying the restart run discussed in Section `5 <#sec:restart>`__.

Make a directory and untar the output.tar file.

::

     [user@comp]$ mkdir /scratch/user/..../CRESM/test_case
     [user@comp]$ cd /scratch/user/..../CRESM/test_case
     [user@comp]$ tar -xvf /path/to/this/file/E01_run1_output.tar

Directories E01_run1_output and E01_run2_output correspond to the run
described in Section `3 <#sec:run>`__ and in Section
`5 <#sec:restart>`__ respectively. The output netCDF file string has the
following meaning:

+===================================+====================================+
| TXGLO:                            | name of the domain/experiment     |
+===================================+====================================+
| atm/ocn/cpl:                      | component model (atm=atmosphere,  |
|                                   | ocn=ocean, cpl=coupler)           |
+-----------------------------------+-----------------------------------+
| hi/r:                             | file type (hi=history, r=restart) |
+-----------------------------------+-----------------------------------+
| YYYY-MM-DD_hh:mm:ss:              | date and time for first record in |
|                                   | the file                          |
+-----------------------------------+-----------------------------------+

For More details about the CRESM output files, please see Section
`[sec:output] <#sec:output>`__. Compare the output netCDF files in this
directory with that from the test run (described in Section
`3 <#sec:run>`__). Make sure it matches perfectly. If the test case is
made on a different machine than TAMU Ada, you may notice slight
missmatch in values (eg. difference around 3rd or 4th decimal place for
K=50 temp in ocean history files) which is quite normal.

Also compare the log files to see they have similar integrated values
(eg. ocn.log) and run times.

.. _sec:restart:

Restarting CRESM
================

Assume E01_run1 is the existing run directory, with restart files (from
the run described inSection `3 <#sec:run>`__). In order to make a
restart run, make a new run directory E01_run2.

::

     :red:`[user@comp]$` mkdir -p /scratch/..../CRESM/run/E01_run2
     [user@comp]$ cd /scratch/..../CRESM/run/E01_run2

Now use the mkcopy.csh tool (see Section `[sec:mkcopy] <#sec:mkcopy>`__)
to copy all the input files from previous run directory
(/scratch/..../CRESM/run/E01_run1) to current restart run directory.
Here gom03 is the prefix for ROMS files, "../E01_run1" is the source run
directory and "./" is the destination run directory.

::

     [user@comp]$ ~/bin/mkcopy.csh gom03 ../E01_run1 ./

Manually copy the map files from E01_run01

::

     [user@comp]$ cp -a ../E01_run1/map_???_????.nc .

Please note that "rpointer.*" files in E01_run1 will have the most
recent restart dates (here 2010-01-04_00:00:00). If the restart date is
different, please edit the "rpointer.*" files.

From E01_run1 directory, link ROMS, WRF, CPL restart file to E01_run2.
Use the restart date string to copy just the required restart files.

::

     [user@comp]$ ln -s ../E01_run1/TXGLO.*.r.2010-01-04*.nc ./

ROMS model can handle multiple boundary (BRYNAME) and nudging (CLMNAME)
netCDF files (eg. one file for each month and 12 such files for a 1-year
run). For the restart example here, the BRYNAME and CLMNAME for ROMS can
be the same but for real-life applications, it can be different. Also
note that the CLMNAME is required only if ROMS is running with
3D-nudging (see Section `[sec:3Dnudg] <#sec:3Dnudg>`__).

For the 3 day run from the restart file for 2010-01-04_00:00:00, the
model start date/time is 2010-01-04_00:00:00 and the end time is
2010-01-07_00:00:00. Edit the following files as suggested (for details
on the input files discussed below, please see Chapter
`[cha:io] <#cha:io>`__).

ocean.in

change NTIMES from 8640 to 17280 (note that ROMS need cumulative number
of timesteps from the very first run).

change NRREC from 0 to 1 (check ROMS restart file to make sure)

| change ININAME from ./gom03_N050_md15m_ini.....201001.nc to
| ./TXGLO.ocn.r.2010-01-04_00:00:00.nc

Keep BRYNAME the same

Keep CLMNAME the same

drv_in (&seq_timemgr_inparm)

start date: change start_ymd from 20100101 to 20100104

run length: Keep stop_n the same, 3 (if run length is different change
this)

restart interval: Keep restart_n the same, 3 (if run length is different
change this)

ocn_in (&ocn_timemgr)

change start_day from 01 to 04 (in general, edit the start year, month,
day, hour, minute, and second according to the restart time)

docn_ocn_in

keep strems the same (the numeric entries represent yrAlign, yrFirst &
yrLast, please see Section `[sec:docnyr] <#sec:docnyr>`__ for details on
editing this for a multi-year run)

namelist.input (&time_control)

change start_day from 01 to 04 (in general, edit the start year, month,
day, hour, minute, and second according to the restart time)

change end_day from 04 to 07 (in general, edit the end year, month, day,
hour, minute, and second according to the run end time)

change restart from .false. to .true. (this is a restart run).

keep restart_interval_d same at 3 (restart interval in days, change if
needed)

rpointer.

Make sure the entries in every rpointer file refers to the model restart
date and time (2010-01-04_00:00:00).

Please mind the syntax difference between rpointer files.

Edit if required (see Section `[sec:rpointer] <#sec:rpointer>`__ for
details).

With these changes the restart run is ready to be submitted. After
submitting the job, monitor its progress and completion as described in
Section `3 <#sec:run>`__. It is highly recommended to check the output
as described in Section `4 <#sec:chk>`__.