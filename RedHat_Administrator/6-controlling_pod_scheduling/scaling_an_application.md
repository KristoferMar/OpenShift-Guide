# Scaling an Application

We start off by logging in and creating a project
<pre>
[student@workstation ~]$ oc login -u developer -p developer \
>    https://api.ocp4.example.com:6443
Login successful.

...output omitted...

[student@workstation ~]$ oc new-project schedule-scale
Now using project "schedule-scale" on server "https://api.ocp4.example.com:6443".

...output omitted...
</pre>

Deploy a test application for this exercise which explicitly requests container resources for CPU and memory.

Modify the resource file located at ~/DO280/labs/schedule-scale/loadtest.yaml to set both requests and limits for CPU and memory usage.
<pre>
[student@workstation ~]$ vim ~/DO280/labs/schedule-scale/loadtest.yaml
</pre>

Replace the resources: {} line with the highlighted lines listed below. Ensure that you have proper indentation before saving the file.
<pre>
...output omitted...
    spec:
      containers:
      - image: quay.io/redhattraining/loadtest:v1.0
        name: loadtest
        resources:
          requests:
            cpu: "25m"
            memory: 25Mi
          limits:
            cpu: "100m"
            memory: 100Mi
status: {}
</pre>

Create the new application using your resource file.
<pre>
[student@workstation ~]$ oc create --save-config \
>    -f ~/DO280/labs/schedule-scale/loadtest.yaml
deployment.apps/loadtest created
service/loadtest created
route.route.openshift.io/loadtest created
</pre>

Verify that your application pod has a status of Running. You might need to run the oc get pods command multiple times.
<pre>
[student@workstation ~]$ oc get pods
NAME                        READY   STATUS    RESTARTS   AGE
loadtest-5f9565dbfb-jm9md   1/1     Running   0          23s
</pre>

Verify that your application pod specifies resource limits and requests.
<pre>
[student@workstation ~]$ oc describe pod/loadtest-5f9565dbfb-jm9md | \
>    grep -A2 -E "Limits|Requests"
    Limits:
      cpu:     100m
      memory:  100Mi
    Requests:
      cpu:        25m
      memory:     25Mi
</pre>

Manually scale the loadtest deployment by first increasing and then decreasing the number of running pods.

Scale the loadtest deployment up to five pods.
<pre>
[student@workstation ~]$ oc scale --replicas 5 deployment/loadtest
deployment.apps/loadtest scaled
</pre>

Verify that all five application pods are running. You might need to run the oc get pods command multiple times.
<pre>
[student@workstation ~]$ oc get pods
NAME                        READY   STATUS    RESTARTS   AGE
loadtest-5f9565dbfb-22f9s   1/1     Running   0          54s
loadtest-5f9565dbfb-8l2rn   1/1     Running   0          54s
loadtest-5f9565dbfb-jm9md   1/1     Running   0          3m17s
loadtest-5f9565dbfb-lfhns   1/1     Running   0          54s
loadtest-5f9565dbfb-prjkl   1/1     Running   0          54s
</pre>

Scale the loadtest deployment back down to one pod.
<pre>
[student@workstation ~]$ oc scale --replicas 1 deployment/loadtest
deployment.apps/loadtest scaled
</pre>

Verify that only one application pod is running. You might need to run the oc get pods command multiple times while waiting for the other pods to terminate.
<pre>
[student@workstation ~]$ oc get pods
NAME                        READY   STATUS    RESTARTS   AGE
loadtest-5f9565dbfb-prjkl   1/1     Running   0          72s
</pre>

Configure the loadtest application to automatically scale based on load, and then test the application by applying load.

Create a horizontal pod autoscaler that ensures the loadtest application always has 2 application pods running; that number can be increased to a maximum of 10 pods when CPU load exceeds 50%.
<pre>
[student@workstation ~]$ oc autoscale deployment/loadtest \
>    --min 2 --max 10 --cpu-percent 50
horizontalpodautoscaler.autoscaling/loadtest autoscaled
</pre>

Wait until the loadtest horizontal pod autoscaler reports usage in the TARGETS column.
<pre>
[student@workstation ~]$ watch oc get hpa/loadtest
</pre>

Press Ctrl+C to exit the watch command after <'unknown> changes to a percentage.
<pre>
Every 2.0s: oc get hpa/loadtest             ...

NAME      REFERENCE            TARGETS    MINPODS  MAXPODS  REPLICAS  ...
loadtest  Deployment/loadtest  0%/50%     2        10       2         ...
</pre>

The loadtest container image is designed to either increase CPU or memory load on the container by making a request to the application API. Identify the fully-qualified domain name used in the route.
<pre>
[student@workstation ~]$ oc get route/loadtest
NAME       HOST/PORT                                      ...
loadtest   loadtest-schedule-scale.apps.ocp4.example.com  ...
</pre>

Access the application API to simulate additional CPU pressure on the container. Move on to the next step while you wait for the curl command to complete.
<pre>
[student@workstation ~]$ curl -X GET \
>    http://loadtest-schedule-scale.apps.ocp4.example.com/api/loadtest/v1/cpu/1
curl: (52) Empty reply from server
</pre>

Open a second terminal window and continuously monitor the status of the horizontal pod autoscaler.
<pre>
[student@workstation ~]$ watch oc get hpa/loadtest
</pre>

As the load increases (visible in the TARGETS column), you should see the count under REPLICAS increase. Observe the output for a minute or two before moving on to the next step. Your output will likely be different from what is displayed below.
<pre>
Every 2.0s: oc get hpa/loadtest             ...

NAME      REFERENCE            TARGETS    MINPODS  MAXPODS  REPLICAS  ...
loadtest  Deployment/loadtest  172%/50%   2        10       9         ...
</pre>

Back in the first terminal window, create a second application named scaling. Scale the application, and then verify the responses coming from the application pods.

Create a new application with the oc new-app command using the container image located at quay.io/redhattraining/scaling:v1.0.
<pre>
[student@workstation ~]$ oc new-app --name scaling \
>    --image quay.io/redhattraining/scaling:v1.0
...output omitted...
--> Creating resources ...
    imagestream.image.openshift.io "scaling" created
    deployment.apps "scaling" created
    service "scaling" created
--> Success
    Application is not exposed.
    You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/scaling'
    Run 'oc status' to view your app.
</pre>

Create a route to the application by exposing the service for the application.
<pre>
[student@workstation ~]$ oc expose svc/scaling
route.route.openshift.io/scaling exposed
</pre>

Scale the application up to three pods using the deployment resource for the application.
<pre>
[student@workstation ~]$ oc scale --replicas 3 deployment/scaling
deployment.apps/scaling scaled
</pre>

Verify that all three pods for the scaling application are running, and identify their associated IP addresses.
<pre>
[student@workstation ~]$ oc get pods -o wide -l deployment=scaling
NAME             READY  STATUS   RESTARTS  AGE   IP          NODE      ...
scaling-1-bm4m2  1/1    Running  0         45s   10.10.0.29  master01  ...
scaling-1-w7whl  1/1    Running  0         45s   10.8.0.45   master03  ...
scaling-1-xqvs2  1/1    Running  0         6m1s  10.9.0.58   master02  ...
</pre>

Display the host name used to route requests to the scaling application.
<pre>
[student@workstation ~]$ oc get route/scaling
NAME      HOST/PORT                                     ...
scaling   scaling-schedule-scale.apps.ocp4.example.com  ...
</pre>

When you access the host name for your application, the PHP page will output the IP address of the pod that replied to the request. Send several requests to your application, and then sort the responses to count the number of requests sent to each pod. Run the script located at ~/DO280/labs/schedule-scale/curl-route.sh.
<pre>
[student@workstation ~]$ ~/DO280/labs/schedule-scale/curl-route.sh
     34 Server IP: 10.10.0.29
     34 Server IP: 10.8.0.45
     32 Server IP: 10.9.0.58
</pre>

 Check the status of the horizontal pod autoscaler running for the loadtest application.

 If the watch oc get hpa/loadtest command is still running in the second terminal window, switch to it and observe the output. Provided enough time has passed, the replica count should be back down to two.
 <pre>
 Every 2.0s: oc get hpa/loadtest             ...

NAME      REFERENCE            TARGETS  MINPODS  MAXPODS  REPLICAS  ...
loadtest  Deployment/loadtest  0%/50%   2        10       2         ...
</pre>

When finished, press Ctrl+C to exit the watch command, and then close the second terminal window.