We start of by logging into our cluster

```
[kris@workstation DO288-apps]$ oc login -u ${RHT_OCP4_DEV_USER} \
-p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

We create a new project

```
[kris@workstation DO288-apps]$ oc new-project ${RHT_OCP4_DEV_USER}-docker-build
Now using project "developer-docker-build" on server "https://api.ocp4.example.com:6443".
```

We create a new application from a Dockerfile and call it echo.

```
[kris@workstation DO288-apps]$ oc new-app --name echo \
https://github.com/${RHT_OCP4_GITHUB_USER}/DO288-apps#docker-build \
--context-dir ubi-echo
...output omitted...
--> Creating resources ...
    imagestream.image.openshift.io "ubi" created
    imagestream.image.openshift.io "echo" created
    buildconfig.build.openshift.io "echo" created
    deployment.apps.openshift.io "echo" created
--> Success
...output omitted...
```

We can follow the build logs in the following way

```
[kris@workstation DO288-apps]$ oc logs -f bc/echo
Cloning "https:/github.com/youruser/DO288-apps#docker-build" ...
Replaced Dockerfile FROM image registry.access.redhat.com/ubi8/ubi:8.0
Caching blobs under "/var/cache/blobs".

...output omitted...
Pulling image registry.access.redhat.com/ubi8/ubi@sha256:1a2a...75b5
...output omitted...
STEP 1: FROM registry.access.redhat.com/ubi8/ubi@sha256:1a2a...75b5  1
STEP 2: USER 1001
STEP 3: CMD bash -c "while true; do echo test; sleep 5; done"
STEP 4: ENV "OPENSHIFT_BUILD_NAME"="echo-1" ... 2
STEP 5: LABEL "io.openshift.build.commit.author"=...
STEP 6: COMMIT temp.builder.openshift.io/developer-docker-build/echo-1:... 3
...output omitted...
Pushing image image-registry.openshift-image-registry.svc:5000/developer-docker-build/echo:latest ...  4
Push successful
```

1. The oc new-app command correctly identified the Git repository as a Dockerfile project and the OpenShift build performs a Dockerfile build.
2. OpenShift appends metadata to the application container image using ENV and LABEL instructions.
3. OpenShift commits the application image to the node's container engine.
4. OpenShift pushes the application image from the node's container engine to the cluster's internal registry.

Now verify that application works inside OpenShift

```
[kris@workstation DO288-apps]$ oc status
In project youruser-docker-build on server
  https://api.ocp4.example.com:6443

deployment/echo deploys istag/echo:latest <-
  bc/echo docker builds https://github.com/your-GitHub-user/DO288-apps#docker-build on istag/ubi:8.0
  deployment #2 running for 20 minutes - 1 pod
  deployment #1 deployed 21 minutes ago
...output omitted...
```

Wait for pod to be ready

```
[kris@workstation DO288-apps]$ oc get pod
NAME            READY     STATUS      RESTARTS   AGE
echo-1-build    0/1       Completed   0          1m
echo-555xx      1/1       Running     0          14s
```

Display application pod logs 

```
[kris@workstation DO288-apps]$ oc logs echo-555xx | tail -n 3
test
test
test
```

Review the build configuration

```
[kris@workstation DO288-apps]$ oc describe bc echo
Name:            echo
...output omitted...
Labels:          app=echo
...output omitted...
Strategy:        Docker
URL:             https://github.com/your-GitHub-user/DO288-apps
Ref:             docker-build  1
ContextDir:	     ubi-echo      2
From Image:      ImageStreamTag ubi:8.0   3
Output to:       ImageStreamTag echo:latest  4
...output omitted...
```

1. Builds start from the docker-build branch from the Git repository in the URL attribute.
2. Builds take only the ubi-echo folder from the the Git repository in the URL attribute.
3. Builds take an image stream that points to the parent image from the Dockerfile so that new builds can be triggered by image changes.
4. Builds generate a new container image and push it to the internal registry through an image stream.

Review the image stream

```
[kris@workstation DO288-apps]$ oc describe is echo
Name:                 echo
...output omitted...
Labels:               app=echo
...output omitted...
Image Repository:	image-registry.openshift-image-registry.svc:5000/developer-docker-build/echo1
...output omitted...
latest
  no spec tag

  * image-registry.openshift-image-registry.svc:5000/developer-docker-build/echo@sha256:5bbf...ef0b 2
...output omitted...
```

1. The image stream points to the OpenShift internal registry using the service DNS name.
2. A SHA256 hash identifies the latest image. Using this hash, the image stream can detect whether the image was changed.

Review the deployment

```
[kris@workstation DO288-apps]$ oc describe deployment echo
Name:             echo
...output omitted...
Labels:           app=echo
Annotations:      deployment.kubernetes.io/revision: 2
                  image.openshift.io/triggers: 1
                  [{"from":{"kind":"ImageStreamTag","name":"echo:latest"},
                  "fieldPath":"spec.template.spec.containers[?(@.name==\"echo\")].image"}]
...output omitted...
Pod Template:
  ...output omitted...
  Containers:
   echo:
    Image:        image-registry.openshift-image-registry.svc:5000/developer-docker-build/echo@sha256:5bbf...ef0b  2
...output omitted...
Deployment #1 (latest):
...output omitted...
```

1. The deployment has a trigger in the image stream. If the image stream changes, then a new deployment is performed.
2. The pod template inside the deployment configuration specifies the SHA256 hash of the container image in order to support deployment strategies such as rolling upgrades.

Delete all applcation resources

```
[kris@workstation ~]$ oc delete all -l app=echo
deployment.apps "echo" deleted
buildconfig.build.openshift.io "echo" deleted
build.build.openshift.io "echo-1" deleted
build.build.openshift.io "echo-2" deleted
imagestream.image.openshift.io "echo" deleted
imagestream.image.openshift.io "ubi" deleted
```

Verify deletion 

```
[kris@workstation ~]$ oc get all
No resources found.
```