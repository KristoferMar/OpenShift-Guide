# Controlling Pod Scheduling Behavior

First we login and create a project
<pre>
[kris@workstation ~]$ oc login -u developer -p developer \
>    https://api.ocp4.example.com:6443
Login successful.

...output omitted...

[kris@workstation ~]$ oc new-project schedule-pods
Now using project "schedule-pods" on server "https://api.ocp4.example.com".

...output omitted...
</pre>

## Deploy and scale a test application
Create a new application named hello using the container located at quay.io/redhattraining/hello-world-nginx:v1.0
<pre>
[kris@workstation ~]$ oc new-app --name hello \
>    --image quay.io/redhattraining/hello-world-nginx:v1.0
...output omitted...
--> Creating resources ...
    imagestream.image.openshift.io "hello" created
    deployment.apps "hello" created
    service "hello" created
--> Success
...output omitted...
</pre>

Create a route to the application
<pre>
[kris@workstation ~]$ oc expose svc/hello
route.route.openshift.io/hello exposed
</pre>

Manually scale the application so there are four running pods
<pre>
[kris@workstation ~]$ oc scale --replicas 4 deployment/hello
deployment.apps/hello scaled
</pre>

Verify that the four running pods are distributed between the nodes.
<pre>
[kris@workstation ~]$ oc get pods -o wide
NAME                     READY   STATUS    ...   IP           NODE       ...
hello-6c4984d949-78qsp   1/1     Running   ...   10.9.0.30    master02   ...
hello-6c4984d949-cf6tb   1/1     Running   ...   10.10.0.20   master01   ...
hello-6c4984d949-kwgbg   1/1     Running   ...   10.8.0.38    master03   ...
hello-6c4984d949-mb8z7   1/1     Running   ...   10.10.0.19   master01   ...
</pre>

Prepare the nodes so that application workloads can be distributed to either development or production nodes by assigning the env label. Assign the env=dev label to the master01 node and the env=prod label to the master02 node.

Log in to your OpenShift cluster as the admin user. A regular user does not have permission to view information about nodes and cannot label nodes.
<pre>
[kris@workstation ~]$ oc login -u admin -p redhat
Login successful.

...output omitted...
</pre>

Verify that none of the nodes use the env label
<pre>
[kris@workstation ~]$ oc get nodes -L env
NAME       STATUS   ROLES           AGE   VERSION           ENV
master01   Ready    master,worker   42d   v1.23.3+e419edf
master02   Ready    master,worker   42d   v1.23.3+e419edf
master03   Ready    master,worker   42d   v1.23.3+e419edf
</pre>

Add the env=dev label to the master01 node to indicate that it is a development node.
<pre>
[kris@workstation ~]$ oc label node master01 env=dev
node/master01 labeled
</pre>

Add the env=prod label to the master02 node to indicate that it is a production node.
<pre>
[kris@workstation ~]$ oc label node master02 env=prod
node/master02 labeled
</pre>

Verify that the nodes have the correct env label set. Make note of the node that has the env=dev label, as you will check later to see if the application pods have been deployed to that node.
<pre>
[kris@workstation ~]$ oc get nodes -L env
NAME       STATUS   ROLES           AGE   VERSION           ENV
master01   Ready    master,worker   42d   v1.23.3+e419edf   dev
master02   Ready    master,worker   42d   v1.23.3+e419edf   prod
master03   Ready    master,worker   42d   v1.23.3+e419edf
</pre>

Switch back to the developer user and modify the hello application so that it is deployed to the development node. After verifying this change, delete the schedule-pods project.

Log in to your OpenShift cluster as the developer user.
<pre>
[kris@workstation ~]$ oc login -u developer -p developer
Login successful.

...output omitted...

Using project "schedule-pods".
</pre>

Modify the deployment resource for the hello application to select a development node. Make sure to add the node selector in the spec group in the template section.
<pre>
[kris@workstation ~]$ oc edit deployment/hello
</pre>

Add the highlighted lines below to the deployment resource, indenting as shown.
<pre>
...output omitted...
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      nodeSelector:
        env: dev
      restartPolicy: Always
...output omitted...
</pre>

The following output from oc edit is displayed after you save your changes.
<pre>
deployment.apps/hello edited
</pre>

Verify that the application pods are deployed to the node with the env=dev label. Although it make take a little time to redeploy, the application pods must be deployed to the master01 node.
<pre>
[kris@workstation ~]$ oc get pods -o wide
NAME                   READY  STATUS   RESTARTS  AGE  IP          NODE      ...
hello-b556ccf8b-8scxd  1/1    Running  0         80s  10.10.0.14  master01  ...
hello-b556ccf8b-hb24w  1/1    Running  0         77s  10.10.0.16  master01  ...
hello-b556ccf8b-qxlj8  1/1    Running  0         80s  10.10.0.15  master01  ...
hello-b556ccf8b-sdxpj  1/1    Running  0         76s  10.10.0.17  master01  ...
</pre>

Remove the schedule-pods project
<pre>
[kris@workstation ~]$ oc delete project schedule-pods
project.project.openshift.io "schedule-pods" deleted
</pre>

Finish cleaning up this portion of the exercise by removing the env label from all nodes.

Log in to your OpenShift cluster as the admin user.
<pre>
[kris@workstation ~]$ oc login -u admin -p redhat
Login successful.

...output omitted...
</pre>

Remove the env label from all nodes that have it.
<pre>
[kris@workstation ~]$ oc label node -l env env-
node/master01 labeled
node/master02 labeled
</pre>

The schedule-pods-ts project contains an application that runs only on nodes that are labeled as client=ACME. In the following example, the application pod is pending and you must diagnose the problem using the following steps:

Log in to your OpenShift cluster as the developer user and ensure that you are using the schedule-pods-ts project.
<pre>
[kris@workstation ~]$ oc login -u developer -p developer
Login successful.

...output omitted...

Using project "schedule-pods-ts".
</pre>

If the output above does not show that you are using the schedule-pods-ts project, switch to it.
<pre>
[kris@workstation ~]$ oc project schedule-pods-ts
Now using project "schedule-pods-ts" on server ...
</pre>

Verify that the application pod has a status of Pending.
<pre>
[kris@workstation ~]$ oc get pods
NAME                       READY   STATUS    RESTARTS   AGE
hello-ts-5dbff9f44-w6csj   0/1     Pending   0          6m19s
</pre>

Because a pod with a status of pending does not have any logs, check the details of the pod using the oc describe pod command to see if describing the pod provides any useful information.
<pre>
[kris@workstation ~]$ oc describe pod hello-ts-5dbff9f44-8h7c7
...output omitted...
QoS Class:       BestEffort
Node-Selectors:  client=acme
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            ...               Message
  ----     ------            ...               -------
  Warning  FailedScheduling  ...  0/3 nodes are available: 3 node(s) didn't match node selector.
</pre>

Based on this information, the pod should be scheduled to a node with the label client=acme, but none of the three nodes have this label.

Log in to your OpenShift cluster as the admin user and verify compute node label. To verify it, run the oc get nodes -L client to list the details of the available nodes.
<pre>
[kris@workstation ~]$ oc login -u admin -p redhat
Login successful.

...output omitted...

[kris@workstation ~]$ oc get nodes -L client

NAME       STATUS   ROLES           AGE   VERSION           CLIENT
master01   Ready    master,worker   10d   v1.23.3+e419edf   ACME
</pre>

The information provided indicates that at least one compute node has the label client=ACME. You have found the problem. The application must be modified so that it uses the correct node selector.

Log in to your OpenShift cluster as the developer user and edit the deployment resource for the application to use the correct node selector.
<pre>
[kris@workstation ~]$ oc login -u developer -p developer
Login successful.

...output omitted...

[kris@workstation ~]$ oc edit deployment/hello-ts
</pre>

Change acme to ACME as shown below.
<pre>
...output omitted...
      dnsPolicy: ClusterFirst
      nodeSelector:
        client: ACME
      restartPolicy: Always
...output omitted...
</pre>

The following output from oc edit is deiplayed after jeg save your changes.
<pre>
deployment.apps/hello-ts edited
</pre>

Verify that the a new application pod is deployed and has a status of Running.
<pre>
[kris@workstation ~]$ oc get pods
NAME                        READY   STATUS    RESTARTS   AGE
hello-ts-69769f64b4-wwhpc   1/1     Running   0          11s
</pre>