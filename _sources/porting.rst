.. _porting:

====================================
 Porting R-CESM to new machines
====================================


An RCESM case directory contains all of the configuration xml files, case control scripts, and namelists to start a RCESM run. It also contains the README document which contains information about the case as it was created, and the CaseStatus document that keeps track of changes as you go.

 - Edit the ``my_rcesm_sandbox/cime/config/cesm/machines/config_machines.xml`` file to point to correct input file directories and output file locations before creating a case. An example on Ada. 

 .. code-block:: console

      <machine MACH="ada">
      <DESC>  TAMU IBM NextScale Cluster, os is Linux, 20 pes/node, batch system is LSF  </DESC>
      <NODENAME_REGEX>.*.ada.tamu.edu</NODENAME_REGEX>
      <OS>LINUX</OS>
      <COMPILERS>intel,gnu,pgi</COMPILERS>
      <MPILIBS compiler="intel" >impi,mpt,openmpi</MPILIBS>
      <MPILIBS compiler="pgi" >mpt</MPILIBS>
      <MPILIBS compiler="gnu" >openmpi</MPILIBS>
      <CIME_OUTPUT_ROOT>/scratch/user/$USER/RCESM/RegionalCESM-2.0.0</CIME_OUTPUT_ROOT>
      <DIN_LOC_ROOT>/scratch/user/liu6/RCESM/inputdata/</DIN_LOC_ROOT>
      <DIN_LOC_ROOT_CLMFORC>>/scratch/user/liu6/RCESM/inputdata/lmwg</DIN_LOC_ROOT_CLMFORC>
      <DOUT_S_ROOT>$CIME_OUTPUT_ROOT/archive/$CASE</DOUT_S_ROOT>
      <BASELINE_ROOT>>/scratch/user/liu6/RCESM/inputdata/cesm_baselines</BASELINE_ROOT>
      <CCSM_CPRNC>>$CIME_OUTPUT_ROOT/../my_rcesm_sandbox2/cime/tools/cprnc</CCSM_CPRNC>
      <GMAKE_J>4</GMAKE_J>

