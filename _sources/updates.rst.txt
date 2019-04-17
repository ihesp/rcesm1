.. _updates:

===============================
Issues or processes for updates
===============================

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

Updating This Document
----------------------

The source files for this webiste and the RCESM documentation are contained within the GitHub repository at: https://github.com/NCAR/TAMURegionalCESM . To update the content of these web pages follow these simple steps.

1. Check out a copy of the repository, and go to the docs/source sub directory.
   
.. code-block:: console

    git clone https://github.com/NCAR/TAMURegionalCESM.git my_cresm_sandbox
    cd my_cresm_sandbox/docs/source

2. Edit the file or files that you are interested in. You can create new sections of the site by creating a new file with a ".rst" extension, and adding it's name to the Table of Contents in the index.rst file. The new file must have the following code at the top of the file and content can be specified via HTML or `markdown <https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet>`_. So, a new file about ROMS configuration options, for example could be named "roms_config.rst" and the top of the file would look like:

.. code-block:: console

    .. _roms_config:

    ==========================
    ROMS Configuration Options
    ==========================

3. Save and commit your changes to the repository master branch.

.. code-block:: console

   git add filename.rst
   git commit
   git push

4. Build the HTML pages from the documentation. (As of this writing, this does not work on Cheyenne, but does work on any CGD/NCAR linux cluster. The machine to build the source must have python and Sphinx installed.) The example code below works on any CGD linux machine:

.. code-block:: console

   module load lang/python/2.7.14
   make html

5. Copy the newly built html source into the gh-pages branch of the GitHub repository. This can be done by making an external copy of the source, switching to the gh-pages branch and then copying it back over.

.. code-block:: console

   cp -r build/html/* ~/temp_html/
   git checkout --track origin/gh-pages
   cp -r ~/temp_html/* .

6. Commit and push the changes up to the remote.

.. code-block:: console

   git add *
   git commit -m "Updating documentation"
   git push

Congratulations! The documentation website at https://ncar.github.io/TAMURegionalCESM should now be updated! You may need to wait a minute or two, or refresh the page a few times to clear the cashe before your changes show up.


