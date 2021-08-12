.. _prepost_tools:

============================================
Creating a new R-CESM application/domain
============================================

In this tutorial, we will walk through the steps involved in adding the low-resolution Gulf of Mexico domain to R-CESM. In order to add an application to focus on a new region domain, we would first need to create the necessary grids and mapping files, then change the R-CESM configuration files to support the new domain. We would also need to modify the namelist options for the components in use. 





Overview of steps involved
=============================================

1. Determine the extents of the atmosphere (WRF) and ocean (ROMS) model domains.

The ROMS domain needs to be slighly smaller than the WRF domain, on all side by 0.2-1\degrees, or a few grid points. This is required to reduce the impact of any boundary effects from the WRF model affecting the ROMS interior solution.

2. Generate the grid, initial and boundary conditions for the WRF model using the WRF pre-processing tools (WPS). Then generate the domain file for the atmosphere. Follow the steps in the `WRF preprocessing tutorial <wrf_grid.rst>`__

3. Generate the grid, intial and boundary conditions, and nudging files for the ROMS model using your preferred tool. Follow the steps outlined in the `ROMS preprocessing tutorial <roms_grid.rst>`__.
.. Edit ROMS grid coastline to match that from WRF model grid. This is easier than otherway around, since land points on WRF grid need lot of additional information like vegetation type, land cover etc.

4. If the application will be configured to use XROMS, then create the domain file for the data ocean from the ROMS grid. XROMS is also expected to provide SST and ICE cover (if applicable) data to WRF. The procedures for XROMS are detailed in `the XROMS guide <xroms.rst>`__.
5. Generate the domain files for the atmosphere and ocean grids in the SCRIP format, and then use the ESMF tool to create mapping files to interpolate between the two domains.
6. Create the domain files using the ``gen_domain`` tool.
7. Create mapping files from the input data sets to the land grid, using the ``mkmapdata`` tool in CLM. Then use the ``mksurfdata_map`` to create the surface datasets required to run CLM. These steps are detailed `here <clm_grid.rst>`__.
8. Compile CRESM with proper options for the new domain and copy executable to new run directory.
9. Copy input files (like namelist, \*\_in, job/pbs etc. files, not the netCDF files) for CRESM, WRF and ROMS from Gulf of Mexico test case to the new run directory and edit it with proper optios/fields for the new grid/domain.



-----------------
Next Steps
-----------------

Next Steps
=============================================

.. toctree::
  :maxdepth: 2

  wrf_grid.rst
  roms_grid.rst
  xroms.rst
  clm_grid.rst
  edit_config.rst