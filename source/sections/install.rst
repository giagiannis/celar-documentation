Installation & Configuration
============================
This section describes the procedures through which you can manually install and configure the CELAR Platform modules.

Prerequisites
-------------
Before the installation and the configuration of the CELAR Platform, we first need to have a running instance of SlipStream, a provisioning tool used by the platform to deploy and scale applications. The user may want to visit the SlipStream `guides <http://ssdocs.sixsq.com/en/latest/>`_ first, to deploy and configure a SlipStream instance pointing to the cloud provider the user wants to deploy their application into.


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
The CELAR Server VM is the static part of the platform. A single CELAR Server VM must be installed into the cloud provider and it is responsible for launching new application deployments. In this section, details are provided regarding the installation and configuration procedures of the modules hosted into the CELAR Server VM.

CELAR DataBase
^^^^^^^^^^^^^^
The CELAR DataBase is part of the CELAR Manager module. It is distributed in a different rpm package to enable the installation of this module into a dedicated VM. 

Installation
~~~~~~~~~~~~
You can install the CELAR Database rpm package from the package management
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

CELAR Manager
^^^^^^^^^^^^^
The CELAR Manager is the endpoint of the platform. It retains an API used to describe and deploy new applications, provide monitoring metrics to the clients, etc. 

Installation
~~~~~~~~~~~~
To install the CELAR Manager you must run the following command (as root):

``$ yum install -y celar-server-rpm``

This command will install the CELAR Manager, along with its dependencies. 

Configuration
~~~~~~~~~~~~~
The configuration file of the CELAR Manager is located at ``/opt/celar/celar-server/conf/celar-server.properties``. The available configuration options, along with their default values are listed below:

::

 # celar server properties file

 # unencrypted traffic port
 server.plain.port = 8080

 # SSL configurations
 server.ssl.port = 8443
 server.ssl.keystore.path = 
 server.ssl.keystore.password = 
 
 # SlipStream properties
 slipstream.username = 
 slipstream.password = 
 slipstream.connector.name = 
 slipstream.url = 


 #DB properties
 backend = postgresql
 postgresql.host = localhost
 postgresql.port = 5432
 postgresql.username = celaruser
 postgresql.password = celar-user
 postgresql.db_name =  celardb

The user must define the url and the connector name of the running SlipStream installation, and -optionally- the username and the password of their SlipStream account. If those credentials are not defined into the configuration file, they must be provided through CAMF, else every request will occur for the specified user (used for standalone installations and debugging purposes). 

The ``server.ssl`` properties are filled by the installer during the installation process, since a new java keystore is generated with a random password and placed under the root directory of the CELAR Manager. The user can override those default certificates with their own. Finally the user must define the DB properties, as updated during the installation of the celar-db component. 

After the configuration of the module, the user must restart the CELAR Manager by issuing the following command (as root):

``$ service celar-server restart``

Information System
^^^^^^^^^^^^^^^^^^

The CELAR Information System  consists of two components, the **Information System Service** and the **Information System Frontend**. Each one is a separate application, which is distributed in its own package. Both components are written in Java and so **Java 1.7** should be present before installation. Additionally the *Information System Frontend* requires a Web Server, which provides a HTTP server and Servlet container capable of serving static and dynamic content. We recommend any of **Tomcat 7.0.xx** versions, but we strongly advise to use the latest one (*7.0.64 currently*).

\* Both the *Information System Service* and the *Information System Frontend* installation scripts will try to fill out these prerequisites by downloading and installing Java and / or tomcat from external repositories.


Installation
~~~~~~~~~~~~
To install or update the *Information System Service* you have to issue the following command

::

  yum install cloud-is-core

\*For the *Information System Service* to operate correctly the CELAR Server Manager Module must be installed also and be accessible from the CELAR Information System Service.


To install or update the *Information System Frontend* you have to issue the following command
::

 yum install cloud-is-web


\*For the *Information System Frontend* to operate correctly the *Information System Service* must be installed also and be accessible from the *Information System Frontend*.


Configuration
~~~~~~~~~~~~~


**Information System Service**

In any case, the default values in the configuration files can be changed, to customize the *Information System Service* behaviour. The Table below lists the available configuration properties. Excluding the ``*.port`` properties, any other properties can be changed at the runtime.  

.. csv-table:: **Properties Options**
   :header: Property Name,Default Value,Type,Description
   :widths: 20, 10, 5, 40
   :stub-columns: 1
   :delim: ;

   
    common.mode;multi;String;The property indicates whether the IS server will run in 'single' or 'multi' mode. **single:** *1 user, 1 application, 1 deployment.* **multi:** *Multiple users, applications and deployments*. When operating in multi mode an extra data source endpoint is needed to provide this information. For the purposes of CELAR the IS can only operate in multi mode
    common.collector;celar;String;Indicates the 'bundle of' connectors that will be used to obtain the needed data
    dev.debug;TRUE;Boolean;If this option is true the service with log additional information for debugging purposes
    log.location;/;String;The path where the log files will be saved
    srv.port;8282;Integer ;The port which the service will listen to.
    mgm.port;8383;Integer ;Management Interface / Socket Properties.
    sampling.presampling;FALSE;Boolean;Indicates whether the sampling will be applied before the statistical operations or after. 
    trend.sma.window;10;Integer;Sampling Moving Average window defines the smoothing windows for creating the trending line. **0:** *automatic*
    trend.parallel.threads;4;Integer;The number of parallel that will be used during the trend calculation. **0:** *automatic*
    sampling.threshhold;0.9;Double ;Sampling threshold defines the portion of the initial data that will be used as the sample.


   
To configure *Information System Service* a user must alter the files in
::
  
  /usr/local/bin/celarISServerDir/resources/config

The file ``server.properties`` contains the initialization and configuration values of the Inforamtion System Service. More specifically the property ``common.collector`` needs to be set to ``celar`` (which is the default value) if the Information System is installed under the CELAR umbrella or it should be set to ``dunmmy`` if someone wants to run Information System in a standalone mode e.g. for testing purposes. While the `mode` is set to ``dummy`` the service generates random data to showcase its functionality.  

The property ``srv.port``, in the same configuration file, indicates the port where the service listens for Rest API Calls.  

In a second step the properties at the path
::

   /usr/local/bin/celarISServerDir/resources/config/celar/endpoint.celarmanager.properties
	
need to be set to the correct CELAR Manager url parameters


**Information System Frontend**

The only parameter that needs to be configured for the *Information System Frontend* is the *Information System Service* address (isserver.ip) in order for those two to communicate. For the purposes of CELAR, the Information System Frontend is installed alongside with the Information System Service, at the CELAR Server. Thus, the default value of the ``isserver.ip`` is ``localhost``.

The *Information System Frontend* can be configured after its installation, by altering the files in

    {extracted_webapp_folder}/config

More specifically the property ``issendpoint.ip`` in the ``init.properties`` file should be set to the address that the *Information System Service* listens.


CELAR Orchestrator
^^^^^^^^^^^^^^^^^^
The CELAR Orchestrator Module is responsible for the enforcement of the resizing actions, as those are decided by the Decision Making Module. Furthermore, the CELAR Orchestrator provides an API used by any interested module to obtain information regarding the current deployment state, past resizing actions along with their status, etc.

Installation
~~~~~~~~~~~~
You can install the CELAR Orchestrator module through the following command (run as root):

``$ yum install -y celar-orchestrator-rpm``

This command installs the CELAR Orchestrator along with all the needed dependencies.

Configuration
~~~~~~~~~~~~~
The configuration file of the CELAR Orchestrator module is located at ``/opt/celar/celar-orchestrator/conf/orchestrator.properties``. Below you can find a sample configuration file along with the default options:

::

 # At least one option from server.{plain,ssl}.port must be provided!
 # unencrypted traffic port
 server.plain.port = 80

 # SSL configurations -- password and path will be filled during the installation
 server.ssl.port = 443
 server.ssl.keystore.path = 
 server.ssl.keystore.password = 

 # SlipStream properties
 slipstream.deployment.id = 
 slipstream.server.host = 
 slipstream.username = 
 slipstream.connector.name = 

 # CELAR Server properties
 celar.server.host = 
 celar.server.port = 


 # RSybl properties
 rsybl.host = localhost
 rsybl.port = 8280

 # CELAR DB properties
 backend=postgresql
 postgresql.host = 109.231.126.66
 postgresql.port=5432
 postgresql.username=celaruser
 postgresql.password=celar-user
 postgresql.db_name=celardb

 # CSAR properties
 # if csar.path variable is set the orchestrator will not need to contact
 # the CELAR Server for fetching it. Plz use it with your own risk: the CSAR
 # files used to describe, deploy and forwarded to the DM  must be identical else
 # you might face Undefined behavior.
 csar.path = 

The ``server.*`` parameters refer to the server configurations regarding its connectivity. By default, the server wait for HTTP connections in port 80 (unencrypted communication). SSL connections are also enabled if the ssl port is set (by default port 443); in that case the java keystore path must be set along with the keystore password. These values are initialized during the installation process of the rpm package, where a keystore with a self signed certificate is created, protected with a random password. The user can freely change this keystore with their own.

The ``slipstream.*`` properties are dynamically configured when the command ``$ service celar-orchestrator start`` command is issued: the init script parses a configuration file dynamically created  by SlipStream and fills the necessary fields. These fields are necessary for the connectivity of the Orchestrator to SlipStream.

The ``celar.*`` properties define the host and the port of the CELAR Manager (also auto filled during the init process) and the ``rsybl.*`` properties are used to point to a running Decision Making Module (by default, the Decision Making Module runs at the same host with the CELAR Orchestrator module). The ``backend`` and ``postgresql.*`` properties point to a running CELAR DB instance, where information regarding the deployment state, the enforced actions, etc. are stored and become available to the rest CELAR Modules. 

Finally, the ``csar.path`` property points to a CSAR file containing the Application Description along with the deployment policies and deployment artifacts. This field is, by default, empty. It can be used for debugging reasons where the user must specify the path of a valid CSAR file. 

After the configuration of the CELAR Orchestrator, the user must restart the Orchestrator by issuing:

``$ service celar-orchestrator restart``


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
