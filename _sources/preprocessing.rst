.. _prepost_tools:

============================================
Creating new grids and cases
============================================


The best method is to try the Gulf of Mexico test case first and then adapt
it (including cresm\_1.0.0 source code) to the new grids and application. 
The guidelines for this are provided in this section.


Changes to Input Files
======================

Please take a look at Chapter to know the basic details and
requirements of CRESM modeling framework. Based on the specific options 
selected for the application with CRESM, WRF and ROMS, prepare input files. 
Input files are discussed in detail in Chapter. Use the input 
files from the Gulf of Mexico test case as a reference. Tools and scripts to
create input netCDF files are provided in Chapter.

As with any models, all CRESM, WRF and ROMS files are model grid dependent.
Please make new input files for experiments with a new component model grid
(WRF or ROMS). Also note that the xroms files and mapping weight files 
(Section ) are also grid dependend and needs to be recreated 
with change in any component model grids.


Steps to Make New Model Grids and Input Files
=============================================

1. Determine the ROMS and WRF model domains. 
      Make ROMS domain slighly (0.2-1\degrees, or few grid points) smaller (all four sides) than the WRF domain, in order to reduce the impact of WRF boundary effects affecting ROMS interior solution.
2. Make WRF model grid using WRF tools. 
3. Make WRF initial and boundary conditions using WRF/WPS tools.
4. Make ROMS grid using ROMS tools. 
 Edit ROMS grid coastline to match that from WRF model grid. This is easier than otherway around, since land points on WRF grid need lot of additional information like vegetation type, land cover etc.
5. Make ROMS initial and boundary conditions using matlab/python based ROMS pre-processing tools.
6. If planning to use xroms, then create data ocean domain file.
7. Create mapping weight files.
8. Compile CRESM with proper options for the new domain and copy executable to new run directory.
9. Copy input files (like namelist, \*\_in, job/pbs etc. files, not the netCDF files) for CRESM, WRF and ROMS from Gulf of Mexico test case to the new run directory and edit it with proper optios/fields for the new grid/domain.
10. Make a test run for few days and make sure the results are as expected.
11. Now edit the input files for the long production run as required and proceed with production run.



Next Steps
=============================================

.. toctree::
  :maxdepth: 2

  wrf_grid.rst
  roms_grid.rst
  clm_grid.rst
   