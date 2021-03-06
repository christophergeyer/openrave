.. _release_organization:

Release Organization
====================

This page describes the OpenRAVE release organization and platform requirements. The :ref:`testing_framework` is responsible for auto-generating the releases and making them available to the general public.

All releases are compiled, signed, and uploaded using:

* email: openrave.testing@gmail.com
* sourceforge id: `openravetesting <https://sourceforge.net/users/openravetesting>`_
* launchpad id: `openrave <https://launchpad.net/~openrave>`_
* GPG id: ``OpenRAVE Testing (Testing and Releasing of OpenRAVE Packages) <openrave.testing@gmail.com>``

Versioning
----------

Official releases have all even MINOR/PATCH version numbers like 0.6.0, 0.6.2, 0.8.0, 1.2.0. Development releases have odd version numbers like 0.5.0, 0.5.1, 0.6.1, 0.8.3. Any API changes require the MINOR version to increase. OpenRAVE patch releases should always be binary compatible so that newer versions can read old plugins. In otherwords all 0.3.x releases should be binary compatible with each other. See `ABI Compatibility <http://www.ros.org/reps/rep-0009.html>`_ for an excellent discussion.

Allow multiple openrave versions to be installed simultaneously. This requires using suffixing several files and executables with $MAJOR.$MINOR similar to how python and `boost <https://svn.boost.org/trac/boost/wiki/ImprovingPractices>`_ work.

For Linux systems, SOVERSION is always "0" since the important version numbers are part of the library name itself.

There is an `ABI compliance checker <http://ispras.linux-foundation.org/index.php/ABI_compliance_checker>`_ that can report compatibility errors.  The `API Sanity Autotest <http://ispras.linuxfoundation.org/index.php/API_Sanity_Autotest>`_ is also an interesting tool that runs functions with reasonable inputs based on a static analysis of the code.

Windows Installer
-----------------

Official releases are uploaded on `Sourceforge <http://sourceforge.net/projects/openrave/files/>`_.

Use `Nullsoft Scriptable Install System <http://nsis.sourceforge.net>`_ for generating an executable. Once the regular NSIS setup is run, install the `Large String Build <http://nsis.sourceforge.net/Special_Builds>`_. The NSI file is generated by running **release/generate_installer_windows.py** on the installation directory produced by cmake.

The installer:

* automatically installs python, boost, numpy, and sympy,
* unpacks the openrave files in a user-specified location,
* modifies **openrave/config.h** to point to the user-installed location,
* adds options to install Octave/Python bindings,
* registers the necessary DLLs and adds registry keys for OpenRAVE
 * Under **HKEY_LOCAL_MACHINE\\SOFTWARE\\OpenRAVE** will be every OpenRAVE version that is installed. For example "|version|". Under that will be **InstallRoot**.
* and modifies the PYTHONPATH and Path environment variables.

Note that the provides DLLs are all compiled in Multithreaded DLL runtime.

Debian Source Packages
----------------------

`ppa:openrave/release <https://launchpad.net/~openrave/+archive/release>`_ - official release archive

`ppa:openrave/testing <https://launchpad.net/~openrave/+archive/testing>`_ - testing archive holding the newest builds

Each package is separated into:

* openrave$V: all openrave related packages
* openrave$V-base: core libraries and tools
* openrave$V-data: basic scenes required for examples
* openrave$V-dev: development files and examples
* openrave$V-ikfast: Robot Kinematics Compiler
* openrave$V-octave: octave bindings
* openrave$V-plugin-X: individual plugins and their versions
* openrave$V-plugins-all: all plugins
* openrave$V-python: openravepy python bindings
* openrave-robots-extra: extra robot CAD model files

Building
~~~~~~~~

Debian source packages for Ubuntu/Debian can be prepared by calling cmake with

.. code-block:: bash
  
  cmake -DOPT_BUILD_PACKAGES=ON

To upload the packges on the server do

.. code-block:: bash
  
  make dput

Many times, a special 4th distribution version number ``w`` is attached to the OpenRAVE version ``x.y.z``

.. code-block:: bash
  
  cmake -DOPT_BUILD_PACKAGES=ON -DPACKAGE_VERSION=w

It is possible to customize the PGP signer, the host to upload them, and what distributions to compile them for using

.. code-block:: bash
  
  cmake -DOPT_BUILD_PACKAGES=ON -DCPACK_DEBIAN_DISTRIBUTION_RELEASES="lucid;maverick;natty" -DCPACK_PACKAGE_CONTACT="new signer" -DDPUT_HOST="ppa:new_signer/name"


To compile and upload single precision side-by-side a double precision build do

.. code-block:: bash
  
  cmake -DOPT_BUILD_PACKAGES=ON -DPACKAGE_VERSION=w -DOPT_BUILD_PACKAGE_DEFAULT=OFF -DOPT_DOUBLE_PRECISION=OFF

`Ubuntu Debian Packaging Guide <https://wiki.ubuntu.com/PackagingGuide/Complete>`_

Sourceforge Releases
--------------------

Releases source code and windows installers.

Once a build is stable and all the release packages have been generated, they are uploaded to sourceforge using the **release/sendtosourceforge.sh** script:

.. literalinclude:: ../../../release/sendtosourceforge.sh
  :language: bash

Release Process
===============

1. Run Jenkins and create the latest_stable tag, windows installers, and documentation

2. Update and upload the documentation. Make sure to set the correct revision in **changelog.rst**, and change Latest Official Version in **index.rst**. To generate the zip files do:

.. code-block:: bash

  LANG=en_US.UTF-8 make json_en html_en
  LANG=ja_JP.UTF-8 make json_ja html_ja
  make openravejsonzip openravehtmlzip

3. Create svn tag:

.. code-block:: bash

  svn cp tags/latest_stable tags/X.Y.Z

4. sourceforge: Create a X.Y.Z folder inside the files and copy the latest uploaded files from Jenkins into it, remove the revision numbers from the filenames.

5. Create the Debian packages in openrave/testing

.. code-block:: bash

  cmake -DOPT_BUILD_PACKAGES=ON -DPACKAGE_VERSION=W
  make dput
  cmake -DOPT_BUILD_PACKAGES=ON -DPACKAGE_VERSION=W -DOPT_BUILD_PACKAGE_DEFAULT=OFF -DOPT_DOUBLE_PRECISION=OFF
  make dput

6. Copy the debian packages from openrave/testing to openrave/release

7. Create a new tag:

  git tag -a vX.Y.Z -m "OpenRAVE Release X.Y.Z"
  git push --tags
