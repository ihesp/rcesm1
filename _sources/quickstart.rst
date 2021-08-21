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

   cd my_rcesm_sandbox
   cime/scripts/create_newcase --case ~/my_case_dirs/PBSGULF2010_gom27k --compset PBSGULF2010 --res txlw27k_gom27k --mach grace --run-unsupported


The result of the above command is the newly-created case directory `~/my_case_dirs/PBSGULF2010_gom27k`. 


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

    cd ~/my_case_dirs/PBSGULF2010_gom27k
    ./case.setup  



.. note::

    1. Prior to running ``case.setup``, the variable ``DIN_LOC_ROOT`` must point to an existing directory. If you get the message: ``ERROR: inputdata root is not a directory``, create the directory and run ``case.setup`` again.

    2. If you have modified any variables since the original ``case.setup``, you can run the command again using the ``--reset`` argument

    ``./case.setup --reset``




3. Obtaining the Input Files
==============================


Before proceeding with the rest of the model workflow, we need to make sure that we have created the inputdata root, and copied the required input files to it. 

For illustrative puproses, the standard test case for R-CESM is a Gulf of Mexico configuration with 27 km WRF and 27 km ROMS. This simulation is initialized on 2010-01-01_00:00:00 This can be obtained from this website. Extract this file to your inputdata root using the following command. 

These files are for a run initialized on 2010-01-01_00:00:00. Time
varying data is available till 2010-01-11_00:00:0.


Please ensure that the input files are present at the following directories. 

1. ``$DIN_LOC_ROOT/ocn/roms/gom27k/20100101_00/ocean.in``

2. ``$SRCROOT/components/roms/Apps/gom27k/ocn_in``

3. ``$DIN_LOC_ROOT/atm/wrf/txlw27k`` and ``$DIN_LOC_ROOT/atm/wrf/txlw27k_g27k``

4. ``$DIN_LOC_ROOT/lnd/clm2/surfdata``

5. ``$DIN_LOC_ROOT/share/domains``


See `Section: IO files <io_files.rst>`_ for a full listing of the files in each of these directories


If you would like to create your own application or a new domain, please see `<prepost_tools>`_ for details. However, we would recommend following along with the rest of this tuotorial before attempting to setup your own application. 



Please note that this test case have 3D-nuding option for ROMS (nudging
3D temperature and salinity to HYCOM data). This is not normally used
for ROMS runs. See Section `[sec:3Dnudg] <#sec:3Dnudg>`__ for tuning off
the 3D-nudging.



4. View/edit case parameters
=============================


The configuration options that control the various settings of R-CESM and its individual components are available through several XML files and text files. 




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




6. Submitting the case and monitoring model output
=====================================================

After the model build completes successfully, you can submit a model run to the job queue with the command

.. code-block:: console

      ./case.submit

from the case directory. This will rebuild all of the model namelists and check that all of the correct input data has been linked and moved to the correct places within the run directory. Finally, it submits the job script ``.case.run`` to the job queue. 


We recommend checking on the status of your model runs using a variety of methods:

- Check if the job is currently running or queue using the job status command specific to your batch scheduler. For SLURM, you can use `squeue`, and for LSF, it's `bjobs`.

- The `$CASEROOT/CaseStatus` file contains a log of all the job states and xmlchange commands in chronological order. Check this file to get a quick idea of the model state and the location of the log files. 

- *Log files*: Each component of CESM logs all the standard output and error messages to separate log files in the $RUNDIR. If the model successfully completes the simulation, log files will be zipped and copied to the ``logs/`` subdirectory of the case directory. 

- *WRF per process output*: If the WRF component is running as the atmosphere, it produces two output files for each process, an rsl.out.XXXX file and an rsl.error.XXXX file (where XXXX is the process rank, ie. 0036). The standard output and standard error streams can be found in these files, which will remain in the run directory regardless of the success or failure of the model run.

- *History files*: During a successful model run, the model writes out history files at the specified frequency to the $RUNDIR. 




If the job completes successfully, the following files are written out:


- The model performance stats are written out to the $CASEROOT/timing directory.

- The restart files for each component are written out to $RUNDIR

- Submit the short-term archiver script case.st_archive to the batch queue if $DOUT_S is TRUE. Short-term archiving will copy and move component history, log, diagnostic, and restart files from $RUNDIR to the short-term archive directory $DOUT_S_ROOT

This behavior can be turned off (and history files remain in the run directory) by setting the xml variable ``DOUT_S`` to False in the case directory before submition. For more information on XML variables and how to query or change them, see `View/edit case parameters`_.




7. Restarting the case
========================

Currently, restarts have not been tested and are not supported in the RCESM. This is an important "to do" item on the list of `Bugs, Issues and Future work`_. Restart files are written and copied into the archive directory at ``\glade\scratch\{$user}\case_name\{$component_name}\rest``

But there is no guarentee they will work currently.


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



---------
ocean.in
---------

===================  ========================
  Variable            Description
===================  ========================
   NTIMES               17280
   NRREC                1 
   ININAME             $CASE.ocn.r.<time>.nc
===================  ========================


-------------------------------
drv_in (&seq_timemgr_inparm)
-------------------------------

start date: change start_ymd from 20100101 to 20100104


-------------------------------
ocn_in (&ocn_timemgr)
-------------------------------

change start_day from 01 to 04 (in general, edit the start year, month, day, hour, minute, and second according to the restart time)

-------------------------------
docn_ocn_in
-------------------------------

keep strems the same (the numeric entries represent yrAlign, yrFirst &
yrLast, please see Section `[sec:docnyr] <#sec:docnyr>`__ for details on
editing this for a multi-year run)

-------------------------------
rpointer.*
-------------------------------

Make sure the entries in every rpointer file refers to the model restart
date and time (2010-01-04_00:00:00).



Edit if required (see Section `[sec:rpointer] <#sec:rpointer>`__ for
details).

With these changes the restart run is ready to be submitted. After
submitting the job, monitor its progress and completion as described in
Section `3 <#sec:run>`__. It is highly recommended to check the output
as described in Section `4 <#sec:chk>`__.