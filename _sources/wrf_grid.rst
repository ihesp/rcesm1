.. _wrf_grid:


1. Generating the Atmospheric input files
==========================================


------------

We summarize the essential steps required to generate the initial and boundary conditions for the WRF model run. For a more comprehensive tutorial, please follow along with 

`WRF/WPS Online Tutorial<https://www2.mmm.ucar.edu/wrf/OnLineTutorial/compilation_tutorial.php>`_




1. Obtain and build WRF Preprocessing System (WPS) source
----------------------------------------------------------

Download version 3.5.1 of the code.


2. WRF Preprocessing System
----------------------------



sidebar:: Sidebar Title
    :subtitle: Optional Sidebar Subtitle

    Subsequent indented lines comprise
    the body of the sidebar, and are
    interpreted as body elements.

Step 1: Download and build WRF source code


note:: This is a note admonition.
    This is the second line of the first paragraph.

    - The note contains all indented body elements
      following.
    - It includes this bullet list.


  Step 2: Download and build WPS source code
dfsdfsd
sdfsdfsdfsdfd
  



Step 3: Download static geographic data and real time GRIB data

Step 5: Edit namelist.wps in WPS directory

Step 6: Run geogrid.exe

Step 7: Run ungrib.exe

Step 8: Run metgrid.exe

Step 9: Generate IC and BC files



.. code-block:: console
    ./geogrid.exe
    cat *.tar | tar xvf - -i
    ln -sf ungrib/Variable_Tables/Vtable.GFS Vtable



.. code-block:: console

    cat *.tar | tar xvf - -i
    ln -sf ungrib/Variable_Tables/Vtable.GFS Vtable




Step:1 - Activate cesm virtual environment

.. code-block:: console
        ./link_grib.csh path_to_data


Step:2 - Create your postprocessing instance

.. code-block:: console

    ./ungrib.exe


.. code-block:: console

    ncl util/plotfmt_nc.ncl GFS:2016-12-28_18
    util/rd_intermediate GFS:2016-12-28_18

Note: depending on the duration of the simulation, you may
also need to increase the wallclock time.

Step:4.2 - Submit the job:

.. code-block:: console

    ./metgrid.exe

Step:4.3 - Check the latest log file to see if averaging was
completed successfully: [your-case-directory]/logs/

Step:5 - Generate the diagnostics

Step:5.1: - Enter your project id in ocn_diagnostics file by
replacing "None" in the following line with your project id:

.. code-block:: console

    ln -sf ../../../WPS_788/met_em.d01.201* .

modify namelist.inp
https://esrl.noaa.gov/gsd/wrfportal/namelist_input_options.html

.. code-block:: console

    ./real.exe


Replacing fields in wrfinput

https://wiki.uio.no/mn/geo/geoit/index.php/Replacing_fields_in_wrfinput


 -----------------------------------------------------------------------------


Common issues and their fixes
------------------------------

- **Issue #1:**

.. code-block:: console

 Domain  1: Current date being processed: 2016-01-01_00:00:00.0000, which is loop #   1 out of 1465
 configflags%julyr, %julday, %gmt:        2016           1   0.00000000
  metgrid input_wrf.F first_date_input = 2016-01-01_00:00:00
  metgrid input_wrf.F first_date_nml = 2016-01-01_00:00:00
 -------------- FATAL CALLED ---------------
 FATAL CALLED FROM FILE:  <stdin>  LINE:     851
  input_wrf.F: SIZE MISMATCH:  namelist ide,jde,num_metgrid_levels= 90  75  32 ; input data ide,jde,num_metgrid_levels=90 75 27
 -------------------------------------------
 -------------- FATAL CALLED ---------------
 FATAL CALLED FROM FILE:  <stdin>  LINE:     851
  input_wrf.F: SIZE MISMATCH:  namelist ide,jde,num_metgrid_levels= 90  75  32 ; input data ide,jde,num_metgrid_levels=90 75 27
 -------------------------------------------
  STOP wrf_abort


**Fix:** num_metgrid_levels in namelist.input



- **Issue #2:**
.. code-block:: console

  -------------- FATAL CALLED ---------------
   FATAL CALLED FROM FILE:  <stdin>  LINE:     851
    input_wrf.F: SIZE MISMATCH:  namelist ide,jde,num_metgrid_levels= 90  75  32; input data ide,jde,num_metgrid_levels=90  75  27
   -------------------------------------------
   -------------- FATAL CALLED ---------------
   FATAL CALLED FROM FILE:  <stdin>  LINE:     851
    input_wrf.F: SIZE MISMATCH:  namelist ide,jde,num_metgrid_levels= 90  75  32; input data ide,jde,num_metgrid_levels=90  75  27
   -------------------------------------------
  Note: The following floating-point exceptions are signalling: IEEE_OVERFLOW_FLAG
  STOP wrf_abort


**Fix:**
Upgrades to GFS on May 11, 2016. Moved from 27 vertical levels to 32.

https://www.perspectaweather.com/blog/2016/5/19/noaa-upgrades-its-primary-computer-forecast-model
https://www.enviroware.com/how-to-deal-with-GFS-upgrades-when-running-WRF/

https://forum.wrfforum.com/viewtopic.php?f=6&t=9761
You can request customizable data and in the "Vertical Level(s):" you can select all fields, except:
- all available ( obviously)
- isobaric surface: 7 mbar
- isobaric surface: 5 mbar
- isobaric surface: 3 mbar
- isobaric surface: 1 mbar

And submit a request.

This way, you will have your entire data base with 27 vertical levels.


- **Issue #3:**
.. code-block:: console

  *************************************
  d01 2014-05-12_00:00:00 *** Initializing nest domain # 2 from an input file. ***
  -------------- FATAL CALLED ---------------
  FATAL CALLED FROM FILE:  <stdin>  LINE:      70
  program wrf: error opening wrfinput_d02 for reading ierr=          -8
  -------------------------------------------

**Fix:** ./wrf.exe called before ./real.exe was called
