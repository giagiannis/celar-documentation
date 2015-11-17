HA Deployment
=====================
In this section we will describe a Highly Available installation of the CELAR Platform. The general architecture of the HA installation is provided in the following figure.

.. image:: https://placeholdit.imgix.net/~text?txtsize=33&txt=350%C3%97150&w=350&h=150


The installation consists of a cluster of endpoint servers, a cluster of CELAR Server instances and a cluster of database servers. In this tutorial, we are describing an architecture where the database server instances are located in dedicated VMs; However, according to the scale and the usage of the CELAR Platform, the database servers could be placed into the same hosts with the CELAR Manager instances. 

Before the installation and the configuration of the HA components, the following prerequisites should be satisfied:

1.  A hot-plugable public IP address, notated as ``PUBLIC_IP`` and used as the endpoint of the platform, should be available.
2.  A cluster of (at least) two VM instances used as the endpoint servers should be initialized. Each VM should contain at least 2 cores, 2GB of RAM and 10GB disk. The IP addresses of the nodes of this cluster will be referenced as ``endpoint1`` and ``endpoint2`` from  now on.
3.  A cluster of (at least) two VM instances used as the CELAR Server instances should be initialized. Each VM should contain at least 4 cores, 4GB of RAM and 10GB disk. The IP addresses of the nodes of this cluster will be referenced as ``celar-server1`` and ``celar-server2`` from  now on.
4.  A cluster of (at least) two VM instances used as the Database Server instances should be initialized. Each VM should contain at least 4 cores, 4GB of RAM and 50GB disk. The IP addresses of the nodes of this cluster will be referenced as ``celar-db1`` and ``celar-db2`` from  now on.

Each of the endpoint servers contains an **HAProxy** instance that redirects the traffic to the CELAR Server instances, a **pgpool** instance, used to redirect the traffic to the Postgresql Servers and a **keepalived** daemon, used to ensure that, in case of failure, another available endpoint server will be utilized. 

The installation details refer to installation over CentOS systems (tested for CentOS 6.4 and CentOS 7). If you want to install the HA platform into different distributions, please refer to the web sites of each component separately, as listed below.  

HAProxy
-------
First, we need to download and install HAProxy. From the ``endpoint1`` console, we are executing (as root):

``$ yum install -y haproxy``

If you want to compile from source, or install another version please visit `haproxy.org <http://www.haproxy.org/#docs>`_.

After the installation is finished, we need to edit the configuration file of HAProxy, located at ``/etc/haproxy/haproxy.cfg``. We open it and append the following lines:

::

 listen celar-server 0.0.0.0:8080
   mode http
   stats enable
   stats uri /haproxy?stats
   stats realm Strictly\ Private
   stats auth username:password
   balance roundrobin
   option httpclose
   option forwardfor
   server celar-server1 celar-server1-ip:8080 check
   server celar-server2 celar-server2-ip:8080 check

You need to replace the **username** and **password** strings above with your own choices (used to have access to the stats screen of HAProxy). If the CELAR Manager is configured to point to a different point than the default 8080, then you need to alter the conf file accordingly. Finally, make sure that the **celar-server1** and **celar-server2** entries on the ``/etc/hosts`` file, point to ``celar-server1`` and ``celar-server2`` respectively. 
We then need to restart the HAProxy instance,

``$ service haproxy restart``

And then check the stats screen of the by pointing to http://``endpoint1``:5000/haproxy?stats.
Finally, we repeat the same procedure for ``endpoint2`` and the rest of the machines of the endpoint cluster (if more than two).

keepalived
----------
`keepalived.org <http://www.keepalived.org/documentation.html>`_


pgpool
------
1. Master Configuration
""""""""""""""""""""""""""

``/var/lib/pgsql/data/postgresql.conf``
::

 listen_addresses = '*'
 wal_level = hot_standby
 synchronous_commit = local
 max_wal_senders = 3
 wal_keep_segments = 8
 synchronous_standby_names = '*'

``/var/lib/pgsql/data/pg_hba.conf``
::

 local   all             all                                     trust
 host    all             all             127.0.0.1/32            trust
 host    all             all             ::1/128                 trust
 host   replication     replicator        0.0.0.0/0        md5
 host   all     all        0.0.0.0/0     md5


Restart the PostgreSQL service to load changes

$ ``service postgresql restart`` 

Create replicator user with REPLICATION permissions (run as user "postgres")
 
$ ``psql -U postgres -c "CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'celar-db';"``

Flush iptables 

$ ``iptables -F #flush iptables``

Create testuser (as user "postgres")

$ ``psql -U postgres -c "CREATE USER testuser createdb PASSWORD 'test';"``

Create the "testuser" database (as user "testuser")

$ ``psql postgres testuser -c "CREATE database  testuser;"``

1. Slave Configuration
""""""""""""""""""""""""""

3. PGPool Server  Configuration
"""""""""""""""""""""""""""""""

We will need to edit the PGPool configuration file in order to reflect the master-slave streaming replication configuration that we have set-up to our PostgreSQL cluster.

(The following settings are for a cluster of 1 master ``celardb-master`` and 1 replica node ``celardb-replica-1``).

Sample entries for the PGPool config file in ``/usr/local/etc/pgpool.conf``
::

 listen_addresses = '*'
 port = 9999
 # - Backend Connection Settings -
 backend_hostname0 = 'celardb-master'
 backend_port0 = 5432
 backend_weight0 = 1
 backend_data_directory0 = '/var/lib/pgsql/data'
 backend_flag0 = 'ALLOW_TO_FAILOVER'
 backend_hostname1 = 'celardb-replica-1'
 backend_port1 = 5432
 backend_weight1 = 1
 backend_data_directory1 = '/var/lib/pgsql/data'
 backend_flag1 = 'ALLOW_TO_FAILOVER'
 
 pool_passwd = 'pool_passwd'        # File name of pool_passwd for md5 authentication. "" disables pool_passwd.
 connection_cache = on
 replication_mode = off        
 replicate_select = off
 load_balance_mode = on
 # Master-Slave mode 
 master_slave_mode = on
 master_slave_sub_mode = 'stream'
 # Streaming replication check user/password
 sr_check_user = 'testuser'
 sr_check_password = 'test'
 # Health check user password
 health_check_user = 'testuser'
 health_check_password = 'test'
 # failover called with %d: node id, %M old master node ID, %m new master node ID
 failover_command = '/usr/local/etc/my_failover.sh %d %M %m \n $(date)
 fail_over_on_backend_error = on
 recovery_user = 'nobody'

