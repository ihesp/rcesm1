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





configure.wrf.grace 








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
