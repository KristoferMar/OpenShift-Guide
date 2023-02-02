# Limiting Resource Usage by an Application

We start of by with login and project creation
<pre>
[kris@workstation ~]$ oc login -u developer -p developer \
>    https://api.ocp4.example.com:6443
Login successful.

...output omitted...

[kris@workstation ~]$ oc new-project schedule-limit
Now using project "schedule-limit" on server "https://api.ocp4.example.com:6443".

...output omitted...
</pre>

Deploy a test application for this exercise that explicitly requests container resources for CPU and memory.

Create a deployment resource file and save it to ~/DO280/labs/schedule-limit/hello-limit.yaml. Name the application hello-limit and use the container image located at quay.io/redhattraining/hello-world-nginx:v1.0.
<pre>
[kris@workstation ~]$ oc create deployment hello-limit \
>    --image quay.io/redhattraining/hello-world-nginx:v1.0 \
>    --dry-run=client -o yaml > ~/DO280/labs/schedule-limit/hello-limit.yaml
</pre>

Edit ~/DO280/labs/schedule-limit/hello-limit.yaml to replace the resources: {} line with the highlighted lines below.
<pre>
[kris@workstation ~]$ vim ~/DO280/labs/schedule-limit/hello-limit.yaml
</pre>

Ensure that you have proper indentation before saving the file.
<pre>
...output omitted...
    spec:
      containers:
      - image: quay.io/redhattraining/hello-world-nginx:v1.0
        name: hello-world-nginx
        resources:
          requests:
            cpu: "3"
            memory: 20Mi
status: {}
</pre>

Create the new application using your resource file.
<pre>
[kris@workstation ~]$ oc create --save-config \
>    -f ~/DO280/labs/schedule-limit/hello-limit.yaml
deployment.apps/hello-limit created
</pre>

Although a new deployment was created for the application, the application pod should have a status of Pending.
<pre>
[kris@workstation ~]$ oc get pods
NAME                            READY   STATUS      RESTARTS   AGE
hello-limit-d86874d86b-fpmrt    0/1     Pending     0          10s
</pre>

The pod cannot be scheduled because none of the compute nodes have sufficient CPU resources. This can be verified by viewing warning events.
<pre>
[kris@workstation ~]$ oc get events --field-selector type=Warning
LAST SEEN   TYPE      REASON             OBJECT                            MESSAGE
88s         Warning   FailedScheduling   pod/hello-limit-d86874d86b-fpmrt  0/3 nodes are available: 3 Insufficient cpu.
</pre>

## Redploy application so that it requires fewer CPU resources

Edit ~/DO280/labs/schedule-limit/hello-limit.yaml to request 1.2 CPUs for the container.
<pre>
[kris@workstation ~]$ vim ~/DO280/labs/schedule-limit/hello-limit.yaml
</pre>

Change the cpu: "3" line to match the highlighted line below.
<pre>
...output omitted...
        resources:
          requests:
            cpu: "1200m"
            memory: 20Mi
</pre>

Apply the changes to your application.
<pre>
[kris@workstation ~]$ oc apply -f ~/DO280/labs/schedule-limit/hello-limit.yaml
deployment.apps/hello-limit configured
</pre>

Verify that your application deploys successfully. You might need to run oc get pods multiple times until you see a running pod. The previous pod with a pending status will terminate and eventually disappear.
<pre>
[kris@workstation ~]$ oc get pods
NAME                           READY   STATUS        RESTARTS   AGE
hello-limit-d86874d86b-fpmrt   0/1     Terminating   0          2m19s
hello-limit-7c7998ff6b-ctsjp   1/1     Running       0          6s
</pre>

Attempt to scale your application from one pod to four pods. After verifying that this change would exceed the capacity of your cluster, delete the resources associated with the hello-limit application.

Manually scale the hello-limit application up to four pods.
<pre>
[kris@workstation ~]$ oc scale --replicas 4 deployment/hello-limit
deployment.apps/hello-limit scaled
</pre>

Check to see if all four pods are running. You might need to run oc get pods multiple times until you see that at least one pod is pending. Depending on the current cluster load, multiple pods might be in a pending state.
<pre>
[kris@workstation ~]$ oc get pods
NAME                         READY   STATUS    RESTARTS   AGE
hello-limit-d55cd65c-765s9   1/1     Running   0          12s
hello-limit-d55cd65c-hmblw   0/1     Pending   0          12s
hello-limit-d55cd65c-r8lvw   1/1     Running   0          12s
hello-limit-d55cd65c-vkrhk   0/1     Pending   0          12s
</pre>

Warning events indicate that one or more pods cannot be scheduled because none of the nodes has sufficient CPU resources. Your warning messages might be slightly different.
<pre>
[kris@workstation ~]$ oc get events --field-selector type=Warning
LAST SEEN   TYPE      REASON             OBJECT                             MESSAGE
...output omitted...
76s         Warning   FailedScheduling   pod/hello-limit-d55cd65c-vkrhk     0/3 nodes are available: 3 Insufficient cpu.
</pre>

Delete all of the resources associated with the hello-limit application.
<pre>
[kris@workstation ~]$ oc delete all -l app=hello-limit
pod "hello-limit-d55cd65c-765s9" deleted
pod "hello-limit-d55cd65c-hmblw" deleted
pod "hello-limit-d55cd65c-r8lvw" deleted
pod "hello-limit-d55cd65c-vkrhk" deleted
deployment.apps "hello-limit" deleted
replicaset.apps "hello-limit-5cc86ff6b8" deleted
replicaset.apps "hello-limit-7d6bdcc99b" deleted
</pre>

Deploy a second application to test memory usage. This second application sets a memory limit of 200MB per container.

Use the resource file located at ~/DO280/labs/schedule-limit/loadtest.yaml to create the loadtest application. In addition to creating a deployment, this resource file also creates a service and a route.
<pre>
[kris@workstation ~]$ oc create --save-config \
>    -f ~/DO280/labs/schedule-limit/loadtest.yaml
deployment.apps/loadtest created
service/loadtest created
route.route.openshift.io/loadtest created
</pre>

The loadtest container image is designed to increase either CPU or memory load on the container by making a request to the application API. Identify the fully-qualified domain name used in the route.
<pre>
[kris@workstation ~]$ oc get routes
NAME       HOST/PORT                       ...
loadtest   loadtest.apps.ocp4.example.com  ...
</pre>

## Generate additional memory load that can be handled by the container.

Open two additional terminal windows to continuously monitor the load of your application. Access the application API from the first terminal to simulate additional memory pressure on the container.

- From the second terminal window, run watch oc get pods to continuously monitor the status of each pod.

- From the third terminal, run watch oc adm top pod to continuously monitor the CPU and memory usage of each pod.

In the first terminal window, use the application API to increase the memory load by 150MB for 60 seconds. Use the fully-qualified domain name previously identified in the route. While you wait for the curl command to complete, observe the output in the other two terminal windows.
<pre>
[kris@workstation ~]$ curl -X GET \
>    http://loadtest.apps.ocp4.example.com/api/loadtest/v1/mem/150/60
curl: (52) Empty reply from server
</pre>

In the second terminal window, observe the output of watch oc get pods. Because the container can handle the additional load, you should see that the single application pod has a status of Running for the entire curl request.

<pre>
Every 2.0s: oc get pods             ...

NAME                      READY   STATUS    RESTARTS   AGE
loadtest-f7495948-tlxgm   1/1     Running   0          7m34s
</pre>

In the third terminal window, observe the output of watch oc adm top pod. The starting memory usage for the pod is about 20-30Mi.
<pre>
Every 2.0s: oc adm top pod             ...

NAME                      CPU(cores)   MEMORY(bytes)
loadtest-f7495948-tlxgm   0m           20Mi
</pre>

As the API request is made, you should see memory usage for the pod increase to about 170-180Mi.
<pre>
Every 2.0s: oc adm top pod             ...

NAME                      CPU(cores)   MEMORY(bytes)
loadtest-f7495948-tlxgm   0m           172Mi
</pre>

A short while after the curl request completes, you should see memory usage drop back down to about 20-30Mi.
<pre>
Every 2.0s: oc adm top pod             ...

NAME                      CPU(cores)   MEMORY(bytes)
loadtest-f7495948-tlxgm   0m           20Mi
</pre>

## Generate additional memory load that cannot be handled by the container
Use the application API to increase the memory load by 200MB for 60 seconds. Observe the output in the other two terminal windows.
<pre>
[kris@workstation ~]$ curl -X GET \
>    http://loadtest.apps.ocp4.example.com/api/loadtest/v1/mem/200/60
<'html><'body><'h1>502 Bad Gateway<'/h1>
The server returned an invalid or incomplete response.
<'/body><'/html>
</pre>

In the second terminal window, observe the output of watch oc get pods. Almost immediately after running the curl command, the status of the pod will transition to OOMKilled. You might even see a status of Error. The pod is out of memory and needs to be killed and restarted. The status might change to CrashLoopBackOff before returning to a Running status. The restart count will also increment.
<pre>
Every 2.0s: oc get pods                ...

NAME                      READY   STATUS      RESTARTS   AGE
loadtest-f7495948-tlxgm   0/1     OOMKilled   0          9m13s
</pre>

In some cases the pod might have restarted and changed to a status of Running before you have time to switch to the second terminal window. The restart count will have incremented from 0 to 1.
<pre>
Every 2.0s: oc get pods                ...

NAME                      READY   STATUS    RESTARTS   AGE
loadtest-f7495948-tlxgm   1/1     Running   1          9m33s
</pre>

In the third terminal window, observe the output of watch oc adm top pod. After the pod is killed, metrics will show that the pod is using little to no resources for a brief period of time.
<pre>
Every 2.0s: oc adm top pod             ...

NAME                      CPU(cores)   MEMORY(bytes)
loadtest-f7495948-tlxgm   8m           0Mi
</pre>

In the first terminal window, delete all of the resources associated with the second application.
<pre>
[kris@workstation ~]$ oc delete all -l app=loadtest
pod "loadtest-f7495948-tlxgm" deleted
service "loadtest" deleted
deployment.apps "loadtest" deleted
route.route.openshift.io "loadtest" deleted
</pre>

In the second and third terminal windows, press Ctrl+C to end the watch command. Optionally, close the second and third terminal windows.

## As a cluster administrator, create quotas for the schedule-limit project.
Log in to OpenShift cluster as the admin user.
<pre>
[kris@workstation ~]$ oc login -u admin -p redhat
Login successful.

...output omitted...
</pre>

Create a quota named project-quota that limits the schedule-limit project to 3 CPUs, 1 GB of memory, and 2 configuration maps.
<pre>
[kris@workstation ~]$ oc create quota project-quota \
>    --hard cpu="3",memory="1G",configmaps="2" \
>    -n schedule-limit
resourcequota/project-quota created
</pre>

As a developer, attempt to exceed the configuration map quota for the project.

Log in to OpenShift cluster as the developer user.
<pre>
[kris@workstation ~]$ oc login -u developer -p developer
Login successful.

...output omitted...
</pre>

Attempt to create a configuration map. All namespaces in the cluster have two configuration maps containing certificates required for the operation of the cluster. Creating a configuration map fails because a third configuration map exceeds the quota of two configuration maps.
<pre>
[kris@workstation ~]$ oc create configmap my-config
error: failed to create configmap: configmaps "my-config" is forbidden: exceeded quota: project-quota, requested: configmaps=1, used: configmaps=2, limited: configmaps=2
</pre>

As a cluster administrator, configure all new projects with default quota and limit range resources.

Log in to OpenShift cluster as the admin user.
<pre>
[kris@workstation ~]$ oc login -u admin -p redhat
Login successful.

...output omitted...
</pre>

Redirect the output of the oc adm create-bootstrap-project-template command so that you can customize project creation.
<pre>
[kris@workstation ~]$ oc adm create-bootstrap-project-template \
>    -o yaml > /tmp/project-template.yaml
</pre>

Edit the /tmp/project-template.yaml file to add the quota and limit range resources defined in the ~/DO280/labs/schedule-limit/quota-limits.yaml file. Add the lines before the parameters section.
<pre>
...output omitted...
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: ${PROJECT_NAME}-quota
  spec:
    hard:
      cpu: 3
      memory: 10G
- apiVersion: v1
  kind: LimitRange
  metadata:
    name: ${PROJECT_NAME}-limits
  spec:
    limits:
      - type: Container
        defaultRequest:
          cpu: 30m
          memory: 30M
parameters:
- name: PROJECT_NAME
- name: PROJECT_DISPLAYNAME
- name: PROJECT_DESCRIPTION
- name: PROJECT_ADMIN_USER
- name: PROJECT_REQUESTING_USER
</pre>

Use the /tmp/project-template.yaml file to create a new template resource in the openshift-config namespace.
<pre>
[kris@workstation ~]$ oc create -f /tmp/project-template.yaml \
>    -n openshift-config
template.template.openshift.io/project-request created
</pre>

Update the cluster to use the custom project template.
<pre>
[kris@workstation ~]$ oc edit projects.config.openshift.io/cluster
</pre>

Modify the spec section to use the following lines in bold.
<pre>
...output omitted...
spec:
  projectRequestTemplate:
    name: project-request
</pre>

Ensure proper indentation, and then save your changes.
<pre>
project.config.openshift.io/cluster edited
</pre>

After a successful change, the apiserver pods in the openshift-apiserver namespace are recreated.
<pre>
[kris@workstation ~]$ watch oc get pods -n openshift-apiserver
</pre>

Wait until all three new apiserver pods are ready and running.
<pre>
Every 2.0s: oc get pods -n openshift-apiserver             ...

NAME                         READY   STATUS    RESTARTS   AGE
apiserver-868dccf5fb-885dz   2/2     Running   0          63s
apiserver-868dccf5fb-8j4vh   2/2     Running   0          39s
apiserver-868dccf5fb-r4j9b   2/2     Running   0          24s
</pre>

Press Ctrl+C to end the watch command.

Create a test project to confirm that the custom project template works as expected.
<pre>
[kris@workstation ~]$ oc new-project template-test
Now using project "template-test" on server "https://api.ocp4.example.com:6443".

...output omitted...
</pre>

List the resource quota and limit range resources in the test project.
<pre>
[kris@workstation ~]$ oc get resourcequotas,limitranges
NAME                                AGE   REQUEST                   LIMIT
resourcequota/template-test-quota   87s   cpu: 0/3, memory: 0/10G

NAME                              CREATED AT
limitrange/template-test-limits   2021-06-02T15:46:37Z
</pre>