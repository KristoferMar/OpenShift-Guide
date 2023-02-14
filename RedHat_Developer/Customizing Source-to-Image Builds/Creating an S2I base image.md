# Creating an S2I Base Image

The S2I build process combines application source code with an appropriate S2I builder image to produce the final application container image that is deployed to an Red Hat OpenShift Container Platform (RHOCP) cluster.

## The S2I Tool

Get the version
<pre>
[kris@workstation ~]$ s2i version
s2i v1.3.1
</pre>

<pre>
[user@host ~]$ s2i create image_name directory
</pre>

The above command creates a folder named directory and populates it with the following template files that you can update as needed
<pre>
directory
├── Dockerfile
├── Makefile
├── README.md
├── s2i
│   └── bin
│       ├── assemble
│       ├── run
│       ├── save-artifacts
│       └── usage
└── test
    ├── run
    └── test-app
        └── index.html
</pre>

ou can build the builder image using the podman build command
<pre>
[user@host ~]$ podman build -t builder_image .
</pre>

When the builder image is ready, you can build an application container image using the s2i build command. This allows you to also test the S2I builder image locally. 
<pre>
[user@host ~]$ s2i build src builder_image tag_name
</pre>

# Example of building an Nginx S2I builder Image
Run the s2i create command to create the skeleton
<pre>
[user@host ~]$ s2i create s2i-do288-nginx s2i-do288-nginx
</pre>

Edit the Dockerfile to include instructions to install Nginx and the S2I scripts. 
<pre>
FROM registry.access.redhat.com/ubi8/ubi:8.0

ENV X_SCLS="rh-nginx18" \
    PATH="/opt/rh/rh-nginx18/root/usr/sbin:$PATH" \
    NGINX_DOCROOT="/opt/rh/rh-nginx18/root/usr/share/nginx/html"

LABEL io.k8s.description="A Nginx S2I builder image" \
      io.k8s.display-name="Nginx 1.8 S2I builder image for DO288" \
      io.openshift.expose-services="8080:http" \
      io.openshift.s2i.scripts-url="image:///usr/libexec/s2i" \
      io.openshift.tags="builder,webserver,nginx,nginx18,html"

ADD nginxconf.sed /tmp/
COPY ./.s2i/bin/ /usr/libexec/s2i

RUN yum install -y --nodocs rh-nginx18 \
  && yum clean all \
  && sed -i -f /tmp/nginxconf.sed /etc/opt/rh/rh-nginx18/nginx/nginx.conf \
  && chgrp -R 0 /var/opt/rh/rh-nginx18 /opt/rh/rh-nginx18 \ 5
  && chmod -R g=u /var/opt/rh/rh-nginx18 /opt/rh/rh-nginx18 \ 6
  && echo 'Hello from the Nginx S2I builder image' > ${NGINX_DOCROOT}/index.html

EXPOSE 8080

USER 1001

CMD ["/usr/libexec/s2i/usage"]
</pre>

Create an assemble script in the .s2i/bin directory of the application source with the following content. 
<pre>
#!/bin/bash -e

echo "---> Copying source HTML files to web server root..."
cp -Rf /tmp/src/. /opt/rh/rh-nginx18/root/usr/share/nginx/html/
</pre>

Create a run script in the .s2i/bin directory of the application source with the following content.
<pre>
#!/bin/bash -e

exec nginx -g "daemon off;"
</pre>

## Building and Testing the S2I Builder Image
Build
<pre>
[user@host ~]$ podman build -t s2i-do288-nginx .
</pre>

Run the s2i build command to build a test application container image.
<pre>
[user@host ~]$ s2i build test/test-app s2i-do288-nginx nginx-test \
--as-docker-file /path/to/Dockerfile
</pre>

Build test container with podman
<pre>
[user@host ~]$ podman build -t nginx-test /path/to/Dockerfile
</pre>

Test the container by booting it up
<pre>
[user@host ~]$ podman run -u 1234 -d -p 8080:8080 nginx-test
</pre>

### Making the S2I Builder Image Available to RHOCP
Push the S2I builder image to an enterprise registry ex Quay.io or the internal openshift registry.
<pre>
[user@host ~]$ skopeo copy containers-storage:localhost/s2i-do288-httpd \
docker://quay.io/${RHT_OCP4_QUAY_USER}/s2i-do288-httpd
</pre> 

Create image stream for the Nginx S2I builder image.
<pre>
[user@host ~]$ oc import-image s2i-do288-nginx \
--from quay.io/${RHT_OCP4_QUAY_USER}/s2i-do288-nginx \
--confirm
</pre>

Use the imagestream to create applictaions using the Nginx S2I builder image
<pre>
[user@host ~]$ oc new-app --name nginx-test \
s2i-do288-nginx~git_repository
</pre>