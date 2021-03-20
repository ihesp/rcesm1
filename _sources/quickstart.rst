.. _quickstart:

====================================
 Quick Start (R-CESM Model Workflow)
====================================

This quick start guide walks through the steps necessary in setting up an R-CESM model run. The R-CESM model was developed and tested around a fully-coupled Gulf of Mexico (GoM) case. We will use this case as the example throughout this quick start guide. Before you can proceed, you will need to download the R-CESM source code (see `Downloading R-CESM <downloading_cesm.html>`_).

.. note:: 

   This tutorial assumes that the R-CESM model code has been ported to your computational cluster. Please refer to the `CIME Porting Documentation <http://esmci.github.io/cime/users_guide/porting-cime.html>`_ if CIME has not yet been ported to the target machine.



If you are unfamiliar with the CIME Case Control system, we strongly recommend reading the `CIME basic usage guide <https://esmci.github.io/cime/index.html>`_ first.





Select a component set, and a resolution for your case.  Details of available
component sets and resolutions are available from the *query_config* tool located
in the ``my_rcesm_sandbox/cime/scripts`` directory

.. code-block:: console

    cd my_rcesm_sandbox/cime/scripts
    ./query_config --help

See the `available RCESM component sets <cesm_configurations.html>`_,
`supported model resolutions <cesm_configurations.html>`_ and `supported
machines <cesm_configurations.html>`_ for a list of RCESM
supported component sets, grids and computational platforms.

.. note::

    There are options
   for other grids and configurations available. Please see the
   `available RCESM component sets <cesm_configurations.html>`_ and
   `supported model resolutions <cesm_configurations.html>`_ for more information.




Relevant directories
======================

Just like for CESM, the following directories are relevant for every model run:

- ``$SRCROOT`` The directory containing the R-CESM source code. This is typically placed in a location that is backed up (Ex: user's home directory). In the current example, `$SRCROOT=~/my_rcesm_sandbox`
- ``$CIMEROOT`` The subdirectory of `$SRCROOT` containing the cime source code. Typically, `CIMEROOT=~/my_rcesm_sandbox/cime`
- ``$CASEROOT`` refers to the full pathname of the root directory where the R-CESM case (`$CASE`) will be created. Note that this is different from `$SRCROOT`.
- ``$CIME_OUTPUT_ROOT`` is the base directory for the CESM case output. Contains the **bld** and **run** directories. **$CIME_OUTPUT_ROOT/run** is where R-CESM generates .nc and log files.
- ``$DIN_LOC_ROOT`` The CESM inputdata directory
- ``$DOUT_S_ROOT``  The CESM short-term archive directory. If variable $DOUT_S was set to TRUE, then the log, history, and restart files will be copied into this directory by the short-term archiver. 





.. note:: 

   It's strongly recommend to point the ``$CIME_OUTPUT_ROOT`` to a location using a parallel Lustre or GPFS filesystems. On most HPC clusters, this is usually the scratch directory.



Assuming the R-CESM model has already been ported to your cluster, you may need to edit the paths in ``CIME_OUTPUT_ROOT``, ``DIN_LOC_ROOT`` and ``DOUT_S_ROOT`` corresponding to your cluster in ``$CIMEROOT/config/cesm/machines/config_machines.xml``. For a full list of modifications, please consult the R-CESM porting guide.



Create a new case
==================

An R-CESM case directory contains all of the configuration xml files, case control scripts, and namelists to start a R-CESM run. It also contains the README document which contains information about the case as it was created, and the CaseStatus document that keeps track of changes as you go.


The **create_newcase** command creates a case directory containing the scripts and XML
files to configure a case for the requested resolution, component set, and
machine. In the following step, we are going to create a new case on the TAMU cluster Ada, based on the **PBSGULF2010** compset and the **tx9k_g3x** grid.


.. code-block:: console

   my_rcesm_sandbox/cime/scripts/create_newcase --case ./my_case_dirs/PBSGULF2010 
   --compset PBSGULF2010 -res tx9k_g3x -mach ada --run-unsupported


 **create_newcase** has three required arguments: ``--case``, ``--compset`` and
``--res`` (invoke **create_newcase --help** for help).

On machines where a project or account code is needed (including NCAR's machines), you
must either specify the ``--project`` argument to **create_newcase** or set the
``$PROJECT`` variable in your shell environment.

If running on a supported machine, that machine will
normally be recognized automatically and therefore it is *not* required
to specify the ``--machine`` argument to **create_newcase**. 

Invoke **create_newcase** as follows:

.. code-block:: console

    ./create_newcase --case CASENAME --compset COMPSET --res GRID --mach MACH

where:

- ``CASENAME`` defines the name of your case (stored in the ``$CASE`` XML variable). This
  is a very important piece of metadata that will be used in filenames, internal metadata
  and directory paths. **create_newcase** will create the *case directory* with the same
  name as the ``CASENAME``. If ``CASENAME`` is simply a name (not a path), the case
  directory is created in the directory where you executed create_newcase. If ``CASENAME``
  is a relative or absolute path, the case directory is created there, and the name of the
  case will be the last component of the path. The full path to the case directory will be
  stored in the ``$CASEROOT`` XML variable. RCESM case names should follow the CESM naming  conventions. See `CESM2 Experiment Casenames
  <http://www.cesm.ucar.edu/models/cesm2/naming_conventions.html#casenames>`_ for
  details regarding CESM experiment case naming conventions.

- ``COMPSET`` is the name of the compset used. Compsets in CESM/RCESM describe which components are active and their basic configurations for the run. See `component set <cesm_configurations.html>`_ for a list of available compsets in RCESM. 

- ``GRID`` is the name of model resolution set used, which describes the grids and domains used in the experiment. See `resolution <cesm_configurations.html>`_. for the list of currently available resolutions.

- ``MACH`` is the machine where the build and run is happening. This allows CIME to load the correct environment and libraries, set up applicable node and task configurations, and configure submission scripts for the correct queues. On many NCAR-supported machines (such as Cheyenne) this flag is optional, as CIME can determine what machine it is on through the shell. For more information on porting to a new machine, see "Porting CIME and the RCESM to a new machine"_ below.

- ``run-unsupported`` is required for all RCESM compsets as these should all be considered scientifically experimental within the RCESM/CESM code base.
  
**Here is an example** on TAMU machine Ada. Check that your ``$USER`` shell environment variable is set to your Ada login name:



Setting up the case run script
==============================

Issuing the *case.setup* command creates scripts needed to run the model
along with namelist ``user_nl_xxx`` files, where xxx denotes the set of components
for the given case configuration. Before invoking **case.setup**, modify
the ``env_mach_pes.xml`` file in the case directory using the *xmlchange* command
as needed for the experiment.

cd to the case directory. Following **the example from above**:

.. code-block:: console

    cd my_case_dirs/PBSGULF2010

Modify settings in ``env_mach_pes.xml`` (optional). (Note: To edit any of
the env xml files, use the *xmlchange* command.
invoke **xmlchange --help** for help.)

Invoke the **case.setup** command.

.. code-block:: console

    ./case.setup  


Build the executable using the case.build command
=================================================

Modify build settings in ``env_build.xml`` (optional).

Run the build script.

.. code-block:: console

    ./case.build 

Users of the NCAR cheyenne system should consider using 
`qcmd <https://www2.cisl.ucar.edu/resources/computational-systems/cheyenne/running-jobs/submitting-jobs-pbs>`_
to compile RCESM on a compute node as follows:

.. code-block:: console

    qcmd -- ./case.build

The RCESM executable will appear in the directory given by the XML variable ``$EXEROOT``,
which can be queried using:

.. code-block:: console
   
   ./xmlquery EXEROOT


View/edit case parameters
=========================

All of the required configuration options for an experiment with the RCESM are encapsulated in XML variables within various files in the case directory. While it is possible to edit these files directly, it is recommended that users use the "xmlquery" and "xmlchange" scripts to access and manipulate the xml variables. These scripts give more information about each variable, do error checking on changes, and keep track of changes in the CaseStatus file so it is easy to see exactly what has been changed from the default in any given experiment.


As an example, the model is set to run for 5 days by default. This is controlled by the ``$STOP_N`` and
``$STOP_OPTION`` variables. To view the current values of these variables, use the ``xlmquery`` command

.. code-block:: console

   ./xmlquery STOP_OPTION,STOP_N

The **Gulf of Mexico example** from above has mostly been tested for this 5 day period. If you wanted to run it for an
entire month, you can use ``xmlchange``

.. code-block:: console

      ./xmlchange STOP_OPTION=nmonths,STOP_N=1
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



Running an RCESM case and Looking at model output
=====================================================================

After the model builds successfully, you can submit a run to the compute queue with the command ::

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

   ./xmlquery REST_OPTION,REST_N

The default values for these variables are set to the end of the simulation (so they should
be the same as ``$STOP_N`` and ``$STOP_OPTION`` variables initially). To set the
frequency so that restart files are written more often, change the variables to reflect the
values you want. For example, to write daily restart files, set:

.. code-block:: console

      ./xmlchange REST_OPTION=ndays,REST_N=1

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
