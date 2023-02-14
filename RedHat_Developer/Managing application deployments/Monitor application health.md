# Managing Application Deployments

A probe is a periodic check that monitors the health of an application.

### Startup probe
A startup probe verifies whether the application within a container is started.

### Readiness probe
Readiness probes determine whether or not a container is ready to serve requests.

### Liveness probe
Liveness probes determine whether or not an application running in a container is in a healthy state.

## Methods for checking application health
### HTTP check
<pre>
...contents omitted...
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 15
  timeoutSeconds: 1
...contents omitted...
</pre>

### Container execution checks
<pre>
...contents omitted...
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/health
  initialDelaySeconds: 15
  timeoutSeconds: 1
...contents omitted...
</pre>

### TCP Socket Checks
<pre>
...contents omitted...
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  timeoutSeconds: 1
...contents omitted...
</pre>

## Creating Proves By Using CLI
The "oc set probe" command provides an alternative approach to edit the deployment YAML definitions directly.
### Example
<pre>
[user@host ~]$ oc set probe deployment myapp --readiness \
--get-url=http://:8080/healthz --period=20
</pre>

<pre>
[user@host ~]$ oc set probe deployment myapp --liveness \
--open-tcp=3306 --period=20 \
--timeout-seconds=1
</pre>

<pre>
[user@host ~]$ oc set probe deployment myapp --liveness \
--get-url=http://:8080/healthz --initial-delay-seconds=30 \
--success-threshold=1 --failure-threshold=3
</pre>

After executing the manual commands above you can navigate to the "YAML" section for a pod through the web console and see and copy the yaml definition for the probe that gets created.