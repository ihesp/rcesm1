.. _io_files:

================================
R-CESM Input/Output Files
================================

All files required during CRESM running phase are called input files and
files created by CRESM during a run are called output files. Please note
that for models like ROMS configurable options are chosen during both
compilation and running phases. Since the source code always contain
files required during compilation stage, their list is not presented
here. But the important files for the compilation phase are discussed in
detail. Since the files needed during model running phase (for a new
experiment) are not available as a package like the CRESM source code,
their list is presented for each component model. Again, most important
among these are discussed in detail. Please use the input files for the
Gulf of Mexico test case (Chapter `[cha:qkstart] <#cha:qkstart>`__) as a
reference for setting up a new experiment. Only a basic description is
provided in this chapter and for information on tools and procedure for
creating some of these input files, please see Chapter
`[cha:tools] <#cha:tools>`__.

: If the model run involves many restart runs, keep the input files (for
the running phase) which are invariant across the runs in a separate
directory and make symbolic links in the run directory as required.

.. _sec:inpR:

ROMS Input Files
================

.. _sec:inpRcomp:

Compilation Phase
-----------------

For the ROMS model, some of the scheme/parametrization choices are made
pre-compilation using header (file with ‘.h’ extension) files. File
txgla_3dnudg.h in src/cresm-1.0.0/ directory is the ROMS header file for
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

   +-------------+-------------+-------------+-------------+-------------+
   | Sl.         | Filename    | Component   | File        | File        |
   +-------------+-------------+-------------+-------------+-------------+
   | No.         |             | Model       | Type        | Description |
   +-------------+-------------+-------------+-------------+-------------+
   | 1           | :math:`*`\  | ROMS        | netCDF      | Grid file   |
   |             | \_roms_grd. |             |             |             |
   |             | nc          |             |             |             |
   +-------------+-------------+-------------+-------------+-------------+
   | 2           | :math:`*`\  | ROMS        | netCDF      | Boundary    |
   |             | \_roms_bry. |             |             | condition   |
   |             | nc          |             |             | file        |
   +-------------+-------------+-------------+-------------+-------------+
   | 3           | :math:`*`\  | ROMS        | netCDF      | Initial     |
   |             | \_roms_ini/ |             |             | condition   |
   |             | rst.nc      |             |             | or restart  |
   |             |             |             |             | file        |
   +-------------+-------------+-------------+-------------+-------------+
   | 4           | #\ :math:`* | ROMS        | netCDF      | 3D nudging  |
   |             | `\ \_roms_n |             |             | file        |
   |             | udg.nc      |             |             | (temp/salt/ |
   |             |             |             |             | u/v)        |
   +-------------+-------------+-------------+-------------+-------------+
   | 5           | ocean.in    | ROMS        | txt         | Namelist/op |
   |             |             |             |             | tions       |
   |             |             |             |             | file        |
   +-------------+-------------+-------------+-------------+-------------+
   | 6           | varinfo.dat | ROMS        | txt         | Input &     |
   |             |             |             |             | Output      |
   |             |             |             |             | variable    |
   |             |             |             |             | details     |
   +-------------+-------------+-------------+-------------+-------------+

.. _sec:gridR:

Grid File
~~~~~~~~~

ROMS grid file (netCDF) contains all the grid related information needed
by ROMS model. In addition model grid cell latitude and logitude, grid
file contains several parameters like grid cell area, model bathymetry
and land-sea mask.

.. _sec:bryR:

Boundary Condition File
~~~~~~~~~~~~~~~~~~~~~~~

For open lateral boundaries, ROMS require time varying data for
temperature, salinity, u and v velocities and sea surface height. This
is provided through the boundary condition netCDF file.

.. _sec:iniR:

Inidial Condition or Restart File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For the very first run, ROMS need initial condition file with full 3D
model state (temperature, salinity, u nad v velocities and sea surface
height). For a restart run, this is provided through model generated
restart files.

.. _sec:nudgR:

3D Nudging File
~~~~~~~~~~~~~~~

ROMS can be configured with 3-dimentional nudging to specified data to
make sure model is not drifting too far away from the specified data
(typically from ocean reanalysis products). The data can be climatology
or inter-annually varying depending on the application. Nudging can be
along few grid points along the open boundaries or throughout the entire
ROMS domain (like in the Gulf of Mexico test case). Irrespective of this
choice, the 3D nudging data should always be for the entire 3D model
domain (a ROMS requirement) and time varying.

.. _sec:nlistR:

ocean.in
~~~~~~~~

ROMS model options, including domain decomposition are specified in the
input file ocean.in. The total PE in ocean.in, computed as the product
of NtileI and NtileJ (eg. 16*24=384) should be same as ocn_tasks in
drv_in (&ccsm_pes namelist) (see Section `3.2.3 <#sec:drvin>`__).

.. _sec:inpW:

WRF Input Files
===============

.. _sec:inpWcomp:

Compilation Phase
-----------------

Files during compilation WRF

.. _sec:inpWrun:

Running Phase
-------------

WRF input netCDF and namelist files are listed in Table
`[tab:inpW] <#tab:inpW>`__, which are mandatory for all cases.

.. table:: List of WRF input netCDF and namelist files. Item 4 is
required only for restart runs.

   +-------------+-------------+-------------+-------------+-------------+
   | Sl.         | Filename    | Component   | File        | File        |
   +-------------+-------------+-------------+-------------+-------------+
   | No.         |             | Model       | Type        | Description |
   +-------------+-------------+-------------+-------------+-------------+
   | 1           | wrfinput_d0 | WRF         | netCDF      | Grid file   |
   |             | 1           |             |             |             |
   +-------------+-------------+-------------+-------------+-------------+
   | 2           | wrflowinp_d | WRF         | netCDF      | Lower       |
   |             | 01          |             |             | boundary    |
   |             |             |             |             | conditions  |
   +-------------+-------------+-------------+-------------+-------------+
   | 3           | wrfbdy_d01  | WRF         | netCDF      | Lateral     |
   |             |             |             |             | boundary    |
   |             |             |             |             | conditions  |
   +-------------+-------------+-------------+-------------+-------------+
   | 4           | #atm.r.2010 | WRF         | netCDF      | Restart     |
   |             | -01-04.\ :m |             |             | file        |
   |             | ath:`*`.nc  |             |             |             |
   +-------------+-------------+-------------+-------------+-------------+
   | 5           | namelist.in | WRF         | txt         | Namelist/op |
   |             | put         |             |             | tions       |
   |             |             |             |             | file        |
   +-------------+-------------+-------------+-------------+-------------+

For WRF there are serveral table files in addition to input netCDF and
namelist files. All files in a WRF directory
(cresm-1.0.0/models/atm/wrf/3.5.1/WRFV3/run/*) are linked to run
directory as table files and no check has been done to see which all of
these files are mandatory for running CRESM and which all reduntant. A
list of table files are given in Table
`[tab:inpWtable] <#tab:inpWtable>`__.

.. table:: List of WRF table input files.

   +-----+----------------------------------+-----+-------------------------+
   | Sl. | Filename                         | Sl. | Filename                |
   +-----+----------------------------------+-----+-------------------------+
   | No. |                                  | No. |                         |
   +-----+----------------------------------+-----+-------------------------+
   | 1   | aerosol.formatted                | 24  | ETAMPNEW_....._rain     |
   +-----+----------------------------------+-----+-------------------------+
   | 2   | aerosol_lat.formatted            | 25  | ETAMPNEW_....._rain_DBL |
   +-----+----------------------------------+-----+-------------------------+
   | 3   | aerosol_lon.formatted            | 26  | GENPARM.TBL             |
   +-----+----------------------------------+-----+-------------------------+
   | 4   | aerosol_plev.formatted           | 27  | grib2map.tbl            |
   +-----+----------------------------------+-----+-------------------------+
   | 5   | CAM_ABS_DATA                     | 28  | gribmap.txt             |
   +-----+----------------------------------+-----+-------------------------+
   | 6   | CAM_AEROPT_DATA                  | 29  | LANDUSE.TBL             |
   +-----+----------------------------------+-----+-------------------------+
   | 7   | CAMtr_volume_mixing_ratio.A1B    | 30  | MPTABLE.TBL             |
   +-----+----------------------------------+-----+-------------------------+
   | 8   | CAMtr_volume_mixing_ratio.A2     | 31  | ozone.formatted         |
   +-----+----------------------------------+-----+-------------------------+
   | 9   | CAMtr_volume_mixing_ratio.RCP4.5 | 32  | ozone_lat.formatted     |
   +-----+----------------------------------+-----+-------------------------+
   | 10  | CAMtr_volume_mixing_ratio.RCP6   | 33  | ozone_plev.formatted    |
   +-----+----------------------------------+-----+-------------------------+
   | 11  | CAMtr_volume_mixing_ratio.RCP8.5 | 34  | RRTM_DATA               |
   +-----+----------------------------------+-----+-------------------------+
   | 12  | CLM_ALB_ICE_DFS_DATA             | 35  | RRTM_DATA_DBL           |
   +-----+----------------------------------+-----+-------------------------+
   | 13  | CLM_ALB_ICE_DRC_DATA             | 36  | RRTMG_LW_DATA           |
   +-----+----------------------------------+-----+-------------------------+
   | 14  | CLM_ASM_ICE_DFS_DATA             | 37  | RRTMG_LW_DATA_DBL       |
   +-----+----------------------------------+-----+-------------------------+
   | 15  | CLM_ASM_ICE_DRC_DATA             | 38  | RRTMG_SW_DATA           |
   +-----+----------------------------------+-----+-------------------------+
   | 16  | CLM_DRDSDT0_DATA                 | 39  | RRTMG_SW_DATA_DBL       |
   +-----+----------------------------------+-----+-------------------------+
   | 17  | CLM_EXT_ICE_DFS_DATA             | 40  | SOILPARM.TBL            |
   +-----+----------------------------------+-----+-------------------------+
   | 18  | CLM_EXT_ICE_DRC_DATA             | 41  | tr49t67                 |
   +-----+----------------------------------+-----+-------------------------+
   | 19  | CLM_KAPPA_DATA                   | 42  | tr49t85                 |
   +-----+----------------------------------+-----+-------------------------+
   | 20  | CLM_TAU_DATA                     | 43  | tr67t85                 |
   +-----+----------------------------------+-----+-------------------------+
   | 21  | co2_trans                        | 44  | URBPARM.TBL             |
   +-----+----------------------------------+-----+-------------------------+
   | 22  | ETAMPNEW_DATA                    | 45  | URBPARM_UZE.TBL         |
   +-----+----------------------------------+-----+-------------------------+
   | 23  | ETAMPNEW_DATA_DBL                | 46  | VEGPARM.TBL             |
   +-----+----------------------------------+-----+-------------------------+

.. _sec:nlistW:

namelist.input
~~~~~~~~~~~~~~

Please adapt the namelist.input file provided with Gulf of Mexico test
case for a new application rather than using namelist.input file
available with independant WRF distribution. Please note that the domain
decomposition (processor tiling) is automatically determined by CRESM
and there is no field in namelist.input to control this aspect for the
WRF component. Please see Section `3.2.3 <#sec:drvin>`__ for details
about processor specifying options for CRESM.

.. _sec:inpC:

CRESM Input Files
=================

.. _sec:inpCcomp:

Compilation Phase
-----------------

Files during compilation CRESM

.. _sec:inpCrun:

Running Phase
-------------

For CRESM and its coupler, there are several input files which are
listed in Table `[tab:inpC] <#tab:inpC>`__. Please provide all of these
files even if some of the component models (like ice) are not used. All
files which do not belong exclusively to either ROMS or WRF is included
in this category. The acronym "IO" implies input-output. (Some words are
used interchangably: ocean/ROMS, atmosphere/WRF, data ocean/xroms in
Table `[tab:inpC] <#tab:inpC>`__ and this will be corrected in the
future.)

.. table:: List of CRESM specific input netCDF and namelist files. Item
24 is only required for restart runs.

   +-------------+-------------+-------------+-------------+-------------+
   | Sl.         | Filename    | Component   | File        | File        |
   +-------------+-------------+-------------+-------------+-------------+
   | No.         |             | Model       | Type        | Description |
   +-------------+-------------+-------------+-------------+-------------+
   | 1           | domain.txgl | CRESM       | netCDF      | Domain file |
   |             | o.nc        |             |             |             |
   +-------------+-------------+-------------+-------------+-------------+
   | 2           | map_a2o_aav | CRESM       | netCDF      | Mapping     |
   |             | e.nc        |             |             | weight,     |
   |             |             |             |             | atm2ocn,    |
   |             |             |             |             | area-averag |
   |             |             |             |             | e           |
   +-------------+-------------+-------------+-------------+-------------+
   | 3           | map_a2o_bli | CRESM       | netCDF      | Mapping     |
   |             | n.nc        |             |             | weight,     |
   |             |             |             |             | atm2ocn,    |
   |             |             |             |             | bi-linear   |
   +-------------+-------------+-------------+-------------+-------------+
   | 4           | map_o2a_aav | CRESM       | netCDF      | Mapping     |
   |             | e.nc        |             |             | weight,     |
   |             |             |             |             | ocn2atm,    |
   |             |             |             |             | area-averag |
   |             |             |             |             | e           |
   +-------------+-------------+-------------+-------------+-------------+
   | 5           | seq_maps.rc | CRESM       | txt         | path to     |
   |             |             |             |             | mapping     |
   |             |             |             |             | weight      |
   |             |             |             |             | files       |
   +-------------+-------------+-------------+-------------+-------------+
   | 6           | #\ :math:`* | CRESM       | netCDF      | SST, ice &  |
   |             | `\ \_xroms_ |             |             | mask for    |
   |             | sstice.nc   |             |             | xroms       |
   +-------------+-------------+-------------+-------------+-------------+
   | 7           | drv_in      | CRESM       | txt         | driver file |
   |             |             |             |             | with date,  |
   |             |             |             |             | time,       |
   |             |             |             |             | processor   |
   |             |             |             |             | information |
   +-------------+-------------+-------------+-------------+-------------+
   | 8           | ocn_in      | CRESM       | txt         | ocean model |
   |             |             |             |             | time        |
   |             |             |             |             | manager &   |
   |             |             |             |             | IO          |
   +-------------+-------------+-------------+-------------+-------------+
   | 9           | lnd_in      | CRESM       | txt         | path to     |
   |             |             |             |             | land model  |
   |             |             |             |             | files       |
   +-------------+-------------+-------------+-------------+-------------+
   | 10          | ice_in      | CRESM       | txt         | path to ice |
   |             |             |             |             | model files |
   +-------------+-------------+-------------+-------------+-------------+
   | 11          | docn_in     | CRESM       | txt         | data ocean  |
   |             |             |             |             | namelist    |
   |             |             |             |             | file        |
   +-------------+-------------+-------------+-------------+-------------+
   | 12          | docn_ocn_in | CRESM       | txt         | data ocean  |
   |             |             |             |             | namelist    |
   |             |             |             |             | file & map  |
   |             |             |             |             | scheme      |
   +-------------+-------------+-------------+-------------+-------------+
   | 13          | docn.stream | CRESM       | txt         | path to     |
   |             | s.txt.presc |             |             | CRESM       |
   |             | ribed       |             |             | domain and  |
   |             |             |             |             | xroms files |
   +-------------+-------------+-------------+-------------+-------------+
   | 14          | ocn_modelio | CRESM       | txt         | ocean model |
   |             | .nml        |             |             | IO settings |
   +-------------+-------------+-------------+-------------+-------------+
   | 15          | atm_modelio | CRESM       | txt         | atmospheric |
   |             | .nml        |             |             | model IO    |
   |             |             |             |             | settings    |
   +-------------+-------------+-------------+-------------+-------------+
   | 16          | cpl_modelio | CRESM       | txt         | coupler IO  |
   |             | .nml        |             |             | settings    |
   +-------------+-------------+-------------+-------------+-------------+
   | 17          | lnd_modelio | CRESM       | txt         | land model  |
   |             | .nml        |             |             | IO settings |
   +-------------+-------------+-------------+-------------+-------------+
   | 18          | ice_modelio | CRESM       | txt         | ice model   |
   |             | .nml        |             |             | IO settings |
   +-------------+-------------+-------------+-------------+-------------+
   | 19          | glc_modelio | CRESM       | txt         | ice-sheet   |
   |             | .nml        |             |             | model IO    |
   |             |             |             |             | settings    |
   +-------------+-------------+-------------+-------------+-------------+
   | 20          | rpointer.oc | CRESM       | txt         | ocean       |
   |             | n           |             |             | restart     |
   |             |             |             |             | pointer     |
   +-------------+-------------+-------------+-------------+-------------+
   | 21          | rpointer.ro | CRESM       | txt         | ROMS        |
   |             | ms          |             |             | restart     |
   |             |             |             |             | pointer     |
   |             |             |             |             | (same as    |
   |             |             |             |             | rpointer.oc |
   |             |             |             |             | n)          |
   +-------------+-------------+-------------+-------------+-------------+
   | 22          | rpointer.at | CRESM       | txt         | atmosphere/ |
   |             | m           |             |             | WRF         |
   |             |             |             |             | restart     |
   |             |             |             |             | pointer     |
   +-------------+-------------+-------------+-------------+-------------+
   | 23          | rpointer.dr | CRESM       | txt         | coupler/dri |
   |             | v           |             |             | ver         |
   |             |             |             |             | restart     |
   |             |             |             |             | pointer     |
   +-------------+-------------+-------------+-------------+-------------+
   | 24          | rpointer.do | CRESM       | txt         | data ocean  |
   |             | cn          |             |             | restart     |
   |             |             |             |             | pointer     |
   +-------------+-------------+-------------+-------------+-------------+
   | 25          | #cpl.r.2010 | CRESM       | netCDF      | coupler     |
   |             | -\ :math:`* |             |             | restart     |
   |             | `.nc        |             |             | file        |
   +-------------+-------------+-------------+-------------+-------------+
   | 26          | cresm       | CRESM       | exe         | cresm       |
   |             |             |             |             | executable  |
   +-------------+-------------+-------------+-------------+-------------+
   | 27          | run_cresm.j | CRESM       | txt         | job         |
   |             | ob          |             |             | submission  |
   |             |             |             |             | file        |
   +-------------+-------------+-------------+-------------+-------------+

.. _sec:rpointer:

rpointer Files
~~~~~~~~~~~~~~

The rpointer here means "restart pointer" which informs CRESM about
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

The number of processors/cores (PEs) for running CRESM and its component
models should be clearly mentioned in drv_in (&ccsm_pes namelist). If
drv_in is edited to update PE count or layout, pleae edit the ocean.in
(Section `1.2.5 <#sec:nlistR>`__) and run_cresm.job file (Section
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

Mapping Weight Files
~~~~~~~~~~~~~~~~~~~~

Coupled model components can have different resolutions. CRESM requires
precomputed interpolation weights to map surface quantities between
different coupled model components. Interpolation options like bilinear
and averaging options like area-average are available with the ESMF
tool.

For a detailed discussion on mapping weight files and how to make them,
please see Section 5.2 in :raw-latex:`\citet{montuoro17}`.

.. _sec:xromssst:

xroms_sstice.nc
~~~~~~~~~~~~~~~

CRESM need data for SST and ice over entire domain. With xROMS set up
(Section `[sec:frxroms] <#sec:frxroms>`__), user has to provide an xROMS
file with SST and ice for the entire xROMS domain. SST for the bigger
domain is typycally available in WRF lower boundary input files. Current
test cases use ice as 0 everywhere.

A simple approach is to use matlab to interpolate WRF SST onto xROMS
grid and then write the interpolated SST to a proper xROMS SST netCDF
file (use the file from Gulf of Mexico test case as a reference).

.. _sec:inpO:

Other Input Files
=================

Other files required by CRESM during running phase are listed in Table
`[tab:inpO] <#tab:inpO>`__.

.. table:: List ofother machine dependent input files, including CRESM
executable.

   +-----+---------------+-----------+------+---------------------+
   | Sl. | Filename      | Component | File | File                |
   +-----+---------------+-----------+------+---------------------+
   | No. |               | Model     | Type | Description         |
   +-----+---------------+-----------+------+---------------------+
   | 1   | cresm         | CRESM     | exe  | cresm executable    |
   +-----+---------------+-----------+------+---------------------+
   | 2   | run_cresm.job | CRESM     | txt  | job submission file |
   +-----+---------------+-----------+------+---------------------+

.. _sec:jobfl:

run_cresm.job
-------------

This is the file used to submit a CRESM job to the job scheduler on the
supercomputer. The total PEs requested should be in agreement with the
total PEs in drv_in (Section `3.2.3 <#sec:drvin>`__) computed as the sum
of atm_ntasks and ocn_ntasks.

.. _sec:output:

All Output Files
================

Complete list of output files from a CRESM run is provided in Table
`[tab:outC] <#tab:outC>`__.

.. table:: List of all output files from a CRESM run.

   +-------------+-------------+-------------+-------------+-------------+
   | Sl.         | Filename    | Component   | File        | File        |
   +-------------+-------------+-------------+-------------+-------------+
   | No.         |             | Model       | Type        | Description |
   +-------------+-------------+-------------+-------------+-------------+
   | 1           | atm.log     | CRESM       | txt         | atm model   |
   |             |             |             |             | log file    |
   +-------------+-------------+-------------+-------------+-------------+
   | 2           | cpl.log     | CRESM       | txt         | coupler log |
   |             |             |             |             | file, run   |
   |             |             |             |             | statistics  |
   +-------------+-------------+-------------+-------------+-------------+
   | 3           | ocn.log     | ROMS        | txt         | ocean log   |
   |             |             |             |             | file        |
   +-------------+-------------+-------------+-------------+-------------+
   | 4           | roms.ocn.lo | ROMS        | txt         | ocean log   |
   |             | g           |             |             | file (same  |
   |             |             |             |             | as ocn.log) |
   +-------------+-------------+-------------+-------------+-------------+
   | 5           | cresm.log   | CRESM       | txt         | CRESM log   |
   |             |             |             |             | file        |
   +-------------+-------------+-------------+-------------+-------------+
   | 6           | data.ocn.lo | CRESM       | txt         | data ocean  |
   |             | g           |             |             | log file    |
   +-------------+-------------+-------------+-------------+-------------+
   | 7           | main.ocn.lo | CRESM       | txt         | mct log     |
   |             | g           |             |             |             |
   +-------------+-------------+-------------+-------------+-------------+
   | 8           | namelist.ou | WRF         | txt         | WRF options |
   |             | tput        |             |             | summary     |
   +-------------+-------------+-------------+-------------+-------------+
   | 9           | rsl.error.? | WRF         | txt         | WRF std.    |
   |             | ???         |             |             | error       |
   +-------------+-------------+-------------+-------------+-------------+
   | 10          | rsl.out.??? | WRF         | txt         | WRF std.    |
   |             | ?           |             |             | out         |
   +-------------+-------------+-------------+-------------+-------------+
   | 11          | :math:`*`.o | ROMS        | netCDF      | ROMS/ocn    |
   |             | cn.hi.\ :ma |             |             | history     |
   |             | th:`*`.nc   |             |             | files       |
   +-------------+-------------+-------------+-------------+-------------+
   | 12          | :math:`*`.o | ROMS        | netCDF      | ROMS/ocn    |
   |             | cn.r.\ :mat |             |             | restart     |
   |             | h:`*`.nc    |             |             | files       |
   +-------------+-------------+-------------+-------------+-------------+
   | 13          | :math:`*`.a | WRF         | netCDF      | WRF/atm     |
   |             | tm.hi.\ :ma |             |             | history     |
   |             | th:`*`.nc   |             |             | files       |
   +-------------+-------------+-------------+-------------+-------------+
   | 14          | :math:`*`.a | WRF         | netCDF      | WRF/atm     |
   |             | tm.r.\ :mat |             |             | restart     |
   |             | h:`*`.nc    |             |             | files       |
   +-------------+-------------+-------------+-------------+-------------+
   | 15          | :math:`*`.c | CRESM       | netCDF      | coupler     |
   |             | pl.r.\ :mat |             |             | restart     |
   |             | h:`*`.nc    |             |             | files       |
   +-------------+-------------+-------------+-------------+-------------+
   | 16          | :math:`*`.d | CRESM       | binary      | data ocean  |
   |             | ocn.rs1.\ : |             |             | restart     |
   |             | math:`*`.bi |             |             | files       |
   |             | n           |             |             |             |
   +-------------+-------------+-------------+-------------+-------------+
