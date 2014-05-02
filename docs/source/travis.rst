============================
 Using Conda with Travis CI
============================

Using conda with `Travis CI <https://travis-ci.org/>`_ is often a preferable
alternative, as the Debian repos provided by Travis often do not include
versions of Python packages for all versions of Python, or may not be updated
fast enough.  `pip` installing such packages may also be undesirable, as this
can take a while, eating up a large chunk of the 50 minutes that Travis allows
for each build.  Using conda also lets you test the building of conda recipes
on Travis.

Conda is cross-platform, so it can be used for anything, not just Python, but
the following guide shows how to use it to test a Python package on Travis CI.

The .travis.yml
===============

The basic configuration for ``.travis.yml``, e.g., for a project that supports
Python 2.6, 2.7, 3.3, and 3.4 is as follows:

.. code-block:: yaml

   language: python
   python:
     # We don't actually use the Travis Python, but this keeps it organized.
     - "2.6"
     - "2.7"
     - "3.3"
     - "3.4"
   install:
     - sudo apt-get update
     # You may want to periodically update this, although the conda update
     # conda line below will keep everything up-to-date.
     - if [[ "$TRAVIS_PYTHON_VERSION" == "2.7" ]]; then
         # This saves us some downloading for this version
         wget http://repo.continuum.io/miniconda/Miniconda-3.4.2-Linux-x86_64.sh -O miniconda.sh;
       else
         wget http://repo.continuum.io/miniconda/Miniconda3-3.4.2-Linux-x86_64.sh -O miniconda.sh;
       fi
     - bash miniconda.sh -b -p $HOME/miniconda
     - export PATH="$HOME/miniconda/bin:$PATH"
     - hash -r
     - conda config --set always_yes yes --set changeps1 no
     - conda update conda
     # Useful for debugging any issues with conda
     - conda info -a

     # Replace dep1 dep2 ... with your dependencies
     - conda create -n test-environment python=$TRAVIS_PYTHON_VERSION dep1 dep2 ...
     - source activate test-environment
     - python setup.py install

   script:
     # Your test script goes here

Additional Steps
================

If you need to support a package that doesn't have official Continuum builds,
you may need to build it yourself, and add it to a Binstar channel. You can
then add

.. code-block:: yaml

   - conda config --add channels your_binstar_username

to the install steps in ``.travis.yml`` so that it finds the packages on that
channel.

For more information on building conda packages, see the :doc:`build` docs and
the example recipes in the `conda-recipes repo
<https://github.com/conda/conda-recipes>`_.

Building a Conda Recipe
=======================

If you support official conda packages for your project, you may want to use
``conda build`` in Travis, so that the building of your recipe is tested as
well.  It is suggested to include the conda recipe along side your source
code. You would then replace

.. code-block:: yaml

   - python setup.py install

with

.. code-block:: yaml

   - conda build your-conda-recipe
   - conda install your-package --use-local