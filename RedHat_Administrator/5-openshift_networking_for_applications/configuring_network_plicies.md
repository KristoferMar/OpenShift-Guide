# Configuring Network Policies

We start off by logging in and creating a project
<pre>
[kris@workstation ~]$ oc login -u developer -p developer \
>    https://api.ocp4.example.com:6443
Login successful.

...output omitted...

[kris@workstation ~]$ oc new-project network-policy
Now using project "network-policy" on server "https://api.ocp4.example.com:6443".

...output omitted...
</pre>

We run two existing deployment files to create two apps.
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

[kris@workstation ~]$ oc new-app --name test \
>    --image quay.io/redhattraining/hello-world-nginx:v1.0
...output omitted...
--> Creating resources ...
    imagestream.image.openshift.io "test" created
    deployment.apps "test" created
    service "test" created
--> Success
...output omitted...
</pre>

Use the oc expose command to create a route to the hello service
<pre>
[kris@workstation ~]$ oc expose service hello
route.route.openshift.io/hello exposed
</pre>

Use the oc rsh and curl commands to confirm that the test pod can access the IP address of the hello pod.
<pre>
[kris@workstation ~]$ oc rsh test-c4d74c9d5-5pq9s \
>    curl 10.8.0.13:8080 | grep Hello
    <'h1>Hello, world from nginx!</'h1>
</pre>

Use the oc rsh and curl commands to confirm that the test pod can access the IP address of the hello service.
<pre>
[kris@workstation ~]$ oc rsh test-c4d74c9d5-5pq9s \
>    curl 172.30.137.226:8080 | grep Hello
    <'h1>Hello, world from nginx!<'/h1>
</pre>

Verify access to the hello pod using the curl command against the URL of the hello route.
<pre>
[kris@workstation ~]$ curl -s hello-network-policy.apps.ocp4.example.com | \
>    grep Hello
    <'h1>Hello, world from nginx!<'/h1>
</pre>

Create the network-test project and a deployment named sample-app.

Create the network-test project.
<pre>
[kris@workstation ~]$ oc new-project network-test
Now using project "network-test" on server "https://api.ocp4.example.com:6443".
...output omitted...
</pre>

Create the sample-app deployment with the quay.io/redhattraining/hello-world-nginx:v1.0 image. The web app listens on port 8080.
<pre>
[kris@workstation ~]$ oc new-app --name sample-app \
>    --image quay.io/redhattraining/hello-world-nginx:v1.0
...output omitted...
--> Creating resources ...
    imagestream.image.openshift.io "sample-app" created
    deployment.apps "sample-app" created
    service "sample-app" created
--> Success
...output omitted...
</pre>

### Verify that pods in a different namespace can access the hello and test pods in the network-policy namespace.

Returning to the first terminal, use the oc rsh and curl commands to confirm that the sample-app pod can access the IP address of the hello pod.
<pre>
[kris@workstation ~]$ oc rsh sample-app-d5f945-spx9q \
>    curl 10.8.0.13:8080 | grep Hello
    <'h1>Hello, world from nginx!<'/h1>
</pre>

Use the oc rsh and curl commands to confirm access to the test pod from the sample-app pod. Target the IP address previously retrieved for the test pod.
<pre>
[kris@workstation ~]$ oc rsh sample-app-d5f945-spx9q \
>    curl 10.8.0.14:8080 | grep Hello
    <'h1>Hello, world from nginx!<'/h1>
</pre>

From the network-policy project, create the deny-all network policy using the resource file available at ~/DO280/labs/network-policy/deny-all.yaml.

Switch to the network-policy project.
<pre>
[kris@workstation ~]$ oc project network-policy
Now using project "network-policy" on server "https://api.ocp4.example.com:6443".
</pre>

Go to the ~/DO280/labs/network-policy directory.

Use a text editor to update the deny-all.yaml file with an empty podSelector to target all pods in the namespace.
<pre>
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-all
spec:
  podSelector: {}
</pre>

Create the network policy with the oc create command.
<pre>
[kris@workstation network-policy]$ oc create -f deny-all.yaml
networkpolicy.networking.k8s.io/deny-all created
</pre>

Verify there is no longer access to the pods in the network-policy namespace.

Verify there is no longer access to the hello pod via the exposed route. Wait a few seconds, and then press Ctrl+C to exit the curl command that does not reply.
<pre>
[kris@workstation network-policy]$ curl -s \
>    hello-network-policy.apps.ocp4.example.com | grep Hello
^C
</pre>

Verify that the test pod can no longer access the IP address of the hello pod. Wait a few seconds, and then press Ctrl+C to exit the curl command that does not reply.
<pre>
[kris@workstation network-policy]$ oc rsh test-c4d74c9d5-5pq9s \
>    curl 10.8.0.13:8080 | grep Hello
^C
command terminated with exit code 130
</pre>

Switch to the network-test project.
<pre>
[kris@workstation network-policy]$ oc project network-test
Now using project "network-test" on server "https://api.ocp4.example.com:6443".
</pre>

Confirm that the sample-app pod can no longer access the IP address of the test pod. Wait a few seconds, and then press Ctrl+C to exit the curl command that does not reply.
<pre>
[kris@workstation network-policy]$ oc rsh sample-app-d5f945-spx9q \
>    curl 10.8.0.14:8080 | grep Hello
^C
command terminated with exit code 130
</pre>

Create a network policy to allow traffic to the hello pod in the network-policy namespace from the sample-app pod in the network-test namespace over TCP on port 8080. Use the resource file available at ~/DO280/labs/network-policy/allow-specific.yaml.

Use a text editor to replace the CHANGE_ME sections in the allow-specific.yaml file as follows.
<pre>
...output omitted...
spec:
  podSelector:
    matchLabels:
      deployment: hello
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            name: network-test
        podSelector:
          matchLabels:
            deployment: sample-app
      ports:
      - port: 8080
        protocol: TCP
</pre>

Create the network policy with the oc create command.
<pre>
[kris@workstation network-policy]$ oc create -n network-policy -f \
>    allow-specific.yaml
networkpolicy.networking.k8s.io/allow-specific created
</pre>

View the network policies in the network-policy namespace.
<pre>
[kris@workstation network-policy]$ oc get networkpolicies -n network-policy
NAME             POD-SELECTOR       AGE
allow-specific   deployment=hello   11s
deny-all         <'none>             5m6s
</pre>

As the admin user, label the network-test namespace with the name=network-test label.

Log in as the admin user.
<pre>
[kris@workstation network-policy]$ oc login -u admin -p redhat
Login successful.

...output omitted...
</pre>

Use the oc label command to apply the name=network-test label.
<pre>
[kris@workstation network-policy]$ oc label namespace network-test \
>    name=network-test
namespace/network-test labeled
</pre>

Confirm the label was applied.
<pre>
[kris@workstation network-policy]$ oc describe namespace network-test
Name:         network-test
Labels:       name=network-test
...output omitted...
</pre>

Log in as the developer user 
<pre>
[kris@workstation network-policy]$ oc login -u developer -p developer
Login successful.

...output omitted...
</pre>

Verify that the sample-app pod can access the IP address of the hello pod, but cannot access the IP address of the test pod.

Switch to the network-test project.
<pre>
[kris@workstation network-policy]$ oc project network-test
Already on project "network-test" on server "https://api.ocp4.example.com:6443".
</pre>

Verify access to the hello pod in the network-policy namespace.
<pre>
[kris@workstation network-policy]$ oc rsh sample-app-d5f945-spx9q \
>    curl 10.8.0.13:8080 | grep Hello
    <'h1>Hello, world from nginx!</'h1>
</pre>

Verify there is no response from the hello pod on another port. Because the network policy only allows access to port 8080 on the hello pod, requests made to any other port are ignored and eventually time out. Wait a few seconds, and then press Ctrl+C to exit the curl command that does not reply.
<pre>
[kris@workstation network-policy]$ oc rsh sample-app-d5f945-spx9q \
>    curl 10.8.0.13:8181 | grep Hello
^C
command terminated with exit code 130
</pre>

Verify there is no access to the test pod. Wait a few seconds, and then press Ctrl+C to exit the curl command that does not reply.
<pre>
[kris@workstation network-policy]$ oc rsh sample-app-d5f945-spx9q \
>    curl 10.8.0.14:8080 | grep Hello
^C
command terminated with exit code 130
</pre>

Create a network policy to allow traffic to the hello pod from the exposed route. Use the resource file available at ~/DO280/labs/network-policy/allow-from-openshift-ingress.yaml.

Use a text editor to replace the CHANGE_ME values in the allow-from-openshift-ingress.yaml file as follows.
<pre>
...output omitted...
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          network.openshift.io/policy-group: ingress
</pre>

Create the network policy with the oc create command.
<pre>
[kris@workstation network-policy]$ oc create -n network-policy -f \
>    allow-from-openshift-ingress.yaml
networkpolicy.networking.k8s.io/allow-from-openshift-ingress created
</pre>

View the network policies in the network-policy namespace.
<pre>
[kris@workstation network-policy]$ oc get networkpolicies -n network-policy
NAME                           POD-SELECTOR       AGE
allow-from-openshift-ingress   <'none>             10s
allow-specific                 deployment=hello   8m16s
deny-all  
</pre>

Log in as the admin user and apply the network.openshift.io/policy-group=ingress label to the default namespace.
<pre>
[kris@workstation network-policy]$ oc login -u admin -p redhat
Login successful.

...output omitted...

[kris@workstation network-policy]$ oc label namespace default \
>    network.openshift.io/policy-group=ingress
namespace/default labeled
</pre>

Verify access to the hello pod via the exposed route.
<pre>
[kris@workstation network-policy]$ curl -s \
>    hello-network-policy.apps.ocp4.example.com | grep Hello
        <'h1>Hello, world from nginx!<'/h1>
</pre>