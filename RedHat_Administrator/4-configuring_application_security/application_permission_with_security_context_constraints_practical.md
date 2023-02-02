# Controlling Application Permissions with Security Context Constraints example

Let's start off by logging in and creating a new project
<pre>
[kris@workstation ~]$ oc login -u developer -p developer \
>    https://api.ocp4.example.com:6443
Login successful.
...output omitted...

[kris@workstation ~]$ oc new-project authorization-scc
Now using project "authorization-scc" on server ...
...output omitted...
</pre>

We create an application called gitlab 
<pre>
[kris@workstation ~]$ oc new-app --name gitlab \
>    --image quay.io/redhattraining/gitlab-ce:8.4.3-ce.0
...output omitted...
--> Creating resources ...
    imagestream.image.openshift.io "gitlab" created
    deployment.apps "gitlab" created
    service "gitlab" created
--> Success
...output omitted...
</pre>

We will se that this application pod reports an error due to the image needing root privileges to properly deploy
<pre>
[kris@workstation ~]$ oc get pods
NAME                     READY   STATUS              RESTARTS   AGE
gitlab-d89cd88f8-jwqbp   0/1     ContainerCreating   0          19s
gitlab-d89cd88f8-jwqbp   1/1     Running             0          30s
gitlab-d89cd88f8-jwqbp   0/1     Error   
</pre>

We check the logs
<pre>
[kris@workstation ~]$ oc logs pod/gitlab-7d67db7875-gcsjl
...output omitted...
================================================================================
Recipe Compile Error in /opt/gitlab/embedded/cookbooks/cache/cookbooks/gitlab/recipes/default.rb
================================================================================

Chef::Exceptions::InsufficientPermissions
-----------------------------------------
directory[/etc/gitlab] (gitlab::default line 26) had an error: Chef::Exceptions::InsufficientPermissions: Cannot create directory[/etc/gitlab] at /etc/gitlab due to insufficient permissions
...output omitted...
</pre>

Now login as the admin user
<pre>
[kris@workstation ~]$ oc login -u admin -p redhat \
>    https://api.ocp4.example.com:6443
Login successful.
...output omitted...
</pre>

Check if using a different SCC can resolve the permissions problem
<pre>
[kris@workstation ~]$ oc get pod/gitlab-7d67db7875-gcsjl -o yaml | \
>    oc adm policy scc-subject-review -f -
RESOURCE                      ALLOWED BY
Pod/gitlab-7d67db7875-gcsjl   anyuid
</pre>

We now create a new service acocunt and assign the anyuid SCC to it
<pre>
[kris@workstation ~]$ oc create sa gitlab-sa
serviceaccount/gitlab-sa created
</pre>

We assign the anyuid SCC to the gitlab-sa service account.
<pre>
[kris@workstation ~]$ oc adm policy add-scc-to-user anyuid -z gitlab-sa
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "gitlab-sa"
</pre>

Modify the gitlab application so that it uses the newly created servicea ccount. Verify that the new deployment succeeds. 
<pre>
[kris@workstation ~]$ oc login -u developer -p developer
Login successful.
...output omitted...

[kris@workstation ~]$ oc set serviceaccount deployment/gitlab gitlab-sa
deployment.apps/gitlab serviceaccount updated
</pre>

Verify that gitlab redeployment succeeds. You might need to run the oc get pods command multiple times until you see a running application pod.
<pre>
[kris@workstation ~]$ oc get pods
NAME                   READY   STATUS    RESTARTS   AGE
gitlab-86d6d65-zm2fd   1/1     Running   0          55s
</pre>

## We now verify the application 

First we expose the gitlab application. Becuase the gitlab service listens on ports 22, 80 and 443, you must use the --port option.
<pre>
[kris@workstation ~]$ oc expose service/gitlab --port 80 \
>    --hostname gitlab.apps.ocp4.example.com
route.route.openshift.io/gitlab exposed
</pre>

We now get the routes and verify
<pre>
[kris@workstation ~]$ oc get routes
NAME     HOST/PORT                      PATH   SERVICES   PORT   ...
gitlab   gitlab.apps.ocp4.example.com          gitlab     80     ...

[kris@workstation ~]$ curl -sL http://gitlab.apps.ocp4.example.com/ | \
>    grep '<'title>'
<'title>Sign in · GitLab<'/title>
</pre>

