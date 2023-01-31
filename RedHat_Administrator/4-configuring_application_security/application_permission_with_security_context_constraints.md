# Controlling Application Permissions with Security Context Constraints

## Security Context Constraints (SCCs)

Red Hat OpenShift provides security context constraints (SCCs), a security mechanism that
restricts access to resources, but not to operations in OpenShift.

SCCs limit the access from a running pod in OpenShift to the host environment. SCCs control:
• Running privileged containers.
• Requesting extra capabilities for a container
• Using host directories as volumes.
• Changing the SELinux context of a container.
• Changing the user ID.

You can run the following command as a cluster administrator to list the SCCs defined by
OpenShift

<pre>
[user@host ~]$ oc get scc
</pre>

OpenShift provides eight default SCCs:
• anyuid
• hostaccess
• hostmount-anyuid
• hostnetwork
• node-exporter
• nonroot
• privileged
• restricted

To get additional information about an SCC, use the oc describe command
<pre>
[user@host ~]$ oc describe scc anyuid
Name:
anyuid
Priority:
10
Access:
Users:
<none>
Groups:
system:cluster-admins
Settings:
...output omitted...
</pre>

Most pods created by OpenShift use the SCC named restricted, which provides limited access
to resources external to OpenShift. Use the oc describe command to view the security context
constraint that a pod uses.

<pre>
[user@host ~]$ oc describe pod console-5df4fcbb47-67c52 \
>
-n openshift-console | grep scc
openshift.io/scc: restricted
</pre>

The random
user ID used by the restricted SCC cannot start a service that listens on a privileged network
port (port numbers less than 1024). Use the scc-subject-review subcommand to list all the
security context constraints that can overcome the limitations of a container
<pre>
[user@host ~]$ oc get pod podname -o yaml | \
>
oc adm policy scc-subject-review -f -
</pre>

For the anyuid SCC, the run as user strategy is defined as RunAsAny, which means that the
pod can run as any user ID available in the container.

To change the container to run using a different SCC, you must create a service account bound to
a pod. Use the oc create serviceaccount command to create the service account, and use
the -n option if the service account must be created in a namespace different than the current
one
<pre>
[user@host ~]$ oc create serviceaccount service-account-name
</pre>


## Privileged Containers
Some containers might need to access the runtime environment of the host. For example, the
S2I builder containers are a class of privileged containers that require access beyond the limits of
their own containers. These containers can pose security risks because they can use any resources
on an OpenShift node. Use SCCs to enable access for privileged containers by creating service
accounts with privileged access.