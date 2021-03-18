.. _downloading:

====================
 Downloading R-CESM
====================

The R-CESM code is publicly available through our `github repository <https://github.com/ihesp/cesm>`_ . To simply download and use the model, no git knowledge is required other than what is documented in the following steps.


1. Navigate to a suitable working directory, and clone the current stable tag of the R-CESM model source code

.. code-block:: console
		
    git clone https://github.com/ihesp/cesm.git -b ihesp-regional-master_200911 my_rcesm_sandbox
	
This will create a new directory *my_rcesm_sandbox* in your current working directory.


2. Optionally, you can checkout the current unstable, developmental branch of R-CESM using

.. code-block:: console

    git clone https://github.com/ihesp/cesm.git -b ihesp-regional-master my_rcesm_sandbox
   

3. Navigate into the source directory, and use the ``manage_externals/checkout_externals`` script to download the individual R-CESM components

.. code-block:: console
		
    ./manage_externals/checkout_externals
   
The ``checkout_externals`` script is a package manager that pulls the appropriate versions of the CIME and active component (WRF, ROMS, CLM) source codes into your R-CESM working directory. The script reads the ``Externals.cfg`` configuration file to obtain the versions and repositories of the code that will be downloaded.


At this point you have a working version of R-CESM.

For general information on using the CIME framework in the context of CESM, see the CIME documentation at http://esmci.github.io/cime/ .

More details on checkout_externals
----------------------------------

The file **Externals.cfg** in your top-level RCESM directory tells
**checkout_externals** which tag/branch of CIME should be
brought in to generate your sandbox. (This file serves the same purpose
as SVN_EXTERNAL_DIRECTORIES when CESM was in a subversion repository.)

NOTE: Just like svn externals, checkout_externals will always attempt
to make the working copy exactly match the externals description. 

**You need to rerun checkout_externals whenever Externals.cfg has
changed** (unless you have already manually updated the relevant
external(s) to have the correct branch/tag checked out). Common times
when this is needed are:

* After checking out a new RCESM branch/tag

* After merging some other RCESM branch/tag into your currently
  checked-out branch

**checkout_externals** must be run from the root of the source
tree. For example, if you cloned RCESM with

.. code-block:: console
        
  git clone https://github.com/ihesp/rcesm1.git my_rcesm_sandbox

then you must run **checkout_externals** from
``/path/to/my_rcesm_sandbox``.

To see more details of **checkout_externals**, issue

.. code-block:: console
        
  ./manage_externals/checkout_externals --help
