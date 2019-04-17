.. _unfinished:

==============================================
Known Issues, Troubleshooting, and Future Work
==============================================

*This page last updated Jan 25, 2019*

Known Issues
------------

    Due to the experimental nature of the RCESM, not all of CESM's features have been implemented. Here are a few known issues that result in features missing from RCESM:

+ The short term archiving script does not currently work with RCESM.
+ The user_nl substituting scripts do not currently work in RCESM for WRF and ROMS. This needs the buildnml.csh scripts for each of these components to be updated to python scripts similar to other CESM build namelist scripts.
+ Usermods for WRF and ROMS are not implemented and have not been tested yet. This needs the buildlib.csh scripts for each of these components to be updated to python scripts similar to other CESM build scripts.
+ Not all of the CESM xml variables are properly routed into the WRF and ROMS configurations. In particular, WRF history file configuration is still done through the WRF namelist and not XML variables. Also, because the short term archiving script is not working correctly, the CIME ``$DSOUT`` variable does not behave correctly in this model. There are probably other CIME xml variables that are not correctly supported in this model, or WRF and ROMS configuration options that should be moved to CIME xml variables.
+ There may be WRF or ROMS build and namelist code that is specific to Cheyenne. So far, RCESM has been ported to three TAMU machines, so there may not be too much. The build and namelist processes for both of these models should be generalizd so the only code that needs to be ported is CIME.
+ The ROMS compset is hard-coded to only allow the ROMS%XROMS configuration. Need to get the %NULL compset configuration working again.
+ CESM testing has not been built for this model, so the create_test command does not currently work.
  

Troubleshooting
---------------

Create_newcase, Setup, and Submit Issues
========================================

Most of the errors produced by these scripts are self-explanitory. Use the "--help" option with any of these scripts to learn about more options provided, or the expected arguments. These scripts require python version 2.7 to run, and strange errors will occur if the version of python used to run the scripts is different. Make sure that the python module loaded by your machine via the ``cime/config/cesm/machines/config_machines.xml`` file is in the 2.7 series.

Case.build Errors
=================

There are a few places where the build steps can go wrong. The first step in building this model is building the component namelists. Errors about missing fields or incorrect configurations of namelists could show up in this step. For WRF and ROMS, the namelist generating scripts are in the ``components/wrf/bld/wrf.buildnml.csh`` (and likewise for ROMS) scripts. Any configurations issues should be addressed here. For other components, changes might need to be made to configuration xml files within that component (for example, ``components/clm/bld/namelist_files/namelist_definition_clm4_0.xml``) or by adding lines to the user_nl file for that component in the case directory.

Errors that occur while the model is building will show up in the build logs. These are written to the build directory, which is parallel to the run directory (check the xml variable ``$RUNDIR`` in the case directory). Some common problems with building are:
- Having an incomplete or incorrect Manage_externals checkout. See `getting the CRESM code <https://ncar.github.io/TAMURegionalCESM/downloading_cesm.html>`_ for more information about manage_externals.
- NetCDF errors can be caused by having a NetCDF library that is built with a different compiler or different compiler version than what you are using to build RCESM.
- Some build errors are the result of compiler bugs in older compilers. The PGI compiler is particularly difficult to build with. Consider trying a newer version of your compiler or a different FORTRAN compiler if strange build errors appear. 

Runtime Errors
==============

Errors that occur during the model run will be written to log files. If the simulation encounters an error, all log and output files will remain in the run directory. If the WRF component is running as the atmosphere, it produces two output files for each process, an rsl.out.XXXX file and an rsl.error.XXXX file (where XXXX is the process rank, ie. 0036). The standard output and standard error streams can be found in these files, which will remain in the run directory regardless of the success or failure of the model run. The XROMS ocean component will write to three different logs: data.ocn.log, ocn.log, and roms.ocn.log. Check each of these for errors in case of a runtime failure. However, most run-time errors send the results to the full CESM log (cesm.log). Some suggestions to help debug runtime errors:

- Make sure all of the required data are in the appropriate locations to be linked for the simulation. The location for the data input directory can be found in the ``$DIN_LOC_ROOT`` xml variable in the case directory. The paths to the required files can be found in the namelist building scripts for WRF (``wrf/bld/wrf.bldnamelist.csh``) and ROMS (``roms/bld/roms.bldnamelist.csh``). 
- Check that your ``$JOB_WALLCLOCK_TIME`` value is long enough for your entire simulation to finish.
- To try to get more information about a run crash, set the xml variable ``$DEBUG`` to ``TRUE`` in the case directory. If you are running with a WRF atmosphere, you can also increase the debug output level of WRF in the namelist build script from 0 to 100 or even 200 (``wrf/bld/wrf.bldnamelist.csh``). This can add a lot of output to the rsl.out.XXXX and rsl.error.XXXX log files. 
- Make sure there is enough space in your ``/glade/scratch/$user`` directory (or generally your run directory) to support all of the history files, restart files, output logs, and any copied forcing data. The ``gladequota`` command can help with this on Cheyenne.
- Occasionally, Cheyenne will cause a run to abort with a simple and vague ``MPT Error`` found in a log. All runs crash with MPT errors in the submission output file, so this file is less helpful. A Cheyenne MPT error could be just a blip in inter-node communication, and restarting the run should be tried.
  
Other Sources for Troubleshooting Info
======================================

Here are several other sources of information that could help with troubleshooting problems in the RCESM:

- The CIME documentation ( https://esmci.github.io/cime/index.html ) and CESM2 documentation ( https://escomp.github.io/cesm/release-cesm2/index.html ) Both contain a lot of information applicable to the RCESM modeling system, porting the code to other computers, common usage, and troubleshooting problems.
- The CRESM 1.0 documentation from Texas A&M university is available in the RCESM code repository. First, download the repository contents using `these instructions <https://ncar.github.io/TAMURegionalCESM/downloading_cesm.html>`_, and then look for the document in the ``docs/cresm1.0/cresm_manual.pdf`` file.
- There is a lot of documentation on CLM 4.0 at their website here: http://www.cesm.ucar.edu/models/cesm1.1/clm/
- The CESM community also discusses issues on the forums here: https://bb.cgd.ucar.edu/
- Documentation for the WRF model used in RCESM is on the web here: http://www2.mmm.ucar.edu/wrf/users/pub-doc.html
- Documentation for the ROMS model used in RCESM is on the web here: https://www.myroms.org/wiki/Documentation_Portal


Future Work
-----------

    There are plenty of directions for software engineering work needed with the RCESM code going forward. Below is a list of items from the version we currently have, in no particular order.

+ Move CIME to another fork, and/or issue a pull request for RCESM changes to be brought into the main CIME distribution.
+ Remove the "csh script" step in WRF and ROMS builds. This is left over from old versions of CESM and should be replaced with python code.
+ Update to CLM 5.0.
+ Set up nightly or some form of automated testing infrastructure.
+ Finish implementing changes to reduce surface wind instabilities (fish-scale pattern) in WRF.
+ Investigate PE layouts for WRF-ROMS coupled runs. Can we find a layout that runs more efficiently?
