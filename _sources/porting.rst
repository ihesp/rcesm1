.. _porting:

====================================
 Porting R-CESM to new machines
====================================


An RCESM case directory contains all of the configuration xml files, case control scripts, and namelists to start a RCESM run. It also contains the README document which contains information about the case as it was created, and the CaseStatus document that keeps track of changes as you go.

 - Edit the ``my_rcesm_sandbox/cime/config/cesm/machines/config_machines.xml`` file to point to correct input file directories and output file locations before creating a case. An example on Ada. 

 .. code-block:: XML

    <machine MACH="grace">
        <OS>LINUX</OS>
        <COMPILERS>intel</COMPILERS>
        <MPILIBS>impi</MPILIBS>
        
        <CIME_OUTPUT_ROOT>/ihesp/shared/rundir/rcesm</CIME_OUTPUT_ROOT>
        <DIN_LOC_ROOT>/ihesp/obs_root/inputdata</DIN_LOC_ROOT>
        
        <DOUT_S_ROOT>$ENV{SCRATCH}/archive/$CASE</DOUT_S_ROOT>
        
        <BATCH_SYSTEM>slurm</BATCH_SYSTEM>
        <mpirun mpilib="impi">
          <executable>mpirun</executable>
          <arguments>
             <arg name="num_tasks"> -np $TOTALPES</arg>
          </arguments>
        </mpirun>
        <module_system type="module">
          <modules>
            <command name="purge"></command>
            <command name="load">intel/2019b</command>
            ....
            <command name="load">PnetCDF/1.12.1</command>
            <command name="load">netCDF/4.7.4</command>
            <command name="load">netCDF-Fortran/4.5.3</command>
          </modules>
       </module_system>
        <environment_variables>
          <env name="NETCDF_PATH">$IHESP/software/easybuild/eb/x86_64/sw/netCDF/4.7.4-iimpi-2019b</env>
          <env name="PNETCDF_PATH">$IHESP/software/easybuild/eb/x86_64/sw/PnetCDF/1.12.1-iimpi-2019b</env>
          <env name="NETCDFF_INCDIR">$IHESP/software/easybuild/eb/x86_64/sw/netCDF-Fortran/4.5.3-iimpi-2019b/include</env>
        </environment_variables>

    </machine>


.. code-block:: XML

    <compiler MACH="grace" COMPILER="intel">
        <MPIFC>mpiifort</MPIFC>
        <MPICC>mpiicc</MPICC>
        <MPICXX>mpiicpc</MPICXX>
        <SFC>ifort</SFC>
        <SCC>icc</SCC>
        <SCXX>icpc</SCXX>
        <NETCDF_PATH>$IHESP/software/easybuild/eb/x86_64/sw/netCDF/4.7.4-iimpi-2019b</NETCDF_PATH>
        <NETCDF_C_PATH>$IHESP/software/easybuild/eb/x86_64/sw/netCDF/4.7.4-iimpi-2019b</NETCDF_C_PATH>
        <NETCDF_FORTRAN_PATH>$IHESP/software/easybuild/eb/x86_64/sw/netCDF-Fortran/4.5.3-iimpi-2019b</NETCDF_FORTRAN_PATH>
        <PNETCDF_PATH>$IHESP/software/easybuild/eb/x86_64/sw/PnetCDF/1.12.1-iimpi-2019b</PNETCDF_PATH>
        <SLIBS>
            <append>-L$(NETCDF_C_PATH)/lib -Wl,-rpath,$(NETCDF_PATH)/lib -lnetcdf</append>
            <append>-L$(NETCDF_FORTRAN_PATH)/lib -Wl,-rpath,$(NETCDF_FORTRAN_PATH)/lib -lnetcdff</append>
            <append>-L$(PNETCDF_PATH)/lib -Wl,-rpath,$(PNETCDF_PATH)/lib -lpnetcdf</append>
        </SLIBS>
        <PIO_FILESYSTEM_HINTS>lustre</PIO_FILESYSTEM_HINTS>
    </compiler>


.. code-block:: XML

    <batch_system MACH="grace" type="slurm" >
        <batch_submit>sbatch</batch_submit>
        <submit_args>
          <arg flag="--time" name="$JOB_WALLCLOCK_TIME"/>
          <arg flag="-p" name="$JOB_QUEUE"/>
        </submit_args>
        <directives>
        <directive>--mem=360GB </directive>
        </directives>
        <queues>
          <queue walltimemax="02:00:00" nodemin="1" nodemax="32">short</queue>
          <queue walltimemax="24:00:00" nodemin="1" nodemax="128" default="true">medium</queue>
          <queue walltimemax="168:00:00" nodemin="1" nodemax="64" >long</queue>
          <queue walltimemax="504:00:00" nodemin="1" nodemax="32" >xlong</queue>
          <queue walltimemax="24:00:00" nodemin="1" nodemax="800" >special</queue>
        </queues>
    </batch_system>




The default CRESM code and test cases are tuned to work on TAMU Ada. For
porting it on to other machines, some changes may be required as
detailed in this section.

.. _sec:ada:

TAMU Ada
--------

On TAMU Ada, the following settings worked fine (last test Sep/2017) for
Fortran, C, MPI, netCDF and pnetcdf libraries.

::

     [user@comp]$ module purge
     [user@comp]$ module load wrf-deps/intel-2015B
     [user@comp]$ module load pnetcdf/1.6.1-intel-2015B 

The default CRESM source code and test cases may be used without any
modification on TAMU Ada.

A note about pnetcdf on Ada. On Ada “filesystem-hints=gpfs" (while
installing pio), and “stripingunit=16777216" (run_cresm.job). For
reference, the complete environmental variables for run_cresm.job on Ada
is provided below:

::

      ulimit -s unlimited 
      module purge
      module load wrf-deps/intel-2015B
      module load pnetcdf/1.6.1-intel-2015B 
      export I_MPI_HYDRA_BOOTSTRAP=lsf
      export I_MPI_LSF_USE_COLLECTIVE_LAUNCH=1
      # set following to the number of hosts ie. (-n value)/(ptile value)
      export I_MPI_HYDRA_BRANCH_COUNT=18
      export PNETCDF_HINTS="striping_unit=16777216"

.. _sec:cheyenne:

UCAR Cheyenne
-------------

Compared to the default version (tuned for TAMU Ada), NCAR Cheyenne
needs some modification (as of Oct/10/2017) for the following reasons:

On Cheyenne, mpiifort and mpiicc are present in proper location (path
set using “module load" command) but they do not work. So, some of the
files need to be edited to set mpif90 and mpicc as default compilers.

pnetcdf library is available on Cheyenne (module load pnetcdf/1.8.0) but
there is absolutely no documentation online about the “filesystem-hints"
(while installing pio) and the environmental variables (needed at
runtime, like “striping_unit"). Hence use CRESM without pnetcdf on
Cheyenne.

makefile for ROMS includes a check for the version of “make" but it do
not have the version 4.0 (which is the one available on Cheyenne) in the
check list.

The default modules worked fine on Cheyenne. A list of loaded modules
are given below.

::

     [user@comp]$ module list
     1) ncarenv/1.2    3) ncarcompilers/0.4.1   5) netcdf/4.4.1.1
     2) intel/17.0.1   4) mpt/2.15f             

The complete list of modifications to port default CRESM on to NCAR
Cheyenne is given below.

Before installing libraries (Section `[sec:lib] <#sec:lib>`__)

| Edit build_mct.sh entry for “MPIFC" as follows (before mct
  installation)
| export MPIFC=mpif90

Edit build_pio.sh with correct entries for MPIFC and MPICC and remove
any reference to pnetcdf (before installing pio). The edited
build_pio.sh is given below.

::

     #!/bin/bash
     export CC=icc
     export FC=ifort
     export MPIFC=mpif90
     export MPICC=mpicc
     export MCT_PATH=/glade/p/work/...../Models/CRESM/lib/mct
     export NETCDF_PATH=/glade/u/apps/ch/opt/netcdf/...../17.0.1
     ./configure --prefix=/glade/p/work/...../Models/CRESM/lib/pio

Before compilation (Section `[sec:comp] <#sec:comp>`__)

Edit the “configure" file (cresm-1.0.0/configure) as follows:

::

     change following line from:
         for ac_prog in mpiifort mpif90 mpfort
     to
         for ac_prog in mpif90 mpiifort mpfort
     and following line from
         for ac_prog in mpiicc mpicc mpcc
     to 
         for ac_prog in mpicc mpiicc mpcc

Edit ROMS makefile (cresm-1.0.0/models/ocn/roms/makefile) as follows:

::

     change following line from:
         NEED_VERSION := 3.80 3.81 3.82
     to
         NEED_VERSION := 3.80 3.81 3.82 4.0

Edit “run_config.sh" (cresm-1.0.0/run_config.sh) to remove the entry for
pnetcdf (“:math:`--`\ with-pnetcdf=/path/to/pnetcdf")(also see Section
`[sec:nopnet] <#sec:nopnet>`__)

Before running (Section `[sec:run] <#sec:run>`__)

Edit WRF input file (namelist.input in run directory) entries for
io_form_history & restart as follows (value 2 is for non-pnetcdf case
and value 11 is for pnetcdf case (see Section
`[sec:nopnet] <#sec:nopnet>`__)

::

         io_form_history = 2
         io_form_restart = 2

Edit “run_cresm.job" (run directory) to remove any entries specific to
pnetcdf (like striping_unit, see Section `1.1 <#sec:ada>`__).

.. _sec:fixmdl:

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
