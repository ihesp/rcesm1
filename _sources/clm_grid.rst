.. _clm_grid:

2. Creating New Domain and Land Surface Data Files for CLM
===========================================================

In order to build and run a new domain in the RCESM model, currently the CLM component and coupler requires domain files and surface data files for the regional domain. For more information on these files (and a description of how to create them) consider browsing through these websites:
 - `Creating mappping files that mksurfdata_map will use <http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x11775.html>`_
 - `Creating a domain file for CLM and DATM <http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x11812.html>`_
 - `Using mksurfdata_map to create surface datasets from grid datasets <http://www.cesm.ucar.edu/models/cesm1.2/clm/models/lnd/clm/doc/UsersGuide/x11868.html>`_

These websites are older, a bit out of date, and not exactly what we need for our model. Below are more specific instructions for generating the domain and surface dataset files needed for a CESM run that includes CLM.

Step 1: Generate SCRIP grid mapping files for your domain. This can be done with a python script available at TAMU. Contact Jaison Kurian if needed (jaisonk@tamu.edu).

Step 2: Create the domain files using the gen_domain tool. This tool is available from many places, and it is not entirely necessary to use the one from an older version of CLM. The version available through CIME in a full checkout of RCESM should work. The tool is located in: ``cime/tools/mapping/gen_domain_files`` .

Step 2-a: Build the gen_domain tool using the directions in the INSTALL file that is located in the same directory.

Step 2-b: Run the gen_domain tool using the command:
``$ gen_domain -m <input mapping file name> -o <ocean grid name> -l <land grid name>``
This will produce three domain files, one for the land and atmosphere, one for the ocean and sea ice, and one for the ocean on the atmospheric grid.

Step 3: Create mapping files from the input data sets to the land grid. In order to map the surface data into a file on the land grid, the mapping weights must be generated with the ``mkmapdata.sh`` script. This is the tool that needs to use a version of CLM that matches the one in the repository. 

One way to get this is to checkout an older tag of clm (such as ``svn co https://svn-ccsm-models.cgd.ucar.edu/clm2/trunk_tags/clm4_5_13_r211``) and build the tool on your machine from its location in that repository (``components/clm/tools/shared/mkmapdata``). However, this could require some detailed porting to a new machine, so the easiest way is to use some older versions of this tool that were previously ported from Yellowstone to Cheyenne by the Paleo working group. These are located on Cheyenne at: ``/gpfs/fs1/p/cesm/palwg/cesm1_2_0/models/lnd/clm/tools/shared/mkmapdata/`` . The script ``mkmapdata-tamu-gom3x-02.sh`` was successfully used to create the surface data to 3km wrf grid mapping files on April 8, 2019. 

Step 3-a: Copy the ``mkmapdata-tamu-gom3x-02.sh`` script to a new file and edit it as needed for your new grid and application.

Step 3-b: Make a pbs submission script to submit this job to a compute node on cheyenne. This job takes a while and will need the memory from compute nodes, which is larger than the login nodes. The script should call the mkmapdata.sh script as:
``/gpfs/fs1/p/cesm/palwg/cesm1_2_0/models/lnd/clm/tools/shared/mkmapdata/mkmapdata-tamu-gom3x-02.sh -p clm4_0 -t regional -v -r [your grid name] -f [full path to atm scrip grid file] &> output.txt`` . It will need 1 CPU and approximately 4 hours to complete.

Step 3-c: Submit the pbs script. You can run `tail -f output.txt` to track the script as it runs. When it is finished, it should have created a mapping file from each input data file grid to your new grid.

Step 4: Create the land surface data file using the mksurfdata_map tool. This is another tool that is useful to have the older version of. Again, you can checkout an older version of clm (such as ``svn co https://svn-ccsm-models.cgd.ucar.edu/clm2/trunk_tags/clm4_5_13_r211``). The mksurfdata_map tool is located in ``components/clm/tools/clm4_0/mksurfdata_map`` .

Step 4-a: Follow the instructions in the README file to build the mksurfdata_map tool. This worked on Cheyenne without porting the entire clm tag. 

Step 4-b: Call the mksurfdata.pl script with user generated grid specifiers as:
``$ ./mksurfdata.pl -res usrspec -usr_gname [your grid name] -usr_gdate [date on mapping files] -usr_mapdir [directory containing map files]``
This script should run quickly on a login node, and produce four files. Two log files and two netcdf surface data files for CLM.
