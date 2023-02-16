# Building Applications for OpenShift
A typical service in OpenShift has both a name and a selector. A service uses its selector to identify pods that should receive application requests sent to the service. OpenShift applications use the service name to connect to the service endpoints.

- Pods from the same project can use the myapi service name as a short host name, without any domain suffix.
- Pods from a different project can use the service name and myapi.myproject project name as a short host name, without the svc.cluster.local domain suffix.

## Creating an external service
<pre>
[user@host ~]$ oc create service externalname myservice \
--external-name myhost.example.com
</pre>

As long as an external source is publically available you simply just create a service within openshift that points towards the source to make the source available as a service internally.
<pre>
[kris@workstation ~]$ oc create svc externalname tododb \
--external-name ${RHT_OCP4_MYSQL_SERVER}
service/tododb created
</pre>