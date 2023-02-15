# Publish Enterprise Container Images

Copy the OCI image to the external registry at Quay.io by using Skopeo and tag it as 1.0.
<pre>
[kris@workstation ~]$ skopeo copy --format v2s1 \
oci:/home/kris/DO288/labs/external-registry/ubi-sleep \
docker://quay.io/${RHT_OCP4_QUAY_USER}/ubi-sleep:1.0
...output omitted...
Writing manifest to image destination
Storing signatures
</pre>

Inspect the image in the external registry by using Skopeo
<pre>
[kris@workstation ~]$ skopeo inspect \
docker://quay.io/${RHT_OCP4_QUAY_USER}/ubi-sleep:1.0
{
  "Name": "quay.io/yourquayuser/ubi-sleep",
  "Tag": "1.0",
  ...output omitted...
</pre>

Verify that Podman can run images from the external registry.

Start a test container from the image in the external registry.
<pre>
[kris@workstation ~]$ podman run -d --name sleep quay.io/${RHT_OCP4_QUAY_USER}/ubi-sleep:1.0
Trying to pull quay.io/youruser/ubi-sleep:1.0...Getting image source signatures
...output omitted...
</pre>

Now login to OCP and create new project
<pre>
[kris@workstation ~]$ oc login -u ${RHT_OCP4_DEV_USER} \
-p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
Login successful.
...output omitted...

[kris@workstation ~]$ oc new-project ${RHT_OCP4_DEV_USER}-external-registry
Now using project "youruser-docker-build" on server "https://api.cluster.domain.example.com:6443".
</pre>

Create a secret from the container registry API access token that was stored by Podman.
<pre>
[kris@workstation ~]$ oc create secret generic quayio \
--from-file .dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json \
--type kubernetes.io/dockerconfigjson
secret/quayio created
</pre>

Link the new secret to the default service account.
<pre>
[kris@workstation ~]$ oc secrets link default quayio --for pull
</pre>

deploy application from the container image
<pre>
[kris@workstation ~]$ oc new-app --name sleep \
--image quay.io/${RHT_OCP4_QUAY_USER}/ubi-sleep:1.0
...output omitted...
--> Creating resources ...
imagestream.image.openshift.io "sleep" created
deployment.apps "sleep" created
--> Success
...output omitted...
</pre>

Delete the container image from the external registry
<pre>
[kris@workstation ~]$ skopeo delete \
docker://quay.io/${RHT_OCP4_QUAY_USER}/ubi-sleep:1.0
</pre>
