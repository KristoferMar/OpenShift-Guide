## Troubleshooting OpenShift Software-defined Networking

In this example we will spin up a frontend and database pod and investigate networking between them.

I login and create a project
<pre>
[kris@workstation ~]$ oc login -u developer -p developer \
>    https://api.ocp4.example.com:6443
Login successful.

...output omitted...

[kris@workstation ~]$ oc new-project network-sdn
Now using project "network-sdn" on server "https://api.ocp4.example.com:6443".
...output omitted...
</pre>

In the following location i have deployment files for my images.
<pre>
[kris@workstation ~]$ cd ~/DO280/labs/network-sdn

[kris@workstation network-sdn]$ ls
db-data.sql  todo-db.yaml  todo-frontend.yaml

[kris@workstation network-sdn]$ oc create -f todo-db.yaml
deployment.apps/mysql created
service/mysql created

[kris@workstation network-sdn]$ oc status
In project network-sdn on server https://api.ocp4.example.com:6443

svc/mysql - 172.30.223.41:3306
  deployment/mysql deploys registry.redhat.io/rhel8/mysql-80:1
    deployment #1 running for 4 seconds - 0/1 pods
...output omitted...

[kris@workstation network-sdn]$ oc get pods
NAME                    READY   STATUS    RESTARTS   AGE
mysql-94dc6645b-hjjqb   1/1     Running   0          33m
</pre>

Now copy sql script to populate the database and test everyhing looks good.
<pre>
[kris@workstation network-sdn]$ oc cp db-data.sql mysql-94dc6645b-hjjqb:/tmp/

[kris@workstation network-sdn]$ oc rsh mysql-94dc6645b-hjjqb /bin/bash
bash-4.4$ mysql -u root items < /tmp/db-data.sql

bash-4.4$ mysql -u root items -e "SHOW TABLES;"
+-----------------+
| Tables_in_items |
+-----------------+
| Item            |
+-----------------+

bash-4.4$ exit
exit
</pre>

Now i deploy the frontend application using the todo-frontend.yaml file and then i expose it to the outside world.
<pre>
[kris@workstation network-sdn]$ oc create -f todo-frontend.yaml
deployment.apps/frontend created
service/frontend created

[kris@workstation network-sdn]$ oc get pods
NAME                        READY   STATUS    RESTARTS   AGE
frontend-57b8b445df-f56qh   1/1     Running   0          34s
...output omitted...

[kris@workstation network-sdn]$ oc expose service frontend \
>    --hostname todo.apps.ocp4.example.com
route.route.openshift.io/frontend exposed

[kris@workstation network-sdn]$ oc get routes
NAME       HOST/PORT                    PATH   SERVICES   PORT   ...
frontend   todo.apps.ocp4.example.com          frontend   8080   ...
</pre>

The application should now be accesable on the following url but it will show that the application is not reachable on purpose.
<pre>
http://todo.apps.ocp4.example.com/todo/ 
</pre>

## The investigation
I new retrieve the database service IP. In a later sep we will use the curl command to access the database endpoint. The JSONPath expression allows you to retrieve the service IP.
<pre>
[kris@workstation network-sdn]$ oc get service/mysql \
>    -o jsonpath="{.spec.clusterIP}{'\n'}"
172.30.103.29
</pre>

We run the oc debug command against the frontend deployment, which runs the web application pod. 
<pre>
[kris@workstation network-sdn]$ oc debug -t deployment/frontend
Starting pod/frontend-debug ...
Pod IP: 10.131.0.144
If you don't see a command prompt, try pressing enter.
sh-4.4$
</pre>

One way to test the connectivity between the frontend and the database is using curl, which supports a veriety of protocols.

Use the curl to connect to the database over 3306, which is the MySQL default port. Make sure to replace the IP address with the one that you obtained previously for the mysql service. When finished, type Ctrl+c to exit the session, and then type exit to exit the debug pod.
<pre>
sh-4.4$ curl -v telnet://172.30.103.29:3306
* About to connect() to 172.30.103.29 port 3306 (#0)
*   Trying 172.30.103.29
* Connected to 172.30.103.29 (172.30.103.29) port 3306 (#0)
J
8.0.21
* RCVD IAC 2
* RCVD IAC 199
^C
sh-4.4$ exit
exit

Removing debug pod ...
</pre>

The output indicates that the database is up and running, and that it is accessible from the frontend deployment.

In the following steps, you ensure that the network connectivity inside the cluster is operational by connecting to the front end container from the database container.

Obtain some information about the frontend pod, and then use the oc debug command to diagnose the issue from the mysql deployment.

Before creating a debug pod, retrieve IP address of the frontend service.
<pre>
[kris@workstation network-sdn]$ oc get service/frontend \
>    -o jsonpath="{.spec.clusterIP}{'\n'}"
172.30.23.147
</pre>

Run the oc debug command to create a container for troubleshooting based on the mysql deployment. You must override the container image because the MySQL Server image does not provide the curl command.
<pre>
[kris@workstation network-sdn]$ oc debug -t deployment/mysql \
>    --image registry.access.redhat.com/ubi8/ubi:8.4
Starting pod/mysql-debug ...
Pod IP: 10.131.0.146
If you don't see a command prompt, try pressing enter.
sh-4.4$
</pre>

Use the curl command to connect to the frontend application over port 8080. Make sure to replace the IP address with the one that you obtained previously for the frontend service.

The following output indicates that the curl command times out. This could indicate either that the application is not running or that the service is not able to access the application.
<pre>
sh-4.4$ curl -m 10 -v http://172.30.23.147:8080
* Rebuilt URL to: http://172.30.23.147:8080/
*   Trying 172.30.23.147...
* TCP_NODELAY set
* Connection timed out after 10000 milliseconds
* Closing connection 0
curl: (28) Connection timed out after 10000 milliseconds
</pre>

We now exit the debug pod
<pre>
sh-4.4$ exit
exit

Removing debug pod ...
</pre>

In the following steps, we now connect to the frontend pod through its private IP. This allows testing, whether or not the issue is related to the service.

Retrieve the IP of the frontend pod.
<pre>
[kris@workstation network-sdn]$ oc get pods -o wide -l name=frontend
NAME                        READY   STATUS    RESTARTS   AGE   IP             ...
frontend-57b8b445df-f56qh   1/1     Running   0          39m   10.128.2.61   ...
</pre>

We create a debug pod from the mysql deployment
<pre>
[kris@workstation network-sdn]$ oc debug -t deployment/mysql \
>    --image registry.access.redhat.com/ubi8/ubi:8.4
Starting pod/mysql-debug ...
Pod IP: 10.131.1.27
If you don't see a command prompt, try pressing enter.
sh-4.4$
</pre>

We run the curl command in verbose mode against the frontend pod on port 8080. Replace the IP address with the one that you obtained previously for the frontend pod.
<pre>
sh-4.4$ curl -v http://10.128.2.61:8080/todo/
*   Trying 10.128.2.61...
* TCP_NODELAY set
* Connected to 10.128.2.61 (10.128.2.61) port 8080 (#0)
> GET /todo/ HTTP/1.1
> Host: 10.128.2.61:8080
> User-Agent: curl/7.61.1
> Accept: /
>
< HTTP/1.1 200 OK
...output omitted...
</pre>

The curl command can access the application through the pod's private IP.

## Review the configuration of the frontend service

List the services in the project and ensure that the frontend service exists.
<pre>
[kris@workstation network-sdn]$ oc get svc
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
frontend   ClusterIP   172.30.23.147   <'none>        8080/TCP    93m
mysql      ClusterIP   172.30.103.29   <'none>        3306/TCP    93m
</pre>

Review the configuration and status of the frontend service. Notice the value of the service selector that indicates to which pod the service should forward packets.
<pre>
[kris@workstation network-sdn]$ oc describe svc/frontend
Name:              frontend
Namespace:         network-sdn
Labels:            app=todonodejs
                   name=frontend
Annotations:       <'none>
Selector:          name=api
Type:              ClusterIP
IP:                172.30.23.147
Port:              <'unset>  8080/TCP
TargetPort:        8080/TCP
Endpoints:         <'none>
Session Affinity:  None
Events:            <'none>
</pre>

This output also indicates that the service has no endpoint, so it is not able to forward incoming traffic to the application.

Retrieve the labels of the frontend deployment. The output shows that pods are created with a name label that has a value of frontend, whereas the service in the previous step uses api as the value.
<pre>
[kris@workstation network-sdn]$ oc describe deployment/frontend | \
>    grep -A1 Labels
Labels:                 app=todonodejs
                        name=frontend
--
  Labels:  app=todonodejs
           name=frontend
</pre>

## Update the frontend service and access the application.

Run the oc edit command to edit the frontend service. Update the selector to match the correct label.
<pre>
[kris@workstation network-sdn]$ oc edit svc/frontend
</pre>

Locate the section that defines the selector, and then update the name: frontend label inside the selector. After making the changes, exit the editor.
<pre>
...output omitted...
  selector:
    name: frontend
...output omitted...
</pre>

Save your changes and verify that the oc edit command applied them.
<pre>
service/frontend edited
</pre>

Review the service configuration to ensure that the service has an endpoint.
<pre>
[kris@workstation network-sdn]$ oc describe svc/frontend
Name:              frontend
Namespace:         network-sdn
Labels:            app=todonodejs
                   name=frontend
Annotations:       <'none>
Selector:          name=frontend
Type:              ClusterIP
IP:                172.30.169.113
Port:              <'unset>  8080/TCP
TargetPort:        8080/TCP
Endpoints:         10.128.2.61:8080
Session Affinity:  None
Events:            <'none>
</pre>

From the workstation machine, open Firefox and access the To Do application at the following URL.

http://todo.apps.ocp4.example.com/todo/

and we now see it works.