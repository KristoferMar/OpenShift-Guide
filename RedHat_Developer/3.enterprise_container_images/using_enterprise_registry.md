Login 

```
[kris@workstation ~]$ podman login -u ${RHT_OCP4_QUAY_USER} quay.io
Password:
Login Succeeded!
```

We copy the OCI image to the external registry at Quay.io by using Skopeo and tag it as 1.0

```
[kris@workstation ~]$ skopeo copy --format v2s1 \
oci:/home/kris/DO288/labs/external-registry/ubi-sleep \
docker://quay.io/${RHT_OCP4_QUAY_USER}/ubi-sleep:1.0
...output omitted...
Writing manifest to image destination
Storing signatures
```

Inspect the image in the external registry with Skopeo

```
[kris@workstation ~]$ skopeo inspect \
docker://quay.io/${RHT_OCP4_QUAY_USER}/ubi-sleep:1.0
{
  "Name": "quay.io/yourquayuser/ubi-sleep",
  "Tag": "1.0",
  ...output omitted...
```

Verify that Podman can run images from the external registry

```
[kris@workstation ~]$ podman run -d --name sleep \
quay.io/${RHT_OCP4_QUAY_USER}/ubi-sleep:1.0
Trying to pull quay.io/youruser/ubi-sleep:1.0...Getting image source signatures
...output omitted...
```

Verify that the new container is running 

```
[kris@workstation ~]$ podman ps
CONTAINER ID        IMAGE                            ...   NAMES
63c5167376e5        quay.io/youruser/ubi-sleep:1.0   ...   sleep
```

Verify that the new container produces log output 

```
[kris@workstation ~]$ podman logs sleep
...output omitted...
sleeping
sleeping
```

Now we can deploy an application to OpenShift based on the image from the external registry

```
[kris@workstation ~]$ oc login -u ${RHT_OCP4_DEV_USER} \
-p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
Login successful.
...output omitted...
[kris@workstation ~]$ oc new-project ${RHT_OCP4_DEV_USER}-external-registry
Now using project "youruser-docker-build" on server "https://api.cluster.domain.example.com:6443".
```

Now we can try to deploy an application from the container image in the external registry. But it will fail since OpenShify needs credentials to access the external registry. 

```
[kris@workstation ~]$ oc new-app --name sleep \
--image quay.io/${RHT_OCP4_QUAY_USER}/ubi-sleep:1.0
...output omitted...
error: unable to locate any local docker images with name "quay.io/yourquayuser/ubi-sleep:1.0"
...output omitted...
```

We create a secret from the container registry API access token that was stored by Podman.

```
[kris@workstation ~]$ oc create secret generic quayio \
--from-file .dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json \
--type kubernetes.io/dockerconfigjson
secret/quayio created
```

We link the new secret to the default service account 

```
[kris@workstation ~]$ oc secrets link default quayio --for pull
```

Now we can deploy an application from the container image in the external registry.

```
[kris@workstation ~]$ oc new-app --name sleep \
--image quay.io/${RHT_OCP4_QUAY_USER}/ubi-sleep:1.0
...output omitted...
--> Creating resources ...
imagestream.image.openshift.io "sleep" created
deployment.apps "sleep" created
--> Success
...output omitted...
```

We can then close everything down and also delete container from external registry using skopeo.

```
[kris@workstation ~]$ skopeo delete \
docker://quay.io/${RHT_OCP4_QUAY_USER}/ubi-sleep:1.0
```