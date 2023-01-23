# Deploying and Managing Applications on an OpenShift Cluster


<h1>OpenShift</h1>

In the folder in this repo you can find usefull commands to remember
<br>
- OpenShift projects are Kubernetes namespaces with additional administrative functions. Therefore, projects also provide isolation within an OpenShift cluster.

- OpgenShift comes with a with a copy of kubectl so all the kubectl commands can be run with oc.

<h2>Operators</h2>
An Operator is essentially a custom controller.

- But not all controllers are operators.

- It's a pattern used to automate tasks in a cluster. <br>
- It's a way to package, deploy and manage a Kubernetes native application <br>

<h2>Pods</h2>
Each pod is allocated its own internal IP address, therefore owning its entire port space.

Compute resources managed by Quota Across pods in a nonTerminal state
- Cpu
    - requests.cpu
- memory
    - requests.memory
- limits.cpu
- limits.memory

<h3>Containers</h3>
Containers are located within pods and they can share their local storage ant networking.

Default setting for a container:


<h2>Services</h3>
It's an internal load balancer. It identifies a set of replicated pods in order to proxy the connections it receives to them.

<h2>Labels</h2>
A label is a key-pair value tag taht can be applied to most resources in OpenShift to give extra meaning or context to that resource for later filtering for selecting. A selector is a prameter used by many resources in OpenShift to associate a resource with another resource by specifying a label.

<h2>ectd</h2>
It's the key-value store for OpenShift Container Platform, which persists the state of all resource objects. Back up your cluster's etcd data regularly and store in a secure location ideally outside the OphenShift Container Platform Environment.

<h2>Scheduler</h2>
The default OpenShift Container Platform pod scheduler is responsible for determining placement of new pods onto nodes within the cluster. It reads data from the pod and tries to find a node that is a good fit based on configured policies. It is completely independent and exists as a standalone/pluggable solution.

<h2>Registry</h2>
OpenShift Container Platform provides an integrated container registry called OpenShift Container Registry (OCR) that adds the ability to automatically provision new image repositories on demand. This provides users with a built-in location for their application builds to push the resulting images.

<h2>Router</h2>
The OpenShift router is the ingress point for all external traffic destined for services in your OpenShift installation. OpenShift provides and supports the following two router plug-ins: The HAProxy template router is the default plug-in.

<h2>Replication Controller</h2>
A replication controller ensures that a specified number of replicas of a pod are running at all times. If pods exit or are deleted, the replica controller acts to instantiate more up to the desired number. ... The number of replicas desired (which can be adjusted at runtime).

<h2>OpenShift Networking SDN</h2>
Overview. OpenShift uses a software-defined networking (SDN) approach to provide a unified cluster network that enables communication between containers across the OpenShift cluster. This cluster network is established and maintained by the OpenShift SDN, which configures an overlay network using Open vSwitch (OVS).

<h2>quota</h2>
A resource quota, defined by a ResourceQuota object, provides constraints that limit aggregate resource consumption per project. It can limit the quantity of objects that can be created in a project by type, as well as the total amount of compute resources that may be consumed by resources in that project.

<h2>Logging</h2>
<h3>Fluentd</h3>
OpenShift Container Platform uses Fluentd to collect operations and application logs from your cluster which OpenShift Container Platform enriches with Kubernetes Pod and Namespace metadata. You can configure log rotation, log location, use an external log aggregator, and make other configurations.

- It gathers logs from nodes and feeds them to Elasticsearch

<h2>OpenShift metrics collection facilities</h2>
The kubelet exposes metrics that can be collected and stored in back-ends by Heapster. As an OpenShift Container Platform administrator, you can view a cluster's metrics from all containers and components in one user interface.

<h3>Kibana</h3>
OpenShift Container Platform uses Kibana to display the log data collected by Fluentd and indexed by Elasticsearch. You can scale Kibana for redundancy and configure the CPU and memory for your Kibana nodes. You must set cluster logging to Unmanaged state before performing these configurations, unless otherwise noted.

<h2>Projects</h2>
 project allows a community of users to organize and manage their content in isolation from other communities. Projects are OpenShift extentions to Kubernetes namespaces, with additional features to enable user self-provisioning. For the vast majority of purposes, they are interchangeable.

 <h1>Application deployment</h1>

<h3>Recreate deployment stratigy</h3>
Recreate. The recreate strategy is a dummy deployment which consists of shutting down version A then deploying version B after version A is turned off. This technique implies downtime of the service that depends on both shutdown and boot duration of the application.

<h3>Triggers</h3>
Webhook triggers allow you to trigger a new build by sending a request to the OpenShift Container Platform API endpoint. You can define these triggers using GitHub webhooks or Generic webhooks.

<h3>BuildConfig</h3>
A build in OpenShift Container Platform is the process of transforming input parameters into a resulting object. Most often, builds are used to transform source code into a runnable container image. A build configuration, or BuildConfig , is characterized by a build strategy and one or more sources.

<h3>Image stream</h3>
An image stream comprises any number of Docker-formatted container images identified by tags. It presents a single virtual view of related images, similar to an image repository, and may contain images from any of the following: Its own image repository in OpenShift Enterprise's integrated registry.
