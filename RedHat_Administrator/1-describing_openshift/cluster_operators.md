# Describing Cluster Operators

## Introducing Kubernetes Operators

## Introducing the Operator Framework

See if there is any operators in our default namespace
<pre>
oc get operators
</pre>

Cluster wide
<pre>
oc get operators --all-namespaces
</pre>

Get ALL cluster operators
<pre>
oc get co | less
</pre>

### OperatorHub
Provided web interface to discover and publish operators that follow the Operator Framework standards.

## Introducing Red Hat Marketplace

## Introducing OpenShift Cluster Operators
OpenShift cluster operators provide OpenShift extension APIs and infrastructure services such as
- The OAuth server, which authenticates access to the control plane and extensions APIs.
- The core DNS server, which manages service discovery inside the cluster.
- the web console, which allows graphical management of the cluster.
- The internal image registry, which allow developers to host container images inside the cluster, using either S2I or another mechanism.
- The monitoring stack, which generates metrics, and alerts about the cluster health. 