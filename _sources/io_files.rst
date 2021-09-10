.. _io_files:

================================
R-CESM Input/Output Files
================================

In this section, we will discuss the grid, mapping and other datasets required for running the model, and the files that result from a successful or unsuccessful model run. We also have files that control model configuration, which is discussed in the next section `Modifying R-CESM parameters <modify_params.rst>`__. 

Please use the input files for the 27km, low-resolution, Gulf of Mexico test case (`<#cha:qkstart>`__) as a
reference for setting up a new experiment. This section is intended as an overview, while `Section: Preprocessing Tools <prepost_tools.rst>`_ covers how to generate the grid and mapping files. For a full description of the files in each component, we recommend consulting the individual model documentations. 


.. note:: 

   If the model run involves many restart runs, keep the input files (for the running phase) which are invariant across the runs in a separate directory and make symbolic links in the run directory as required.


.. _sec:inpR:

ROMS Input Files
================


The ROMS model requires the following files: 



1. Grid File
----------------

ROMS grid file (netCDF) contains all the grid related information needed
by ROMS model. In addition model grid cell latitude and logitude, grid
file contains several parameters like grid cell area, model bathymetry
and land-sea mask.

.. _sec:bryR:

2. Boundary Condition File
--------------------------------

For open lateral boundaries, ROMS require time varying data for
temperature, salinity, u and v velocities and sea surface height. This
is provided through the boundary condition netCDF file.

.. _sec:iniR:

3. Initial Condition or Restart Files
------------------------------------------------

For the very first run, ROMS need initial condition file with full 3D
model state (temperature, salinity, u nad v velocities and sea surface
height). For a restart run, this is provided through model generated
restart files.

.. _sec:nudgR:

4. 3D Nudging File
--------------------------------

ROMS can be configured with 3-dimentional nudging to specified data to
make sure model is not drifting too far away from the specified data
(typically from ocean reanalysis products). The data can be climatology
or inter-annually varying depending on the application. Nudging can be
along few grid points along the open boundaries or throughout the entire
ROMS domain (like in the Gulf of Mexico test case). Irrespective of this
choice, the 3D nudging data should always be for the entire 3D model
domain (a ROMS requirement) and time varying.



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



.. _sec:inpW:

WRF Input Files
===============


.. _sec:inpWrun:

Grid, Initial and Boundary Conditions
------------------------------------------

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



WRF Static Geographic Datasets
------------------------------------------   

For WRF there are serveral table files in addition to input netCDF and
namelist files. All files in a WRF directory
(R-CESM-1.0.0/models/atm/wrf/3.5.1/WRFV3/run/*) are linked to run
directory as table files and no check has been done to see which all of
these files are mandatory for running R-CESM and which all reduntant. A
list of table files are given in Table
`[tab:inpWtable] <#tab:inpWtable>`__.

.. table:: List of WRF table input files.

   +-----+----------------------------------+-----+-------------------------+
   |  #  | Filename                         |  #  | Filename                |
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



.. _sec:inpdocn:

XROMS/ Data Ocean files 
===============================

1. Mapping Weight Files
----------------------------

Coupled model components can have different resolutions. R-CESM requires
precomputed interpolation weights to map surface quantities between
different coupled model components. Interpolation options like bilinear
and averaging options like area-average are available with the ESMF
tool.

For a detailed discussion on mapping weight files and how to make them,
please see Section 5.2 in :raw-latex:`\citet{montuoro17}`.



2. xroms_sstice.nc
--------------------

R-CESM need data for SST and ice over entire domain. With xROMS set up
(Section `[sec:frxroms] <#sec:frxroms>`__), user has to provide an xROMS
file with SST and ice for the entire xROMS domain. SST for the bigger
domain is typycally available in WRF lower boundary input files. Current
test cases use ice as 0 everywhere.

A simple approach is to use matlab to interpolate WRF SST onto xROMS
grid and then write the interpolated SST to a proper xROMS SST netCDF
file (use the file from Gulf of Mexico test case as a reference).


   +------+-------------------------+--------------+------------------------------------------+-----------------+
   | Sl.  |             Filename    | File Type   |        Purpose                            |   Source        |
   +======+=========================+=============+===========================================+=================+
   |                          X-ROMS (Data Ocean)                                                               |
   +------+-------------------------+-------------+-------------------------------------------+-----------------+
   | 1    |   gom_lr_docn_grd.nc    | netCDF      |            Domain file                    |     User        |
   +------+-------------------------+-------------+-------------------------------------------+-----------------+
   | 2    | map_a2o_aave.nc         | netCDF      | Area-averaged mapping file from atm2ocn   |     User        |
   +------+-------------------------+-------------+-------------------------------------------+-----------------+
   | 3    | map_a2o_blin.nc         | netCDF      | Bi-linear interp mapping file from atm2ocn |     User        |
   +------+-------------------------+-------------+-------------------------------------------+-----------------+
   | 4    | map_o2a_aave.nc         | netCDF      | Area-averaged mapping file from ocn2atm   |     User        |
   +------+-------------------------+-------------+-------------------------------------------+-----------------+
   | 5    | seq_maps.rc             |    txt      | Specifies to above mapping files          |
   +------+-------------------------+-------------+-------------------------------------------+-----------------+
   | 6    | *_xroms_sstice_*_solo.nc | netCDF     | SST, ICE forcing & land-sea               |     User        |
   +------+-------------------------+-------------+-------------------------------------------+-----------------+
   | 7    | docn_in                 | txt     |     DOCN stream-independent namelist file               |     User        |
   +------+-------------------------+-------------+-------------------------------------------+-----------------+
   | 8    | docn_ocn_in             | txt     |     DOCN stream-dependent namelist file       |     User        |
   +------+-------------------------+-------------+-------------------------------------------+-----------------+
   | 8    | docn.streams.txt.prescribed | txt     |  Settings required for running DOCN with prescribed SST and ice-coverage |     User        |
   +------+-------------------------+-------------+-------------------------------------------+-----------------+


.. _sec:inplnd:

CLM input files
===============================

1. Mapping Weight Files
----------------------------

Coupled model components can have different resolutions. R-CESM requires
precomputed interpolation weights to map surface quantities between
different coupled model components. Interpolation options like bilinear
and averaging options like area-average are available with the ESMF
tool.

For a detailed discussion on mapping weight files and how to make them,
please see Section 5.2 in :raw-latex:`\citet{montuoro17}`.



2. xroms_sstice.nc
--------------------

R-CESM need data for SST and ice over entire domain. With xROMS set up
(Section `[sec:frxroms] <#sec:frxroms>`__), user has to provide an xROMS
file with SST and ice for the entire xROMS domain. SST for the bigger
domain is typycally available in WRF lower boundary input files. Current
test cases use ice as 0 everywhere.

A simple approach is to use matlab to interpolate WRF SST onto xROMS
grid and then write the interpolated SST to a proper xROMS SST netCDF
file (use the file from Gulf of Mexico test case as a reference).


.. _sec:inpC:

R-CESM/Coupler Input Files
=========================



Running Phase
-------------

For R-CESM and its coupler, there are several input files which are
listed in Table `[tab:inpC] <#tab:inpC>`__. Please provide all of these
files even if some of the component models (like ice) are not used. All
files which do not belong exclusively to either ROMS or WRF is included
in this category. The acronym "IO" implies input-output. (Some words are
used interchangably: ocean/ROMS, atmosphere/WRF, data ocean/xroms in
Table `[tab:inpC] <#tab:inpC>`__ and this will be corrected in the
future.)

.. table:: List of R-CESM specific input netCDF and namelist files. Item
24 is only required for restart runs.



   +------+-------------------------+--------------+------------------------------------------+-----------------+
   | Sl.  |             Filename    | File Type   |        Purpose                            |   Source        |
   +======+=========================+=============+===========================================+=================+
   | 7    | drv_in                  | txt      | CPL namelist file  general options, time manager options, pe layout, timing output, and parallel IO settings | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 7    | ocn_in                  | txt      | CPL namelist file  general options, time manager options, pe layout, timing output, and parallel IO settings | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 7    | lnd_in                  | txt      | CPL namelist file  general options, time manager options, pe layout, timing output, and parallel IO settings | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 7    | ice_in                  | txt      | CPL namelist file  general options, time manager options, pe layout, timing output, and parallel IO settings | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 8    |  cpl_modelio.nml        | txt      | sets the filename for the primary standard output file | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 8    |  ocn_modelio.nml        | txt      | sets the filename for the primary standard output file | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 8    |  atm_modelio.nml        | txt      | sets the filename for the primary standard output file | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 8    |  lnd_modelio.nml        | txt      | sets the filename for the primary standard output file | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 8    |  ice_modelio.nml        | txt      | sets the filename for the primary standard output file | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 8    |  glc_modelio.nml        | txt      | sets the filename for the primary standard output file | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 8    |  rpointer.ocn           | txt      | ocean restart pointer | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 8    |  rpointer.roms           | txt      | ocean restart pointer | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 8    |  rpointer.atm           | txt      | ocean restart pointer | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 8    |  rpointer.drv           | txt      | ocean restart pointer | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+
   | 8    |  rpointer.docn           | txt      | ocean restart pointer | Code | 
   +------+-------------------------+-------------+-----------------------------------------------------------------------------------------------------------+------+


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



.. _sec:map:




Other Input Files
=================

Other files required by R-CESM during running phase are listed in Table
`[tab:inpO] <#tab:inpO>`__.

.. table:: List ofother machine dependent input files, including R-CESM
executable.

   +-----+---------------+-----------+------+---------------------+
   | Sl. | Filename      | Component | File | File                |
   +-----+---------------+-----------+------+---------------------+
   | No. |               | Model     | Type | Description         |
   +-----+---------------+-----------+------+---------------------+
   | 1   | R-CESM         | R-CESM     | exe  | R-CESM executable    |
   +-----+---------------+-----------+------+---------------------+
   | 2   | run_R-CESM.job | R-CESM     | txt  | job submission file |
   +-----+---------------+-----------+------+---------------------+

.
.. _sec:output:

All Output Files
================

Complete list of output files from a R-CESM run is provided in Table
`[tab:outC] <#tab:outC>`__.

.. table:: List of all output files from a R-CESM run.


   +------+-------------------------+--------------+------------------------------------------+
   | Sl.  |             Filename    | File Type   |        Purpose                            |
   +======+=========================+=============+===========================================+
   | 1    |   *.log                 | txt         |      Log files from each component        |
   +------+-------------------------+-------------+-------------------------------------------+
   | 2    | rsl.error.*              | txt      |     WRF std error                           |
   +------+-------------------------+-------------+-------------------------------------------+
   | 2    | rsl.out.*               | txt         |     WRF std error                         |
   +------+-------------------------+-------------+-------------------------------------------+
   | 2    | rpointer.*              | txt      |    restart file pointers                     |
   +------+-------------------------+-------------+-------------------------------------------+
   | 2    | <case>.atm.hi.<time>.nc | netCDF      |      WRF output file                      |
   +------+-------------------------+-------------+-------------------------------------------+
   | 2    | <case>.ocn.hi.<time>.nc | netCDF      |      ROMS output file                     |
   +------+-------------------------+-------------+-------------------------------------------+
   | 2    | <case>.docn.rs1.<time>.nc | netCDF      |      DOCN restart file                  |
   +------+-------------------------+-------------+-------------------------------------------+
   | 2    | <case>.clm2.h0.<time>.nc | netCDF      |     CLM 4.0 monthly output file          |
   +------+-------------------------+-------------+-------------------------------------------+
   | 2    | <case>.<comp>.r.<time>.nc | netCDF      |     Restart files from each component   |
   +------+-------------------------+-------------+-------------------------------------------+
   | 2    | <case>.clm2.h0.<time>.nc | netCDF      |     CLM 4.0 monthly output file          |
   +------+-------------------------+-------------+-------------------------------------------+
   | 2    | <case>.clm2.rh0.<time>.nc | netCDF      |     CLM 4.0 restart file                |
   +------+-------------------------+-------------+-------------------------------------------+

   