# Customizing an Existing S2I base image

The S2I scripts are packaged within the S2I builder images by default. In certain scenarios, you may want to customize these scripts to change the way your application is built and run, without rebuilding the image.

podman pull command to pull the container image from a conatiner registry to the local system.
<pre>
[user@host ~]$ podman pull \
myregistry.example.com/rhscl/php-73-rhel7
...output omitted...
Digest: sha256:...
[user@host ~]$ podman inspect \
--format='{{ index .Config.Labels "io.openshift.s2i.scripts-url"}}' \
rhscl/php-73-rhel7
image:///usr/libexec/s2i
</pre>

You can use the skopeo inspect command to retrieve the same information directly from a remote registry
<pre>
[user@host ~]$ skopeo inspect \
docker://myregistry.example.com/rhscl/php-73-rhel7 \
| grep io.openshift.s2i.scripts-url
        "io.openshift.s2i.scripts-url": "image:///usr/libexec/s2i",
</pre>

Create a wrapper for the assemble script in the .s2i/bin folder
<pre>
#!/bin/bash
echo "Making pre-invocation changes..."

/usr/libexec/s2i/assemble
rc=$?

if [ $rc -eq 0 ]; then
    echo "Making post-invocation changes..."
else
    echo "Error: assemble failed!"
fi

exit $rc
</pre>

Create a wrapper for the run script in the .s2i/bin folder
<pre>
#!/bin/bash
echo "Before calling run..."
exec /usr/libexec/s2i/run
</pre>


