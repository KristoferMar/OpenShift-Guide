We start off by creating an HTPasswd authentication file named htpasswd in a directory. We use the htpasswd command to populate the HTPasswd authencation file.

<pre>
[kris@workstation ~]$ htpasswd -c -B -b ~/DO280/labs/auth-provider/htpasswd \
>    admin redhat
Adding password for user admin
</pre>

We add the developer user with a password of develeoper to the same file. 
<pre>
[kris@workstation ~]$ htpasswd -b ~/DO280/labs/auth-provider/htpasswd \
>    developer developer
Adding password for user developer
</pre>

We can the review the content of the file
<pre>
[kris@workstation ~]$ cat ~/DO280/labs/auth-provider/htpasswd
admin:$2y$05$QPuzHdl06IDkJssT.tdkZuSmgjUHV1XeYU4FjxhQrFqKL7hs2ZUl6
developer:$apr1$0Nzmc1rh$yGtne1k.JX6L5s5wNa2ye.
</pre>

We then login to our cluster and create a secret from the htpasswd file. 
<pre>
[kris@workstation ~]$ oc create secret generic localusers \
>    --from-file htpasswd=~/DO280/labs/auth-provider/htpasswd \
>    -n openshift-config
secret/localusers created
</pre>

We assign the admin user to the cluster-admin role
<pre>
[kris@workstation ~]$ oc adm policy add-cluster-role-to-user \
>    cluster-admin admin
Warning: User 'admin' not found
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "admin"
</pre>

We now export an existing OAuth resource to a file named oauth.yaml. 
<pre>
[kris@workstation ~]$ oc get oauth cluster \
>    -o yaml > ~/DO280/labs/auth-provider/oauth.yaml
</pre>

We now edit the oauth.yml file and choose the name of identityProviders and fileData structures. We edit the following with vim from the oauth.yaml file
<pre>
apiVersion: config.openshift.io/v1
kind: OAuth
...output omitted...
spec:
  identityProviders:
  - htpasswd:
      fileData:
        name: localusers
    mappingMethod: claim
    name: myusers
    type: HTPasswd
</pre>

We den apply the custom resource defined in the previous step
<pre>
[kris@workstation ~]$ oc replace -f ~/DO280/labs/auth-provider/oauth.yaml
oauth.config.openshift.io/cluster replaced
</pre>

We can now login as the new user 
<pre>
[kris@workstation ~]$ oc login -u admin -p redhat
Login successful.

...output omitted...
</pre>

And get the nodes for instance
<pre>
[kris@workstation ~]$ oc get nodes
NAME       STATUS   ROLES           AGE  VERSION
master01   Ready    master,worker   2d   v1.23.5+9ce5071
master02   Ready    master,worker   2d   v1.23.5+9ce5071
master03   Ready    master,worker   2d   v1.23.5+9ce5071
</pre>

We can also login as developer
<pre>
[kris@workstation ~]$ oc login -u developer -p developer
Login successful.

...output omitted...
</pre>

And we see that we dont have the same accessrights as the admin which is good.
<pre>
[kris@workstation ~]$ oc get nodes
Error from server (Forbidden): nodes is forbidden:
User "developer" cannot list resource "nodes" in API group "" at the cluster scope
</pre>

We log back in as admin and check list of users 
<pre>
[kris@workstation ~]$ oc login -u admin -p redhat
Login successful.

[kris@workstation ~]$ oc get users
NAME       UID                                   FULL NAME  IDENTITIES
admin      31f6ccd2-6c58-47ee-978d-5e5e3c30d617             myusers:admin
developer  d4e77b0d-9740-4f05-9af5-ecfc08a85101             myusers:developer
</pre>

Display the list of current identities.
<pre>
[kris@workstation ~]$ oc get identity
NAME                IDP NAME   IDP USER NAME   USER NAME   USER UID
myusers:admin       myusers    admin           admin       31f6ccd2-6c58-47...
myusers:developer   myusers    developer       developer   d4e77b0d-9740-4f...
</pre>

## As admin create new HTPasswd user named manager

Extract the file data from the secret to the ~/DO280/labs/auth-provider/htpasswd file.
<pre>
[kris@workstation ~]$ oc extract secret/localusers -n openshift-config \
>    --to ~/DO280/labs/auth-provider/ --confirm
/home/kris/DO280/labs/auth-provider/htpasswd
</pre>

We then add our entry to the file for the additional user manager with a password of redhat
<pre>
[kris@workstation ~]$ htpasswd -b ~/DO280/labs/auth-provider/htpasswd \
>    manager redhat
Adding password for user manager
</pre>

We review the content of the file
<pre>
[kris@workstation ~]$ cat ~/DO280/labs/auth-provider/htpasswd
admin:$2y$05$QPuzHdl06IDkJssT.tdkZuSmgjUHV1XeYU4FjxhQrFqKL7hs2ZUl6
developer:$apr1$0Nzmc1rh$yGtne1k.JX6L5s5wNa2ye.
manager:$apr1$CJ/tpa6a$sLhjPkIIAy755ZArTT5EH/
</pre>

And now we update the secret after adding the additional user.
<pre>
[kris@workstation ~]$ oc set data secret/localusers \
>    --from-file htpasswd=~/DO280/labs/auth-provider/htpasswd \
>    -n openshift-config
secret/localusers data updated
</pre>

We can now login as the manager and create a project
<pre>
[kris@workstation ~]$ oc login -u manager -p redhat
Login successful.

[kris@workstation ~]$ oc new-project auth-provider
Now using project "auth-provider" on server https://api.ocp4.example.com:6443".
</pre>

We can now try to login as developer and delete the project created as manager which will not be possible.
<pre>
[kris@workstation ~]$ oc login -u developer -p developer
Login successful.

[kris@workstation ~]$ oc delete project auth-provider
Error from server (Forbidden): projects.project.openshift.io "auth-provider"
is forbidden: User "developer" cannot delete resource "projects"
in API group "project.openshift.io" in the namespace "auth-provider"
</pre>


We can now again try to extract the secrets file and and generate a random user password and assign it to the MANAGER_PASSWD variable.
<pre>
[kris@workstation ~]$ oc extract secret/localusers -n openshift-config \
>    --to ~/DO280/labs/auth-provider/ --confirm
/home/kris/DO280/labs/auth-provider/htpasswd
</pre>

Generate a random user password and assign it to the MANAGER_PASSWD variable
<pre>
[kris@workstation ~]$ MANAGER_PASSWD="$(openssl rand -hex 15)"
</pre>

Now update the manager user to use the password stored in the MANAGER_PASSWD variable
<pre>
[kris@workstation ~]$ htpasswd -b ~/DO280/labs/auth-provider/htpasswd \
>    manager ${MANAGER_PASSWD}
Updating password for user manager
</pre>

Update the secret
<pre>
[kris@workstation ~]$ oc set data secret/localusers \
>    --from-file htpasswd=~/DO280/labs/auth-provider/htpasswd \
>    -n openshift-config
secret/localusers data updated
</pre>

And then login using the new variable and password
<pre>
[kris@workstation ~]$ oc login -u manager -p ${MANAGER_PASSWD}
Login successful.
</pre>

## Remove user

Now we login as admin
<pre>
[kris@workstation ~]$ oc login -u admin -p redhat
Login successful.
</pre>

We exctract the localusers secret
<pre>
[kris@workstation ~]$ oc extract secret/localusers -n openshift-config \
>    --to ~/DO280/labs/auth-provider/ --confirm
/home/kris/DO280/labs/auth-provider/htpasswd
</pre>

We delete the manager user from the htpasswd file
<pre>
[kris@workstation ~]$ htpasswd -D ~/DO280/labs/auth-provider/htpasswd manager
Deleting password for user manager
</pre>

We then update the secret
<pre>
[kris@workstation ~]$ oc set data secret/localusers \
>    --from-file htpasswd=~/DO280/labs/auth-provider/htpasswd \
>    -n openshift-config
secret/localusers data updated
</pre>

We then delete the <b>identity resource</b> for the manager uesr
<pre>
[kris@workstation ~]$ oc delete identity "myusers:manager"
identity.user.openshift.io "myusers:manager" deleted
</pre>

We then delete the <b>resource</b> for the manager user
<pre>
[kris@workstation ~]$ oc delete user manager
user.user.openshift.io manager deleted
</pre>

If we list all users we now see the manager is gone and we cant login as him anymore.
<pre>
[kris@workstation ~]$ oc get users
NAME       UID                                   FULL NAME  IDENTITIES
admin      31f6ccd2-6c58-47ee-978d-5e5e3c30d617             myusers:admin
developer  d4e77b0d-9740-4f05-9af5-ecfc08a85101             myusers:developer
</pre>


We can now delete the OAuth resource aswell as our secret. 

<pre>
[kris@workstation ~]$ oc edit oauth
</pre>

And here we remove everything from the "spec" area so it becomes what is was before.
<pre>
...output omitted...
spec: {}
</pre>

We now delete the localusers secret from the openshift-config namespace
<pre>
[kris@workstation ~]$ oc delete secret localusers -n openshift-config
secret "localusers" deleted
</pre>

We then delete all users
<pre>
[kris@workstation ~]$ oc delete user --all
user.user.openshift.io "admin" deleted
user.user.openshift.io "developer" deleted
</pre>

We now delete all identity resources
<pre>
[kris@workstation ~]$ oc delete identity --all
identity.user.openshift.io "myusers:admin" deleted
identity.user.openshift.io "myusers:developer" del
</pre>