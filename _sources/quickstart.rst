.. _quickstart:

====================================
 Quick Start (RCESM Model Workflow)
====================================

The following quick start guide is for versions of RCESM that have
already been ported to the local target machine. RCESM is built on the
CIME (Common Infrastructure for Modeling Earth) framework.
Please refer to the `CIME Porting Documentation <http://esmci.github.io/cime/users_guide/porting-cime.html>`_ if CIME has not
yet been ported to the target machine. 

If you are new to RCESM or CESM2, please consider reading the
`CIME Case Control System Part 1: Basic Usage guide <https://esmci.github.io/cime/index.html>`_ first.

This is the procedure for quickly setting up and running a RCESM case.

Download the RCESM source (see `Downloading RCESM <downloading_cesm.html>`_).

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

   The RCESM code was developed and tested around a fully-coupled Gulf of Mexico (GOM) case. We
   will use this case as the example throughout this quick start guide. There are options
   for other grids and configurations available. Please see the
   `available RCESM component sets <cesm_configurations.html>`_ and
   `supported model resolutions <cesm_configurations.html>`_ for more information.

.. note:: 

   Variables presented as ``$VAR`` in this guide typically refer to variables in XML files
   in a CESM case. From within a case directory, you can determine the value of such a
   variable with ``./xmlquery VAR``. In some instances, ``$VAR`` refers to a shell
   variable or some other variable; we try to make these exceptions clear.

Create a case
==============

The *create_newcase* command creates a case directory containing the scripts and XML
files to configure a case (see below) for the requested resolution, component set, and
machine. **create_newcase** has three required arguments: ``--case``, ``--compset`` and
``--res`` (invoke **create_newcase --help** for help).

On machines where a project or account code is needed (including NCAR's machines), you
must either specify the ``--project`` argument to **create_newcase** or set the
``$PROJECT`` variable in your shell environment.

If running on a supported machine, that machine will
normally be recognized automatically and therefore it is *not* required
to specify the ``--machine`` argument to **create_newcase**. 

Invoke **create_newcase** as follows:

.. code-block:: console

    ./create_newcase --case CASENAME --compset COMPSET --res GRID

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

- ``COMPSET`` is the `component set <cesm_configurations.html>`_.

- ``GRID`` is the model `resolution <cesm_configurations.html>`_.

- ``run-unsupported`` is required for all RCESM compsets as these should all be considered scientifically experimental within the RCESM/CESM code base.
  
**Here is an example** on NCAR machine cheyenne with the ``$USER`` shell environment variable
set to your cheyenne login name:

.. code-block:: console

   my_rcesm_sandbox/cime/scripts/create_newcase --case my_case_dirs/p.r10.pbsgulf2010.tx9k_g3x.my_new_test.001 --compset PBSGULF2010 -res tx9k_g3x --run-unsupported


Setting up the case run script
==============================

Issuing the *case.setup* command creates scripts needed to run the model
along with namelist ``user_nl_xxx`` files, where xxx denotes the set of components
for the given case configuration. Before invoking **case.setup**, modify
the ``env_mach_pes.xml`` file in the case directory using the *xmlchange* command
as needed for the experiment.

cd to the case directory. Following **the example from above**:

.. code-block:: console

    cd my_case_dirs/p.r10.pbsgulf2010.tx9k_g3x.my_new_test.001

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


Run the case
============

Modify runtime settings in ``env_run.xml`` (optional).

Run length: By default, the model is set to run for 5 days based on the ``$STOP_N`` and
``$STOP_OPTION`` variables:

.. code-block:: console

   ./xmlquery STOP_OPTION,STOP_N

These default settings can be useful in `troubleshooting
<http://esmci.github.io/cime/users_guide/troubleshooting.html>`_ runtime problems
before submitting for a longer time. The **Gulf of Mexico example** from above
has mostly been tested for this 5 day period. If you wanted to run it for an
entire month, you would change:

.. code-block:: console

      ./xmlchange STOP_OPTION=nmonths,STOP_N=1


After setting the run lenght, submit the job to the batch queue using the **case.submit** command.

.. code-block:: console

    ./case.submit

When the job is complete, most output will *NOT* be written under the case directory, but
instead under some other directories (on NCAR's cheyenne machine, these other directories
will be in ``/glade/scratch/$USER``). Review the following directories and files, whose
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


Restart the case
================

The RCESM supports the ability to restart from any point that restart files are written. To
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
