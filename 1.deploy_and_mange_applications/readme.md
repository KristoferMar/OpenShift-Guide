# Deploying and Managing Applications on an OpenShift Cluster

We login to our Openshift cluster

# Managing Applications with the CLI

Get logs from build config
- The logs from a build configuration are the logs from the latest build, whether it was successful or not.
<pre>
[user@host ~]$ oc logs bc/myapp
</pre>

Get logs from the build pod created 
<pre>
[user@host ~]$ oc logs myapp-build-2
</pre>

Retrieve fx an Apache HTTP Server log stored inside an application pod directly
<pre>
[user@host ~]$ oc cp frontend-1-zvjhb:/var/log/httpd/error_log \
/tmp/frontend-server.log
</pre>

Run a command inside a pod, ex "ps ax"
<pre>
[user@host ~]$ oc rsh frontend-1-zvjhb ps ax
</pre>

Start an interactive shell session inside the container 
<pre>
[user@host ~]$ oc rsh -t frontend-1-zvjhb
</pre>
