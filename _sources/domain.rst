.. _xroms:

3. Generating the domain and mapping files
=============================================

Date: May/22/2021
Author: Jaison Kurian and Abishek Gopal



-----------------------------
Generating the mapping files
-----------------------------

Mapping files are required if the ROMS and WRF grid are not exactly same or you are using xROMS. These files store the mapping weights from one type of grid to the other.

1. Convert the WRF and X-ROMS grid files to the SCRIP format domain files so that ESMF can read them. 

.. code-block:: bash

    ./create_map.py -g gom_lr_docn_grd.nc -o scrip_docn_grd.nc -t Data_Ocean_grid_in_scrip_format --var_lat yc --var_lon xc --var_mask mask
    ./create_map.py -g wrfinput_d01.nc -o scrip_atm_grd.nc -t WRF_grid_in_scrip_format --var_lat XLAT --var_lon XLONG --var_mask NONE

 
2. Use the ESMF library to generate the mapping files.


# Conservative remapping (fluxes)

.. code-block:: console


    mpirun -n 8 ESMF_RegridWeightGen --ignore_unmapped -m conserve     -w map_o2a_aave.nc -s scrip_docn_grd.nc -d scrip_atm_grd.nc --src_regional --dst_regional
    mpirun -n 8 ESMF_RegridWeightGen --ignore_unmapped -m conserve     -w map_a2o_aave.nc -s scrip_atm_grd.nc -d scrip_docn_grd.nc --src_regional --dst_regional


# Bilinear remapping (wind stress)

.. code-block:: console


    mpirun -n 8 ESMF_RegridWeightGen --ignore_unmapped -m bilinear     -w map_a2o_blin.nc -s scrip_atm_grd.nc -d scrip_docn_grd.nc --src_regional --dst_regional
                                  
  
Then use run_esmf.pbs to create the mapping files using ESMF library.





-----------------------------
Generating the domain files
-----------------------------



Create the domain files using the gen_domain tool. This tool is available from many places, and it is not entirely necessary to use the one from an older version of CLM. The version available through CIME in a full checkout of RCESM should work. The tool is located in: ``cime/tools/mapping/gen_domain_files`` .

The INSTALL file in the gen_domain_files/ directory contains details on
how to build the gen_domain executable. After you have built it, the README
in the same directory contains details on how to use the tool. The basic usage
is:
$ ./gen_domain -m ../gen_mapping_files/map_ocnname_TO_lndname_aave.yymmdd.nc \
-o ocnname -l lndname
$ ./gen_domain -m ../gen_mapping_files/map_ocnname_TO_atmname_aave.yymmdd.nc \
-o ocnname -l atmname



Step 2-b: Run the gen_domain tool using the command:
``$ gen_domain -m <input mapping file name> -o <ocean grid name> -l <land grid name>``
This will produce three domain files, one for the land and atmosphere, one for the ocean and sea ice, and one for the ocean on the atmospheric grid.


These commands will generate the following domain files:
domain.lnd.lndname_ocnname.yymmdd.nc
domain.ocn.lndname_ocnname.yymmdd.nc
domain.lnd.atmname_ocnname.yymmdd.nc
domain.ocn.atmname_ocnname.yymmdd.nc
domain.ocn.ocnname.yymmdd.nc
Notes:
a. If you are running with the atmosphere and land components on the
same grid, you only need to execute gen_domain once.
b. The input atmosphere grid is assumed to be unmasked (global). Land
cells whose fraction is zero will have land mask = 0.
c. If the ocean and land grids are identical then the mapping file will simply
be unity and the land fraction will be one minus the ocean fraction.

