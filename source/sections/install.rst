Installation & Configuration
============================
Prerequisites
-------------
Distribution
------------
Source code
^^^^^^^^^^^
Binary packages
^^^^^^^^^^^^^^^

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
