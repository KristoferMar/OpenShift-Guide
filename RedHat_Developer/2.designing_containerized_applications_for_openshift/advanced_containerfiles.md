# Building Container Images with Advanced Containerfile Instructions

Let's review the following file.

```
FROM registry.access.redhat.com/ubi8/ubi:8.0 (1)

MAINTAINER kristofer <kristofer@practice.com>

# DocumentRoot for Apache
ENV DOCROOT=/var/www/html (2)

RUN   yum install -y --no-docs --disableplugin=subscription-manager httpd && \ (3)
      yum clean all --disableplugin=subscription-manager -y && \
      echo "Hello from the httpd-parent container!" > ${DOCROOT}/index.html

# Allows child images to inject their own content into DocumentRoot
ONBUILD COPY src/ ${DOCROOT}/ (4)

EXPOSE 80

# This stuff is needed to ensure a clean start
RUN rm -rf /run/httpd && mkdir /run/httpd

# Run as the root user
USER root (5)

# Launch httpd
CMD /usr/sbin/httpd -DFOREGROUND
```

1. The base image is the Universal Base Image (UBI) for Red Hat Enterprise Linux 8.0 from the Red Hat Container Catalog.

2. Environment variables for this container image.

3. The RUN instruction contains several commands that install the Apache HTTP Server and create a default home page for the web server.

4. The ONBUILD instruction allows child images to provide their own customized web server content when building an image that extends from this parent image.

5. The USER instruction runs the Apache HTTP Server process as the root user.

Inspect the ~/DO288-apps/container-build/Containerfile file. The Containerfile has a single instruction, FROM, which uses the redhattraining/httpd-parent image
<pre>
FROM quay.io/redhattraining/http-parent
</pre>

The child container provides its own index.html file in the ~/DO288-apps/container-build/src folder, which overwrites the parent's index.html file. The contents of the child container image's index.html file is shown below

```
<!DOCTYPE html>
<html>
<body>
  Hello from the Apache child container!
</body>
</html>
```

### Build and deploy a container to an OpenShift cluster using the Apache HTTP Server child Containerfile.

Log in to OpenShift using your developer username
<pre>
[kris@workstation DO288-apps]$ oc login -u ${RHT_OCP4_DEV_USER} \
-p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
Login successful.
...output omitted...
</pre>

Create a new project for the application. Prefix the project name with your developer username.
<pre>
[kris@workstation DO288-apps]$ oc new-project \
${RHT_OCP4_DEV_USER}-container-build
Now using project "yourdevuser-container-build" on server "https://api.cluster.domain.example.com:6443".
</pre>

Build the Apache HTTP Server child image:
<pre>
[kris@workstation DO288-apps]$ podman build --layers=false \
-t do288-apache ./container-build
STEP 1: FROM quay.io/redhattraining/httpd-parent (1)
...output omitted...
STEP 2: COPY src/ ${DOCROOT}/ (2)
STEP 3: COMMIT do288-apache
...output omitted...
Storing signatures
...output omitted...
</pre>
1. The Containerfile is automatically identified and podman pull the parent image.

2. The ONBUILD instruction in the parent Containerfile triggers the copying of the child's index.html file, which overwrites the parent index file.

View the the images
<pre>
[kris@workstation DO288-apps]$ podman images
localhost/do288-apache              latest fc114a884288  9 minutes ago  236 MB
quay.io/redhattraining/httpd-parent latest 4346d3cace25  2 years ago    236 MB
</pre>

Tag and push the image to Quay.io
<pre>
[kris@workstation DO288-apps]$ podman tag do288-apache \
quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
[kris@workstation DO288-apps]$ podman login quay.io -u ${RHT_OCP4_QUAY_USER}
Password:
Login Succeeded!
[kris@workstation DO288-apps]$ podman push \
--format v2s1 quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
...output omitted...
Storing signatures
[kris@workstation DO288-apps]$
</pre>

Log into Quay.io and make the new image public

Deploy the Apache HTTP Server child image
<pre>
[kris@workstation DO288-apps]$ oc new-app --name hola \
quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
--> Found Container image 1d6f8d3 (11 minutes old) from quay.io for "quay.io/yourquayuser/do288-apache"
</pre>

Verify that the application pod fails to start. The pod will be in the Error state, but if you wait too long, the pod will move to the CrashLoopBackOff state.
<pre>
[kris@workstation DO288-apps]$ oc get pods
NAME                     READY     STATUS                RESTARTS   AGE
hola-58554c88b9-qxc7n    0/1       CrashLoopBackOff      0          12s
</pre>

Inspect the container's logs to see why the pod failed to start
<pre>
[kris@workstation DO288-apps]$ oc logs hola-13p75f5
AH00558: httpd: Could not reliably determine the server's fully qualified domain name...
(13)Permission denied: AH00072: make_sock: could not bind to address [::]:80 (1)
(13)Permission denied: AH00072: make_sock: could not bind to address 0.0.0.0:80 (2)
no listening sockets available, shutting down
AH00015: Unable to open logs (3)
</pre>

1. Because OpenShift runs containers using a random userid, ports below 1024 are privileged and can only be run as root.

2. The random userid used by OpenShift to run the container does not have permissions to read and write log files in /var/log/httpd (the default log file location for the Apache HTTP Server on RHEL 7).

Delete all resources from the OpenShift project. The next step changes the Containerfile to follow Red Hat recommendations for OpenShift.

Before updating the child Apache HTTP Server Containerfile, delete all the resources in the project that have been created so far
<pre>
[kris@workstation DO288-apps]$ oc delete all -l app=hola
service "hola" deleted
deployment.apps "hola" deleted
imagestream.image.openshift.io "hola" deleted
</pre>

### Change the Containerfile for the child container to run on an OpenShift cluster by updating the Apache HTTP Server process to run as a random, unprivileged user.

Edit the ~/DO288-apps/container-build/Containerfile file and perform the following steps. You can also copy these instructions from the provided ~/DO288/solutions/container-build/Containerfile file.

Override the EXPOSE instruction from the parent image and change the port to 8080.

<pre>
EXPOSE 8080
</pre>

Include the io.openshift.expose-service label to indicate the changed port that the web server runs on
<pre>
LABEL io.openshift.expose-services="8080:http"
</pre>

Update the list of labels to include the io.k8s.description, io.k8s.display-name, and io.openshift.tags labels that OpenShift consumes to provide helpful metadata about the container image
<pre>
LABEL io.k8s.description="A basic Apache HTTP Server child image, uses ONBUILD" \
      io.k8s.display-name="Apache HTTP Server" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="apache, httpd"
</pre>

You need to run the web server on an unprivileged port (that is, greater than 1024). Use a RUN instruction to change the port number in the Apache HTTP Server configuration file from the default port 80 to 8080
<pre>
RUN sed -i "s/Listen 80/Listen 8080/g" /etc/httpd/conf/httpd.conf \
RUN sed -i "s/#ServerName www.example.com:80/ServerName 0.0.0.0:8080/g" \
    /etc/httpd/conf/httpd.conf
</pre>

Change the group ID and permissions of the folders where the web server process reads and writes files
<pre>
RUN chgrp -R 0 /var/log/httpd /var/run/httpd && \
    chmod -R g=u /var/log/httpd /var/run/httpd
</pre>

Add a USER instruction for an unprivileged user. The Red Hat convention is to use userid 1001
<pre>
USER 1001
</pre>

Save the Containerfile and commit the changes to the Git repository from the ~/DO288-apps/container-build folder
<pre>
[kris@workstation DO288-apps]$ cd container-build
[kris@workstation container-build]$ git commit -a -m \
"Changed the Containerfile to enable running as a random uid on OpenShift"
...output omitted...
[kris@workstation container-build]$ git push
...output omitted...
[kris@workstation container-build]$ cd ..
</pre>

Rebuild and redeploy the Apache HTTP Server child container image.

Remove old images
<pre>
[kris@workstation DO288-apps]$ podman rmi -a --force
...output omitted...
</pre>

Re-create the application using the new Containerfile
<pre>
[kris@workstation DO288-apps]$ podman build --layers=false \
-t do288-apache ./container-build
STEP 1: FROM quay.io/redhattraining/httpd-parent
...output omitted...
STEP 2: COPY src/ ${DOCROOT}/
STEP 3: EXPOSE 8080
STEP 4: LABEL io.k8s.description="A basic Apache HTTP Server child image, uses ONBUILD"       io.k8s.display-name="Apache HTTP Server"       io.openshift.expose-services="8080:http"       io.openshift.tags="apache, httpd"
STEP 5: RUN sed -i "s/Listen 80/Listen 8080/g" /etc/httpd/conf/httpd.conf
STEP 6: RUN chgrp -R 0 /var/log/httpd /var/run/httpd &&     chmod -R g=u /var/log/httpd /var/run/httpd
STEP 7: USER 1001
STEP 8: COMMIT do288-apache
...output omitted...
</pre>

Tag the new image and replace the image in Quay.io
<pre>
[kris@workstation DO288-apps]$ podman tag do288-apache \
quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
[kris@workstation DO288-apps]$ podman push \
--format v2s1  quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
...output omitted...
Storing signatures
[kris@workstation DO288-apps]$
</pre>

Redeploy the Apache HTTP Server child container image.
<pre>
[kris@workstation ~]$ oc new-app --name hola \
quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
--> Found container image fe746d5 (7 minutes old) from quay.io for "quay.io/yourquayuser/do288-apache"
...output omitted...
</pre>

Wait until the pod is ready and running. View the status of the application pod
<pre>
[kris@workstation ~]$ oc get pods
NAME                    READY     STATUS       RESTARTS   AGE
hola-58554c88b9-qxc7n   1/1       Running      0          5s
</pre>

Create an OpenShift route to expose the application to external access
<pre>
[kris@workstation ~]$ oc expose --port='8080' svc/hola
route.route.openshift.io/hola exposed
</pre>

Obtain the route URL using the oc get route command
<pre>
[kris@workstation ~]$ oc get route
NAME   HOST/PORT                                                    ...
hola   hola-yourquayuser-container-build.cluster.domain.example.com ...
</pre>