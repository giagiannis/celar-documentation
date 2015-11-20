Example usage
=============
In this section we will provide examples of the platform's usage.

Application description and submission
--------------------------------------
To describe the application, you must first install CAMF, as described `previously <https://docs.celarcloud.eu/sections/install.html#camf>`_. 

+ Switch to the CAMF Perspective (Window -> Open Perspective -> Cloud Application Management Framework)
        **Note:** Make sure that the CELAR platform already runs on the target provider.

        - Now add the target provider Endpoints 
        - Open Window -> Preferences ( Eclipse -> Preferences if you are using an Apple Mac)
        - Click CAMF Preferences
        - Create New Provider connection

                + From the list select *CELAR Provider*

                        - Add Name and HTTP endpoint and click Finish 
                        - Depending on the deployed application, this will be the URL of CELAR Manager endpoint either in Okeanos or Flexiant 

+ Create a new Cloud Project 
        - Right click on the Cloud Project view (left pane)
        - New -> Other -> Cloud Project
                - Enter name
                - Select the Target Cloud Provider (Okeanos or Flexiant)
                - Click Finish

      *Note* Project is now created, and directory structure created on the Cloud Project View

+ Creating you Application Blueprint (TOSCA Representation)
        - Right click on the Application Descriptions folder
        - New -> Other -> Application Description
        - Enter filename and click Finish
        
+ Import any artifacts if needed to the Cloud project (if any)
        - Right click on the Artifact folders -> Deployment Scripts
                - Here you can import Bash scripts (.sh), public keys (.pub), or authorized keys
        - Right click on the Artifacts -> Applications 
                - Here you can import binaries, executables, etc.
        - Right click on the Artifacts -> Reconfiguration Scripts
        
         *Note:* The Application Modelling editor is now open in the centre pane. Also the Resource Palette is now available on the right pane.

+ Create all Application Components from the elements on the palette
        - Select either of these components from the palette App. Server, Database Server, Load Balancer
        - Drag and Drop to the canvas the component(s) you require
        - Drag and Drop form the Image compartment, the most suitable machine images to each component
        - From the Monitoring compartment, add different monitor probes to each component.
        - Add keypairs (if needed)

+ Defining the Properties of each component     (lower pane)

  - Specify Flavor (Main tab) and number of min/max instances
  - Specifying Elasticity policies or strategies (elasticity tab)

    + In the Elasticity Constraints Section (Left)

      - Click *Add* to add a new elasticity contraint to the component
      - Select the *Type* of constraint (depending on the monitoring probes added to the application)
      - Select the equality operator (=, <, >, !=,...)
      - Input the constraint threshold value

    + In the Elasticity Strategy Section (Right)

      - Click *Add* to add a new elasticity strategy to the component
      - Select the predefined elasticity strategy (Scale-In, Scale-Out, Vertical, Diagonal, etc)
      - Add any Pre/Post scale action (script) by clicking the file  
      - *Note:* Pre/Post scale scripts should be added in the Artifacts folder in advance.

  - Repeat the above for all components that need to be elastically scaled.

+ Save your Application Blueprint
+ Right click on the TOSCA file and click Application Submission
+ TOSCA is now saved in the Application Deployment folder
+ Right click on this file and click Application Deployment
+ Select the target provider if not already selected and click Finish.
+ The application now should be deployed on the provider.
+ Use the Application Deployment View (lower pane) to view its status (automatically refreshed).

Example: Cassandra cluster stressed by YCSB
---------------------------------------------
In this example, we are following the guidelines presented at the previous section, and we describe a Cassandra NoSQL cluster. The cluster can scale horizontally (by adding/removing nodes when the load is high/low respectively), and also, in specific time frames the data can be rebalanced, i.e. key value are forced to be equally distributed between the cluster nodes. This is a high level resizing action which does not involve cloud level orchestration, i.e. no resources are allocates/deallocated. Furthermore, we are also describing the installation of multiply YCSB nodes, that run the popular NoSQL benchmark and create load to the application. For more details regarding Cassandra and YCSB you can visit the following links (respectively): `link1 <http://cassandra.apache.org/>`_, `link2 <https://labs.yahoo.com/news/yahoo-cloud-serving-benchmark>`_.

Deployment Artifacts
^^^^^^^^^^^^^^^^^^^^
The "deployment artifacts" consist of any necessary script, configuration file, ssh key, etc. needed during the deployment process of the application. In this section we provide those artifacts for this pilot application. Our application consists of three types of ndoes:
  - the Cassandra seed node, that determines which keys are assigned to which node
  - the Cassandra node, that hold the data
  - the YCSB client, that creates the load to the cluster

The following script is used to install the Cassandra seed node:

::

 #!/bin/bash
 ip=$(ss-get hostname)
 hostname=$(hostname)
 echo $ip $hostname >> /etc/hosts
 echo $ip > /tmp/seedNodeIP

 wget http://javadl.sun.com/webapps/download/AutoDL?BundleId=78697 -O jre.tar.gz
 tar xfz jre.tar.gz
 jre=`ls | grep jre1.7`
 echo $jre
 mkdir /usr/lib/jvm
 mv $jre /usr/lib/jvm/
 ln -s /usr/lib/jvm/$jre/bin/java /etc/alternatives/java
 ln -s /usr/lib/jvm/$jre/bin/java /usr/bin/java
 ln -s /usr/lib/jvm/$jre/bin/java /usr/local/bin/java


 export JAVA_HOME=/usr/lib/jvm/$jre
 echo export JAVA_HOME=/usr/lib/jvm/$jre >> /root/.bashrc
 mkdir /local
 cd /local/

 #download balancer jar
 wget http://147.102.3.52/balancer-0.0.1-SNAPSHOT.jar

 #download hector api jar 
 wget http://147.102.3.52/hector-core-1.1-4.jar

 #download cassandra
 wget http://archive.apache.org/dist/cassandra/1.2.5/apache-cassandra-1.2.5-bin.tar.gz
 tar xfz apache-cassandra-1.2.5-bin.tar.gz

 master=$ip

 # cassandra-env change
 sed -i 's/# JVM_OPTS="$JVM_OPTS -Djava.rmi.server.hostname=<public name>"/JVM_OPTS="$JVM_OPTS -Djava.rmi.server.hostname='$ip'"/g' /local/apache-cassandra-1.2.5/conf/cassandra-env.sh

 sed -i 's/-Xss180k/-Xss228k/g' /local/apache-cassandra-1.2.5/conf/cassandra-env.sh

 # cassandra.yaml change
 sed -i "s/initial_token:/initial_token: '30303030303030303030303030303030303030'/g" /local/apache-cassandra-1.2.5/conf/cassandra.yaml
 sed -i 's/seeds: "127.0.0.1"/seeds: "'$ip'"/g' /local/apache-cassandra-1.2.5/conf/cassandra.yaml
 sed -i 's/listen_address: localhost/listen_address: '$ip'/g' /local/apache-cassandra-1.2.5/conf/cassandra.yaml
 sed -i 's/rpc_address: localhost/rpc_address: '$ip'/g' /local/apache-cassandra-1.2.5/conf/cassandra.yaml
 sed -i 's/rpc_address: localhost/rpc_address: '$ip'/g' /local/apache-cassandra-1.2.5/conf/cassandra.yaml
 sed -i 's/org.apache.cassandra.dht.Murmur3Partitioner/org.apache.cassandra.dht.ByteOrderedPartitioner/g' /local/apache- cassandra-1.2.5/conf/cassandra.yaml 

 apache-cassandra-1.2.5/bin/cassandra 
 sleep 60 
 ss-set ready true

 multiplicity=$(ss-get cassandraNode:multiplicity)
 echo "multiplicity=$multiplicity"
 
 for i in `seq 1 $multiplicity`
 do
    echo "Waiting node $i"
    ss-get --timeout 3600 cassandraNode.$i:ready
 done

 echo -e "create keyspace usertable \n with placement_strategy = 'org.apache.cassandra.locator.SimpleStrategy' \n and  strategy_options = {replication_factor:1}; \nuse usertable; \ncreate column family data; \nshow keyspaces;\n" > schema.cql

 ./apache-cassandra-1.2.5/bin/cassandra-cli -h $master -p 9160 -f schema.cql

 ss-set loaded true
 ./apache-cassandra-1.2.5/bin/nodetool -host $master status


The deployment script for the Cassandra node is the following:
:: 

 #!/bin/bash 
 ip=$(ss-get hostname)
 hostname=$(hostname)
 echo $ip $hostname >> /etc/hosts

 wget http://javadl.sun.com/webapps/download/AutoDL?BundleId=78697 -O jre.tar.gz
 tar xfz jre.tar.gz
 jre=`ls | grep jre1.7`
 echo $jre
 mkdir /usr/lib/jvm
 mv $jre /usr/lib/jvm/
 #rm /etc/alternatives/java
 ln -s /usr/lib/jvm/$jre/bin/java /etc/alternatives/java
 ln -s /usr/lib/jvm/$jre/bin/java /usr/bin/java
 ln -s /usr/lib/jvm/$jre/bin/java /usr/local/bin/java

 export JAVA_HOME=/usr/lib/jvm/$jre
 echo export JAVA_HOME=/usr/lib/jvm/$jre >> /root/.bashrc

 mkdir /local
 cd /local/

 #download balancer jar
 wget http://147.102.3.52/balancer-0.0.1-SNAPSHOT.jar

 #download cassandra
 wget http://archive.apache.org/dist/cassandra/1.2.5/apache-cassandra-1.2.5-bin.tar.gz
 tar xfz apache-cassandra-1.2.5-bin.tar.gz

 master=$(ss-get --timeout 360 cassandraSeedNode.1:hostname)
 echo $master seednode >> /etc/hosts


 ss-get --timeout 360 cassandraSeedNode.1:ready


 multiplicity=$(ss-get cassandraNode:multiplicity)
 echo "multiplicity=$multiplicity"

 #configure cassandra
 # cassandra-env change
 sed -i 's/# JVM_OPTS="$JVM_OPTS -Djava.rmi.server.hostname=<public name>"/JVM_OPTS="$JVM_OPTS -Djava.rmi.server.hostname='$ip'"/g' /local/apache-cassandra-1.2.5/conf/cassandra-env.sh
 sed -i 's/-Xss180k/-Xss228k/g' /local/apache-cassandra-1.2.5/conf/cassandra-env.sh

 # cassandra.yaml change
 # set token
 index=$(ss-get id)
 echo "index=$index"
 pscript="print (2**50 / ($multiplicity * $index))"
 echo $pscript
 token_num=`python -c "$pscript"`
 token=`java -cp balancer-0.0.1-SNAPSHOT.jar gr.ntua.ece.cslab.balancing.cassandra.Utils $token_num`

 echo "token=$token"

 sed -i "s/initial_token:/initial_token: '"$token"'/g" /local/apache-cassandra-1.2.5/conf/cassandra.yaml
 sed -i 's/seeds: "127.0.0.1"/seeds: "'$master'"/g' /local/apache-cassandra-1.2.5/conf/cassandra.yaml
 sed -i 's/listen_address: localhost/listen_address: '$ip'/g' /local/apache-cassandra-1.2.5/conf/cassandra.yaml
 sed -i 's/rpc_address: localhost/rpc_address: '$ip'/g' /local/apache-cassandra-1.2.5/conf/cassandra.yaml
 sed -i 's/var\/lib\/cassandra/local\/cassandra\/data/g' /local/apache-cassandra-1.2.5/conf/cassandra.yaml
 sed -i 's/org.apache.cassandra.dht.Murmur3Partitioner/org.apache.cassandra.dht.ByteOrderedPartitioner/g' /local/apache- cassandra-1.2.5/conf/cassandra.yaml

 #start cassandra
 apache-cassandra-1.2.5/bin/cassandra 
 sleep 60
 ss-set ready true

Finally, this is the script for the YCSB node:

::

 #!/bin/bash 
 ip=$(ss-get hostname)
 hostname=$(hostname)
 echo $ip $hostname >> /etc/hosts

 wget http://javadl.sun.com/webapps/download/AutoDL?BundleId=78697 -O jre.tar.gz
 tar xfz jre.tar.gz
 jre=`ls | grep jre1.7`
 echo $jre
 mkdir /usr/lib/jvm
 mv $jre /usr/lib/jvm/
 #rm /etc/alternatives/java
 ln -s /usr/lib/jvm/$jre/bin/java /etc/alternatives/java
 ln -s /usr/lib/jvm/$jre/bin/java /usr/bin/java
 ln -s /usr/lib/jvm/$jre/bin/java /usr/local/bin/java


 export JAVA_HOME=/usr/lib/jvm/$jre
 echo export JAVA_HOME=/usr/lib/jvm/$jre >> /root/.bashrc

 cd /opt/

 wget http://147.102.3.52/mycsb.tar.gz
 tar xfz mycsb.tar.gz
 cd YCSB-master

 master=$(ss-get --timeout 360 cassandraSeedNode.1:hostname)
 echo $master seednode >> /etc/hosts

 ss-get --timeout 100000 cassandraSeedNode.1:loaded

 bin/ycsb load cassandra-10 -threads 20 -P workloads/workloada -p hosts=$master -p recordcount=1000000 -s &> /opt/ycsb.log 

All the above scripts are compatible with SlipStream: every ``ss-*`` command is provided by the SlipStream client (terminal client) which provides synchronization during an application deployment. The exact same scripts can also be deployed into SlipStream.

Furthermore, deployment artifacts are also provided for the elastic scaling of the application, since each resizing action is accompanied by a resizing script. In this case we have three resizing actions/scripts:
 - addition of a Cassandra node (no resizing script is needed here)
 - removal of a Cassandra node (the daemon must me decommissioned instead of just powered off, since we want a graceful stop and don't want to lose data)
 -  data balancing (that executed a Java class implementing a data balancing algorithm).



Application Structure and Policy Making
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
After providing the necessary deployment artifacts, we can now compile a single application ready to be deployed into a cloud provider. In the following Figure we provide a screenshot of CAMF, depicting the Cassandra application we previously described:

.. image:: https://docs.celarcloud.eu/_static/camf.png
   :height: 600 px
   :width: 800 px
   :scale: 100 %
   :alt: alternate text
   :align: right

The last step before actually deploying, is the definition of the resizing policies. The design of good policies is a tedious task that demands knowledge about the application. The application expert is able to combine application-level metrics (if necessary) with actions and, simultaneously, the Decision Making Module will target to maximize the objectives of the user according to the described policies.

In the depicted example, we created the following policy:
 - Given the constraint that the deployment cost must be kept below 1.5$, minimize the latency.

In this `video <https://www.youtube.com/watch?v=tOVbQ-w6bnw>`_, we provide a video demonstration of this scenario. You can see that the cluster is scaled out when the (sinusoidal) load increases and the cluster is scaled in in the opposite case. Another approach, presented at the second part of the previous video demo, could be:
 - Given the constraint that the cluster latency is below 12 ms, minimize cost.

The big difference between the two policies is highlighted by the application's behavior; In the first case the number of nodes of the cluster follows the load (more nodes on high load, less nodes on low load), where in the second case CELAR increases the number of nodes in order to satisfy the constraint and, when the constraint is satisfied, the cost is minimized. 

After the policies are defined, the application can then be described and deployed. 

**Note**: The final application description is exported as a CSAR file and it can be found `here <https://docs.celarcloud.eu/_static/balancer.csar>`_. 


Deployment and Monitoring
^^^^^^^^^^^^^^^^^^^^^^^^^
After the application is deployed, the user can find information about it through their SlipStream account. Specifically, a successful deployment will create a new application deployment entry into the SlipStream dashboard, and the user can be notified about the deployment status through the SlipStream User Interface. When the deployment is ready, the provisioning process start automatically: the application is already monitored, the Decision Making Modules evaluates the application's health and status and actions are ready to be enforced by the CELAR Orchestrator module. The user can be informed real-time about their applications by using the CELAR User Interfaces. Specifically, the user can:
 * Identify the ``ORCHESTRATOR_IP``, through the SlipStream UI.
 * The application status can be monitored through the following UIs:

  1. http://ORCHESTRATOR_IP:8080/JCatascopia-Web/
  2. http://ORCHESTRATOR_IP:8280/rSYBL - decision module UI
  3. http://ORCHESTRATOR_IP:8180/MELA - MELA Data Service - shows only monitoring data (no requirements )
  4. http://ORCHESTRATOR_IP:8181/MELA - MELA Space and Pathway Analysis Service - shows monitoring data, requirements, and has options to see Space and Pathway
  5. http://ORCHESTRATOR_IP:8182/MELA - MELA Composite Cost Evaluation Service - shows cost decomposition and highlighted cost data along normal monitoring data
  6. http://ORCHESTRATOR_IP:8183/MELA - MELA Elasticity Relationships Analysis Service - shows dependencies between monitored metrics - targeted for offline analysis

Finally, the user can also use the Information System of the CELAR Platform, through Eclipse to obtain more information regarding the monitoring stats, past deployments, executed resizing actions etc.

Links
^^^^^
 * Video demo: https://www.youtube.com/watch?v=tOVbQ-w6bnw
 * CSAR file:  https://docs.celarcloud.eu/_static/balancer.csar

