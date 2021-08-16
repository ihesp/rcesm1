.. _xroms:

3. Generating the X-ROMS (DOCN) grid, domain and mapping files
================================================================

Date: May/22/2021
Author: Jaison Kurian and Abishek Gopal




-----------------------------
Generating the X-ROMS grid
-----------------------------


The ROMS domain in R-CESM is designed to be slightly smaller than that of WRF so that the WRF boundary artifacts (due to a dynamic inconsistency between WRF’s interior solution and prescribed lateral boundary conditions) do not affect the actively coupled ROMS region. To provide the SST for the gap-region between ROMS and WRF domains, a new X-ROMS (extended-ROMS) horizontal-grid, which covers the WRF domain with matching grid points with ROMS grid over the overlapping region, has been used. The X-ROMS blends SST from ROMS with that provided from data (on X-ROMS grid, from the dataset used for ROMS initial and boundary conditions) before passing it to the coupler.


The coupler only "sees" the data ocean DOCN (X-ROMS), atmosphere (WRF) and land (CLM) grids, and not the actual ROMS grid. So, the DOCN grid needs to adhere to a set of requirements. 

- It should match the extents of the WRF grid exactly, or can be slightly bigger than it on all four sides.
- Over the ROMS region, the DOCN grid must have exactly the same lat and lon values as that of ROMS grid.
- If you plan to use the WRF surface flux scheme in R-CESM, it is recommended to have the ROMS and X-ROMS land-sea mask values match closely with the WRF grid.
- If you plan to use the CESM surface flux scheme, the CLM grid is generated after WRF and ROMS, and it do not care about WRF grid. So, the ROMS grid can have a land-sea mask different from that of WRF.

Steps to create the X-ROMS grid
--------------------------------
       - Use ROMS grid generation tools to make a ROMS grid with features listed above.
       - Make sure the xROMS and ROMS grid points match exactly over the ROMS domain.
       - Edit the land-sea mask if needed.
       - Use droms2dom.py to convert the ROMS type grid file to proper xROMS/DATA ocean grid/domain file.



With the TAMU tools, these are the steps together

1. use grid_rcesm_docn_roms_fmt.m (along with func_rcesm_docn_roms_fmt.m) to make gom_lr_docn_grd_raw.nc
       - This is a quick way to get lat and lon for DOCN domain.
       - Mask may not be correct, which is corrected in the next two steps.

2. use write_wrf_mask_docn.jnl to get gom_lr_wrf_mask_on_docn_grd.nc

3. use grid_edit_coastline_rcesm_docn.m to edit coastline & create
       gom_lr_docn_grd_edcoastline.nc
         - Since this use matlab GUI, use it on a local machine instead of Grace.
         - Transfer following files to the local machine before running the script:
                gom_lr_201001_edcoast.nc       : ROMS grid file
                gom_lr_docn_grd_raw.nc         : DOCN grid file (raw)
                gom_lr_wrf_mask_on_docn_grd.nc : WRF mask interpolated on to DOCN grid.
         - Please note that you need to have the ROMS matlab tool package Roms_Tools for this step.
         - Also a need a coastline dataset (use fine resolution coastline dataset if the model
               resolution is high). For current grid Atl_WCL.mat will be okay.

4. use droms2dom.py to make DOCN domain file gch09_docn_domain.nc

  
         ml Anaconda3/2020.07
         source activate /scratch/group/ihesp/shared/conda/envs/py2_utils

.. code-block:: console

         ./droms2dom.py gom_lr_docn_grd_edcoastline.nc gom_lr_docn_grd.nc

This yields the DOCN domain file: gom_lr_docn_grd.nc



-----------------------------
xROMS SST files
-----------------------------


- xROMS is supposed to provide both SST and ICE data. ICE is typically set to 0. Data is needed for the entire R-CESM run period. Please make sure the NetCDF time units for this file should refer the time to "00" hours (otherwise the model will not run, some silly time stuff).
- With WRF surface flux scheme and a land-sea mask similar to WRF grid, there will be lakes over land and those locations needs SST data. So do the following:
- Interpolate the original ocean Re-Analysis data (eg. COPERNICUS/SODA/HYCOM) to the xROMS grid. Mask the data over any possible land-points.
- Interpolate the WRF SST from WRF input files to xROMS grid. Mask data over xROMS ocean points.
- Now combine data from above steps together.
- This method will work only if you make the time-frequency for the ocean data from Reanalysis and WRF SST data from WRF input files the same.
- With CESM surface flux scheme, the land-sea mask do not depend on WRF grid. So in this case you can just use the ocean Reanalysis data to create the xROMS SST field.



STEPS:

1. Use write_xroms_ocn_sst.jnl to get gom_lr_xroms_sstice_201001_ocn.nc for 1 month.

2. For multiple months, please edit write_xroms_ocn_sst.jnl first and then edit
write_xroms_ocn_sst_master.jnl as needed and then run the master file.

3. Then use write_xroms_solo_sst.m to get the SST data in a format that R-CESM needs.

- to process just one file, comment out the function statement (1st line) in
    write_xroms_solo_sst.m and provide proper values for t1_adj, month and year in
    "USER INPUT" section.
- For multiple month/files case, use write_xroms_solo_sst.m as a function (comment
    out all lines in "USER INPUT" Section) and use write_xroms_solo_sst_master.m.
- In both cases, the data paths should be provided in write_xroms_solo_sst.m, right
    after the "USER INPUT" section.
- PLease note that t1_adj (to make the time for very first record of the very first
    file to start from the model initial condition time (typically 00 hrs) from the
    typical Copernicus data time (12 hrs)) need to be used only once for a case.
- The example here produces gom_lr_xroms_sstice_201001_solo.nc. As you can see, the
first time point is 01-JAN-2010 00 and 2nd time point is  02-JAN-2010 12.
                                                                                                                                                                                               97,1          95%

