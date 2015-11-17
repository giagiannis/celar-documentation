Installation & Configuration
============================
This section describes the procedures through which you can manually install and configure the CELAR Platform modules.

Distribution
------------
In this section we describe the distribution methods of the platform. 

Source code
^^^^^^^^^^^
The source code of the CELAR Platform is located in `github <https://github.com/celar>`_. Instructions regarding the compilation and installation of each component can be found in the README file of each subproject.

Binary packages
^^^^^^^^^^^^^^^

The CELAR Platform modules are distributed as rpm packages, through the official CELAR repository. To be able to obtain the CELAR Packages, you have to create a new repository entry by adding the file ``/etc/yum.repos.d/celar.repo`` with the following contents:
::

 [CELAR-releases]
 name=CELAR-releases
 baseurl=http://snf-175960.vm.okeanos.grnet.gr/yum/releases
 enabled=1
 protect=0
 gpgcheck=0
 metadata_expire=30s
 autorefresh=1
 type=rpm-md

After the addition of this file, you can now run ``$ yum install <CELAR module name>`` successfully.


Precooked VM images
^^^^^^^^^^^^^^^^^^^
To avoid manual configuration of the different submodules, the platform is also distributed as a set of precooked VM images, containing all the necessary components for a minimal CELAR installation. In the following table we provide the image IDs for the two different cloud providers CELAR was deployed and tested into.   

===========================  ====================================  ====================================
 \                           Okeanos                               Flexiant 
===========================  ====================================  ====================================
CELAR Server VM Image        072f0e29-faf8-4941-a688-c7e4dcf200e7  072f0e29-faf8-4941-a688-c7e4dcf200e7 
CELAR Orchestrator VM Image  072f0e29-faf8-4941-a688-c7e4dcf200e7  072f0e29-faf8-4941-a688-c7e4dcf200e7 
===========================  ====================================  ====================================


CELAR Server VM
---------------
CELAR Manager
^^^^^^^^^^^^^
Installation
~~~~~~~~~~~~
Configuration
~~~~~~~~~~~~~

CELAR DataBase
^^^^^^^^^^^^^^

Installation
~~~~~~~~~~~~
The Celar project repository must be added in 
 */etc/yum.repos.d/celar.repo*
You can innstall the CELAR Database rpm package from the package management
 $ ``yum install celar-db-rpm`` 

This will also install the *PostgreSQL* database which a package dependency for the CELAR DataBase

Configuration
~~~~~~~~~~~~~
The installation script handles the default configurations, so no further configuration is required for the CELAR Database package to be run in its default settings.

The default configuration is the following:

 :database: celardb
 :user: celaruser
 :password: celar-user

The default user and database is automatically created and so is the database structure

the listen addresses in *postgresql.conf* are set to  *'0.0.0.0'* and all users are given *md5* access from all hosts
                          
Information System
^^^^^^^^^^^^^^^^^^
Installation
~~~~~~~~~~~~
Configuration
~~~~~~~~~~~~~


CELAR Orchestrator VM
---------------------
CELAR Orchestrator
^^^^^^^^^^^^^^^^^^
Installation
~~~~~~~~~~~~
Configuration
~~~~~~~~~~~~~
Decision Making Module
^^^^^^^^^^^^^^^^^^^^^^
Installation
~~~~~~~~~~~~~
Configuration
~~~~~~~~~~~~~
MELA
^^^^
Installation
~~~~~~~~~~~~
Configuration
~~~~~~~~~~~~~
JCatascopia
^^^^^^^^^^^
Installation
~~~~~~~~~~~~
Configuration
~~~~~~~~~~~~~

Client Tools
------------
CAMF
^^^^
Installation
~~~~~~~~~~~~
Configuration
~~~~~~~~~~~~~
