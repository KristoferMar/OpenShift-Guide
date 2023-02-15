# Selecting the Appropriate Deployment Strategy

The "oc new-app creates a Deployment resource but OpenShift also provides "DeploymentConfig". 

The DeploymentConfig resource enables you to use additional features, such as 
- Custom deployment stratigies
- Lifecycle hooks

## Deployment stratigies

### Rolling 
The rolling strategy is the default strategy.
Use a rolling deployment strategy when:
- You require no downtime during an application update.
- Your application supports running an older version and a newer version at the same time.


### Recreate
In this strategy, Red Hat OpenShift first stops all the pods that are currently running and only then starts up pods with the new version of the application. This strategy incurs downtime because, for a brief period, no instances of your application are running.

Use a recreate deployment strategy when:
- Your application does not support running an older version and a newer version at the same time.
- Your application uses a persistent volume with RWO (ReadWriteOnce) access mode, which does not allow writes from multiple pods.

### Custom
If neither the rolling nor the recreate deployment strategies suit your needs, you can use the custom deployment strategy to deploy your applications.

## Integrate deployments with Life-cycle Hooks 
The Recreate and Rolling strategies support life-cycle hooks. 

## Pre-Lifecycle Hook
Red Hat OpenShift executes the pre-life-cycle hook before any new pods for a deployment start, and also before any older pods shut down.

## Mid-Lifecycle Hook
The mid-life-cycle hook is executed after all the old pods in a deployment have been shut down, but before any new pods are started. Mid-Lifecycle Hooks are only available for the Recreate strategy.

## Post-Lifecycle Hook
The post-life-cycle hook is executed after all new pods for a deployment have started, and after all the older pods have shut down.

# Practical
Verify the default deployment strateigy
<pre>
[student@workstation ~]$ oc describe dc/mysql | grep -i strategy:
Strategy:	Rolling
</pre>

How to change the default deployment config 
<pre>
[student@workstation ~]$ oc patch dc/mysql --patch \
'{"spec":{"strategy":{"type":"Recreate"}}}'
deploymentconfig.apps.openshift.io/mysql patched
</pre>

Remove the rollingParams attribute from the deployment configuration.
<pre>
[student@workstation ~]$ oc patch dc/mysql --type=json \
-p='[{"op":"remove", "path": "/spec/strategy/rollingParams"}]'
deploymentconfig.apps.openshift.io/mysql patched
</pre>

force a new deployment to test changes to a strategy and post life-cycle hooks
<pre>
[student@workstation ~]$ oc rollout latest dc/mysql
deploymentconfig.apps.openshift.io/mysql rolled out
</pre>