.. _downloading:

==================
 Downloading RCESM
==================

Downloading the code and scripts
--------------------------------

The RCESM code is available from a private repository in github. The basic components used in this model are included in the repo (unlike CESM), but the scripting infrastructure is maintained in an external repository and managed with the "manage_externals" tool. To access the repository, please email one of the collaborators with your git account information.

You will need some familiarity with git in order to modify the code and commit any changes. However, to simply checkout and run the code, no git knowledge is required other than what is documented in the following steps.

To obtain the RCESM release code you need to do the following:

1. Clone the repository.

.. code-block:: console
		
    git clone https://github.com/NCAR/TAMURegionalCESM.git my_cresm_sandbox
	
This will create a directory my_cresm_sandbox/ in your current working directory.

2. Go into the newly created directory and determine what version of the RCESM you want. To see what cesm tags are available, simply issue the git tag command.

.. code-block:: console
		
    cd my_cresm_sandbox git tag
	
3. OPTIONAL: Do a git checkout of the tag you want. If you want to checkout RCESM_release_v0_1, you would issue the following.

.. code-block:: console

    git checkout RCESM_release_v0_1
   
(It is normal and expected to get a message about being in 'detached HEAD' state. For now you can ignore this, but it becomes important if you want to make changes to your Externals.cfg file and commit those changes to a branch.)

4. Run the script manage_externals/checkout_externals.

.. code-block:: console
		
    ./manage_externals/checkout_externals
   
The checkout_externals script is a package manager that will populate the cresm directory with the relevant version of the CIME infrastructure code.

At this point you have a working version of RCESM.

For general information on using the CIME framework in the context of CESM and RCESM, see the CIME documentation at http://esmci.github.io/cime/ .



The **checkout_externals** script will read the configuration file called ``Externals.cfg`` and
will download all the external component models and CIME into /path/to/my_cesm_sandbox. 

To see more details regarding the checkout_externals script from the command line, type:

.. code-block:: console

    ./manage_externals/checkout_externals --help

To confirm a successful download of all components, you can run ``checkout_externals``
with the status flag to show the status of the externals:

.. code-block:: console

    ./manage_externals/checkout_externals -S

Check the ``manage_externals.log`` file to see what errors are reported.
