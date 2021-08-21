.. _updates:

======================================
Future work and processes for updates
======================================



Future Work
=============

    There are plenty of directions for software engineering work needed with the RCESM code going forward. Below is a list of items from the version we currently have, in no particular order.

+ Move CIME to another fork, and/or issue a pull request for RCESM changes to be brought into the main CIME distribution.
+ Remove the "csh script" step in WRF and ROMS builds. This is left over from old versions of CESM and should be replaced with python code.
+ Update to CLM 5.0.
+ Set up nightly or some form of automated testing infrastructure.
+ Finish implementing changes to reduce surface wind instabilities (fish-scale pattern) in WRF.
+ Investigate PE layouts for WRF-ROMS coupled runs. Can we find a layout that runs more efficiently?



Process for updates
=====================


    Each of the components for this model will be developed by their independant working groups, and updates will need to be periodically performed in our model to take advantage of new features, bug fixes, and machine support in a component update. In order to do updates, certain issues will need to be addressed, as described here.

WRF Updates
-----------

    While the NCAR integration of the TAMU WRF model did not make many changes to the actual WRF source, yet, there is currently work going on to fix surface-level instabilities in wind fields that will make changes to the source. In addition, there seems to be many change in the WRF code that were made by software engineers at TAMU. All of these changes may or may not need to be propagated forward to newer versions of WRF. This will likely be the most difficult part of any upgrade to the WRF code base in this model. Also, changes should be made to the namelist and build scripts in our model to make them more flexible and resilient to future upgrades. This is discussed further in Section 6. 

ROMS Updates
------------

    Similar to WRF, our project did not make many changes to the ROMS code base. However, it is possible (likely, even) that TAMU made some changes to ROMS in the distribution given to us. These changes may or may not need to be propagated forward to new versions of ROMS. Based on my (limited) browsing of the ROMS code in this distribution, there are not as many RCESM specific additions to the ROMS code base as there are to WRF, so it is likely ROMS updates would be much easier. Also, as in WRF, changes should be made to the namelist and build scripts in our model to make them more flexible and resilient to future upgrades. This is discussed further in Section 6.

CLM Updates
-----------

    The RCESM uses CLM version 4.0 because earlier versions of similar models did as well, but this is not a good version to use going forward as it is no longer supported by the land model working group. Current releases of the CLM code do not even include the CLM 4.0 code base any more. Scripts and tools to create domain and surface data files have been removed, and most of these do not work on Cheyenne, with no plans to update them to work on modern machines. So updating to a modern version of CLM (perhaps 5.0) should be a relatively high priority. The good news is that very few changes were made to the CLM source code during the creation of the RCESM, so updating should be pretty straight forward. However, an updated version of CLM will likely need an updated version of CIME as well, and that process is discussed in the next section.

CIME Updates
------------

    Currently, the Master branch of the RCESM code repository points to the CIME code base as an external repository. The external used is my personal fork (https://github.com/Katetc/cime). Going forward, this is probably not the best way to handle the CIME code, as I am no longer working on this project. The best way to manage the CIME code would probably be to create a branch off the main CIME repository, with our changes added in. Eventually, the goal would be to have our changes incorporated into the main CIME distribution, so we would not need to stay separated at all. The goal should be to import the latest CIME release as an external each time the code is cloned. The current changes to CIME for RCESM are minor, so doing a pull request to get them integrated would be more of a discussion of priorities than of difficulties.



