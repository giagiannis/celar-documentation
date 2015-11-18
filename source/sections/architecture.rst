#####################
General Architecture
#####################

=============
Overview
=============

 The following schematic presents an overview of CELAR's architecture
 
 .. image:: architecture.png

 The next sections will explain the function of the various components of CELAR, on the following conceptual layers.

 1. Application Management Platform
 2. Cloud Information and Performance Monitoring
 3. Elasticity Platform

==========================
1. Application Management Platform
==========================

------------------------------------------------------------------------------------------
Cloud Application Management Framework (CAMF)
-----------------------------------------------------------------------------------------
Application deployment and management in Infrastructure as a Service (IaaS) Clouds
can be a complex and time consuming endeavor, typically requiring manual effort on the
users' behalf and relying on vendor-specific, proprietary tools.
Therefore, it becomes evident
that there is a need for application management tools that facilitate the description of
applications in a vendor neutral manner, enabling easy application deployment and
management, considering elasticity.

CELAR has 
invested significant effort towards developing an infrastructure-agnostic Application
Management Framework and making it available to the open-source community. c-
Eclipse is now an official Eclipse Technology project and is now called the Cloud
Application Management Framework (CAMF).

CAMF provides a generic Application Management Framework that: 
 (i) is open- source and implemented on top of the reliable Eclipse platform 
 (ii) offers graphical tools to facilitate the description of an application's structure and its lifecycle management operations 
 (iii) adopts the TOSCA open specification for blueprinting Cloud applications and consequently packaging them in portable archives that can be processed by any compliant provider
 (iv) adopts the SYBL language that enables the description of  elasticity policies for Cloud applications and provides the necessary tools for elasticity  policy specification at different levels of an application's structure.

CAMF can be provided by the Cloud vendors in order to enable their users to
configure, deploy and manage applications on their infrastructures. This is beneficial for
both vendors and users; Cloud vendors can integrate CAMF to their Cloud architectures
to attract more customers to use their infrastructure via its GUI; and users are able to
describe the deployment and management lifecycle of their applications with minimal
effort and in a portable way.



================================
2. Cloud Information and Performance Monitoring
================================

The CELAR platform incorporates intelligent decision-
making algorithms to support multi-dimensional and multi-grained elasticity control.
CELAR evaluates elasticity at multiple levels of the cloud stack and considers the impact
of enforced actions in relation to cost, quality and allocated resources for elastic cloud
services. 

To support such a complex, automated and intelligent platform, as
well as, provide this functionality in a vendor-neutral manner, real-time monitoring and
elasticity behavior analysis of heterogeneous types of information collected from
different and multiple data sources, is required. This important role, in
the CELAR software stack, is the job of the Cloud Information and Performance Monitor
Layer. The Cloud Information and Performance Monitor Layer runs alongside all the layers
of the cloud infrastructure, in order to provide the intelligent decision making mechanisms of the
CELAR System with real-time, cost-enriched, monitoring metrics.

The Cloud Information and Performance Monitor
Layer is comprised of 3 components:

 (i) The CELAR Monitoring System, which is a fully-automated, multi-layer and platform  independent monitoring system that runs in a non-intrusive and transparent manner to any underlying virtualized infrastructure;
 (ii) The Multi-Level Metrics Evaluation Module, which provides composable cost-related monitoring  information, by mapping collected monitoring metrics to the structure of the application and then enriching cost-wise the obtained metrics based on the cloud provider’s pricing  schemes; and 
 (iii) The CELAR Information System, which provides users with high-level analytic insights to their deployed cloud applications based on collected  monitoring information, cost-enriched metrics, as well as, cloud-platform specific  information, which is used to evaluate the behaviour of a specific deployment and even  compare multiple deployments in relation to performance, quality and cost.

In contrast to existing Cloud monitoring systems, or systems with
workarounds to support Cloud monitoring, the CELAR MS provides self-adaptive
mechanisms to minimize the intrusiveness of its presence, and enhanced filtering to
reduce communication costs for metric distribution. The Monitoring System’s elastic
properties are further enhanced with the ability to dynamically add/remove monitoring
instances located on multiple levels of the infrastructure without the need to restart the
monitoring system. Furthermore, the CELAR MS supports aggregation and grouping of
low-level metrics to generate high-level metrics at runtime

------------------------------------------------
2.1 Monitoring System (JCatascopia)
------------------------------------------------
The CELAR Monitoring System (CELAR MS), which is a fully-automated, multi-
layer and platform independent monitoring system that runs in a non-intrusive
and transparent manner to any underlying virtualized infrastructure. The MS is
utilized by CELAR to collect monitoring metrics from multiple levels of the
underlying cloud platform as well as performance metrics from deployed cloud
services and afterwards it distributes them to subscribed users and platform
operators. 

Subsequently, the CELAR Monitoring System is in charge of low-level system-wise monitoring information (i.e. CPU utilization, memory allocation, network traffic) and application-specific behavior and performance metrics (i.e. response time, latency, throughput, number of active users).

The following figure depicts an abstract view of the JCatascopia Monitoring System
architecture. JCatascopia is the implemented prototype of the CELAR Monitoring System.
Briefly, the architecture follows an agent-based approach. This approach provides a
scalable, real-time, cloud monitoring system that can be utilized to collect monitoring
metrics from multiple layers of the underlying infrastructure, as well as performance
metrics from the deployed cloud services. JCatascopia is platform independent and
during the metric collection process it takes into consideration the rapid changes that
occur due to the enforcement of elasticity actions on the application execution
environment and the cloud infrastructure.
 .. image:: jcatascopia.png

The main components which comprise the JCatascopia Monitoring System are
defined as follows:
 * Monitoring Agents: light-weight monitoring instances deployable on any cloud
   elements to be monitored, such as physical nodes or virtual instances. Monitoring
   Agents are the entities responsible for managing the metric collection process on
   the respective cloud element. Additional functionality of Monitoring Agents
   includes metric aggregation and metric distribution via a pub/sub metric
   delivery mechanism.
 * Monitoring Probes: the actual metric collectors managed by Monitoring Agents.
   Probes are responsible for gathering system-level and application performance
   metrics. Metrics are forwarded to the respected Monitoring Agent in either a
   push or pull manner. Additionally Probes offer a filtering mechanism, which
   allows for metric values to be filtered, in place, rather than distributing metrics
   and later discarding them at the Monitoring Server level. Users can take
   advantage of the JCatascopia Java Probe API to implement their own custom
   Monitoring Probes and then, deploy them dynamically, at runtime, to Monitoring
   Agents without the need to restart the monitoring process.
 * Monitoring Servers: entities which receive monitoring metrics from their
   respected Monitoring Agents. The acquired metrics are then processed and
   stored in the monitoring database. Monitoring Servers additionally, handle user
   metric and configuration requests. A hierarchy of Monitoring Servers can be
   utilized to enhance greater scalability.
 * Finally, JCatascopia provides a REST Web Service that allows for external
   entities such as application users or the Decision Module to access monitoring
   information stored in the Monitoring Database.

---------------------------------------------------------------
2.2 Multi-Level Metrics Evaluation Module
---------------------------------------------------------------

Multi-Level Metric Evaluation Module, provides monitoring information mapped to the application structure for the CELAR
Decision Module to address the task of Composable Cost
Evaluation. In this section we provide a short overview of the Multi-Level Metric
Evaluation Module and its implementation (MELA).

In general, elastic cloud applications can re-configure individual components or the
whole application at run-time, due to various elasticity requirements. Such run-time re-
configuration can either focus on: 

 (i) vertical scaling, adding/removing resources to existing components or 
 (ii) horizontal scaling, duplicating components/instantiating more instances of the same type. 

Due to individual horizontal scaling mechanisms, one
application component could “burst” by a controller, creating a new instance in unused
virtual machines along other components, or in new machines. This generates two main
issues. First, as components can dynamically appear/disappear on virtual machines, a
monitoring system for elastic applications must be able to dynamically start/stop
collecting metrics on individual virtual machines. Secondly, as virtual machines
themselves are created/destroyed dynamically at run-time, if monitoring information is
associated only with each virtual machine, it will be lost during scale-in operations
which destroy virtual machines.
While the first issue is solved by JCatascopia, which supports dynamic addition and
removal of metrics collected by a Monitoring Agent, for solving the Virtual Machine (VM)
volatility issue, we must associate monitoring information with the application
structure. Addressing this second issue is the initial objective of the CELAR Multi-Level
Metric Evaluation Module, implemented by MELA. Based on monitoring information
mapped to the application structure, MELA provides both real-time and historical
Composable Cost Evaluation according to different cloud pricing schemes.

The architecture of MELA is presented in the following figure:

 .. image:: mela.png

---------------------------------------------------------------
2.3  Information System
---------------------------------------------------------------

An Information System is the entity which purpose is to help the user interact with
the data stored in a system. Through such an Information System, the user has the
ability to search, view, and navigate through the available data of the underlying system.
The CELAR Information System (CELAR IS) is the CELAR component in charge of
providing, per cloud application, usage analytics by processing information regarding
resource utilization and cost-enriched monitoring data, collected during the lifecycle of
an elastic cloud service deployment. Furthermore, the CELAR IS enables Application
Users with the ability to query, explore, compare, and visualize historic data collected
from past deployments 1 and previous versions 2 of a cloud service.
Application Version Data and Deployment Data stored in the CELAR DataBase are 
accessible from the CELAR IS.

Deployment data consist of: 
 (i) deployment id
 (ii) start/end time of the deployment
 (iii) instantiated resources along with the duration they were allocated and the resource type.

The application version data,include:
 (i) version and application id
 (ii) submission time
 (iii) application structure along with the defined resources to be utilized

The following table provides an overview of the CELAR IS features and functionality
 
 .. image:: is_features.png

The next figure presents the High-Level architecture of CELAR IS.

 .. image:: is_arch.png

The CELAR IS consists of two high-level components, the Information System
Service and the Information System Frontend. Each one is a separate application, which
is distributed in its own package. The communication between these components is
achieved through the CELAR IS REST API that the first one exposes. Furthermore, any
other interested party can use the REST API to get data from the CELAR IS. The
Information System Service handles the computational tasks of the CELAR IS and it
includes the 
 (i) Data Collection Modules
 (ii) the Analysis Module 
 (iii) Controller modules.

Each module is further separated, to sub-modules and units which are responsible for the actual Information System Service
functionality. The Information System Frontend is responsible to visualize the data and
provide the means for the user to interact with it. The Application User can access this
functionality through the c-Eclipse Information Tool.

==========================
3. Elasticity Platform
==========================

The Elasticity Provisioning Platform hosts all central to CELAR operations and takes input
from almost all other CELAR modules. Its main functionality is to perform the appropriate
elasticity actions according to the monitored workload and the user-defined policies and to
ensure the correct application deployment according to user-defined descriptions. The
major platform modules are:

 1. the Decision Module, responsible for taking the elasticity decisions and
 2. the Resource Provisioner, responsible for the application of the elasticity decisions to the deployed application.

Apart from these modules, the complete CELAR platform also contains the Monitoring
System, which provides real-time metrics of the application's performance; this module
however is not considered as a part of the Elasticity Provisioning Platform (it belongs to
Cloud Information and Performance Monitor System).

An overview of the Elasticity Platform is presented in the following figure:

 .. image:: platform_arch.png

The CELAR platform comprises of the CELAR Server and CELAR Orchestrator container
components. The CELAR server is installed as a single virtual machine in each IaaS
deployment. For every different application execution, a new virtual machine is launched
and configured by the CELAR Server. This virtual machine is referred to as the CELAR
Orchestrator. In the next figure we provide a visualization of the deployment status of the
platform for an IaaS provider.

 .. image:: platform_deployment.png

The Elasticity Platform consists of the following discrete modules:
 1. CELAR Server
 2. CELAR DataBase
 3. CELAR Orchestrator
 4. Application Profiler
 5. Decision Making Module

The function of each of those components is explained in the next sections.

-----------------
3.1 CELAR Server
-----------------

The CELAR Server is the "static" part of the CELAR platform. It is deployed into a
dedicated VM inside the IaaS provider and acts as an endpoint between the platform and its
users. Its functionality can be summarized as follows:
 - Initiates new deployments.
 - Stores and serves data about past and current deployments.

CELAR Server is a container component and hosts all the static modules needed to trigger a
new application deployment. It contains:
 - the SlipStream Server, which is used for IaaS interaction and application orchestration for the newly deployed applications
 - the CELAR Manager, which acts as an endpoint between the platform and its clients,
 - the CELAR DB which stores application related data.

The CELAR Manager communicates both with the SlipStream Server and CELAR DB in order
to achieve the aforementioned objectives. Specifically, it will communicate with SlipStream
Server in order to deploy new applications and it will serve data from the CELAR DB. From a
technical aspect, their roles are separate. In order to enhance the readability,
we will use the term CELAR Server to indicate these three modules as a whole.

The CELAR Server exports a number of services to any interested party. Some of them are
used by the c-Eclipse platform, in order to describe and deploy new applications and some of
them are used by other modules (e.g., Information System, Decision-making module, etc).
Those services are exposed via a REST API.

---------------------
3.2 CELAR DataBase
---------------------

The CELAR DB is the central repository where information about any component or the
deployments done through the CELAR Platform is stored. The CELAR Manager, Orchestrator,
the Decision Module, the Monitoring System and the Information System all use information
stored in CELAR DB. However, not all components access this information in the same
manner.

Only the CELAR Manager and CELAR Orchestrator can access CELAR DB directly through a
native SQL interface. In particular, the CELAR Manager is tightly coupled with CELAR DB. It is
the gateway through which the rest of the CELAR Platform can access information stored in
CELAR DB. Nearly all the REST calls of the CELAR server retrieve or store data to the Database. The
intuition behind that is that the rest of the CELAR Platform is offered a comprehensive and
easy-to-use web interface that hides the complexity of managing data from the relational
model of the database. As a result, nearly all of the data available in CELAR DB go through
CELAR Manager. The CELAR Orchestrator also uses CELAR DB directly to retrieve and store
information regarding an application's current state.

The deveopment focus of the CELAR DB is given in its interoperability with other modules and
the flexibility of its deployment. It can  be deployed as its own package, independently of
CELAR-Server, in a different host.


-----------------------
3.3 CELAR Orchestrator
-----------------------

The CELAR Orchestrator is responsible for elastically scaling a specific application from its
initial deployment until its termination. This procedure entails a number of sub processes
(e.g., monitoring the application, taking decisions and applying them to the existing
 application, etc.), cooperating with each other to achieve their objective.
These processes are actually a part of the Monitoring System, the Decision Module
and the Application and Cloud Orchestrator. 

The CELAR Orchestrator VM is provisioned
when a new deployment request is received by the CELAR Server, and runs all the
aforementioned components as services. The CELAR Application Orchestrator process is
responsible for the deployment and resizing of the specific application. The CELAR
Orchestrator provides an API used by the Decision Module to enforce resizing actions and
retrieve information regarding the state of the resizing actions.

-------------------------
3.4 Application Profiler
-------------------------

The goal of the Application Profiler is to construct a knowledge-base for a newly described
application and enable the Decision Module to take "wiser" decisions at the first steps of the
deployment. The idea is that the user describes the configuration space into which the
application can be deployed along with some representative workloads that will be used to
stress the application. The profiler then deploys and stresses the application for some of
those configurations and uses statistical models to approximate the application performance
for the entire configuration space. This is then given as input to the Decision and Smart
Deployment Modules.

---------------------------
3.5 Decision Making Module
---------------------------

The Decision Making Module (DMM), part of the Elasticity Provisioning Platform, is in charge
of controlling the elasticity of applications, based on monitoring information coming from
CELAR Monitoring System, and elasticity requirements coming from CELAR Manager.
The next figure shows the Decision Making Module architecture.

 .. image:: dmm_arch.png

The DMM contributes to application’s elasticity from design time to run-time. At design time, the Smart Deployment
Service is in charge with application enrichment, enriching application scripts and application
description (e.g., adding missing software installation details), and with selecting the
appropriate set of resources that gives enough flexibility during runtime for achieving
application elasticity. 

The application description is received from the CELAR Manager, together with the
application deployment description, the application profile, and elasticity requirements
related to various components or composite components of the application.

The DMM core, rSYBL, takes this information and evaluates whether control actions are
necessary. The control actions can be easily defined in the primitive action configuration of
the DMM, and are enforced by the DMM through CELAR Manager, if CELAR Manager
exposes them. The action plan is generated in the
Planning Engine component of rSYBL, and enforced from the Cloud Interaction Unit of rSYBL.
The Data Elasticity Control sub-component is in charge with managing data elasticity
properties of the application, such as data balancing and data quality.

The DMM exposes a REST API, with its methods. Those are used by the other CELAR Modules.
DMM is by design able to control multiple applications at once (i.e., from the same user).
