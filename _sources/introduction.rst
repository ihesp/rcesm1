.. _introduction:

==============
 Introduction
==============

How To Use This Document
------------------------

This guide is intended to instruct both novice and experienced users on downloading,
building and running the **Regional Community Earth System Model (RCESM)**.

RCESM is built on the `CIME framework <http://github.com/ESMCI/cime>`_.

If you are a new user, we recommend reading the first few sections of
the `CIME`_ documentation which is written so that, as much as
possible, individual sections stand on their own and the `CIME`_
documentation guide can be scanned and sections read in a relatively
ad hoc order.

.. code-block:: console 

    Throughout the guide, this presentation style indicates shell
    commands and options, fragments of code, namelist variables, etc.

.. note:: 

   Variables presented as ``$VAR`` in this guide typically refer to variables in XML files
   in a CESM case. From within a case directory, you can determine the value of such a
   variable with ``./xmlquery VAR``. In some instances, ``$VAR`` refers to a shell
   variable or some other variable; we try to make these exceptions clear.
    


RCESM Overview
--------------

The RCESM is a fully coupled regional model developed jointly by the National Center for Atmospheric Research (NCAR) and Texas A&M University (TAMU). This model is built on the coupling framework and software infrastructure used by the Community Earth System Model (CESM). To learn more about the base model for this project, CESM, please see the `CESM webiste <http://www.cesm.ucar.edu>`_.

This documentation provides both a quickstart guide, including how to get the code and how to build and run an experiment, and information on the system architecture and the development changes made by software engineers at NCAR.

The original TAMU model (CRESM 1.0) is documented in the GitHub repo. Once you `Download the RCESM source <downloading_cesm.html>`_ then go to the ``docs/cresm1.0/`` sub-directory and find the ``cresm_manual.pdf`` file. 


RCESM Software/Operating System Prerequisites
---------------------------------------------

The following are the external system and software requirements for
installing and running RCESM.

-  UNIX style operating system such as CNL, AIX or Linux

-  python >= 2.7

-  perl 5 

-  subversion client (version 1.8 or greater but less than v1.11) for downloading CAM, POP, and WW3

-  git client (1.8 or greater)

-  Fortran compiler with support for Fortran 2003

-  C compiler

-  MPI (although CESM does not absolutely require it for running on one processor)

-  `NetCDF 4.3 or newer <http://www.unidata.ucar.edu/software/netcdf/>`_.

-  `ESMF 5.2.0 or newer (optional) <http://www.earthsystemmodeling.org/>`_.

-  `pnetcdf 1.7.0 is required and 1.8.1 is optional but recommended <http://trac.mcs.anl.gov/projects/parallel-netcdf/>`_

-  `Trilinos <http://trilinos.gov/>`_ may be required for certain configurations 

-  `LAPACK <http://www.netlib.org/lapack/>`_ and `BLAS <http://www.netlib.org/blas/>`_

-  `CMake 2.8.6 or newer <http://www.cmake.org/>`_ 

.. warning:: NetCDF must be built with the same Fortran compiler as CESM. In the netCDF build the FC environment variable specifies which Fortran compiler to use. CESM is written mostly in Fortran, netCDF is written in C. Because there is no standard way to call a C program from a Fortran program, the Fortran to C layer between CESM and netCDF will vary depending on which Fortran compiler you use for CESM. When a function in the netCDF library is called from a Fortran application, the netCDF Fortran API calls the netCDF C library. If you do not use the same compiler to build netCDF and CESM you will in most cases get errors from netCDF saying certain netCDF functions cannot be found.

Parallel-netCDF, also referred to as pnetcdf, is optional. If a user
chooses to use pnetcdf, version 1.7.0 or later should be used with CESM.
It is a library that is file-format compatible with netCDF, and provides
higher performance by using MPI-IO. Pnetcdf is enabled by setting the
``$PNETCDF_PATH`` Makefile variable in the ``Macros.make`` file.

.. _CIME: http://esmci.github.io/cime
