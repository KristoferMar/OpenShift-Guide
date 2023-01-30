After logging in as admin 

List all cluster role bindings that reference the self-provisioner
<pre>
[kris@workstation ~]$ oc get clusterrolebinding -o wide | \
>    grep -E 'NAME|self-provisioner'
NAME                      ROLE                           ...
self-provisioners   ...   ClusterRole/self-provisioner   ...
</pre>


Remove the privilege to create new projects from all users who are not cluster administrators by deleting the self-provisioner cluster role from the system:authenticated:oauth

Confirm that the self-provisioners cluster role binding assigns the self-provisioner cluster role to the system:authenticated:oauth
<pre>
[kris@workstation ~]$ oc describe clusterrolebindings self-provisioners
Name:         self-provisioners
Labels:       <'none>
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  self-provisioner
Subjects:
  Kind   Name                        Namespace
  ----   ----                        ---------
  Group system:authenticated:oauth
</pre>


Remove the self-provisioner cluster role from the system:authenticated:oauth virtual group. 
<pre>
[kris@workstation ~]$ oc adm policy remove-cluster-role-from-group \
>    self-provisioner system:authenticated:oauth
Warning: Your changes may get lost whenever a master is restarted, unless you prevent reconciliation of this rolebinding using the following command:
oc annotate clusterrolebinding.rbac self-provisioner 'rbac.authorization.kubernetes.io/autoupdate=false' --overwrite
clusterrole.rbac.authorization.k8s.io/self-provisioner removed: "system:authenticated:oauth"
</pre>

Verify the role has been removed from the group. The cluster role binding self-provisioners should not exists. 
<pre>
[kris@workstation ~]$ oc describe clusterrolebindings self-provisioners
Error from server (NotFound): clusterrolebindings.rbac.authorization.k8s.io "self-provisioners" not found
</pre>

Determine if any other cluster role bindings reference the self-provisioner cluster role.
<pre>
[kris@workstation ~]$ oc get clusterrolebinding -o wide | \
>    grep -E 'NAME|self-provisioner'
NAME                      ROLE    ...
</pre>



Now if we login as the user leader we find that he does not have the access to create a new project
<pre>
[kris@workstation ~]$ oc login -u leader -p redhat
Login successful.

[kris@workstation ~]$ oc new-project test
Error from server (Forbidden): You may not request a new project via this API.
</pre>


Then we can login as admin and add him to the policy
<pre>
[kris@workstation ~]$ oc login -u admin -p redhat
Login successful.

[kris@workstation ~]$ oc new-project auth-rbac
Now using project "auth-rbac" on server "https://api.ocp4.example.com:6443".

[kris@workstation ~]$ oc policy add-role-to-user admin leader
clusterrole.rbac.authorization.k8s.io/admin added: "leader"
</pre>


### Creating groups
Now we create a group called dev-group
<pre>
[kris@workstation ~]$ oc adm groups new dev-group
group.user.openshift.io/dev-group created
</pre>

Add the developer user to dev-group
<pre>
[kris@workstation ~]$ oc adm groups add-users dev-group developer
group.user.openshift.io/dev-group added: "developer"
</pre>

Now we createa new group and call it qa-group
<pre>
[kris@workstation ~]$ oc adm groups new qa-group
group.user.openshift.io/qa-group created
</pre>

We then add the qa-engineer user to the qa-group
<pre>
[kris@workstation ~]$ oc adm groups add-users qa-group qa-engineer
group.user.openshift.io/qa-group added: "qa-engineer"
</pre>

Review all existing OpenShift groups to verify that they have the correct memebers.
<pre>
[kris@workstation ~]$ oc get groups
NAME        USERS
dev-group   developer
qa-group    qa-engineer
</pre>


## Local privileges

We now login as the leader user
<pre>
[kris@workstation ~]$ oc login -u leader -p redhat
Login successful.
</pre>

Add write privileges to dev-group on the auth-rbac project
<pre>
[kris@workstation ~]$ oc policy add-role-to-group edit dev-group
clusterrole.rbac.authorization.k8s.io/edit added: "dev-group"
</pre>

Add read privileges to qa-group on the auth-rbac project.
<pre>
[kris@workstation ~]$ oc policy add-role-to-group view qa-group
clusterrole.rbac.authorization.k8s.io/view added: "qa-group"
</pre>

Review all role bindings on the auth-rbac project to verify that they assign roles to the correct groups and users. The following output omits default role bindings assigned by OpenShift to service accounts.
<pre>
[kris@workstation ~]$ oc get rolebindings -o wide
NAME      ROLE                AGE   USERS    GROUPS      ...
admin     ClusterRole/admin   58s   admin
admin-0   ClusterRole/admin   51s   leader
edit      ClusterRole/edit    12s            dev-group
...output omitted...
view      ClusterRole/view    8s             qa-group
</pre>

Now as the developer user we will deploy  an Apache HTTP Server to prove that the developer user has write privileges in the project. 
<pre>
[kris@workstation ~]$ oc login -u developer -p developer
Login successful.

[kris@workstation ~]$ oc new-app --name httpd httpd:2.4
...output omitted...
--> Creating resources ...
    imagestreamtag.image.openshift.io "httpd:2.4" created
    deployment.apps "httpd" created
    service "httpd" created
--> Success
...output omitted..
</pre>

If we try to grant write privileges to the qa-engineer user we see it fail since we dont have the right.
<pre>
[kris@workstation ~]$ oc policy add-role-to-user edit qa-engineer
Error from server (Forbidden): rolebindings.rbac.authorization.k8s.io is forbidden: User "developer" cannot list resource "rolebindings" in API group "rbac.authorization.k8s.io" in the namespace "auth-rbac"
</pre>

If we now login as qa-engieer and for instance try to scale up the apache application we see we cant do that.
<pre>
[kris@workstation ~]$ oc login -u qa-engineer -p redhat
Login successful.

[kris@workstation ~]$ oc scale deployment httpd --replicas 3
Error from server (Forbidden): deployments.apps "httpd" is forbidden: User "qa-engineer" cannot patch resource "deployments/scale" in API group "apps" in the namespace "auth-rbac"
</pre>

We now login as admin and restore project createion privileges for all users by recreating the self-provisioners cluster role binding created by the OpenShift installer.
<pre>
[kris@workstation ~]$ oc login -u admin -p redhat
Login successful.

[kris@workstation ~]$ oc adm policy add-cluster-role-to-group \
>    --rolebinding-name self-provisioners \
>    self-provisioner system:authenticated:oauth
Warning: Group 'system:authenticated:oauth' not found
clusterrole.rbac.authorization.k8s.io/self-provisioner added: "system:authenticated:oauth"
</pre>