# Troubleshooting OpenShift Clusters and Applications

Verify all notes in cluster
<pre>
[kris@workstation ~]$ oc get nodes
</pre>

Verify whether any of your nodes are close to using all of the CPU and memory avaiable to them.
<pre>
[kris@workstation ~]$ oc adm top node
</pre>

Verify that all of the conditions that might indicate problems are false 
<pre>
[kris@workstation ~]$ oc describe node master01
</pre>

Review the logs of the internal registry operator, the internal registry serverm and the Kubelet of a node.

<pre>
[kris@workstation ~]$ oc get pod -n openshift-image-registry

NAME                                               READY   STATUS      ...
cluster-image-registry-operator-6f7784dc86-qq258   1/1     Running   ...
image-pruner-27506880-dcn8n                        0/1     Completed   ...
image-pruner-27508320-2vfph                        0/1     Completed   ...
image-registry-867979b75b-xk4ns                    1/1     Running   ...
</pre>

Follow the logs of the opreator pod (cluster-image-registry-operator-xxx)
<pre>
[kris@workstation ~]$ oc logs --tail 3 -n openshift-image-registry cluster-image-registry-operator-564bd5dd8f-s46bz
</pre>

Follow the logs of the image registry server pod (image-registry-xxx from the output of the oc get pod command run previously)
<pre>
[kris@workstation ~]$ oc logs --tail 1 -n openshift-image-registry image-registry-794dfc7978-w7w69
</pre>

Follow the logs of the Kubelet from the same node that you inspected for CPU and memory usage in the previous step
<pre>
[kris@workstation ~]$ oc adm node-logs --tail 1 -u kubelet master01
</pre>

Start a shell session on use the chroot command to enter the local file system of the host.
<pre>
[kris@workstation ~]$ oc debug node/master01
Creating debug namespace/openshift-debug-node-5zsch ...
Starting pod/master01-debug ...
To use host binaries, run `chroot /host`
Pod IP: 192.168.50.10
If you do not see a command prompt, try pressing enter.
sh-4.4# chroot /host
sh-4.4#
</pre>

Using the session verify kubelet and the CRI-O container engine are running.
<pre>
sh-4.4# systemctl status kubelet
● kubelet.service - Kubernetes Kubelet
   Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-mco-default-madv.conf, 20-logging.conf, 20-nodenet.conf
   Active: active (running) since Wed 2022-04-20 20:36:05 UTC; 5h 47min ago
...output omitted...
q
</pre>

<pre>
sh-4.4# systemctl status crio
● crio.service - Container Runtime Interface for OCI (CRI-O)
   Loaded: loaded (/usr/lib/systemd/system/crio.service; disabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/crio.service.d
           └─10-mco-default-madv.conf, 10-mco-profile-unix-socket.conf, 20-nodenet.conf
   Active: active (running) since Wed 2022-04-20 20:35:59 UTC; 5h 59min ago
...output omitted...
q
</pre>

Verify that the etcd pod is running
<pre>
sh-4.4# crictl ps --name etcd
CONTAINER      ... STATE     NAME           ATTEMPT  POD ID
5e18a8220a9d5  ... Running   etcd-operator  2        90b6b1150e3fe8d9adceeb5549
19bc3ed5e8643  ... Running   etcd-metrics   3        c94baa4f55618
...output omitted...
</pre>

Enter the install-troubleshoot project to diagnose a pod that is in a error state
<pre>
[kris@workstation ~]$ oc project install-troubleshoot
Now using project "install-troubleshoot" on server ...
</pre>

Verify that the project has a single pod in either the ErrImagePull or ImagePullBackOff status.
<pre>
[kris@workstation ~]$ oc status
...output omitted...
svc/psql - 172.30.13.140:5432

deployment/psql deploys registry.redhat.io/rhel8/postgresq-13:1
  deployment #1 running for 7 hours - 0/1 pods
...output omitted...
</pre>

List all events from the current project and look for error messages related to the pod
<pre>
oc get events
</pre>

In this case we login to the registry and verify the image and see that our image has a miss spelling 

Login to the RedHat Container catalog 
<pre>
[kris@workstation ~]$ podman login registry.redhat.io
Username: your_username
Password: your_password
Login Succeeded!
</pre>

use skopeo to find information about the container image
<pre>
[kris@workstation ~]$ skopeo inspect \
>    docker://registry.redhat.io/rhel8/postgresq-13:1
FATA[0000] Error parsing image name "docker://registry.redhat.io/rhel8/postgresq-13:1": Error reading manifest 1 in registry.redhat.io/rhel8/postgresq-13: unknown: Not Found
</pre>

We find the missspelling
<pre>
[kris@workstation ~]$ skopeo inspect \
>    docker://registry.redhat.io/rhel8/postgresql-13:1
{
    "Name": "registry.redhat.io/rhel8/postgresql-13",
...output omitted...
</pre>


To verify the name image name is our root cause of error we edit the psql deployment to correct the name of the container image. In a real-world scenario,you would ask whoever deployed the PostgreSQL database to fix their YAML and redeploy their application.
<pre>
[kris@workstation ~]$ oc edit deployment/psql
...output omitted...
    spec:
      containers:
      - env:
        - name: POSTGRESQL_DATABASE
          value: db
        - name: POSTGRESQL_PASSWORD
          value: pass
        - name: POSTGRESQL_USER
          value: user
        image: registry.redhat.io/rhel8/postgresql-13:1-7
...output omitted...
</pre>

We can now verify our work.
<pre>
[kris@workstation ~]$ oc status
...output omitted...
deployment/psql deploys registry.redhat.io/rhel8/postgresql-13:1
  deployment #2 running for 30 seconds - 1 pod
  deployment #1 deployed 7 hours ago
</pre>

<pre>
[kris@workstation ~]$ oc get pods
NAME                             READY   STATUS    RESTARTS   AGE
psql-544c9c666f-btlw8  1/1     Running   0          55s
</pre>