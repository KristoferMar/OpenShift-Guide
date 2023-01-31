## Create, manage and configure dynamic storage

Verify the default storage class
<pre>
[kris@workstation ~]$ oc get storageclass
NAME                    PROVISIONER                                    ...
nfs-storage (default)   k8s-sigs.io/nfs-subdir-external-provisioner    ...
</pre>

We create a new database deployment using the container image located at registry.redhat.io/rhel8/postgresql-13:10-7
<pre>
[kris@workstation ~]$ oc new-app --name postgresql-persistent \
>   --image registry.redhat.io/rhel8/postgresql-13:1-7 \
>   -e POSTGRESQL_USER=redhat \
>   -e POSTGRESQL_PASSWORD=redhat123 \
>   -e POSTGRESQL_DATABASE=persistentdb
...output omitted...
--> Creating resources ...
    imagestream.image.openshift.io "postgresql-persistent" created
    deployment.apps "postgresql-persistent" created
    service "postgresql-persistent" created
--> Success
...output omitted...
</pre>

Create a new persistent volume 
<pre>
[kris@workstation ~]$ oc set volumes deployment/postgresql-persistent \
>   --add --name postgresql-storage --type pvc --claim-class nfs-storage \
>   --claim-mode rwo --claim-size 10Gi --mount-path /var/lib/pgsql \
>   --claim-name postgresql-storage
deployment.apps/postgresql-persistent volume updated
</pre>

Verify that successfull creation of the new PVC
<pre>
[kris@workstation ~]$ oc get pvc
NAME                 STATUS   ...  CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgresql-storage   Bound    ...  10Gi       RWO            nfs-storage    25s
</pre>

Verify that you successfully created the new PV
<pre>
[kris@workstation ~]$ oc get pv \
>   -o custom-columns=NAME:.metadata.name,CLAIM:.spec.claimRef.name
NAME                                       CLAIM
pvc-26cc804a-4ec2-4f52-b6e5-84404b4b9def   image-registry-storage
pvc-65c3cce7-45eb-482d-badf-a6469640bd75   postgresql-storage
</pre>

or just

<pre>
[kris@workstation ~]$ oc get pv 
</pre>

Delete all resources that contain the app postgresql-persistent label
<pre>
[kris@workstation install-storage]$ oc delete all -l app=postgresql-persistent
service "postgresql-persistent" deleted
deployment.apps "postgresql-persistent" deleted
imagestream.image.openshift.io "postgresql-persistent" deleted
</pre>

When a new app is then created you can add the existing postgresql-storage persistent volume claim to the new deployment.
<pre>
[kris@workstation install-storage]$ oc set volumes \
>   deployment/postgresql-persistent2 \
>   --add --name postgresql-storage --type pvc \
>   --claim-name postgresql-storage --mount-path /var/lib/pgsql
deployment.apps/postgresql-persistent2 volume updated
</pre>