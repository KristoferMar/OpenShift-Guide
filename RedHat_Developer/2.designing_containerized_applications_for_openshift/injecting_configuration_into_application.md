# How to inject configuration data into an application

Assuming you have an application up and running

This is how we create a configmap and verify what it contains
<pre>
[kris@workstation DO288-apps]$ oc create configmap myappconf \
--from-literal APP_MSG="Test Message"

[kris@workstation DO288-apps]$ oc describe cm/myappconf
Name:		myappconf
...output omitted...
Data
====
APP_MSG:
---
Test Message
...output omitted...
</pre>

This is how we create a new secret and verify what it contains
<pre>
[kris@workstation DO288-apps]$ oc create secret generic myappfilesec \
--from-file /home/kris/DO288-apps/app-config/myapp.sec
secret/myappfilesec created

[kris@workstation DO288-apps]$ oc get secret/myappfilesec -o json
{
    "apiVersion": "v1",
    "data": {
        "myapp.sec": "dXNlcm5hbWU9dXNlcjEKcGFzc3dvcmQ9cGFzczEKc2...
    },
    "kind": "Secret",
    "metadata": {
        ...output omitted...
        "name": "myappfilesec",
        ...output omitted...
    },
    "type": "Opaque"
}
</pre>

Now this is how we inject the configuration map and the secret into the application container.
<pre>
[kris@workstation ~]$ oc set env deployment/myapp \
--from configmap/myappconf
deployment.apps.openshift.io/myapp updated
</pre>

Now use "oc set volume to add the secret to the deployment configuration.
<pre>
[kris@workstation DO288-apps]$ oc set volume deployment/myapp --add \
-t secret -m /opt/app-root/secure \
--name myappsec-vol --secret-name myappfilesec
deployment.apps/myapp volume updated
</pre>

Verify that the application is redeployed and uses the data from the configuration map and the secret.
<pre>
[kris@workstation DO288-apps]$ oc status
In project yourdevuser-app-config on server ...output omitted...

http://myapp-yourdevuser-app-config.apps.cluster.domain.example.com to pod port 8080-tcp (svc/myapp)
  deployment/myapp deploys istag/myapp:latest <-
    bc/myapp source builds https://github.com/yourgithubuser/DO288-apps#app-config on openshift/nodejs:16-ubi8
    deployment #4 running for 10 seconds - 1 pod
    deployment #3 deployed 44 seconds ago
    deployment #2 deployed 3 minutes ago
    deployment #1 deployed 4 minutes ago
</pre>

## Change the information stored in the configuration map and restart the application
Use the oc edit configmap command to change the value of the APP_MSG key:
<pre>
[kris@workstation DO288-apps]$ oc edit cm/myappconf
</pre>

The above command opens a Vim-like buffer with the configuration map attributes in YAML format. Edit the value associated with the APP_MSG key under the data section and change the value as follows
<pre>
...output omitted...
apiVersion: v1
data:
  APP_MSG: Changed Test Message
kind: ConfigMap
...output omitted...
</pre>

Save and close the file. 

Verify that the value in the APP_MSG key is updated
<pre>
[kris@workstation DO288-apps]$ oc describe cm/myappconf
Name:		myappconf
...output omitted...
Data
====
APP_MSG:
---
Changed Test Message
...output omitted...
</pre>