# Verifying health of a cluster

- Red Hat OpenShift Container Platform provides two main installation methods: full-stack automation, and pre-existing infrastructure.

- Future releases are expected to add more cloud and virtualization providers, such as VMware, Red Hat Virtualization, and IBM System Z.

- An OpenShift node based on Red Hat Enterprise Linux CoreOS runs very few local services that would require direct access to a node to inspect their status. Most of the system services run as containers, the main exceptions are the CRI-O container engine and the Kubelet.

- The oc get node, oc adm top, oc adm node-logs, and oc debug commands provide troubleshooting information about OpenShift nodes.

# Troubleshooting OpenShift Clusters and Applications
