Firstly login and create a project

<pre>
[kris@workstation ~]$ oc login -u developer -p developer \
>    https://api.ocp4.example.com:6443
Login successful.

[kris@workstation ~]$ oc new-project authorization-secrets
Now using project "authorization-secrets" on server
"https://api.ocp4.example.com:6443".
...output omitted...
</pre>

Create a secret with the credentials and connection information to access a MySQL database 
<pre>
[kris@workstation ~]$ oc create secret generic mysql \
>   --from-literal user=myuser --from-literal password=redhat123
>   --from-literal database=test_secrets --from-literal hostname=mysql
secret/mysql created
</pre>

We then try to deploy a database and add the secret for user and database configuration

This will fail because we need to add the default database initial configurations.
<pre>
[kris@workstation ~]$ oc new-app --name mysql \
>   --image registry.redhat.io/rhel8/mysql-80:1
...output omitted...
--> Creating resources ...
    imagestream.image.openshift.io "mysql" created
    deployment.apps "mysql" created
    service "mysql" created
--> Success
...output omitted...
</pre>

Run the oc get pods command with the -w option to retrieve the status of the deployment in real time. Here we notice how the database pod is in failed state due to missing configuration.
<pre>
[kris@workstation ~]$ oc get pods -w
NAME                     READY   STATUS             RESTARTS      AGE
mysql-7b58b9b68b-ftp6j   0/1     CrashLoopBackOff   3 (46s ago)   106s
mysql-7b58b9b68b-ftp6j   1/1     Running            4 (53s ago)   113s
mysql-7b58b9b68b-ftp6j   0/1     Error              4 (54s ago)   114s
mysql-7b58b9b68b-ftp6j   0/1     CrashLoopBackOff   4 (16s ago)   2m9s
</pre>

Use the mysql secret to initialize environment variables on the mysql deployment. The deployment needs the MYSQL_USER, MYSQL_PASSWORD and MYSQL_DATABASE environment variables for a succesful initialization. 
<pre>
[kris@workstation ~]$ oc set env deployment/mysql --from secret/mysql --prefix MYSQL_
deployment.apps/mysql updated
</pre>

To demonstrate how a secret can be mounted as a volume, mount the mysql secret to the /run/secrets/mysql directory within the pod. This step only shows how to mount a secret as a volume, it is not required to fix the deployment.
<pre>
[kris@workstation ~]$ oc set volume deployment/mysql --add --type secret \
>   --mount-path /run/secrets/mysql --secret-name mysql
info: Generated volume name: volume-nrh7r
deployment.apps/mysql volume updated
</pre>


## We try again

Now we create a new application using one of redhats projects.
<pre>
[kris@workstation ~]$ oc new-app --name quotes \
>   --image quay.io/redhattraining/famous-quotes:2.1
--> Found container image 7ff1a7b (19 months old) from quay.io for "quay.io/redhattraining/famous-quotes:2.1"
...output omitted...
--> Creating resources ...
    imagestream.image.openshift.io "quotes" created
    deployment.apps "quotes" created
    service "quotes" created
--> Success
...output omitted...
</pre>

Verify the status of the quotes application pod. The pod displays an error because it cannot connect to the database. This might take a while to display in the output. Press Ctrl=C to exit the command.
<pre>
[kris@workstation ~]$ oc get pods -l deployment=quotes -w
NAME                      READY   STATUS             RESTARTS      AGE
quotes-66b987df5d-59flf   0/1     CrashLoopBackOff   2 (20s ago)   50s
quotes-66b987df5d-59flf   0/1     Error              3             59s
quotes-66b987df5d-59flf   0/1     CrashLoopBackOff   3 (16s ago)   74s
</pre>

The quotes application requires several environment variables. The mysql secret can initialize environment variables for the quotes application by adding the "QUOTES_" prefix.

Use the mysql secret to initialize the follwoing environment variables that the quotes application needs to connect to the database and hostname keys of the mysql secret.
<pre>
[kris@workstation ~]$ oc set env deployment/quotes --from secret/mysql \
>   --prefix QUOTES_
deployment.apps/quotes updated
</pre>

Wait until the quotes application pod redeploys. The older pods terminate automatically.
<pre>
[kris@workstation ~]$ oc get pods -l deployment=quotes
NAME                      READY   STATUS    RESTARTS   AGE
quotes-77df54758b-mqdtf   1/1     Running   3          7m17s
</pre>

Check the logs to verify succesfull database connection
<pre>
[kris@workstation ~]$ oc logs quotes-77df54758b-mqdtf | head -n2
... Connecting to the database: myuser:redhat123@tcp(mysql:3306)/test_secrets
... Database connection OK
</pre>

Expose the quotes service so that it can be accessed from outside the cluster
<pre>
[kris@workstation ~]$ oc expose service quotes \
>    --hostname quotes.apps.ocp4.example.com
route.route.openshift.io/quotes exposed
</pre>

Verify the application host name.
<pre>
[kris@workstation ~]$ oc get route quotes
NAME     HOST/PORT                      PATH   SERVICES   PORT       ...
quotes   quotes.apps.ocp4.example.com          quotes     8000-tcp   ...
</pre>

Test the application with curl
<pre>
[kris@workstation ~]$ curl -s http://quotes.apps.ocp4.example.com/status
Database connection OK
</pre>

Delete the project 
<pre>
[kris@workstation ~]$ oc delete project authorization-secrets
project.project.openshift.io "authorization-secrets" deleted
</pre>