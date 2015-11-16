HA Deployment
=====================
HAProxy
-------
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

keepalived
----------
