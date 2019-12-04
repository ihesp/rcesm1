.. _prepost_tools:

============================================
Pre-processing and Post-processing Utilities
============================================


Post-processing tools
=====================



NCAR CESM Postprocessing
-------------------------

See https://github.com/NCAR/CESM_postprocessing

Requires https://github.com/NCAR/ASAPPyTools



Instructions




Step:0 - Install a local copy of asaptools
This step is needed only the first time you run the tool

.. code-block:: console

    module purge
    module load python/2.7.13-intel-2017b
    cd ~
    git clone https://github.com/NCAR/ASAPPyTools
    cd /ihesp/altuntas/CESM_postprocessing//cesm-env2/bin
    ./activate
    cd ~/ASAPPyTools
    python setup.py install --user

Step:1 - Activate cesm virtual environment

.. code-block:: console

    module load python/2.7.13-intel-2017b
    export POSTPROCESS_PATH=/ihesp/altuntas/CESM_postprocessing/
    alias cesm_pp_activate='. $POSTPROCESS_PATH/cesm-env2/bin/activate'
    cesm_pp_activate

Step:2 - Create your postprocessing instance

.. code-block:: console

    create_postprocess -case [your-case-directory]

Step:3 - Edit env_postprocess.xml and env_diags_ocn.xml files
see the files in the following directory as an example:
/scratch/user/altuntas/g.e20.G.TL319_t13.control.001_contd

Step:4 - Generate average files

Step:4.1: - Enter your project id in ocn_averages file by
replacing "None" in the following line with your project id:

.. code-block:: console

    bsub -P None

Note: depending on the duration of the simulation, you may
also need to increase the wallclock time.

Step:4.2 - Submit the job:

.. code-block:: console

    bsub < ocn_averages

Step:4.3 - Check the latest log file to see if averaging was
completed successfully: [your-case-directory]/logs/

Step:5 - Generate the diagnostics

Step:5.1: - Enter your project id in ocn_diagnostics file by
replacing "None" in the following line with your project id:

.. code-block:: console

    bsub -P None

Step:5.2 - Submit the job:

.. code-block:: console

    bsub < ocn_diagnostics
