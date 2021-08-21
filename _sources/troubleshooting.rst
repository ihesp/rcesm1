.. _troubleshooting:

==============================================
Known Issues and Troubleshooting
==============================================

*This page last updated Jan 25, 2019*

Known Issues
------------

    Due to the experimental nature of the RCESM, not all of CESM's features have been implemented. Here are a few known issues that result in features missing from RCESM:

+ The short term archiving script does not currently work with RCESM.
+ The user_nl substituting scripts do not currently work in RCESM for WRF and ROMS. This needs the buildnml.csh scripts for each of these components to be updated to python scripts similar to other CESM build namelist scripts.
+ Usermods for WRF and ROMS are not implemented and have not been tested yet. This needs the buildlib.csh scripts for each of these components to be updated to python scripts similar to other CESM build scripts.
+ Not all of the CESM xml variables are properly routed into the WRF and ROMS configurations. In particular, WRF history file configuration is still done through the WRF namelist and not XML variables. Also, because the short term archiving script is not working correctly, the CIME ``$DSOUT`` variable does not behave correctly in this model. There are probably other CIME xml variables that are not correctly supported in this model, or WRF and ROMS configuration options that should be moved to CIME xml variables.
+ There may be WRF or ROMS build and namelist code that is specific to Cheyenne. So far, RCESM has been ported to three TAMU machines, so there may not be too much. The build and namelist processes for both of these models should be generalizd so the only code that needs to be ported is CIME.
+ The ROMS compset is hard-coded to only allow the ROMS%XROMS configuration. Need to get the %NULL compset configuration working again.
+ CESM testing has not been built for this model, so the create_test command does not currently work.
  

Troubleshooting
---------------

Create_newcase, Setup, and Submit Issues
========================================


.. code-block:: console

    $ ./case.setup 
      ERROR: inputdata root is not a directory: /ihesp/user/agopal/inputdata




Most of the errors produced by these scripts are self-explanitory. Use the "--help" option with any of these scripts to learn about more options provided, or the expected arguments. These scripts require python version 2.7 to run, and strange errors will occur if the version of python used to run the scripts is different. Make sure that the python module loaded by your machine via the ``cime/config/cesm/machines/config_machines.xml`` file is in the 2.7 series.

Case.build Errors
=================


.. code-block:: console

    $DIN_LOC_ROOT/ocn/roms/gom27k/20100101_00/ocean.in roms input file not found




.. code-block:: console

    File not found: atm2ocn_fmapname = "atm/wrf/tx27k_g27x/map_a2o_aave.nc", will attempt to download in check_input_data phase



.. code-block:: console

    make: *** No rule to make target `$casename/bld/lib/libatm.a', needed by `$casename/bld/cesm.exe'. Stop.

There are a few places where the build steps can go wrong. The first step in building this model is building the component namelists. Errors about missing fields or incorrect configurations of namelists could show up in this step. For WRF and ROMS, the namelist generating scripts are in the ``components/wrf/bld/wrf.buildnml.csh`` (and likewise for ROMS) scripts. Any configurations issues should be addressed here. For other components, changes might need to be made to configuration xml files within that component (for example, ``components/clm/bld/namelist_files/namelist_definition_clm4_0.xml``) or by adding lines to the user_nl file for that component in the case directory.

Errors that occur while the model is building will show up in the build logs. These are written to the build directory, which is parallel to the run directory (check the xml variable ``$RUNDIR`` in the case directory). Some common problems with building are:
- Having an incomplete or incorrect Manage_externals checkout. See `getting the CRESM code <https://ncar.github.io/TAMURegionalCESM/downloading_cesm.html>`_ for more information about manage_externals.
- NetCDF errors can be caused by having a NetCDF library that is built with a different compiler or different compiler version than what you are using to build RCESM.
- Some build errors are the result of compiler bugs in older compilers. The PGI compiler is particularly difficult to build with. Consider trying a newer version of your compiler or a different FORTRAN compiler if strange build errors appear. 

Runtime Errors
==============

Errors that occur during the model run will be written to log files. If the simulation encounters an error, all log and output files will remain in the run directory. If the WRF component is running as the atmosphere, it produces two output files for each process, an rsl.out.XXXX file and an rsl.error.XXXX file (where XXXX is the process rank, ie. 0036). The standard output and standard error streams can be found in these files, which will remain in the run directory regardless of the success or failure of the model run. The XROMS ocean component will write to three different logs: data.ocn.log, ocn.log, and roms.ocn.log. Check each of these for errors in case of a runtime failure. However, most run-time errors send the results to the full CESM log (cesm.log). Some suggestions to help debug runtime errors:

- Make sure all of the required data are in the appropriate locations to be linked for the simulation. The location for the data input directory can be found in the ``$DIN_LOC_ROOT`` xml variable in the case directory. The paths to the required files can be found in the namelist building scripts for WRF (``wrf/bld/wrf.bldnamelist.csh``) and ROMS (``roms/bld/roms.bldnamelist.csh``). 
- Check that your ``$JOB_WALLCLOCK_TIME`` value is long enough for your entire simulation to finish.
- To try to get more information about a run crash, set the xml variable ``$DEBUG`` to ``TRUE`` in the case directory. If you are running with a WRF atmosphere, you can also increase the debug output level of WRF in the namelist build script from 0 to 100 or even 200 (``wrf/bld/wrf.bldnamelist.csh``). This can add a lot of output to the rsl.out.XXXX and rsl.error.XXXX log files. 
- Make sure there is enough space in your ``/glade/scratch/$user`` directory (or generally your run directory) to support all of the history files, restart files, output logs, and any copied forcing data. The ``gladequota`` command can help with this on Cheyenne.
- Occasionally, Cheyenne will cause a run to abort with a simple and vague ``MPT Error`` found in a log. All runs crash with MPT errors in the submission output file, so this file is less helpful. A Cheyenne MPT error could be just a blip in inter-node communication, and restarting the run should be tried.




Known Issues and Fixes
======================

.. _sec:nocompiler:

Cannot Find Compiler
--------------------

: On UCAR Cheyenne, with default modules loaded (ncarenv, ncarcompilers,
intel & mpt), the "which mpiifort" command returns the complete path to
mpiifort. However, while compiling the model (Section
`[sec:comp] <#sec:comp>`__) or trying to install mct or pio (see Section
`[sec:lib] <#sec:lib>`__), the following error pops up:

::

     which: no mpiifort in (/glade/u/apps/ch/opt/vim/8.0.0273/........

Same error message appear with mpiicc too.

: If mpif90 and mpicc are working, then use them with
cresm-1.0.0/tools/build_mct.sh, cresm-1.0.0/tools/build_pio.sh, &
cresm-1.0.0/configure (see Section `1.2 <#sec:cheyenne>`__) instead of
mpiifort and mpiicc.

.. _sec:nocompiler:

Make version Error with ROMS
----------------------------

: CRESM compilation (Section `[sec:comp] <#sec:comp>`__) fails for ROMS
component with the following error:

::

     Compilation Status : Error : makefile:32: *** This makefile
           requires one of GNU make version 3.80 3.81 3.82..  Stop.

: ROMS makefile (cresm-1.0.0/models/ocn/roms/makefile) has a check for
the version of “make" command against a predefined list, which includes
3.80, 3.81 & 3.82. On machines with a different version of make, this
check will fail. For make versions closer to 3.8 (say 4.0) simply add
the version number to the existing list to make it work (Section
`1.2 <#sec:cheyenne>`__):

::

     change following line from:
         NEED_VERSION := 3.80 3.81 3.82
     to
         NEED_VERSION := 3.80 3.81 3.82 4.0

More careful editing of the makefile may be required if the version of
make command is very different from 3.8.

.. _sec:segerr:

CRESM SIGSEGV Error
-------------------

: CRESM fails during run time with the following error:

::

   forrtl: severe (174): SIGSEGV, segmentation fault occurred
   Image              PC                Routine         Line     Source             
   cresm              0000000002F4E4A1  Unknown         Unknown  Unknown
   .....................................................................
   libpthread-2.19.s  00002AAAAAEE3870  Unknown         Unknown  Unknown
   libmpi_mt.so       00002AAAAB57C9ED  MPI_SGI_bcast   Unknown  Unknown

: This error happens when pio and CRESM were compiled with pnetcdf
options but runtime pnetcdf options are not set up correctly. So, either
compile without pnetcdf (Sections `[sec:nopnet] <#sec:nopnet>`__ and
`1.2 <#sec:cheyenne>`__) or set pnetcdf environment properly (Section
`1.1 <#sec:ada>`__).

.. _sec:blowup:

ROMS Blow-up Issue
------------------

: The ROMS component of CRESM sometimes blows up with NaN values in the
Potential Energy and Total Energy fields. If this happens, please search
for the string "NaN" in ocn.log file in run directory. Check whether you
see lines like:

::

    2881    1 00:00:30 1.150065E-02 1.543130E+04 1.543131E+04 3.563578E+15
          (282,410,25) 2.585091E-03 4.415171E-04 9.561279E-02 1.917248E+00
    2882    1 00:01:00 1.150041E-02 1.543130E+04 1.543131E+04 3.563578E+15
          (282,410,25) 2.587113E-03 4.457352E-04 9.571663E-02 1.917091E+00
    2883    1 00:01:30 1.150017E-02          NaN          NaN 3.563578E+15
          (282,410,25) 2.588798E-03 4.498424E-04 9.583168E-02 1.916931E+00

As shown above, typically the blow-up happens at 2883’rd time step from
the start. It could happen for the restart runs too (eg. restart time
step 8640 and blows up at time step 11523). This blow-up is related to
some model bug (but not from model inputs) and the exact reason has not
been figured out yet.

: Resubmitting the run/job again works fine during most of the time.
Please consider cleaning up the run directory (see Section
`[sec:clean] <#sec:clean>`__) before resubmitting the job.

.. _sec:sigerr:

WRF SIGINT Error
----------------

: For some reason, with the intel/MPI libraries on Ada (see Section
`[sec:req] <#sec:req>`__), the CRESM run do not exit gracefully. With
respect to the model run and model output, restart and log files,
everything will be complete but still the job do not exit the queue. The
CRESM log file (cresm.log) do report some memory related issues as shown
below.

::

     forrtl: error (69): process interrupted (SIGINT)
     Image        PC                Routine     Line     Source             
     cresm        0000000002B24EA1  Unknown     Unknown  Unknown

: Could not find a clean solution yet. Please see Section
`[sec:complete] <#sec:complete>`__ to decide whether a CRESM run is
complete in terms of log files. If the job is complete but still showing
up in the queue, please kill the job. As a work around, please make sure
you only request for reasonable wall clock time limits for your job.

.. _sec:mcterr:

MCT error with mapping files
----------------------------

: CRESM fails/hangs with the errors in rsl.err.0000 similar to:

::

     SOIL TEXTURE CLASSIFICATION = STAS FOUND  19 CATEGORIES
       MCT::m_SparseMatrixPlus:: FATAL--length of vector y different 
                from row count of sMat.Length of y =   320295 Number 
                of rows in sMat =   694216
       000.MCT(MPEU)::die.: from MCT::m_SparseMatrixPlus::initDistributed_()
                application called MPI_Abort(MPI_COMM_WORLD, 2) - process 0

: This happens when mapping weight files (Section
`[sec:map] <#sec:map>`__) specified in seq_maps.rc (Table
`[tab:inpC] <#tab:inpC>`__) do not match with the CRESM configuration
(whether using ROMS or xROMS). When using xROMS, the mapping is from WRF
to xROMS (extented ROMS). But with ROMS (configured with
-enable-atm-sst) the mapping is from WRF to ROMS grid. For details about
mapping weight files, please see Section `[sec:map] <#sec:map>`__. Also
see Section `[sec:frocn] <#sec:frocn>`__ to know the difference between
ROMS and xROMS configuration.

.. _sec:docnerr:

Error with Data Ocean Year
--------------------------

: A run starting in 2010 and extending to 2011 (or longer) may fail with
error messages in data.ocn.log like:

::

     (docn_comp_run) ocn: model date 20101231   64800s
     (shr_stream_findBounds) ERROR: limit on and rDateIn gt rDategvd
        20101231.7604167        20101231.7500000     
     (shr_sys_abort) ERROR: (shr_stream_findBounds) ERROR: rDateIn gt 
                          rDategvd limit true
     (shr_sys_abort) WARNING: calling shr_mpi_abort() and stopping

or like:

::

     (shr_sys_abort) ERROR: (shr_stream_findBounds) ERROR: LVD not found,
                         all data is after yearLast
     (shr_sys_abort) WARNING: calling shr_mpi_abort() and stopping

: Please edit "docn_ocn_in" (Section `[sec:docnyr] <#sec:docnyr>`__) as

::

       streams = "docn.streams.txt.prescribed yrAlign yrFirst yrLast"

So, for a run starting anywhere in 2010 and ending in 2011, yrAlign and
yrFirst are 2010 and yrLast is 2011. Please note that yrAlign should be
same as yrFirst always!!!!!

.. _sec:diagerr:

Diagnosing Model Errors
=======================

Please check and verify the input files provided in Table
`[tab:diagerr] <#tab:diagerr>`__ to diagnose reason for error in CRESM
results. Please see Chapter `[cha:io] <#cha:io>`__ for details about
each file in the table.

.. table:: List of input files to check and verify in case of errors in
CRESM results. Please see Chapter `[cha:io] <#cha:io>`__ for details
about each file in the table.

   +-----------------+-----------------+-----------------+-----------------+
   | Sl. No.         | Filename        | Location/Where  |                 |
   |                 |                 | to Find         |                 |
   +=================+=================+=================+=================+
   | 1               | :math:`*`.h     | src compilation |                 |
   |                 | (ROMS header    | dir             |                 |
   |                 | file)           |                 |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 2               | :math:`*`\ \_ro | run or input    |                 |
   |                 | ms_grd.nc       | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 3               | :math:`*`\ \_ro | run or input    |                 |
   |                 | ms_bry.nc       | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 4               | :math:`*`\ \_ro | run or input    |                 |
   |                 | ms_ini/rst.nc   | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 5               | #\ :math:`*`\ \ | run or input    |                 |
   |                 | _roms_nudg.nc   | dir             |                 |
   |                 | ted             |                 |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 6               | ocean.in (ROMS  | run or input    |                 |
   |                 | namelist file)  | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 7               | varinfo.dat     | run or input    |                 |
   |                 |                 | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 8               | wrfinput_d01    | run or input    |                 |
   |                 |                 | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 9               | wrflowinp_d01   | run or input    |                 |
   |                 |                 | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 10              | wrfbdy_d01      | run or input    |                 |
   |                 |                 | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 11              | namelist.input  | run or input    |                 |
   |                 |                 | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 12              | drv_in          | run or input    |                 |
   |                 |                 | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 13              | ocn_in          | run or input    |                 |
   |                 |                 | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 14              | lnd_in          | run or input    |                 |
   |                 |                 | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 15              | ice_in          | run or input    |                 |
   |                 |                 | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 16              | docn_in         | run or input    |                 |
   |                 |                 | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 17              | docn_ocn_in     | run or input    |                 |
   |                 |                 | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 18              | docn.streams.tx | run or input    |                 |
   |                 | t.prescribed    | dir             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 19              | domain\_\ :math | run or input    |                 |
   |                 | :`*`.nc         | dir (path in    |                 |
   |                 |                 | docn_ocn_in)    |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 20              | #\_\ :math:`*`\ | run or input    |                 |
   |                 |  \_xroms_sstice | dir (path in    |                 |
   |                 | .nc             | docn.streams... |                 |
   |                 |                 | ..)             |                 |
   +-----------------+-----------------+-----------------+-----------------+
   | 21              | map_???_????.nc | run or input    |                 |
   |                 |                 | dir (path in    |                 |
   |                 |                 | seq_maps.rc)    |                 |
   +-----------------+-----------------+-----------------+-----------------+


  
Other Sources for Troubleshooting Info
======================================

Here are several other sources of information that could help with troubleshooting problems in the RCESM:

- The CIME documentation ( https://esmci.github.io/cime/index.html ) and CESM2 documentation ( https://escomp.github.io/cesm/release-cesm2/index.html ) Both contain a lot of information applicable to the RCESM modeling system, porting the code to other computers, common usage, and troubleshooting problems.
- The CRESM 1.0 documentation from Texas A&M university is available in the RCESM code repository. First, download the repository contents using `these instructions <https://ncar.github.io/TAMURegionalCESM/downloading_cesm.html>`_, and then look for the document in the ``docs/cresm1.0/cresm_manual.pdf`` file.
- There is a lot of documentation on CLM 4.0 at their website here: http://www.cesm.ucar.edu/models/cesm1.1/clm/
- The CESM community also discusses issues on the forums here: https://bb.cgd.ucar.edu/
- Documentation for the WRF model used in RCESM is on the web here: http://www2.mmm.ucar.edu/wrf/users/pub-doc.html
- Documentation for the ROMS model used in RCESM is on the web here: https://www.myroms.org/wiki/Documentation_Portal
