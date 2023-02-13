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