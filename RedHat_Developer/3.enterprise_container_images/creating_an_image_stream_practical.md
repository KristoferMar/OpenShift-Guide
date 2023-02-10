# Creating an Image Stream

When we create an image stream it's good practice to inspect the image we are about to use if it is somewhere available. 
<pre>
[kris@workstation ~]$ skopeo inspect \
docker://quay.io/kris-practice/hello-world-nginx
{
    "Name": "quay.io/kris-practice/hello-world-nginx",
    "Tag": "latest",
    "Digest": "sha256:4f4f...acc1",
    "RepoTags": [
        "latest"
    ],
...output omitted...
</pre>

Create a image stream that points towards a "hello-world-nginx" container image from Quay.io
<pre>
[kris@workstation ~]$ oc import-image hello-world --confirm \
--from quay.io/kris-practice/hello-world-nginx
imagestream.image.openshift.io/hello-world imported
...output omitted...
Name:           hello-world
Namespace:      youruser-common
...output omitted...
Unique Images:  1
Tags:           1

latest
  tagged from quay.io/kris-practice/hello-world-nginx

...output omitted...
</pre>

Verify image stream tag was created
<pre>
[kris@workstation ~]$ oc get istag
NAME                 IMAGE REF                                               ...
hello-world:latest   quay.io/kris-practice/hello-world-nginx@sha256:4f4f...acc1
</pre>

Verify that the image stream and it's tag contain metadata about the Nginx container image
<pre>
[kris@workstation ~]$ oc describe is hello-world
Name:          hello-world
Namespace:     youruser-common
...output omitted...
Tags:          1

latest
  tagged from quay.io/kris-practice/hello-world-nginx

  * quay.io/kris-practice/hello-world-nginx@sha256:4f4f...acc1
      2 minutes ago

...output omitted...
</pre>

We can now deploy an application from the new imagestream in OpenShift
<pre>
[kris@workstation ~]$ oc new-app --name hello \
-i ${RHT_OCP4_DEV_USER}-common/hello-world
--> Found image 44eaa13 (20 hours old) in image stream "youruser-common/hello-world" under tag "latest" for "youruser-common/hello-world"
...output omitted...
--> Creating resources ...
    imagestreamtag.image.openshift.io "hello:latest" created
    deployment.apps "hello" created
    service "hello" created
--> Success
...output omitted...
</pre>

