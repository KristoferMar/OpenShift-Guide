# Developement 

## Podman
Podman is your best friend during development.

How to build image and test.

<pre>
podman build -t localhost/imagename .
</pre>

<pre>
podman run -d -name imagename localhost/imagename
</pre>

Run with exposed port
<pre>
podman run -d -p 8080:80 --name imagename localhost/imagename
</pre>

### Build and deploy in a pulic repository
Build
<pre>podman build -t quay.io/kristofer24/imagename .</pre>
Push
<pre>podman push quay.io/kristofer24/imagename</pre>