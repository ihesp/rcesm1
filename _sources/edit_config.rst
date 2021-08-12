.. _edit_config:


5. Modifying the configuration files
==========================================


Adding the new grid to R-CESM
------------------------------

In order to add the newly-created domain to R-CESM, we need to make changes to the following configuration files. 


   - ``cime/config/cesm/config_grids.xml``  This configuration file needs to be edited in three sections. Under `<grids>`, we specify the grid name/alias and the name of the grids for each component. 
    

     .. code-block:: XML

        <model_grid alias="txlw27k_gom27k">
            <grid name="atm">txlw27k</grid>
            <grid name="lnd">txlw27k</grid>
            <grid name="ocnice">gom27k</grid>
        </model_grid>

   The actual definition of the grid/domain files occur further down in this file, under `<domains>`. We need to specify the grid dimensions, path and description for each domain. 
     
    .. code-block:: XML

        <domain name="txlw27k">
          <nx>89</nx>  <ny>79</ny>
          <file grid="atm|lnd">$DIN_LOC_ROOT/share/domains/domain.atmlnd.txlw27k_gom27k.210706.nc</file>
          <desc>27km Gulf of Mexico for Wrf, to pair with gom27k in xRoms</desc>
        </domain>
        <domain name="gom27k">
          <nx>273</nx>  <ny>242</ny>
          <file grid="ocnice">$DIN_LOC_ROOT/share/domains/gom_lr_docn_grd.210705.nc</file>
          <desc>27km Gulf of Mexico grid for xRoms</desc>
        </domain>


    The definition of the mapping files, required by the coupler to regrid between components, occurs under the `<gridmaps>` section. 

    .. code-block:: XML
    
        <gridmap atm_grid="txlw27k" ocn_grid="gom27k">
          <map name="ATM2OCN_FMAPNAME">atm/wrf/tx27k_g27x/map_a2o_aave.nc</map>
          <map name="ATM2OCN_SMAPNAME">atm/wrf/tx27k_g27x/map_a2o_aave.nc</map>
          <map name="ATM2OCN_VMAPNAME">atm/wrf/tx27k_g27x/map_a2o_aave.nc</map>
          <map name="OCN2ATM_FMAPNAME">atm/wrf/tx27k_g27x/map_o2a_aave.nc</map>
          <map name="OCN2ATM_SMAPNAME">atm/wrf/tx27k_g27x/map_o2a_aave.nc</map>
        </gridmap>


    .. note:: 

        For Contributors: Note that the CIME code base is maintained in a seperate repository from R-CESM, so changes to this file will need a seperate commit and pull request from any other changes to the code base.


    .. container:: custom

       This paragraph might be rendered in a custom way.
     

   - ``components/wrf/bld/wrf.buildnml.csh`` If adding a grid for the WRF atmospheric component, it is currently required to add an `if endif` clause in the  ``wrf.buildnml.csh`` file to include the following:

        - Copy or symlink the WRF grid and BC files (wrfbdy_d01, wrfinput_d01, wrflowinp_d01) from the `$DIN_LOC_ROOT` to the run directory.
        - Copy or symlink the static data files required to run WRF from the `$DIN_LOC_ROOT` to the run directory.
        - Copy or write the WRF namelist.input configuration file to to the run directory.


    We plan to modify this in the future. 

   

   - ``components/roms/bld/roms.buildnml.csh``  There are a few modifications to this file required if you have a ROMS grid:

         -  You need to add information about the grid size in OCN_NX and OCN_NY in the `if endif` clause

           .. code-block:: bash
                 else if ($OCN_GRID =~ gom27k) then
                    set OCN_NX = 208
                    set OCN_NY = 154
                 endif    

        - Copy or symlink the ROMS grid and boundary data from the `$DIN_LOC_ROOT` to the run directory.
        - Copy or symlink the X-ROMS(DOCN) `docn_ocn_in`, `docn_in` and `docn.streams.txt.prescribed` from the `$DIN_LOC_ROOT` to the run directory.


   - ``components/roms/Apps`` The ROMS header files that specify ROMS settings are based on the domain or grid name. You will
     need to create a new sub directory of this directory by the name of the new grid and create the necessary namelist
     and header files within it. Look at the directory ``components/roms/Apps/gom3x/`` as an example.


   - ``components/clm/bld/namelist_files/namelist_definition_clm4_0.xml`` and ``components/clm/bld/namelist_files/namelist_defaults_clm4_0.xml``  If your grid is applicable to the CLM (land) component, then these files need to be updated. You may need to
     specify a land surface start up file (``fsurdat``) for the grid, or decide to run with a "cold start". For
     more information on creating domain and land surface files see:
     http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x11573.html

     And for more information about CLM 4.0 namelists and start types see:
     http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x1230.html



and ``components/wrf/cime_config/config_pes.xml``  and ``components/roms/cime_config/config_pes.xml``  If your grid is applicable to the
     WRF atmospheric component, then these files need to be updated to include your grid and domain. The WRF namelist is built
     based on the name of the grid WRF is running with, so new grids will need to specify a new section in the wrf.buildnml.csh
     script. The config_pes.xml file specifies the PE layout that is best for a given grid. There is a default value that should
     work for most components, but adding a section for your new grid will give you flexibility to update this in the future.

     The config_pes.xml file specifies the PE layout that is best for a given grid. There is a default value that should
     work for most components, but adding a section for your new grid will give you flexibility to update this in the future.
     Note that it is not common for active atmospheric models to run on the same grid as an active ocean model, so it is very
     likely that your atmospheric grid will be named differently and specified differently in the ``config_grids.xml`` file
     from your ocean grid.



Creating a New Domain or Regional Set Up
------------------------------------------

Changing the domain, time, or "application" for RCESM requires answering a series of questions in order to
determine which files need to be changed or updated to support your new application. Eventually, your goal is
to have a *compset*, a *grid*, and the needed *namelist file* support to create a new experiment case from
which to run your simulation. Consider these questions to direct the changes needed:

1. Do you want to use a set of components that has already been defined and tested or a new set?

   (See the section above: `RCESM Components`_ ). If you want to use any of these configurations, then you can move on to the next question. If you want 

2. Do you want to use a grid resolution and domain that has already been tested or create a new one?

   The RCESM code has been tested and includes support for three different grid/domain sets. These are described in the section above: `RCESM Grids`_ . If one of these supported grids works for your new application, then you can move on to the next question. If you need to add a new domain or grid, there are several files that will need to be updated with grid information, and you will need several coupler domain and mapping files to support it.

   - ``cime/config/cesm/config_grids.xml``  This file needs to be edited in two places. Early in the file, grid names and the
     components to which they apply are specified. Later in the file, the grid domain size and links to the coupler domain file
     are specified. To see what edits need to be made, search the file for the grid "gom3" and consider how this grid was
     added to the ``config_grids.xml`` file. Note that the CIME code base is maintained in a seperate repository from the RCESM,
     so changes to this file will need a seperate commit and pull request from any other changes to the code base.

   - ``components/wrf/bld/wrf.buildnml.csh`` and ``components/wrf/cime_config/config_pes.xml``  If your grid is applicable to the
     WRF atmospheric component, then these files need to be updated to include your grid and domain. The WRF namelist is built
     based on the name of the grid WRF is running with, so new grids will need to specify a new section in the wrf.buildnml.csh
     script. The config_pes.xml file specifies the PE layout that is best for a given grid. There is a default value that should
     work for most components, but adding a section for your new grid will give you flexibility to update this in the future.

   - ``components/clm/bld/namelist_files/namelist_definition_clm4_0.xml`` and ``components/clm/bld/namelist_files/namelist_defaults_clm4_0.xml``  If your grid is applicable to the CLM (land) component, then these files need to be updated. You may need to
     specify a land surface start up file (``fsurdat``) for the grid, or decide to run with a "cold start". For
     more information on creating domain and land surface files see:
     http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x11573.html

     And for more information about CLM 4.0 namelists and start types see:
     http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x1230.html

   - ``components/roms/bld/roms.buildnml.csh`` and ``components/roms/cime_config/config_pes.xml``  Much like in WRF,  if your
     grid is applicable to the
     ROMS ocean component, then these files need to be updated to include your grid and domain. The ROMS namelist is built
     based on the name of the grid ROMS is running with, so new grids will need to specify a new section in the roms.buildnml.csh
     script. The config_pes.xml file specifies the PE layout that is best for a given grid. There is a default value that should
     work for most components, but adding a section for your new grid will give you flexibility to update this in the future.
     Note that it is not common for active atmospheric models to run on the same grid as an active ocean model, so it is very
     likely that your atmospheric grid will be named differently and specified differently in the ``config_grids.xml`` file
     from your ocean grid.

   - ``components/roms/Apps`` The ROMS header files that specify ROMS settings are based on the domain or grid name. You will
     need to create a new sub directory of this directory by the name of the new grid and create the necessary namelist
     and header files within it. Look at the directory ``components/roms/Apps/gom3x/`` as an example.

3. Do you want to change the time, date, or physical/parameterization options of an application?

   Both ROMS and WRF use CIME xml variables and namelist options to specify various parameters for their runs. To change the
   date, length of a run or change parameterizations, you will first need to add support for these changes to the appropriate
   namelist via the namelist generating scripts (``components/wrf/bld/wrf.buildnml.csh`` and/or
   ``components/roms/bld/roms.buildnml.csh``). You may also need to make changes to the ROMS header or namelist files in the
   ``components/roms/Apps`` directory. Once this is all done, you can create an experiment case for your application from which
   you can set up your runs. To learn more about creating a case, changing xml variables, and running the model, see the
   `CRESM Quick Start guide <https://ncar.github.io/TAMURegionalCESM/quickstart.html>`.