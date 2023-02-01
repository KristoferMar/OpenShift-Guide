## Exposing Applications for External Access

We first start off by logging in and creating a project

<pre>
[kris@workstation ~]$ oc login -u developer -p developer \
>    https://api.ocp4.example.com:6443
Login successful.

...output omitted...

[kris@workstation ~]$ oc new-project network-ingress
Now using project "network-ingress" on server "https://api.ocp4.example.com:6443".

...output omitted...
</pre>

Use the oc create command to deploy the application in the network-ingress OpenShift project
<pre>
[kris@workstation ~]$ oc create -f \
>   ~/DO280/labs/network-ingress/todo-app-v1.yaml
deployment.apps/todo-http created
service/todo-http created
</pre>

Wait a couple of minutes, so that the application can start, and then review the resources in the project.
<pre>
[kris@workstation ~]$ oc status
In project network-ingress on server https://api.ocp4.example.com:6443

svc/todo-http - 172.30.247.75:80 -> 8080
  deployment/todo-http deploys quay.io/redhattraining/todo-angular:v1.1
    deployment #1 running for 16 seconds - 1 pod
...output omitted...
</pre>

Run the oc expose command to create a route for accessing the application. Give the route a host name of todo-http.apps.ocp4.example.com.
<pre>
[kris@workstation ~]$ oc expose svc todo-http \
>   --hostname todo-http.apps.ocp4.example.com
route.route.openshift.io/todo-http exposed
</pre>

Retrieve the name of the route and copy it to the clipboard.
<pre>
[kris@workstation ~]$ oc get routes
NAME        HOST/PORT                         PATH   SERVICES    PORT   ...
todo-http   todo-http.apps.ocp4.example.com          todo-http   8080   ...
</pre>

We can now open the application through firefox

    http://todo-http.apps.ocp4.example.com 

Open a new terminal tab and run the tcpdump command with the following options to intercept the traffic on port 80:

- -i eth0 intercepts traffic on the main interface.

- -A strips the headers and prints the packets in ASCII format.

- -n disables DNS resolution.

- port 80 is the port of the application.

Optionally, the grep command allows you to filter on JavaScript resources.

Start by retrieving the name of the main interface whose IP is 172.25.250.9.
<pre>
[kris@workstation ~]$ ip addr | grep 172.25.250.9
inet 172.25.250.9/24 brd 172.25.250.255 scope global noprefixroute eth0

[kris@workstation ~]$ sudo tcpdump -i eth0 -A -n port 80 | grep "angular"
</pre>

On Firefox, refresh the page and notice the activity in the terminal. Press Ctrl+C to stop the capture.

```
    ...output omitted...
    <script type="text/javascript" src="assets/js/libs/angular/angular.min.js">
    <script type="text/javascript" src="assets/js/libs/angular/angular-route.min.js">
    <script type="text/javascript" src="assets/js/libs/angular/angular-animate.min.js">
    ...output omitted...
```

Create a secure edge route. Edge certificates encrypt the traffic between the client and the router, but leave the traffic between the router and the service unencrypted. OpenShift generates its own certificate that it signs with its CA.

In later steps, we extract the CA to ensure the route certificate is signed.

Go to ~/DO280/labs/network-ingress and run the oc create route command to define the new route.

Give the route a host name of todo-https.apps.ocp4.example.com.
<pre>
[kris@workstation ~]$ cd ~/DO280/labs/network-ingress
[kris@workstation network-ingress]$ oc create route edge todo-https \
>   --service todo-http \
>   --hostname todo-https.apps.ocp4.example.com
route.route.openshift.io/todo-https created
</pre>

To test the route and read the certificate, open Firefox and access the application URL.

     https://todo-https.apps.ocp4.example.com 


From the terminal, we can now use the curl command with the -I and -v options to retrieve the connection headers.

The Server certificate section shows some information about the certificate and the alternative name matches the name of the route. The output indicates that the remote certificate is trusted because it matches the CA.
<pre>
[kris@workstation network-ingress]$ curl -I -v \
> https://todo-https.apps.ocp4.example.com
...output omitted...
* Server certificate:
*  subject: O=EXAMPLE.COM; CN=.api.ocp4.example.com
*  start date: May 10 11:18:41 2021 GMT
*  expire date: May 10 11:18:41 2026 GMT
*  subjectAltName: host "todo-https.apps.ocp4.example.com" matched cert's "*.apps.ocp4.example.com"
*  issuer: O=EXAMPLE.COM; CN=Red Hat Training Certificate Authority
*  SSL certificate verify ok.
...output omitted...
</pre>

Although the traffic is encrypted at the edge with a certificate, you can still access the insecure traffic at the service level because the pod behind the service does not offer an encrypted route.

Retrieve the IP address of the todo-http service.
<pre>
[kris@workstation network-ingress]$ oc get svc todo-http \
>   -o jsonpath="{.spec.clusterIP}{'\n'}"
172.30.102.29
</pre>

Create a debug pod in the todo-http deployment. Use the Red Hat Universal Base Image (UBI), which contains some basic tools to interact with containers.
<pre>
[kris@workstation network-ingress]$ oc debug -t deployment/todo-http \
>   --image registry.access.redhat.com/ubi8/ubi:8.4
Starting pod/todo-http-debug ...
Pod IP: 10.131.0.255
If you don't see a command prompt, try pressing enter.
sh-4.4$
</pre>

From the debug pod, use the curl command to access the service over HTTP. Replace the IP address with the one that you obtained in a previous step.

The output indicates that the application is available over HTTP.
<pre>
sh-4.4$ curl -v 172.30.102.29
* Rebuilt URL to: 172.30.102.29/
*   Trying 172.30.102.29...
* TCP_NODELAY set
* Connected to 172.30.102.29 (172.30.102.29) port 80 (#0)
> GET / HTTP/1.1
> Host: 172.30.102.29
> User-Agent: curl/7.61.1
> Accept: /
>
< HTTP/1.1 200 OK
...output omitted...
</pre>

# Generate TLS certificates for the application

In the following steps, we generate a CA-signed certificate that you attach as a secret to the pod. We then configure a secure route in passthrough mode and let the application expose that certificate.

Go to the ~/DO280/labs/network-ingress/certs directory and list the files.
<pre>
[kris@workstation network-ingress]$ cd certs
[kris@workstation certs]$ ls -l
total 20
-rw-rw-r--. 1 kris kris  604 Nov 29 17:35 openssl-commands.txt
-rw-r--r--. 1 kris kris   33 Nov 29 17:35 passphrase.txt
-rw-r--r--. 1 kris kris 1743 Nov 29 17:35 training-CA.key
-rw-r--r--. 1 kris kris 1363 Nov 29 17:35 training-CA.pem
-rw-r--r--. 1 kris kris  406 Nov 29 17:35 training.ext
</pre>

Generate the private key for your CA-signed certificate.
<pre>
[kris@workstation certs]$ openssl genrsa -out training.key 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
...output omitted...
e is 65537 (0x010001)
</pre>

Generate the certificate signing request (CSR) for todo-https.apps.ocp4.example.com.
<pre>
[kris@workstation certs]$ openssl req -new \
>   -key training.key -out training.csr \
>   -subj "/C=US/ST=North Carolina/L=Raleigh/O=Red Hat/\
> CN=todo-https.apps.ocp4.example.com"
</pre>

Finally, generate the signed certificate. Notice the use of the -CA and -CAkey options for signing the certificate against the CA. The -passin option allows you to reuse the password of the CA. The extfile option allows you to define a Subject Alternative Name (SAN).
<pre>
[kris@workstation certs]$ openssl x509 -req -in training.csr \
>   -passin file:passphrase.txt \
>   -CA training-CA.pem -CAkey training-CA.key -CAcreateserial \
>   -out training.crt -days 1825 -sha256 -extfile training.ext
Signature ok
subject=C = US, ST = North Carolina, L = Raleigh, O = Red Hat, CN = todo-https.apps.ocp4.example.com
Getting CA Private Key
</pre>

Ensure that the newly created certificate and key are present in the current directory.
<pre>
[kris@workstation certs]$ ls -lrt
total 36
-rw-r--r--. 1 kris kris  599 Jul 31 09:35 openssl-commands.txt
-rw-r--r--. 1 kris kris   33 Aug  3 12:38 passphrase.txt
-rw-r--r--. 1 kris kris  352 Aug  3 12:38 training.ext
-rw-------. 1 kris kris 1743 Aug  3 12:38 training-CA.key
-rw-r--r--. 1 kris kris 1334 Aug  3 12:38 training-CA.pem
-rw-------. 1 kris kris 1675 Aug  3 13:38 training.key
-rw-rw-r--. 1 kris kris 1017 Aug  3 13:39 training.csr
-rw-rw-r--. 1 kris kris   41 Aug  3 13:40 training-CA.srl
-rw-rw-r--. 1 kris kris 1399 Aug  3 13:40 training.crt
</pre>

## Deploying a new version of our application
The new version of the application expects a certificate and a key inside the container at /usr/local/etc/ssl/certs. The web server in that version is configured with SSL support. Create a secret to import the certificate from the workstation machine. In a later step, the application deployment requests that secret and exposes its content to the container at /usr/local/etc/ssl/certs.

Create a tls OpenShift secret named todo-certs. Use the --cert and --key options to embed the TLS certificates. Use training.crt as the certificate, and training.key as the key.
<pre>
[kris@workstation network-ingress]$ oc create secret tls todo-certs \
>   --cert certs/training.crt --key certs/training.key
secret/todo-certs created
</pre>

The deployment file, accessible at ~/DO280/labs/network-ingress/todo-app-v2.yaml, points to version 2 of the container image. The new version of the application is configured to support SSL certificates. Run oc create to create a new deployment using that image.
<pre>
[kris@workstation network-ingress]$ oc create -f todo-app-v2.yaml
deployment.apps/todo-https created
service/todo-https created
</pre>

Wait a couple of minutes to ensure that the application pod is running. Copy the pod name to your clipboard.
<pre>
[kris@workstation network-ingress]$ oc get pods
NAME                          READY   STATUS    RESTARTS   AGE
...output omitted...
todo-https-59d8fc9d47-265ds   1/1     Running   0          62s
</pre>

Review the volumes that are mounted inside the pod. The output indicates that the certificates are mounted to /usr/local/etc/ssl/certs.
<pre>
[kris@workstation network-ingress]$ oc describe pod \
> todo-https-59d8fc9d47-265ds | grep -A2 Mounts
  Mounts:
    /usr/local/etc/ssl/certs from tls-certs (ro)
    /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qrtkj (ro)
</pre>

## Create a secret route

Run the oc create route command to define the new route.

Give the route a host name of todo-https.apps.ocp4.example.com.
<pre>
[kris@workstation network-ingress]$ oc create route passthrough todo-https \
>   --service todo-https --port 8443 \
>   --hostname todo-https.apps.ocp4.example.com
route.route.openshift.io/todo-https created
</pre>

Use the curl command in verbose mode to test the route and read the certificate. Use the --cacert option to pass the CA certificate to the curl command.

The output indicates a match between the certificate chain and the application certificate. This match indicates that the OpenShift router only forwards packets that are encrypted by the application web server certificate.
<pre>
[kris@workstation network-ingress]$ curl -vv -I \
>   --cacert certs/training-CA.pem \
>   https://todo-https.apps.ocp4.example.com
...output omitted...
* Server certificate:
*  subject: C=US; ST=North Carolina; L=Raleigh; O=Red Hat; CN=todo-https.apps.ocp4.example.com
*  start date: Jun 15 01:53:30 2021 GMT
*  expire date: Jun 14 01:53:30 2026 GMT
*  subjectAltName: host "todo-https.apps.ocp4.example.com" matched cert's "*.apps.ocp4.example.com"
*  issuer: C=US; ST=North Carolina; L=Raleigh; O=Red Hat; CN=ocp4.example.com
*  SSL certificate verify ok.
...output omitted...
</pre>

Create a new debug pod to further confirm proper encryption at the service level.

Retrieve the IP address of the todo-https service.
<pre>
[kris@workstation network-ingress]$ oc get svc todo-https \
>   -o jsonpath="{.spec.clusterIP}{'\n'}"
172.30.121.154
</pre>

Create a debug pod in the todo-https deployment with the Red Hat UBI.
<pre>
[kris@workstation network-ingress]$ oc debug -t deployment/todo-https \
>   --image registry.access.redhat.com/ubi8/ubi:8.4
Starting pod/todo-https-debug ...
Pod IP: 10.128.2.129
If you don't see a command prompt, try pressing enter.
sh-4.4$
</pre>

From the debug pod, use the curl command to access the service over HTTP. Replace the IP address with the one that you obtained in a previous step.

The output indicates that the application is not available over HTTP, and the web server redirects you to the secure version.
<pre>
sh-4.4$ curl -I http://172.30.121.154
HTTP/1.1 301 Moved Permanently
Server: nginx/1.14.1
Date: Tue, 15 Jun 2021 02:01:19 GMT
Content-Type: text/html
Connection: keep-alive
Location: https://172.30.121.154:8443/
</pre>

Finally, access the application over HTTPS. Use the -k option because the container does not have access to the CA certificate.

```
sh-4.4$ curl -s -k https://172.30.121.154:8443 | head -n5
<!DOCTYPE html>
<html lang="en" ng-app="todoItemsApp" ng-controller="appCtl">
<head>
    <meta charset="utf-8">
    <title>ToDo app</title>
```