.. _roms_grid:

2. Generating the Ocean input files
====================================

Date: Nov/24/2020
Author: Jaison Kurian (jaisonk@tamu.edu)

Copyright: This tool is shared within iHESP group for internal use. 


R-CESM uses ROMS as the ocean compotnent and WRF as the atmospheric component. Since WRF and ROMS are coupled within R-CESM, it is ideal to have similar land-sea mask in both models. If ROMS and WRF have identical resolution and overlapping grid points, the land-sea mask can be matched exactly and if the resolutions are different, then the matching will be only possible to some level.

WRF have a dedicated pre-processing system (WPS) to make WRF grid files and decides land sea mask (and so the coastline too) based on the resolution of datasets specified during grid creation. Once a grid is created, WRF needs several variables on the land-grid point (like details of soil, vegetation etc.). So, it is not advisible to edit WRF-land sea mask after creating WRF grid. However, the datasets and methods to define bathymetry and land-sea mask in ROMS are different. This is where the relevance of this tool comes in. Also note that this method will allow to have matching grid points between WRF and ROMS, which is not required in R-CESM but very convenient for budgetting type of analysis (for example, one can define box regions with same coverage in WRF and ROMS).

WRF lateral boudnary conditions are applied over few grid points close to the boundary and there will be some inconsistency between the interios WRF solution and the forcing provided at the lateral boundaries. In order to avoid such inconsistencies forcing or propagating to ROMS solution in R-CESM, the ROMS model domain in R-CESM is always designed to be slightly smaller than (about 1 degree or 10-15 WRF grid points) that of WRF. This is described as the "buffer points" in this documentation.

:test: hello

1) Get ROMS lat-lon grid & land-sea mask

 The main script is grid_wrf2roms.m.  The "USER INPUT" part towards the beginning is the only place a user is supposed to make changes. See the documentation in this script for step-by-step details but some general concepts are explained here.
 
  --> edit the user input part of grid_wrf2roms.m as need and run it

 Step.1: Create WRF grid using standard WRF pre-processing System (WPS).
 Step.2: Using tools like Matlab, find the "buffer points".
               bufr_pts = [41 13 216 43] ; % [S E N W] 
           leave  41 WRF grid points at the South side
           leave  13 WRF grid points at the East side
           leave 216 WRF grid points at the North side
           leave  43 WRF grid points at the West side
 Step.3: Decide a dummy bathymetry dataset, just to fill variable "h" in ROMS grid.
           Proper bathymetry data will be written to the grid file at a later stage.
 Step.4: Resolution Factor --> specify ROMS resolution based on WRF 
             1 --> ROMS_res   = WRF_res --> same resolution ROMS (as that of WRF)
             2 --> ROMS_res*2 = WRF_res --> high-resolution ROMS 
            -2 --> ROMS_res/2 = WRF_res --> coarse-resolutinn ROMS
 Step.5: Dist_method
            - there are sevaral different methods/approximations to compute distance on the Earth surface. "geodesic" method is recommented (which seems to match well with the distance estimates within WRF tools/grid).
            
 Functions/tables needed:
         netcdf tool box: Complete tool box - tool box to handle netcdf files
      x  func_wrf2roms_latlon.m  - compute ROMS grid's lat and lon from that of WRF 
      x  func_create_grid_file.m - create ROMS grid netcdf file
         func_gcdist.m          - distance on earth surface 
         func_spherical_dist.m  - distance on earth surface
         func_geodesic_dist.m   - distance on earth surface 
      x  func_roms_angle.m      - compute ROMS angle if it is a curvilinear grid.
         func_lon_chk.m         - to check the longitude orientation of ROMS grid and bathymetry data.
         func_flip_lon.m        - to flip the longitude orientation of data to match that of ROMS grid.
      x  roms_table             - a handy table with ROMS grid file variables
         Roms Tools package:
              rho2uvp
              uvp_mask

4) Get Bathymetry
-----------------
  - Add bathymetry data to the ROMS grid file from (3) above. This is done as a dedicated step for more control over the process.
  - Use Alexandaer Schpetkin's Fortran tools to sample bathymetry data from the source grid to ROMS grid.
  - Jaison will udpate this part of the documentation later.

5) Edit coastline
-----------------
  - If ROMS and WRF are at same resolution then the land-sea mask interpolated on to ROMS grid in step (3) will be sufficient. If not, then read on.
  - Use a ROMS land-sea mask editing GUI for fine tuning ROMS land-sea mask
  - An example is given in the grid_edit_coastline.m. The actual program used is editmask.m which is a part of ROMS pre-processing toolkit.
  - Jaison will provide documentation for this step later.

5) Smooth Bathymetry
--------------------
  - ROMS is a sigma coordinate model and prone to pressure gradient errors with rough bathymetry.
  - It is customary to smooth the bathymetry to limit rx0 <= 0.25 and rx1 <= AB. AB can be anywhere from 7 to 15 depending on how complex/rough is the bathymetry. Having rx1=7 means the model will run smoothly without blowing up events. But sometime it smooth bathymetry too much and final bathmetry may not be realistic. So, choose rx1 maximum as 12 or 15 and do some test runs. 
 - Jaison will provide the tools and documentation for this step later.









https://www.myroms.org/wiki/easygrid