.. _prepost_tools:

============================================
Creating a new R-CESM application/domain
============================================

In order to add an application to focus on a new region domain, we would first need to create the necessary grids and mapping files, then change the R-CESM configuration files to support the new domain. We would also need to modify the namelist options for the components in use. 

In this tutorial, we will walk through the steps involved in adding the low-resolution Gulf of Mexico domain to R-CESM. 



Steps inolved in creating a new domain
=============================================

1. Determine the ROMS and WRF model domains. 
      Make ROMS domain slighly (0.2-1\degrees, or few grid points) smaller (all four sides) than the WRF domain, in order to reduce the impact of WRF boundary effects affecting ROMS interior solution.

2. Generate the WRF atmosphere grid, initial and boundary conditions using WRF pre-processing tools. Then generate the domain file for the atmosphere. Follow the steps `here <wrf_grid.rst>`__

3. Generate the ROMS ocean grid, and the intial, boundary and nudging files. Detailed in `here <roms_grid.rst>`__.
.. Edit ROMS grid coastline to match that from WRF model grid. This is easier than otherway around, since land points on WRF grid need lot of additional information like vegetation type, land cover etc.

4. If the application will be configured to use XROMS, then create the domain file for the data ocean. 
5. Generate the SCRIP mapping files for the domains.
6. Create the domain files using the ``gen_domain`` tool.
7. Create mapping files from the input data sets to the land grid, using the ``mkmapdata`` tool in CLM. Then use the ``mksurfdata_map`` to create the surface datasets required to run CLM. These steps are detailed `here <clm_grid.rst>`__.
8. Compile CRESM with proper options for the new domain and copy executable to new run directory.
9. Copy input files (like namelist, \*\_in, job/pbs etc. files, not the netCDF files) for CRESM, WRF and ROMS from Gulf of Mexico test case to the new run directory and edit it with proper optios/fields for the new grid/domain.



Next Steps
=============================================

.. toctree::
  :maxdepth: 2

  wrf_grid.rst
  roms_grid.rst
  clm_grid.rst