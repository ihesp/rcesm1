.. _downloading:

==================
 Downloading R-CESM
==================

The R-CESM code is publicly available from its `github repository <https://github.com/ihesp/cesm>`_ . The basic components used in this model are included in the repo (unlike CESM), but the scripting infrastructure is maintained in an external repository and managed with the "manage_externals" tool. 

You will need some familiarity with git in order to modify the code and commit any changes. However, to simply checkout and run the code, no git knowledge is required other than what is documented in the following steps.

To obtain the R-CESM release code you need to do the following:

1. Clone the repository.

.. code-block:: console
		
    git clone https://github.com/ihesp/cesm.git my_rcesm_sandbox
	
This will create a directory my_rcesm_sandbox/ in your current working directory.

2. Go into the newly created directory (using ``cd``) and determine the currently active version of R-CESM using the ``git status`` command.

.. code-block:: console
		
    cd my_rcesm_sandbox 
    git status
	
By default, R-CESM uses the 'igesp-regional-master' branch, which is the most stable configuration.

3. Optionally, you can checkout any specific tag of R-CESM. To see the available R-CESM tags, use the ``git tag`` command. Then use the ``git checkout`` command to checkout the tag of interest. For example:

.. code-block:: console

    git tag
    git checkout release_cesm2.0.0
   
(It is normal and expected to get a message about being in 'detached HEAD' state. For now you can ignore this, but it becomes important if you want to make changes to your Externals.cfg file and commit those changes to a branch.)

4. Download the components required by R-CESM, using the script ``manage_externals/checkout_externals``.

.. code-block:: console
		
    ./manage_externals/checkout_externals
   
The ``checkout_externals`` script is a package manager that clones the source codes for CIME and active component models (WRF, ROMS, CLM) into your R-CESM working directory. The script reads the ``Externals.cfg`` configuration file to obtain the versions and repositories of the code that will be downloaded.


To see more details regarding the checkout_externals script from the command line, type:

.. code-block:: console

    ./manage_externals/checkout_externals --help

To confirm a successful download of all components, you can run ``checkout_externals``
with the status flag to show the status of the externals:

.. code-block:: console

    ./manage_externals/checkout_externals -S

Check the ``manage_externals.log`` file to see what errors are reported.


For general information on using the CIME framework in the context of CESM and R-CESM, see the CIME documentation at http://esmci.github.io/cime/ .

