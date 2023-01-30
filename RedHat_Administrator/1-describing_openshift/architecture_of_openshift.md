# Describing the Red Hat OpenShift Container Platform

## The declerative Architecture of Kubernetes

## The OpenShift Control Plane

## Describing OpenShift Extensions
A lot of functionality depends on external components, such as ingress controllers, storage plug-ins, and authentication plug-ins.

## The OpenShift Default Storage Class
Openshift ships with integrated storage plug-ins and storage classes that rely on the underlying cloud or virtualization platform to provide dynamically provisioned storage.

OpenShift cluster administrators can later define additional storage classes that use different EBS service tiers. For example, you could have one storage class for high-performance storage that sustains a high input-output operations per second (IOPS) rate, and another storage class for low-performance, low-cost storage. Cluster administrators can then allow only certain applications to use the high-performance storage class, and configure data archiving applications to use the low-performance storage class.