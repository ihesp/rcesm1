.. _contribute:

================================
Contributing to RCESM
================================


Contributing to RCESM source code
---------------------------------





Updating This Document
----------------------

The source files for this webiste and the RCESM documentation are contained within the GitHub repository at: https://github.com/ihesp/rcesm1 . To update the content of these web pages follow these simple steps.

1. Check out a copy of the repository, and go to the docs/source sub directory.
   
.. code-block:: console

    git clone https://github.com/ihesp/rcesm1.git rcesm_docs
    git checkout gh-pages
    cd rcesm_docs/_sources

2. Edit the exist re-Structured text files (.rst) or add new sections to the documentation by creating a new file with a ".rst" extension, and adding it's name to the Table of Contents in the `index.rst` file. The new file must have the following code at the top of the file and content can be specified via HTML or `markdown <https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet>`_. So, a new file about ROMS configuration options, for example could be named "roms_config.rst" and the top of the file would look like:

.. code-block:: console

    .. _roms_config:

    ==========================
    ROMS Configuration Options
    ==========================

3. Build the HTML pages from the documentation. The machine to build the source must have python and Sphinx installed. The example code below works on any CGD linux machine:

.. code-block:: console

   module load lang/python/2.7.14
   make html

4. Save and commit your changes to the repository master branch.

.. code-block:: console

   git add _sources/filename.rst
   git add filename.html
   git commit
   git push 


Congratulations! The documentation website at https://ihesp.github.io/rcesm1 should now be updated! You may need to wait a minute or two, or refresh the page a few times to clear the cashe before your changes show up.
