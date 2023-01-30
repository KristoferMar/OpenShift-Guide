# Describing Installation Methods

## Introducing OpenShift Installation Methods

### Full-stack Automation
Openshift installer provisions all compute, storage, and network resources from a cloud or virtualization provider. You provide the installer with minimum data, such as credentials to a cloud provider and the size of the inital clusterm and then the installer deploys a fully functional OpenShift cluster. 

### Pre-exisiting Infrastructure
You configure a set of compute, storage, and network resourdces and the OpenShift installer configures and inital cluster using these resources.

## Comparing OpenShift Installation Methods

## Describing the Deployment Process

## Customizing an OpenShift Installation
The OpenShift installer allows very little customization of the initial cluster that it provisions. Most
customization is performed after installation, including
- Defining custom storage classes for dynamic storage provisioning.
- Changing the custom resources of cluster operators.
- Adding new operators to a cluster.
- Defining new machine sets.