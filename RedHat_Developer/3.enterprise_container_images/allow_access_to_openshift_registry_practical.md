# Allowing Access to the OpenShift Registry

To ease up we can set env virable to define the internal registry
<pre>
[kris@workstation ~]$ INTERNAL_REGISTRY=\
default-route-openshift-image-registry.apps.ocp4.example.com
</pre>

Verify the availability of the route to the RHOCP internal registry by logging in to the registry and using the developer's authentication token and the registry hostname.
<pre>
[kris@workstation ~]$ podman login -u ${RHT_OCP4_DEV_USER} \
-p $(oc whoami -t) $INTERNAL_REGISTRY
Login Succeeded!
</pre>

Create a project to host the image streams that manage the container images that you push to the OpenShift internal registry
<pre>
[kris@workstation ~]$ oc new-project ${RHT_OCP4_DEV_USER}-common
Now using project "youruser-common" on server
"https://api.cluster.domain.example.com:6443".
</pre>

Retrieve your developer user account's OpenShift authentication token to use in later commands
<pre>
[kris@workstation ~]$ TOKEN=$(oc whoami -t)
</pre>

Copy the OCI image to the classroom cluster's internal registry by using Skopeo and tag it as 1.0. Use the hostname and the token that you retrieved in previous steps.
<pre>
[kris@workstation ~]$ skopeo copy --format v2s1 \
--dest-creds=${RHT_OCP4_DEV_USER}:${TOKEN} \
oci:/home/kris/DO288/labs/expose-registry/ubi-info \
docker://${INTERNAL_REGISTRY}/${RHT_OCP4_DEV_USER}-common/ubi-info:1.0
...output omitted...
Writing manifest to image destination
Storing signatures
</pre>

Verify that an image stream was created to manage the new container image
<pre>
[kris@workstation ~]$ oc get is
NAME       IMAGE REPOSITORY
ubi-info   default-route-openshift-image-registry.apps...
</pre>

Create a local container from the image in the OpenShift internal registry.
<pre>
[kris@workstation ~]$ podman pull \
${INTERNAL_REGISTRY}/${RHT_OCP4_DEV_USER}-common/ubi-info:1.0
...output omitted...
</pre>

Start a new container from the ubi-info:1.0 container image
<pre>
[kris@workstation ~]$ podman run --name info \
${INTERNAL_REGISTRY}/${RHT_OCP4_DEV_USER}-common/ubi-info:1.0
...output omitted...
--- Host name:
...output omitted...
--- Free memory
...output omitted...
--- Mounted file systems (partial)
</pre>