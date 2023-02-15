# Managing Application Deployments with CLI Commands
# Managing Application Deployments with CLI Commands

A deployment configuration is desclared within a DeploymentConfig attribute in a resource file, which can be in YAML or JSON format. 
<pre>
kind: "DeploymentConfig"
apiVersion: "v1"
metadata:
  name: "frontend" (1)
spec:
...
  replicas: 5 (2)
  selector:
    name: "frontend"
  triggers:
    - type: "ConfigChange" (3)
    - type: "ImageChange" (4)
      imageChangeParams:
...
  strategy:
    type: "Rolling"
...
</pre>

1. The deployment configuration name.
2. The number of replicas to run.
3. A configuration change trigger that causes a new replication controller to be created when there are changes to the deployment configuration.
4. An image change trigger that causes a new replication controller to be created each time a new version of the image is available.

## Managing Deployments By Using CLI Commands
To start a deployment, use the oc rollout command. The latest option indicates that the newest version of the template must be used
<pre>
[user@host ~]$ oc rollout latest dc/name
</pre>

View the history of deployments for a specific deployment configuration, use the oc rollout history command
<pre>
[user@host ~]$ oc rollout history dc/name
</pre>

Access details about a specific deployment, append the --revision parameter to the oc rollout history command
<pre>
[user@host ~]$ oc rollout history dc/name --revision=1
</pre>

To access details about a deployment configuration and its latest revision, use the oc describe dc command
<pre>
[user@host ~]$ oc describe dc name
</pre>

To access details about a deployment configuration and its latest revision, use the oc describe dc command
<pre>
[user@host ~]$ oc describe dc name
</pre>

To cancel a deployment, run the oc rollout command with the cancel option
<pre>
[user@host ~]$ oc rollout cancel dc/name
</pre>

To retry a deployment configuration that failed, run the oc rollout command with the retry option
<pre>
[user@host ~]$ oc rollout retry dc/name
</pre>

To use a previous version of the application you can roll back the deployment with the oc rollback command
<pre>
[user@host ~]$ oc rollback dc/name
</pre>

To prevent accidentally starting a new deployment process after a rollback is complete, image change triggers are disabled as part of the rollback process.

However, after a rollback, you can re-enable image change triggers with the oc set triggers command
<pre>
[user@host ~]$ oc set triggers dc/name --auto
</pre>

To view deployment logs, use the oc logs command
<pre>
[user@host ~]$ oc logs -f dc/name
</pre>

You can also view logs from older failed deployment processes, provided they have not been pruned or deleted manually
<pre>
[user@host ~]$ oc logs --version=1 dc/name
</pre>

You can scale the number of pods in a deployment by using the oc scale command
<pre>
[user@host ~]$ oc scale dc/name --replicas=3
</pre>

## Deployment Triggers
Use the oc set triggers command to set a deployment trigger for a deployment configuration. For example, to set the ImageChange trigger, run the following command
<pre>
[user@host ~]$ oc set triggers dc/name \
--from-image=myproject/origin-ruby-sample:latest -c helloworld
</pre>

## Setting Deployment Resource Limits 

In the following example, resources required for the deployment are declared under the resources attribute of the deployment configuration
<pre>
type: "Recreate"
resources:
  limits:
    cpu: "100m" (1)
    memory: "256Mi"  (2)
</pre>

1. CPU resource in CPU units. 100m equals 0.1 CPU units.
2. Memory resource in bytes. 256Mi equals 268435456 bytes (256 * 2 ^ 20).